---
name: decision-logging-threshold
description: Deterministic scoring and gating policy for deciding whether a candidate decision is major enough to persist to provenance. Use before any provenance write.
---

# Decision Logging Threshold Skill

Use this skill whenever you need to decide whether a candidate decision should be persisted to provenance.

This file is the canonical runtime contract for threshold semantics.

## Ownership boundary

This skill owns:

- the candidate-decision scoring dimensions
- the mandatory override rules
- the explicit non-decision rules
- the final log / no-log gate
- the standard `threshold_check` output

Other skills may persist, cite, or analyze the threshold result, but they must not redefine the threshold logic locally.

## Core principle

Log only **high-signal decision inflection points**.

A candidate should be logged when it is likely to help with one or more of these later:

- reuse as a prior
- auditability
- explaining why an outcome happened

Do **not** log routine execution noise.

## Deterministic procedure

For each candidate decision, apply the following procedure in order.

### Step 1: Normalize the candidate

Score one candidate decision at a time.

If the task contains multiple explicit choices, split them first and score them separately.

Examples:

- roadmap prompt with `primary` and `secondary` initiative choices -> `2` separate candidates
- rollout prompt with `release scope` and `safety guardrail` choices -> `2` separate candidates
- one strategy plus several mechanical substeps inside that strategy -> usually `1` candidate, not many

Use a lightweight candidate snapshot like:

```yaml
candidate_snapshot:
  decision: <what was chosen>
  context: <relevant situation and trigger>
  alternatives:
    - <option A>
    - <option B>
  rationale: <why this was chosen>
  constraints:
    - <constraint>
  assumptions:
    - <assumption>
  confidence: low|medium|high
  owner: agent|human|mixed
  timestamp: <ISO-8601 or session-time>
```

Keep the snapshot concise. The full persisted schema belongs to provenance, not this threshold skill.

### Step 2: Check mandatory overrides

Always log if **any** of these apply:

- `policy_compliance_boundary`
  - security-sensitive handling
  - privacy-sensitive handling
  - legal or financial implications
  - business-rule enforcement
- `human_interaction_boundary`
  - human override
  - escalation
  - explicit approval or rejection

If an override applies, `should_log` must be `true` even if the numeric score is low.

### Step 3: Check explicit non-decision cases

If **no mandatory override applies**, do **not** log the candidate when it is clearly one of these:

- deterministic transformation
  - formatting
  - parsing
  - mapping
- retrieval-only step with no reasoning
- mechanical execution step
- repeated micro-decision inside an already selected strategy
- low-level tool call with no alternatives considered

If a candidate matches one of these cases, return `should_log: false` with all dimension scores set to `0`.

### Step 4: Apply the usefulness gate

Set `usefulness_gate: true` if **any** of the following are true:

- the decision influences future reasoning or later workflow steps
- the decision encodes a non-trivial trade-off worth auditing later
- the decision cannot be reliably reconstructed from the final output alone

Otherwise set `usefulness_gate: false`.

If `usefulness_gate` is `false` and no mandatory override applies, the candidate should not be logged.

### Step 5: Score the five dimensions

Compute:

`decision_score = impact_radius + reversibility + uncertainty + tradeoff_intensity + longevity`

Score each dimension independently.

Use the **highest value directly supported by the current context**.
If the evidence only weakly suggests a higher score, choose the lower score and explain the uncertainty briefly in `rationale`.

Do **not** inflate one dimension to compensate for another.

#### 1. `impact_radius`

- `0` -> local to one step or one narrow output fragment
- `1` -> affects multiple steps, files, modules, or deliverable sections
- `2` -> affects architecture, workflow shape, release posture, safety/compliance posture, or broad downstream outcomes

#### 2. `reversibility`

- `0` -> easy to undo with negligible cost in the same session
- `1` -> reversible, but with noticeable rework, coordination, or reruns
- `2` -> hard to undo, externally consequential, or likely to invalidate later work if changed

#### 3. `uncertainty`

- `0` -> deterministic or clearly prescribed by policy/rule
- `1` -> some judgment required, but choices are bounded and understandable
- `2` -> high ambiguity, incomplete information, or materially conflicting evidence/constraints

#### 4. `tradeoff_intensity`

- `0` -> no real viable alternative
- `1` -> alternatives exist, but consequences are limited
- `2` -> explicit competing options with materially different consequences, risks, or opportunity costs

#### 5. `longevity`

- `0` -> ephemeral and unlikely to matter after the immediate step
- `1` -> affects the rest of the current task or near-term workflow
- `2` -> persistent, reusable, policy-shaping, or likely to influence future decisions

## Final gate

Use this exact rule:

```text
override_applied = len(mandatory_overrides) > 0

should_log =
  override_applied
  OR (
    usefulness_gate
    AND decision_score >= 5
  )
```

Interpretation:

- mandatory override beats the score
- usefulness gate blocks noisy decisions even if they were discussed at length
- score `>= 5` is the major-decision threshold

## Tie-break and edge-case rules

- Score the **actual selected decision**, not every rejected alternative.
- Treat an assumption as a candidate decision only if accepting it changes scope, risk, behavior, or outcome.
- If a bundled log entry would hide two materially independent choices, split it before scoring.
- If unsure between two adjacent numeric values, choose the lower value unless the higher one is explicitly supported by the context.
- Do not dynamically tune the threshold during a live task. Threshold evolution is a policy-maintenance concern, not a runtime behavior.

## Standard output

Return:

```yaml
threshold_check:
  threshold_policy_ref: decision-logging-threshold/v1
  impact_radius: 0|1|2
  reversibility: 0|1|2
  uncertainty: 0|1|2
  tradeoff_intensity: 0|1|2
  longevity: 0|1|2
  decision_score: <0..10>
  mandatory_overrides: []
  override_applied: true|false
  usefulness_gate: true|false
  should_log: true|false
  rationale: <short reason explaining the decisive factors>
```

## Output expectations

The `rationale` field should briefly explain:

- why the candidate was or was not useful later
- which dimensions drove the score
- whether an override applied

Good rationale examples:

- `System-wide release-scope choice under legal risk; explicit trade-offs and persistent downstream effects.`
- `Human escalation boundary; logged by mandatory override despite modest score.`
- `Deterministic formatting cleanup with no meaningful alternatives; treated as execution noise.`

## What this skill is optimizing for

Optimize for the **signal-to-noise ratio of decision memory**:

- not all reasoning should be stored
- only meaningful inflection points should be stored
- a logged decision should be reusable or auditable later

Treat thresholding as a **precision filter**, not a trace dump.
