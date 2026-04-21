---
model: Claude Sonnet 4.5
date: 2026-04-21
type: decision-threshold-test
---

## Context

A new workflow automation engine is being introduced into a document platform. The decision governs the default execution model — how workflows are triggered, processed, and recovered from failures. This choice has broad downstream consequences: it shapes multiple services, deployment topology, operational runbooks, and approximately two quarters of implementation roadmap. Engineering capacity is available to build either Option B or Option C this quarter, but not both.

## Options Evaluated

| Option | Summary | Key Strength | Key Risk |
|--------|---------|-------------|----------|
| A | Synchronous execution inside the user request | Simplest to implement | User-facing latency; brittle failure handling |
| B | Single shared async queue with retries | Cheap, fast to ship, production-ready quickly | Noisy-neighbor effect under tenant spikes |
| C | Dedicated per-customer worker pools | Strong isolation, cleaner long-term scale | Operationally heavy; slower to deliver |
| D | Postpone the engine; keep manual workflows | Zero engineering risk this quarter | Defers value; accrues process debt |

### Option A — Synchronous execution
Tightly coupling workflow execution to the HTTP request lifecycle is the fastest path to a demo, but it creates unacceptable risk at production scale: a slow workflow blocks the user thread, failures surface directly as HTTP errors, and retry logic must be re-implemented by every caller. Rejected.

### Option C — Dedicated per-customer worker pools
Provides the cleanest isolation model and avoids noisy-neighbor problems by design. However, it requires per-tenant provisioning logic, scaled-down idle pools, quota management, and more complex operational runbooks. The incremental engineering cost exceeds what can be delivered this quarter without cutting scope elsewhere. Prematurely optimising for multi-tenant isolation before the platform has real tenant load is a classic over-engineering trap. Deferred — remains the recommended migration target once B is validated.

### Option D — Postpone
Introduces no new risk in the short term but forfeits the two-quarter window. Manual workflows accumulate coordination overhead and create implicit coupling that becomes costly to unwind. Rejected.

### Option B — Shared async queue with retries ✅ (selected)
A single well-designed queue (e.g., backed by a managed message broker) with idempotent retry semantics decouples workflow execution from the user request, provides durable delivery, and is operationally well-understood. The noisy-neighbor concern is real but manageable: priority lanes, per-tenant rate limiting, and consumer concurrency caps can be added incrementally without re-architecting the queue itself. Critically, the schema and contract established in B are compatible with a later migration to per-tenant pools (Option C), meaning this choice does not foreclose future isolation.

## Final Decision

**Option B — Use one shared asynchronous queue with retries**

Establish a single shared async queue as the default execution model for the workflow automation engine. The queue should support:
- At-least-once delivery with idempotent job handlers
- Configurable retry backoff with dead-letter routing
- Per-tenant concurrency limits to mitigate noisy-neighbor effects
- Structured job metadata to support future queue-partitioning or pool migration

**Trade-offs accepted:**
- Noisy-neighbor risk under tenant spikes is accepted in exchange for faster delivery and simpler operations. Mitigation is available (concurrency caps, priority lanes) and can be added without redesigning the core model.
- Operational burden of queue infrastructure (monitoring, DLQ alerts, replay tooling) is accepted as preferable to synchronous fragility (A) or idle pool cost (C).

**Migration path:**
Option B is explicitly designed as the first step toward Option C. Once real tenant load data is available, per-tenant pools can be introduced as a routing layer on top of the existing queue contract — minimising rework.
