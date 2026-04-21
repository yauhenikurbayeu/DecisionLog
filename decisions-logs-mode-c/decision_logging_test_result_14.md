---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

# Milestone-1 Offline Capability Scope Assumption

## Context

The mobile document-review feature is entering milestone-1 planning this week. The written spec is **silent on whether offline capability is required** in milestone 1 — it neither includes nor excludes it. This silence creates a planning ambiguity with material consequences: the scope of offline support directly affects implementation effort, test surface, architectural constraints, and which later design decisions remain open.

The team faces a hard deadline: planning **must complete this week**. An open-ended wait for product-manager clarification is not viable. A decision must be made explicitly, with enough visibility that the PM can review and override the assumption before implementation begins.

The stakes are high in both directions:
- **Under-scoping** offline (treating it as out of scope) risks a PM correction mid-sprint requiring scope-increase rework.
- **Over-scoping** offline (treating it as in scope) risks pulling in significant complexity — sync conflict handling, local storage, test matrix expansion — that may not be wanted for an initial milestone.

Because the spec is silent and the team cannot wait, a **conservative engineering default** is the appropriate resolution posture.

---

## Options Evaluated

### Option A — Offline viewing is out of scope for milestone 1

**Implication:** The feature ships as online-only for M1. No local caching layer, no offline state management, no sync logic. Simplest scope. If offline turns out to be required, it can be added in a subsequent milestone with lower rework cost than the reverse path (build-then-remove).

**Effort:** Minimal incremental effort beyond core feature. Standard network-availability assumptions apply throughout.

**Risk:** PM may want offline from day one; however, this assumption is explicit and reviewable before implementation locks in.

---

### Option B — Read-only offline viewing is in scope for milestone 1

**Implication:** Documents viewed online are cached and readable offline. No editing or sync-write path required. Moderate added complexity: local storage layer, cache invalidation policy, offline state indicators in the UI.

**Effort:** Moderate. Requires decisions about cache size, TTL, eviction policy, and network-state detection. Test matrix expands to cover online/offline transitions.

**Risk:** No explicit requirement justifies this addition. If the PM doesn't need it, the team absorbs non-trivial effort with no customer value delivered in M1.

---

### Option C — Full offline editing is in scope for milestone 1

**Implication:** Users can read and edit documents while offline; changes sync when connectivity is restored. Requires conflict resolution logic, a sync engine, a local write store, and full offline test coverage.

**Effort:** High. Sync conflict handling alone is a significant engineering and QA workload. Would likely dominate M1 scope and crowd out other features.

**Risk:** Extremely high over-scope risk given spec silence. Sync conflict bugs are among the most expensive to fix post-launch. This option should not be defaulted into without an explicit requirement.

---

### Option D — Stop planning and wait for PM clarification

**Implication:** Planning is paused until the PM responds with an explicit offline stance.

**Effort:** Zero technical effort now; high schedule cost if PM is unavailable or delayed.

**Risk:** Directly violates the planning deadline constraint. The team **must complete planning this week**, making this option inviable as stated. D is appropriate only if no deadline existed — it is not applicable here.

---

## Final Decision

**Selected: Option A — Offline viewing is out of scope for milestone 1.**

### Rationale

This decision directly reuses the prior recorded in `./decision-log.md` under *"Milestone-1 offline capability scope assumption"*, which addressed the identical scenario: spec silent on offline capability, four-option choice, planning deadline this week. The prior concluded Option A for the same reasons applied here.

The **conservative engineering default** is the correct resolution when a spec is silent on a capability and no stakeholder has explicitly requested it:

1. **Silence is not a requirement.** The absence of offline in the spec is not accidental omission — it is the appropriate signal that offline has not been prioritized for M1. Defaulting to inclusion would invert that signal.
2. **Asymmetric rework cost.** Adding offline capability in M2 (if the PM requests it) carries lower rework cost than discovering in sprint that full or partial offline was not wanted and needing to strip it out.
3. **Keeps M1 scope minimal and predictable.** Online-only is the simplest architectural baseline. It avoids pulling in storage, cache, sync, and offline-state UI work with no backing requirement.
4. **Explicit and reviewable.** By documenting this assumption now — before implementation begins — the PM has a clear, concrete artifact to review and override before it becomes costly to change.

### Prior Influence

| Field | Value |
|---|---|
| Prior reviewed | `./decision-log.md` — Milestone-1 offline capability scope assumption |
| Reuse disposition | **Reused** |
| Explanation | The prior is directly applicable: same feature domain (mobile document review), same four options (A/B/C/D), same binding constraint (spec silent, deadline this week). No contextual factors distinguish this instance from the prior. The prior rationale holds without modification. |

### Accepted Trade-offs

- If the PM intended offline from day one, this assumption creates a correction loop. That correction cost is accepted as lower than the cost of building unrequested offline infrastructure.
- The explicit assumption documented here is the mitigation: the PM can override before coding begins.

### PM Review Trigger

This assumption is **explicit and time-bounded**. If the PM does not override it before implementation begins, Option A is confirmed. If the PM overrides to Option B (read-only offline), the scope delta should be evaluated for M1 fit or deferred to M2. Option C (full offline editing) is recommended only for a dedicated milestone given its complexity.
