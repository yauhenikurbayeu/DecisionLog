---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

# Completion Message Selection

## Context

An internal developer helper script needed a single completion message to print after local test setup finishes. The message is internal-only, affects no user-facing output, has no downstream product or workflow consequence, and can be changed instantly at any time. All four options were considered equally acceptable.

## Options Evaluated

| Option | Message            |
|--------|--------------------|
| A      | `Done.`            |
| B      | `Completed.`       |
| C      | `Setup finished.`  |
| D      | `Ready.`           |

## Final Decision

**Option A — `Done.`**

The shortest, most idiomatic option for an internal completion indicator. With all options explicitly equivalent and no downstream consequence, brevity is the only meaningful differentiator.

**Decision-logging threshold result:** `should_log: false` (score 0/10, usefulness_gate false). This is a trivial cosmetic choice with no trade-offs, no future reuse value, and instant reversibility. No entry was appended to the decision log.
