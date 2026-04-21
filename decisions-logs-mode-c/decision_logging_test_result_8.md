---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

# Retry Policy Decision — Internal Document-Export Notification Service

## Context

Internal-only document-export notification service. Failure patterns are not yet well understood. The retry policy choice affects multiple worker steps and on-call behavior. It can be changed later, but requires code changes, retesting, and coordination with operations. The chosen policy will likely remain in place for the rest of this quarter unless incidents force a change.

## Options Evaluated

- **A — No retry:** Simple but exposes on-call to transient-failure paging noise for recoverable errors.
- **B — Retry twice with fixed 30-second delay:** Provides predictable protection against transient failures, short total retry window (~1 min), limits queue amplification, preserves failure visibility for on-call. **Selected.**
- **C — Exponential backoff for up to 10 minutes:** Risks suppressing real incidents and hiding failure signals; adds premature sophistication before failure patterns are understood.
- **D — No automatic retry, manual operator rerun:** Adds disproportionate operational burden for a service where some transient failures are expected.

## Final Decision

**Option B — Retry twice with a fixed 30-second delay.**

Rationale: Option B provides conservative, predictable transient-failure protection without hiding real incidents. The fixed 30-second delay keeps the total retry window short (~1 minute), limiting queue amplification and preserving on-call failure visibility. It serves as a sound baseline until operational data is available. A prior decision in the workspace provenance log (decision-log.md) for the same service and option set reached the same conclusion independently; that prior was reviewed and reused.

**Accepted trade-offs:** Outages longer than ~1 minute will still alert on-call quickly (not suppressed). Fixed delay is likely suboptimal long-term; can be revised with operational data. Slightly more implementation complexity than Option A.
