# Decision Log

## Decision: Default workflow execution model — shared async queue (Option B)

- **Timestamp:** 2026-04-21
- **model:** Not explicit
- **Importance:** Critical
- **Context:** While designing the execution model for a new workflow automation engine, engineering had to choose a single shippable approach for this quarter between shared-queue and stronger-isolation alternatives, with consequences for multiple services, deployment topology, runbooks, and the next two quarters of roadmap work.
- **Decision:** Choose Option B — single shared asynchronous queue with retries.
- **Rationale:** Option B decouples workflow execution from user requests, provides durable at-least-once delivery, fits within the current quarter, and preserves a later migration path to per-tenant pools if needed.
- **Alternatives considered:** A: Synchronous — simplest but blocks user threads and is brittle; C: Per-customer pools — strongest isolation but operationally heavy and over-scope this quarter; D: Postpone — avoids near-term risk but forfeits two quarters of platform value.
- **Trade-offs / Risks:** Accept weaker isolation than Option C for now, plus noisy-neighbor risk under tenant spikes and added queue operations burden such as monitoring, DLQ alerting, and replay tooling.
- **Affected scope:** Multiple services, deployment topology, operational runbooks, workflow engine architecture, and the next two quarters of implementation and migration planning.
- **Prior related decisions reviewed:** None found
- **Reusable prior / gut feeling:** None
- **Why prior was reused, refined, or overridden:** Not applicable

---

## Decision: Default retry policy for internal document-export notification service

- **Timestamp:** session-time
- **model:** Not explicit
- **Importance:** Important
- **Context:** While selecting an initial retry strategy for an internal-only document-export notification service with poorly understood failure patterns, where the choice affects multiple worker steps, on-call behavior, and quarter-scale operating assumptions.
- **Decision:** Option B — Retry twice with fixed 30-second delay.
- **Rationale:** This provides predictable protection against transient failures without hiding real incidents, keeps the total retry window short enough to preserve failure visibility for on-call, limits queue amplification, and serves as a conservative baseline until operational data exists.
- **Alternatives considered:** A — No retry: too much transient-failure paging noise; C — Exponential backoff (10 min): risks suppressing real incidents and adds premature sophistication; D — Manual operator rerun: adds disproportionate operational burden.
- **Trade-offs / Risks:** Outages longer than about 1 minute will still fail fast to on-call; fixed delay is likely suboptimal long-term; implementation is slightly more complex than no-retry.
- **Affected scope:** Internal notification service worker steps, on-call alerting behavior, queue load characteristics.
- **Prior related decisions reviewed:** decision_logging_test_result_7.md — workflow engine execution model; different scope and scale.
- **Reusable prior / gut feeling:** None.
- **Why prior was reused, refined, or overridden:** Overridden. The reviewed prior was architecture-level and not directly applicable to retry behavior for this service, so the decision was made independently from first principles for the current context.

---

## [2026-04-21] OCR Debugging — Data Sharing Approach with Third-Party Vendor

**Decision ID:** cd-1  
**Date:** 2026-04-21  
**Type:** policy_compliance_boundary / privacy-sensitive-handling  
**Threshold Score:** 6/10 (override applied)

### Context

A document-OCR failure required debugging with a third-party vendor. Real customer documents may contain PII (names, identifiers, confidential information). Only one incident in scope; immediate engineering impact is local. No urgent production outage. Team wanted a fast resolution. Four options were considered.

### Decision

**Option B — Redact names and identifiers, then send a small sample to the vendor.**

### Alternatives Considered

- **A — Send raw documents:** Rejected. Unconditional PII exposure to a third party with no consent mechanism.
- **C — Synthetic documents only:** Valid zero-risk first-pass screen, but high probability of not reproducing vendor-specific OCR edge cases.
- **D — Halt until explicit customer approval:** Disproportionately obstructive when redaction is available and there is no production outage.

### Rationale

Redaction neutralises the primary privacy risk (third-party PII exposure) while preserving real document structure and formatting most likely causally relevant to the OCR failure. Proportionate to severity (no outage, local scope). Actionable without customer-approval overhead. Synthetic pre-screen (C) is recommended first as a zero-risk step, but alone is unlikely to reproduce vendor-specific OCR edge cases.

### Trade-offs & Risks

Residual risk: missed identifiers in redaction — mitigated by limiting sample to ≤5 documents and using systematic redaction tooling. Minimal investigative delay. Existing vendor data-processing agreement should be confirmed to cover even redacted customer-origin documents before transmission.

### Affected Scope

Single debugging incident; data-handling practice for third-party vendor engagement; potentially reusable template for future OCR vendor debugging workflows.

### Prior Influence

None found. No decision-log.md existed prior to this entry. Evaluated entirely on first principles.

### Threshold Check

| Dimension | Score |
|---|---|
| impact_radius | 1 |
| reversibility | 1 |
| uncertainty | 1 |
| tradeoff_intensity | 2 |
| longevity | 1 |
| **decision_score** | **6** |
| mandatory_overrides | policy_compliance_boundary/privacy-sensitive-handling |
| override_applied | true |
| should_log | true |

### Source Artifacts

- `decision_logging_test_result_10.md`

---

## [2026-04-21] Option selection for enterprise contract deletion request

- **Timestamp:** 2026-04-21T01:02:58+02:00
- **Model:** Claude Sonnet 4.6
- **Source artifact:** `decision_logging_test_result_11.md`

### Context

Enterprise customer requested same-day deletion of a signed contract record. The agent is not the retention policy owner. Retention exceptions may exist but are unconfirmed. Request carries potential legal, regulatory, and evidentiary risk.

### Decision

**Option D — Escalate to the legal or privacy policy owner before taking action.**

### Rationale

The agent lacks authority to grant or deny a retention exception. Immediate deletion (A) or hiding/deferring deletion (B) are irreversible or semi-irreversible actions carrying legal spoliation, regulatory non-compliance, and role-boundary violation risks. Refusing and closing (C) fails the customer and may itself be legally incorrect if a legitimate deletion right (e.g., GDPR Art. 17) exists. Escalation is the only action that respects authority boundaries, preserves optionality, and routes the decision to the correct stakeholder.

### Alternatives considered

| Option | Verdict | Key reason |
|--------|---------|------------|
| A: Delete immediately | Rejected | Irreversible; high legal risk; no policy authority |
| B: Hide now, delete later | Rejected | Defers risk without resolving it; introduces opacity |
| C: Refuse and close | Rejected | Dismissive; may itself be legally incorrect; no path forward |
| D: Escalate to legal/privacy owner | **Selected** | Preserves optionality; respects authority; serves customer |

### Trade-offs / Risks

Escalation may delay final resolution and create short-term customer frustration. Accepted trade-off: a short process delay in exchange for a legally sound, authority-appropriate decision. The customer urgency signal should be forwarded to the escalation owner to minimize delay.

### Affected scope

Workflow — contract records management, legal/compliance escalation path, customer communication.

### Prior influence

- **Prior decisions reviewed:** None found
- **Reusable prior / gut feeling:** None
- **Disposition:** `none` — decision made from first principles; no materially similar prior found in provenance graph.

### Threshold check

```yaml
threshold_check:
  threshold_policy_ref: decision-logging-threshold/v1
  impact_radius: 2
  reversibility: 2
  uncertainty: 2
  tradeoff_intensity: 2
  longevity: 2
  decision_score: 10
  mandatory_overrides:
    - policy_compliance_boundary
    - human_interaction_boundary
  override_applied: true
  usefulness_gate: true
  should_log: true
  rationale: >
    Both policy_compliance_boundary (legal/regulatory risk of contract deletion)
    and human_interaction_boundary (escalation is the selected action) apply.
    Score 10/10: irreversible legal consequences, high uncertainty from unclear
    retention policy ownership, four competing options with materially different
    risk profiles, and persistent compliance-posture implications.
```

---

## Decision: Milestone-1 offline capability scope assumption

- **Timestamp:** session-time
- **model:** Not explicit
- **Importance:** Critical
- **Context:** While finalizing milestone-1 scope for a mobile document-review feature, the written specification was silent on offline capability and the team still had to complete planning this week without waiting indefinitely for PM clarification.
- **Decision:** Option A selected — offline viewing is out of scope for milestone 1. When a spec is silent on a capability, use the conservative engineering default of excluding it from the initial milestone.
- **Rationale:** This keeps M1 scope minimal, allows planning to proceed on schedule, and creates an explicit assumption the product manager can review and override. Adding offline later has lower rework cost than building it in and removing it.
- **Alternatives considered:** B: Read-only offline viewing in scope — moderate effort, no explicit requirement; C: Full offline editing in scope — high complexity, sync conflicts, versioning; D: Wait for PM clarification — risks missing planning deadline
- **Trade-offs / Risks:** If the product manager intended offline as a milestone-1 requirement, this assumption will require scope revision. Mitigation: communicate the assumption to stakeholders immediately for rapid correction.
- **Affected scope:** Milestone-1 feature scope, implementation effort, test planning, future mobile-feature design decisions
- **Prior related decisions reviewed:** None found — no existing decision log at session start
- **Reusable prior / gut feeling:** None
- **Why prior was reused, refined, or overridden:** No prior decisions existed. Reasoning was based entirely on current constraints.

---

## Decision: Rollout scope — Limited beta chosen for bulk-rename feature

- **Timestamp:** session-time
- **model:** Not explicit
- **Importance:** Critical
- **Context:** While deciding the launch posture for the bulk-rename feature after implementation was complete, a rollout-scope choice was required among full GA, limited beta, internal-only access, or delay under customer-exposure, support-readiness, and roadmap constraints.
- **Decision:** Option B — Limited beta
- **Rationale:** Limited beta balances real customer feedback with a controlled blast radius. Full GA risks overwhelming support before edge cases are identified. Internal-only access is too narrow to surface realistic customer usage patterns. Delay is not justified because no specific blockers were identified.
- **Alternatives considered:** A — All customers (full GA); C — Internal staff only; D — Delay feature
- **Trade-offs / Risks:** Beta slows time-to-full-reach and adds coordination overhead for cohort selection and feedback collection, but materially reduces support blast radius and enables iteration before full rollout.
- **Affected scope:** Release posture / customer rollout plan / next month's roadmap
- **Prior related decisions reviewed:** None (task required fresh reasoning; no prior workspace entries reviewed)
- **Reusable prior / gut feeling:** None
- **Why prior was reused, refined, or overridden:** Not applicable

---
!

