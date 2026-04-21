# Decision Logging Threshold Policy
## Purpose
Provide the narrative rationale behind the `decision-logging-threshold` skill and explain why the threshold policy is shaped the way it is.

The canonical runtime contract now lives in [`.github/skills/decision-logging-threshold/SKILL.md`](./.github/skills/decision-logging-threshold/SKILL.md).

The goal is to capture **high-signal decision points** while avoiding execution noise.

A "major decision" is not defined by importance alone, but by its **impact on future reasoning, auditability, and reuse as a prior**.

---

## Core Principle

Log decisions that:
- Influence future decisions
- Encode non-trivial trade-offs
- Cannot be reliably reconstructed later

Do NOT log routine execution steps.

---

## Decision Scoring Model

For every candidate decision, evaluate the following dimensions:

### 1. Impact Radius
- 0 → Local (single step, no external effect)
- 1 → Cross-component (affects multiple steps/modules)
- 2 → System-wide (affects architecture, flow, or outcomes broadly)

### 2. Reversibility
- 0 → Easily reversible
- 1 → Reversible with cost
- 2 → Hard or irreversible

### 3. Uncertainty / Ambiguity
- 0 → Deterministic (clear correct answer)
- 1 → Some judgment required
- 2 → High ambiguity or incomplete information

### 4. Trade-off Intensity
- 0 → No real alternatives
- 1 → Minor alternatives
- 2 → Explicit competing options with consequences

### 5. Longevity of Effect
- 0 → Ephemeral (short-lived)
- 1 → Medium-lived
- 2 → Persistent or long-term

---

## Logging Threshold Rule

Calculate:

decision_score = sum(all dimensions)

### Log the decision IF:
decision_score >= 5

Otherwise:
→ Do NOT log (treat as execution step)

---

## Mandatory Logging Overrides

Always log a decision regardless of score if:

### 1. Policy / Compliance Boundary
- Security-related decisions
- Privacy-sensitive handling
- Financial or legal implications
- Business rule enforcement

### 2. Human Interaction
- Human override
- Escalation
- Explicit approval or rejection

---

## Explicit Non-Decision Cases (Do NOT Log)

Do NOT log the following:

- Deterministic transformations (formatting, parsing, mapping)
- Retrieval operations without reasoning (e.g., top-k similarity)
- Mechanical execution steps
- Repeated micro-decisions within an already chosen strategy
- Low-level tool calls without alternatives considered

---

## Minimum Candidate Snapshot Before Provenance Handoff

Before handing a decision to the provenance layer, the threshold step only needs a lightweight candidate snapshot such as:

{
  "decision": "What was chosen",
  "context": "Relevant inputs and situation",
  "alternatives": ["Option A", "Option B"],
  "rationale": "Why this option was selected",
  "constraints": ["Constraint 1", "Constraint 2"],
  "assumptions": ["Assumption 1"],
  "confidence": "low | medium | high",
  "owner": "agent | human",
  "timestamp": "ISO-8601"
}

Keep it concise. Avoid unnecessary verbosity.

The full persisted decision schema lives in `decision-provenance`. This section is only the minimal input needed to score whether a decision should be logged at all.

---

## Behavioral Guidance

Before logging, ask:

"Would this decision be useful to reuse as a prior or to audit later?"

If NO → do not log  
If YES → log if threshold or override conditions are met

## Ownership Boundary

The threshold policy owns:

- the dimensions
- override rules
- the scoring formula
- the final log / no-log gate

The provenance layer may store the threshold result for auditability, but should not restate or fork the threshold logic.

## Recommended Threshold Output

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
  rationale: <short reason>
```

---

## Optimization Loop (Self-Improvement)

Agents should implicitly learn:

- If missing a logged decision leads to failure → increase sensitivity
- If logged decisions are never reused → decrease sensitivity

Goal: maximize **signal-to-noise ratio of decision memory**

---

## Summary

- Not all reasoning is worth storing
- Only log **decision inflection points**
- Optimize for **future reuse and explainability**
- Treat decision logging as a **precision filter, not a trace dump**
