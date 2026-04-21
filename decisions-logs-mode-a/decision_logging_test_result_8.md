---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

## Context

The subject service is an **internal-only document-export notification service**. Failure patterns are not yet well understood, so the retry policy must be chosen conservatively and with good observability in mind. The decision affects several worker steps and on-call behavior but does not touch the broader product roadmap. Changing this policy later is possible but non-trivial: it requires code changes, retesting, and coordination with operations. The choice is expected to hold for the rest of the current quarter unless incidents force a revision.

Key tensions to resolve:
- **Simplicity vs. resilience**: simpler policies are easier to reason about and operate; more sophisticated policies may recover from transient failures automatically.
- **Queue load vs. operator burden**: aggressive retries amplify load on downstream systems; no retries push burden onto on-call engineers.
- **Predictability vs. adaptability**: fixed delays are easy to reason about; exponential backoff is more adaptive but harder to predict when failure patterns are unknown.

---

## Options Evaluated

### A — No Retry
**Description**: Any failure is immediately propagated as a terminal error. No automatic recovery attempt.

**Trade-offs**:
- ✅ Simplest possible implementation; no retry state to manage.
- ✅ Failures surface immediately to monitoring and on-call.
- ❌ Every transient failure (brief network blip, momentary dependency unavailability) results in a missed notification and an alert — high false-positive on-call noise.
- ❌ Requires operators to manually assess and reprocess each failure, even trivially recoverable ones.
- ❌ For a notification service, zero-tolerance for transient errors is typically too strict without compensating safeguards.

**Verdict**: Rejected. The operator burden and false-positive alert rate are disproportionate for an internal service where many failures are likely transient.

---

### B — Retry Twice with Fixed 30-Second Delay ✅ SELECTED
**Description**: On failure, the system retries the notification up to 2 additional times, waiting 30 seconds between each attempt. After 3 total attempts, the failure is recorded as terminal.

**Trade-offs**:
- ✅ Handles the most common transient failure modes (brief network interruption, momentary upstream unavailability) without operator intervention.
- ✅ Fixed delay is simple, deterministic, and easy to reason about — especially important when failure patterns are not yet understood.
- ✅ Low amplification risk: at most 3 total attempts per notification; 30-second spacing limits burst pressure on downstream systems.
- ✅ Total window (~1 minute) is short enough that failures are still surfaced quickly to on-call.
- ✅ Low implementation complexity; easy to inspect in logs.
- ⚠️ Does not recover from longer outages (> ~1 minute) — those will still page on-call.
- ⚠️ Fixed delay may not be optimal once failure patterns become clearer; likely a candidate for refinement later.

**Verdict**: Selected. Offers the best balance of operational simplicity, resilience against transient failures, low queue amplification, and fast failure visibility given current unknowns.

---

### C — Exponential Backoff for Up to 10 Minutes
**Description**: On failure, retries with increasing wait intervals, continuing for up to 10 minutes before marking terminal.

**Trade-offs**:
- ✅ Most resilient option; recovers from sustained but transient outages automatically.
- ❌ **Failure patterns are not yet understood**: a 10-minute retry window may silently suppress real incidents, delaying on-call awareness substantially.
- ❌ Queue and worker state must be maintained for up to 10 minutes per job — higher operational complexity.
- ❌ If the downstream system is degraded (not fully down), exponential backoff can create bursty retry storms when it recovers.
- ❌ Harder to reason about behavior and debug in production without established baselines.
- ❌ Premature optimization: the sophistication is not yet warranted given the current knowledge gap.

**Verdict**: Rejected. The 10-minute window introduces unacceptable observability risk when failure modes are unknown, and the complexity is not justified at this stage.

---

### D — No Automatic Retry, Operators Must Rerun Manually
**Description**: All retries are handled by human operator action; no automatic recovery is attempted.

**Trade-offs**:
- ✅ Maximum human oversight; useful when retrying blindly could cause side effects (e.g., financial transactions, idempotency concerns).
- ❌ Highest possible operator burden; every transient failure requires manual triage and intervention.
- ❌ Notification services typically have low stakes per event and high idempotency expectations — manual-only retry is disproportionate.
- ❌ Slow recovery: on-call must be available and responsive even for trivially recoverable failures.
- ❌ Does not scale well as notification volume grows.

**Verdict**: Rejected. Manual-only retry is appropriate for high-stakes or non-idempotent operations, not internal notification delivery.

---

## Final Decision

**Selected: Option B — Retry twice with fixed 30-second delay**

### Rationale

Given that failure patterns are not yet well understood, the safest starting point is a **minimal, predictable retry strategy** that handles common transient failures without introducing operational complexity or masking real incidents. Two retries with a fixed 30-second delay meet this bar: they cover brief network blips and momentary downstream unavailability, keep the total retry window short enough that failures still surface quickly to on-call, and impose minimal queue load (at most 3x per notification).

This choice deliberately avoids premature sophistication. Exponential backoff (Option C) would be the right evolution once failure patterns are understood and baselines are established — likely a natural revisit point next quarter or after the first few incidents. The policy is appropriately scoped: it reduces noise without hiding real failures, and it sets a clear, auditable baseline for future tuning.

### Trade-offs Accepted

| Trade-off | Accepted risk |
|---|---|
| Does not recover from outages > ~1 min | Failures surface to on-call, which is the correct behavior at this stage |
| Fixed delay may not be optimal long-term | Acceptable for current quarter; planned revisit after incident data accrues |
| Slightly higher implementation cost vs. no-retry | Minimal; standard retry loop with counter and sleep |

### Alternatives Rejected

| Option | Primary rejection reason |
|---|---|
| A — No retry | Excessive false-positive on-call noise for transient failures |
| C — Exponential backoff (10 min) | Risk of masking real incidents when failure patterns are unknown |
| D — Manual rerun only | Disproportionate operator burden for a notification service |
