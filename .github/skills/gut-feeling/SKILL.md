---
name: gut-feeling
description: Reviews relevant prior decisions as soft priors before a non-trivial decision and determines whether they should be reused, refined, or overridden based on the current context.
---

# Gut Feeling Skill

## Purpose

Use this skill whenever a task involves judgment and prior decisions may provide useful precedent.

This skill treats prior decisions as soft **gut-feeling priors**:

- they can guide the next decision
- they must not be followed blindly
- current evidence and constraints still win

---

## Core behavior

Before making a non-trivial decision:

1. review the relevant prior decisions available in the current workflow
2. identify which prior decisions are materially similar
3. decide whether each relevant prior should be reused, refined, or overridden
4. carry forward only the parts that still fit the current context
5. record how prior reasoning did or did not influence the final choice

---

## What counts as a similar prior decision

Treat a prior decision as potentially relevant if it overlaps with the current task in one or more of these ways:

- same domain or problem type
- same or similar options being compared
- same stakeholders, constraints, or risk profile
- same files, modules, workflows, or deliverable type
- same ambiguity pattern, escalation boundary, or trade-off shape

If no prior entry is meaningfully similar, proceed normally and note that no useful prior was found.

---

## Reuse, refine, or override

### Reuse

Reuse a prior when the current context is materially similar and the earlier judgment still fits.

### Refine

Refine a prior when part of the earlier judgment still applies, but some constraints, evidence, or priorities have changed.

### Override

Override a prior when the context has changed in a meaningful way or current evidence clearly points elsewhere.

Prior decisions are guidance, not binding rules.

---

## Traceability expectation

When prior reasoning materially influenced a new decision, preserve that traceability by capturing:

- which prior decisions were considered
- what reusable prior or gut feeling, if any, carried forward
- why the prior was reused, refined, or overridden

If no prior meaningfully influenced the choice, record that explicitly rather than inventing influence.

---

## Review loop

Use this internal check:

1. Is this a non-trivial decision?
2. Are any prior decisions relevant?
3. Which prior decisions are materially similar?
4. Should they be reused, refined, or overridden?
5. What effect did that have on the final decision?

---

## Guardrails

- Do not confuse prior consensus with truth.
- Do not let stale precedent override stronger current evidence.
- Do not claim prior influence when none actually existed.
- Do not delete or rewrite history just because a new decision differs.

---

## Scope boundary

This skill defines how to think about prior-decision influence.

It does not define:

- how prior decisions are retrieved
- where they are stored
- how they are persisted after the new decision

Those procedural details belong to the active agent or companion skill.
