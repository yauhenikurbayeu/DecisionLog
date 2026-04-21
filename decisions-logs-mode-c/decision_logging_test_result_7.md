---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

## Context

A new workflow automation engine is being introduced into an existing document platform. The decision affects multiple downstream services, deployment topology, and operational runbooks, and anchors two quarters of implementation planning. Engineering capacity is bounded: the team can realistically ship either a shared async queue (Option B) or dedicated per-customer worker pools (Option C) within the current quarter, but not both. The choice must balance immediate delivery feasibility, tenant isolation quality, operational overhead, and long-term architectural flexibility.

---

## Options Evaluated

| Option | Description | Assessment |
|--------|-------------|------------|
| **A** — Synchronous execution | Execute each workflow inline within the user request lifecycle | Simplest to implement short-term, but directly couples workflow latency to user-facing response time. Brittle failure handling; any workflow hang or retry degrades the user experience. Not viable for a production automation platform. |
| **B** — Shared async queue with retries | Route all workflow jobs through a single durable queue with retry semantics (e.g., DLQ, exponential backoff) | Decouples execution from user requests. Provides at-least-once delivery durability. Fits within current-quarter engineering capacity. Cheaper and faster to ship than C. Accepted risk: noisy-neighbor behavior under per-tenant traffic spikes. |
| **C** — Dedicated per-customer worker pools | Provision isolated worker pools per customer for full tenant separation | Cleaner isolation model, better suited to future scale and enterprise SLAs. However, operationally heavier (pool lifecycle management, resource sizing per tenant, higher infra cost). Beyond realistic current-quarter capacity. |
| **D** — Postpone engine; keep manual workflows | Defer the automation engine to a future quarter | Eliminates near-term engineering risk but forfeits two quarters of platform value delivery. Strategically weak given competitive positioning and roadmap commitments. |

---

## Final Decision

**Selected: Option B — Shared asynchronous queue with retries**

### Rationale

Option B is selected as the execution model for the new workflow automation engine.

By routing workflow jobs through a shared async queue, execution is fully decoupled from user-facing request handling. Users receive immediate acknowledgment while workflows run durably in the background. The at-least-once delivery guarantee, combined with retry logic and a dead-letter queue (DLQ), ensures resilience against transient failures without requiring complex in-request error recovery.

Critically, Option B is achievable within current quarter engineering capacity. Option C, while architecturally superior for multi-tenant isolation, requires substantially more infrastructure work (per-tenant pool provisioning, lifecycle management, resource autoscaling) and is not realistic this quarter. Option B does not foreclose the path to C — the queue abstraction layer can be evolved into per-tenant worker routing once the platform has the operational maturity and demand signal to justify it.

### Accepted Trade-offs

- **Noisy-neighbor risk:** Under heavy per-tenant traffic spikes, a shared queue may cause job delay for other tenants. This is mitigated short-term via queue priority tiers and rate limiting per tenant, and resolved longer-term by migrating to Option C.
- **Queue operational burden:** Running a production async queue requires investment in DLQ monitoring, alerting, replay tooling, and runbook documentation. This overhead is accepted as necessary for a durable delivery model.

### Prior Decision Alignment

This decision is directly reinforced by a prior entry in `decision-log.md`:

> **Entry:** "Default workflow execution model — shared async queue (Option B)" | **Date:** 2026-04-21
>
> *"Option B decouples workflow execution from user requests, provides durable at-least-once delivery, fits within the current quarter, and preserves a later migration path to per-tenant pools if needed."*

The current context is materially identical to the prior decision context — same platform, same option set, same engineering capacity constraints. No new information warrants overriding the prior. The prior is **reused** with full confidence.
