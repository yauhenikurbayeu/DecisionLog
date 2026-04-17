---
seconds_timestamp: 1776374278
model: Claude Opus 4.6
date: 2026-04-16
type: release-decision
---

# Release Decision: AI Document-Classification Feature

## Context

- Offline benchmark quality looks strong.
- Real-user pilot feedback is mixed.
- Two high-severity misclassifications were reported on sensitive documents.
- Sales has promised the feature to one strategic prospect.
- Engineering says fixing the issue might take one week, or might uncover deeper model problems.

## Options Evaluated

| Option | Assessment |
|--------|-----------|
| **A) Release now** | Unacceptable. Two high-severity misclassifications on sensitive documents represent a trust and liability risk that strong benchmarks cannot offset. Mixed pilot feedback confirms the issue is not isolated. |
| **B) Release to a limited beta** | Strong candidate. Contains blast radius while preserving sales timeline flexibility. However, "limited beta" on sensitive documents still carries reputational risk if misclassifications recur with real customer data. |
| **C) Delay release by two weeks for more testing** | Strong candidate. Gives engineering one week to fix + one week to validate. Addresses the root cause rather than containing symptoms. Sales timeline is impacted but manageable with transparent communication. |
| **D) Cancel the release for this quarter** | Disproportionate. The benchmarks are strong and the issue may be fixable in one week. Cancellation abandons sunk investment and breaks a sales commitment without sufficient justification. |

## Final Decision

**Selected: B) Release to a limited beta — with explicit escalation gates and a hard constraint on sensitive-document classes.**

### Rationale

The binding constraint is the intersection of **trust risk** (high-severity misclassifications on sensitive documents) and **incomplete evidence** (engineering cannot yet scope the fix). A full release (A) is irresponsible given the known failures. A two-week delay (C) is reasonable but assumes the fix is achievable in that window — engineering flagged it might uncover deeper problems, making a time-boxed delay a gamble. Cancellation (D) is disproportionate.

A limited beta threads the needle:

1. **Exclude sensitive document classes** from the beta entirely until the misclassification root cause is understood and fixed. This removes the highest-severity risk vector.
2. **Scope the beta to the strategic prospect** (satisfying the sales commitment) plus 2–3 additional willing participants, with explicit informed-consent that this is a beta with known limitations.
3. **Set a hard escalation gate at two weeks:** if engineering has not resolved the misclassification issue by then, escalate to product leadership for a go/no-go on broader release. Do not auto-promote from beta.
4. **Instrument the beta heavily** to collect real-user classification accuracy data that resolves the mixed pilot feedback signal.

### Escalation

This decision explicitly recommends **escalation to product leadership** if any of the following occur during the beta:
- Any misclassification on a sensitive document within the beta population.
- Engineering determines the fix requires architectural changes (the "deeper model problems" scenario).
- The two-week gate is reached without a confirmed fix.

### Trade-offs and Risks

- Sales commitment is partially met but not fully — the prospect gets a beta, not GA. Sales must set expectations carefully.
- If the beta surfaces more issues, the timeline extends further.
- Excluding sensitive document classes reduces the feature's value proposition in the beta.
- If engineering resolves the issue quickly, the beta period may feel unnecessarily cautious — but caution is warranted given the severity of the reported failures.
