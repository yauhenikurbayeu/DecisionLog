---
name: decision-log
description:  Decision log skill.
---

# Decision Log Skill

## Purpose

Use this skill whenever you are executing a task that requires judgment, trade-offs, prioritization, selection among alternatives, rejection of options, assumption-making, escalation, or conflict resolution.

For every meaningful decision, you must:

1. detect that a meaningful decisions should be made
2. gather the all decisions context during the execution
3. assess the decisions importance
4. complete the user task execution
5. record the decisions using the specified field schema

---

## When a decision exists

Treat a step as a **decision** if at least one of the following is true:

- you selected one option over another
- you rejected an alternative
- you made or accepted an assumption
- you prioritized one action, requirement, or constraint over another
- you resolved ambiguity
- you introduced a trade-off
- you chose to defer something
- you escalated or decided not to escalate
- you changed the original plan
- you determined that some information was sufficient even if incomplete
- you applied a threshold, rule, or confidence cutoff
- you decided that something was too risky, too costly, too uncertain, or too low-value

Do **not** log every tiny formatting or wording choice. Only log decisions that materially influence the reasoning, output, workflow, scope, risk, or result.

---

## Importance levels

For every detected decision, classify its importance into one of these levels.

### Critical
Use `Critical` if the decision is:
- hard to reverse
- high impact on correctness, safety, compliance, architecture, or business outcome
- likely to affect many downstream steps
- likely to create meaningful risk if wrong

Examples:
- choosing architecture direction
- deciding to skip validation
- selecting a compliance-sensitive processing path
- escalating to human because of low confidence
- resolving a conflict between major constraints

### Important
Use `Important` if the decision is:
- meaningful but reasonably reversible
- influential on workflow quality, prioritization, delivery path, or interpretation
- local to a task but still significant

Examples:
- prioritizing one initiative over another
- choosing one tool over another
- selecting one decomposition approach
- deciding to delay a non-blocking item
- preferring speed vs completeness in a bounded context

### Minor
Use `Minor` if the decision is:
- low risk
- easy to reverse
- unlikely to materially affect downstream outcomes
- mostly execution convenience

Examples:
- ordering of substeps
- naming of internal sections
- minor presentation structure
- choosing between two equivalent low-risk approaches

If unsure between two levels, prefer the higher one.

---

## Importance scoring heuristic

Evaluate each decision on three dimensions:

- **Impact:** how much this affects output, behavior, architecture, risk, or downstream work
- **Reversibility:** how costly or difficult it would be to undo
- **Propagation:** how many later steps depend on it

Interpretation:
- **Critical:** high on at least two dimensions
- **Important:** medium on at least two dimensions
- **Minor:** low on all or almost all dimensions

---

## Logging format

Record each decision using the following structure:

```md
## Decision: <short title> 

- **Timestamp:** <ISO-8601 if available, otherwise "session-time">
- **model:** <AI model name and version if applicable, otherwise "Not explicit">
- **Importance:** Critical | Important | Minor
- **Context:** <what was happening when this decision was made>
- **Decision:** <what was decided>
- **Rationale:** <why this decision was made>
- **Alternatives considered:** <list or short text>
- **Trade-offs / Risks:** <what was sacrificed, accepted, or risked>
- **Affected scope:** <files, modules, workflow steps, deliverable sections, or "session-wide">
```

If decision evolves materially, and mention that it supersedes or refines an earlier one in the rationale, extend the log entry with:

```md
- **Prior related decisions reviewed:** <headings, timestamps, or "None found">
- **Reusable prior / gut feeling:** <what prior pattern, intuition, or precedent influenced the decision, or "None">
- **Why prior was reused, refined, or overridden:** <short explanation, or "Not applicable">
```

If some fields are unknown, keep the field and write `Not explicit`.


---

## Context rules

The `Context` field should briefly capture:
- the immediate task being performed
- the constraint, ambiguity, or trigger
- the point in execution where the decision occurred

Good context examples:
- "While choosing between two data storage approaches for provenance records under cost and auditability constraints."
- "During document processing workflow design when OCR confidence for scanned PDFs was assumed to be inconsistent."
- "While deciding whether to ask for clarification or proceed with a best-effort implementation."

Avoid vague context like:
- "During work"
- "Agent execution"
- "Task processing"

---

## What not to log

Do not log:
- every sentence rewrite
- every cosmetic formatting adjustment
- trivial wording changes
- repeated restatements of the same decision without meaningful evolution

If a decision evolves materially, add a new entry and mention that it supersedes or refines an earlier one in the rationale.

---

## Brevity rule

Keep each decision entry concise but complete. Aim for clarity over verbosity.

Recommended size:
- 1 line for title
- 1 short bullet per field
- 1–3 alternatives at most

---

## Required behavior

When this skill is active, you must not silently make meaningful decisions without evaluating whether they should be logged.

If no meaningful decisions were made, do not create a fake entry.
