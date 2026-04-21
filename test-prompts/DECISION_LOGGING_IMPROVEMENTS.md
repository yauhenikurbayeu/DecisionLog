# Decision Logging Improvements Proposal

## Purpose

This document captures useful ideas from [arXiv-1804.05741v4/FINAL_VERSION.md](./arXiv-1804.05741v4/FINAL_VERSION.md) that appear worth applying to the current decision provenance flow reviewed in [DESISION_LOGGING_RESULTS_ANANLYSIS.md](./DESISION_LOGGING_RESULTS_ANANLYSIS.md).

It does not propose replacing the current flow. The current flow is already strong in several important ways:

- it distinguishes meaningful decisions from non-decisions
- it supports reuse, refine, and override behavior
- it separates execution from strict logging
- it preserves a no-decision path
- it already identified some important gaps such as `decision_made_by` / `logged_by`, cold-start contamination, and one-record-per-decision validation

The most useful additional insight from the paper is this:

> the next maturity step is to move from isolated decision logs to explicit decision pipelines

The current system is already good at logging single decisions. The paper suggests making the surrounding flow visible too:

- what inputs led to the decision
- which boundaries were crossed
- who or what handled the decision at each stage
- what downstream effects the decision triggered
- which controls or interventions should fire when provenance reveals risk

## High-Value Improvements

### 1. Expand from decision records to decision pipeline records

The paper repeatedly argues that accountability comes from exposing chains of inputs, processing steps, interconnections, and flow-on effects, not just the final decision itself.

The current flow logs:

- the decision
- rationale
- alternatives
- trade-offs
- prior influence

That is good, but still mostly decision-local.

Add pipeline-level fields around each logged decision:

- `upstream_inputs`: prompt files, source docs, prior decisions, retrieved evidence, constraints
- `processing_stage`: screening, prior review, execution, logging, escalation, post-hoc review
- `downstream_effects_expected`: result file written, escalation triggered, schema validation triggered, later decision likely affected
- `downstream_effects_observed`: actual later artifacts or follow-on decisions linked back to this record

Why this matters:

- better post-hoc debugging when a later result looks wrong
- easier explanation of how one decision influenced the next
- stronger auditability than a standalone rationale paragraph
- closer alignment with the paper's core notion of a "decision pipeline"

### 2. Add explicit actor and boundary provenance

The paper emphasizes that accountability problems often appear at boundaries between systems, components, or organisations. In the current flow, the most important boundaries are agent boundaries:

- `main-provenance`
- `user-task-executor`
- `provenance-log`

The analysis already notes that architecture traceability is the biggest remaining audit gap. The paper reinforces that this is not a cosmetic improvement. It is central.

Add fields such as:

- `decision_made_by`
- `decision_reviewed_by`
- `logged_by`
- `delegated_to`
- `handoff_from`
- `handoff_to`
- `run_id`
- `session_id`
- `result_artifact`

Add a small handoff manifest per run:

```yaml
run_id: 2026-04-18T...
task_received_by: main-provenance
priors_reviewed_by: main-provenance
task_executed_by: user-task-executor
decision_logged_by: provenance-log
result_artifact: decision_logging_test_result....
decision_log_artifact: decision-log.md
```

Why this matters:

- makes the split-agent design auditable instead of only implied
- exposes where a decision changed shape across handoffs
- supports later trust, validation, and blame-free debugging

### 3. Normalize reuse applicability and expiry

The paper repeatedly stresses that provenance is useful only when context is visible. A prior should not be reused just because it exists. It should be reused because the earlier context still fits.

The current flow already has reuse / refine / override. The next step is to define reuse boundaries more explicitly.

Add fields such as:

- `applicability_conditions`
- `assumptions`
- `do_not_reuse_when`
- `review_by`
- `staleness_risk`
- `confidence_before_prior_review`
- `confidence_after_prior_review`

Example:

```yaml
applicability_conditions:
  - same option set
  - similar trust constraints
  - no new compliance blocker
do_not_reuse_when:
  - enterprise-risk profile changed materially
  - model safety requirements tightened
review_by: 2026-06-01
staleness_risk: medium
```

Why this matters:

- reduces consensus cascade and recency bias
- makes override decisions easier to justify later
- turns "gut feeling" into bounded, reviewable precedent

### 4. Add observed provenance, not just self-reported provenance

One of the most useful implementation insights in the paper is the distinction between disclosed provenance and observed provenance.

Right now the current system is mostly disclosed provenance:

- the agents say what they reviewed
- the agents say what decision they made
- the agents say how prior reasoning influenced them

That is necessary, but it is not enough for high trust.

Add a lightweight observed layer captured automatically by the orchestration layer or test harness:

- which files were read
- which prompt file was used
- which prior decision timestamps were retrieved
- whether delegation occurred
- which artifacts were written
- when validation succeeded or failed

Why this matters:

- catches cold-start contamination directly
- reduces reliance on agent memory or honesty
- makes provenance more trustworthy without replacing narrative reasoning

### 5. Add proactive policy and intervention hooks

The paper is clear that provenance should not only help after the fact. It should also drive intervention, policy enforcement, and risk mitigation at run time.

That idea maps very well to the current flow.

Potential triggers:

- if multiple material decisions are merged into one log record, block final logging
- if cold-start mode touches prior logs, raise a contamination flag
- if a prior is reused despite a `do_not_reuse_when` match, require override justification
- if a `Critical` decision lacks alternatives or trade-offs, fail validation
- if an escalation-sensitive prompt produces no escalation analysis, require explicit review

Add fields such as:

- `triggered_policies`
- `validation_status`
- `intervention_taken`
- `escalation_required`
- `escalation_reason`

Why this matters:

- shifts provenance from passive memory to active governance
- aligns directly with the paper's ex ante accountability idea
- would help prevent known failure modes already observed in the analysis

### 6. Track downstream effects, not just upstream reasoning

The paper emphasizes flow-on effects. The current logs capture why a decision was made, but only weakly capture what it caused later.

Add fields such as:

- `expected_follow_on_actions`
- `linked_future_decisions`
- `superseded_by`
- `caused_artifacts`
- `affected_workflow_steps`

Example:

```yaml
expected_follow_on_actions:
  - write result artifact
  - inform release recommendation
  - constrain later safety guardrail choice
linked_future_decisions:
  - decision-2026-04-18-002
  - decision-2026-04-18-003
```

Why this matters:

- makes multi-decision tasks easier to audit
- supports real decision chains rather than disconnected entries
- helps explain why an earlier weak choice propagated into later outputs

### 7. Add trust and integrity metadata

The paper is explicit that provenance is only useful if it is reliable. The current flow is already improving structure, but trustworthiness can be made stronger.

Add fields such as:

- `decision_id`
- `parent_decision_ids`
- `log_entry_hash`
- `schema_version`
- `validated_at`
- `validation_tool`
- `completeness_score`

Why this matters:

- easier to detect tampering or accidental drift
- easier to reconcile logs across iterations
- creates a better foundation for future graph-based provenance

### 8. Create audience-specific views

The paper notes that provenance is hard to use if every stakeholder sees the same raw representation. Different users need different views.

The current flow is optimized for the internal experimenter or system designer. That is fine for now, but a more mature system should produce multiple views from the same underlying provenance.

Useful views:

- `operator view`: concise decision summary for daily use
- `audit view`: full schema, boundary trace, and validation metadata
- `research view`: comparable fields for experiment analysis
- `user-facing view`: very short natural-language explanation of why a decision was made

Why this matters:

- improves usability without weakening rigor
- aligns with the paper's focus on oversight, empowerment, and multiple audiences

## Proposed Schema Extension

This is a compact additive extension to the current schema, not a replacement:

```yaml
decision_id: decision-2026-04-18-001
schema_version: decision-log/v2
run_id: run-2026-04-18-001
session_id: session-2026-04-18-a

decision_made_by: main-provenance
decision_reviewed_by: main-provenance
logged_by: provenance-log
delegated_to: user-task-executor

upstream_inputs:
  prompt_files:
    - DECISION_LOGGING_TEST_PROMPT4-multi-decision.md
  prior_decisions_reviewed:
    - 1776369600
    - 1776376800
  source_artifacts:
    - decision-log.md

processing_stage: execution
applicability_conditions:
  - same option set
  - materially similar constraints
do_not_reuse_when:
  - compliance constraints changed
  - prompt explicitly requests cold-start isolation
review_by: 2026-06-01
staleness_risk: medium

confidence_before_prior_review: medium
confidence_after_prior_review: high

expected_follow_on_actions:
  - write result artifact
  - trigger secondary-decision logging
linked_future_decisions:
  - decision-2026-04-18-002

triggered_policies:
  - one-record-per-decision
  - cold-start-isolation
validation_status: passed
completeness_score: 1.0
log_entry_hash: sha256:...
```

## Priority Roadmap

### Near term

These are the highest-value changes to make first:

1. Add `decision_made_by`, `logged_by`, `run_id`, and `session_id`
2. Add normalized `prior_decisions_reviewed` and `prior_corpus_reviewed`
3. Add `applicability_conditions`, `do_not_reuse_when`, and `review_by`
4. Add `validation_status` and `completeness_score`
5. Add a simple handoff manifest per run

### Medium term

These strengthen the flow materially:

1. Add automatic observed capture of files read, priors retrieved, and artifacts written
2. Add `expected_follow_on_actions` and `linked_future_decisions`
3. Add policy-trigger hooks for contamination, merged decisions, and missing critical fields
4. Add `schema_version` and stable `decision_id`

### Later

These are worth doing once the core flow is stable:

1. Build a graph view of linked decisions rather than only append-only markdown
2. Add audience-specific rendered views
3. Add retention and summarization rules for long-term logs
4. Add cross-run aggregation of recurring patterns and known override conditions

## Expected Impact On Current Known Gaps

This proposal would help with problems already surfaced in the current analysis:

- `cold-start contamination`
  - addressed by observed retrieval logs and policy-trigger hooks
- `merged multi-decision entries`
  - addressed by pipeline stages, one-record-per-decision enforcement, and linked future decisions
- `weak architecture traceability`
  - addressed by actor and handoff provenance
- `recency bias in warm reuse`
  - addressed by applicability and expiry metadata
- `schema compliance without strong trust guarantees`
  - addressed by validation metadata, completeness scoring, and hashes
- `good rationale but weak flow visibility`
  - addressed by upstream and downstream provenance fields

## Bottom Line

Yes, the paper contains useful insights worth applying.

The most important one is not "log more decisions." The current flow already does a fair amount of that.

The most important next step is:

- make the current decision log aware of the full decision pipeline

In practice, that means:

- capture who handled the decision at each boundary
- capture what inputs and priors actually flowed into it
- capture when reuse should no longer apply
- capture what the decision caused afterward
- add lightweight observed traces and policy hooks so the provenance can drive intervention, not just retrospective explanation

That would move the current system much closer to the paper's stronger form of decision provenance: accountable, auditable, reusable, and operational rather than just well-structured.
