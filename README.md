# Decision Provenance Workspace

This workspace is a practical exploration of my Decision Provenance ideas and how they can be applied to real agent workflows.

It is inspired by the principles behind [The Provenance Manifesto](https://provenancemanifesto.org/): decisions should be treated as first-class artifacts, preserved together with their context, allowed to evolve without erasing history, and used to create governed memory for human and AI collaboration.

This repository turns that high-level idea into a working setup for running AI tasks with an auditable decision trail.

Important platform note:

- the implementation in this repository is currently GitHub Copilot-specific
- it uses GitHub Copilot-style `.github/agents`, `.github/skills`, and hook-oriented workspace conventions
- the overall pattern is portable to other agent platforms such as Codex, Claude, or custom orchestrators
- what is GitHub-specific here is mostly the packaging and runtime conventions, not the decision-provenance concept itself

Instead of only producing a final answer, the workspace is designed to:

- complete the user task
- identify the meaningful decisions made during that task
- review relevant prior decisions as soft guidance
- append structured decision records to a shared provenance log

The current setup uses a split-agent workflow:

- `main-provenance` coordinates the task and reviews prior decisions
- `user-task-executor` does the actual task work
- `provenance-log` writes strict, structured entries to `decision-log.md`

This README explains what the workspace is, how to use it, what outputs to expect, and why the recommended model split is set up this way.

## Platform scope

This workspace should be read in two layers:

1. Concept layer
   Decision provenance, prior-decision reuse, structured logging, and role-split agent workflows are platform-agnostic ideas.

2. Implementation layer
   This repository packages those ideas using GitHub Copilot workspace conventions, so the checked-in files are directly suited to GitHub Copilot-style agent environments.

If you want to port this to another platform such as Codex or Claude, the main pieces to preserve are:

- a coordinator that reviews prior decisions before non-trivial choices
- an execution agent that does the actual task work
- a strict logger that writes structured decision entries
- a persistent decision log that can act as reusable memory
- a rule set for deciding when priors should be reused, refined, or overridden

## What "decision provenance" means here

In this workspace, decision provenance means recording the important choices made during task execution, together with enough context to understand them later.

A good provenance record captures:

- what decision was made
- when and why it was made
- which alternatives were considered
- what trade-offs or risks were accepted
- what part of the task or workflow was affected
- whether an earlier decision was reused, refined, or overridden

The goal is not to log every tiny wording change. The goal is to preserve the decisions that materially shaped the outcome.

## What this workspace is for

This workspace is a good fit when you want an agent to make or support non-trivial judgments such as:

- product prioritization
- rollout and release decisions
- safety and guardrail choices
- escalation decisions
- scope trade-offs
- decisions that should be reusable as future precedent

It is not meant to force logging onto trivial tasks. If a task is only a rewrite, cleanup, or formatting pass with no meaningful judgment, the correct behavior is to do the task and skip decision logging.

## What users get as a result

When you use this workspace as intended, you typically get two outputs:

1. The requested task artifact
   For the included examples in `test-prompts/`, this is usually a file named like `decision_logging_test_result[seconds timestamp].md`.

2. Structured decision-log entries in [`decision-log.md`](./decision-log.md)
   These entries capture the important decisions made while producing the task artifact.

Important behavior:

- one threshold-qualified decision should be logged as one decision entry
- multi-decision tasks should produce multiple entries (one per threshold-qualified decision)
- prior decisions are treated as soft priors, not hard rules
- if no candidate decision passes the threshold, no fake log entry should be created

## How the workflow works

```text
User task
  -> main-provenance
      -> review context and relevant prior decisions
      -> apply gut-feeling skill: reuse / refine / override
      -> delegate task delivery to user-task-executor
      -> normalize candidates + apply decision-logging-threshold
      -> delegate strict logging to provenance-log
  -> task artifact + structured decision log
```

In practice, the flow is:

1. `main-provenance` inspects the user request and relevant workspace files.
2. If the task is non-trivial, it reviews relevant prior decisions in `decision-log.md`.
3. It applies the `gut-feeling` skill to decide whether those priors should be reused, refined, or overridden.
4. It sends the actual work to `user-task-executor`.
5. After execution, it normalizes explicit candidate decisions and applies `decision-logging-threshold`.
6. It sends only threshold-qualified decisions to `provenance-log`.
7. `provenance-log` appends schema-compliant entries to `decision-log.md` (and re-checks the threshold if needed).
8. If no candidate decision passes the threshold, it records nothing rather than inventing provenance.

## Workspace structure

- [`.github/agents/main-provenance.agent.md`](./.github/agents/main-provenance.agent.md)  
  Main entry point and provenance-aware coordinator.
- [`.github/agents/user-task-executor.agent.md`](./.github/agents/user-task-executor.agent.md)  
  Execution-focused worker that completes the user task.
- [`.github/agents/provenance-log.agent.md`](./.github/agents/provenance-log.agent.md)  
  Strict logger that appends decision entries.
- [`.github/skills/gut-feeling/SKILL.md`](./.github/skills/gut-feeling/SKILL.md)  
  Defines how prior decisions are reviewed and reused.
- [`.github/skills/decision-log/SKILL.md`](./.github/skills/decision-log/SKILL.md)  
  Defines what counts as a meaningful decision and how it must be logged.
- [`.github/skills/decision-logging-threshold/SKILL.md`](./.github/skills/decision-logging-threshold/SKILL.md)  
  Deterministic scoring gate for deciding whether a candidate decision is major enough to persist.
- [`threshold/decision-logging-threshold-policy.md`](./threshold/decision-logging-threshold-policy.md)  
  Narrative rationale for why the threshold policy is shaped the way it is.
- [`threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md`](./threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md)  
  Threshold-focused validation plan and pass criteria.
- [`threshold/DECISION-LOGGING-THRESHOLD-TEST-RESULT.md`](./threshold/DECISION-LOGGING-THRESHOLD-TEST-RESULT.md)  
  Observed results across clean-room and rerun modes.
- [`decision-log.md`](./decision-log.md)  
  Default decision memory for the workspace.
- [`test-prompts/`](./test-prompts/)  
  Example scenarios that exercise the workflow.
- [`decisions-logs-test-1/`](./decisions-logs-test-1/)  
  Saved test results and decision-log artifacts for iteration 1.
- [`decisions-logs-test-2/`](./decisions-logs-test-2/)  
  Saved test results and decision-log artifacts for iteration 2.
- [`decisions-logs-test-3/`](./decisions-logs-test-3/)  
  Saved test results and decision-log artifacts for iteration 3.
- [`decisions-logs-test-4/`](./decisions-logs-test-4/)  
  Saved test results and decision-log artifacts for iteration 4.
- [`decisions-logs-mode-a/`](./decisions-logs-mode-a/)  
  Saved threshold-suite artifacts for Mode A (clean-room run).
- [`decisions-logs-mode-b/`](./decisions-logs-mode-b/)  
  Saved threshold-suite artifacts for Mode B (prior-rich control rerun).
- [`decisions-logs-mode-c/`](./decisions-logs-mode-c/)  
  Saved threshold-suite artifacts for Mode C (mixed-order resilience run).
- [`DESISION_PROVENANCE_EVOLUTION.md`](./DESISION_PROVENANCE_EVOLUTION.md)  
  Background analysis used to shape the current design and model recommendations.

## Agents

| Agent | Recommended model | User-facing? | Purpose |
|---|---|---:|---|
| `main-provenance` | `Claude Opus 4.6` | Yes | Reviews context and priors, coordinates execution, thresholds candidate decisions, and routes logging |
| `user-task-executor` | `Claude Sonnet 4.6` | Usually no | Completes the requested work and reports candidate decisions made |
| `provenance-log` | `GPT-5.4` | Usually no | Writes strict, structured decision entries for threshold-qualified candidates |

### `main-provenance`

Use this as the default entry point.

Its responsibilities are to:

- inspect the task
- decide whether meaningful decisions are likely
- review relevant prior decisions before non-trivial choices
- treat history as a soft prior, never as binding truth
- delegate execution to `user-task-executor`
- normalize and threshold-score candidate decisions using `decision-logging-threshold`
- delegate strict logging of threshold-qualified decisions to `provenance-log`

### `user-task-executor`

This agent is focused on delivery. It completes the task while keeping enough detail for logging.

It should:

- solve the task well
- treat provenance guidance as advisory
- report candidate decisions and relevant context for thresholding/logging
- avoid writing directly to `decision-log.md`

### `provenance-log`

This agent exists to keep the log strict and consistent.

It should:

- log only threshold-qualified decisions
- use the schema from `.github/skills/decision-log/SKILL.md`
- append entries instead of overwriting history
- split multi-decision tasks into multiple entries
- write `Not explicit` when a required field is unknown

## Skills

### `gut-feeling`

This skill defines how prior decisions should influence new work.

Core idea:

- prior decisions are soft priors
- they can be helpful
- they are never automatically correct
- strong current evidence always wins

Before a non-trivial choice, the skill asks the agent to:

1. find materially similar prior decisions
2. decide whether each prior should be reused, refined, or overridden
3. carry forward only the parts that still fit
4. preserve traceability about how the prior influenced the new choice

### `decision-log`

This skill defines what a meaningful decision is and how it must be recorded.

It treats the following as candidate decisions:

- selecting one option over another
- rejecting an alternative
- accepting an assumption
- prioritizing one constraint over another
- resolving ambiguity
- introducing a trade-off
- escalating or deciding not to escalate
- changing the original plan

It explicitly says not to log trivial formatting or wording choices.

### `decision-logging-threshold`

This skill defines a deterministic gate for whether a candidate decision is major enough to persist to provenance.

It scores each candidate decision across five dimensions (0–2 each) and applies:

- mandatory overrides (policy/compliance and human-interaction boundaries)
- explicit non-decision suppression (formatting, retrieval-only, micro-decisions, mechanical execution)
- a usefulness gate (must be reusable/auditable later)

Final rule:

- `should_log = override_applied OR (usefulness_gate AND decision_score >= 5)`

Supporting docs:

- policy rationale: [`threshold/decision-logging-threshold-policy.md`](./threshold/decision-logging-threshold-policy.md)
- test plan: [`threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md`](./threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md)
- test results: [`threshold/DECISION-LOGGING-THRESHOLD-TEST-RESULT.md`](./threshold/DECISION-LOGGING-THRESHOLD-TEST-RESULT.md)

## Decision log format

The logger uses the schema defined in [`.github/skills/decision-log/SKILL.md`](./.github/skills/decision-log/SKILL.md). A typical entry contains:

```md
## Decision: <short title>

- **Timestamp:** <ISO-8601 or session-time>
- **model:** <model name and version>
- **Importance:** Critical | Important | Minor
- **Context:** <specific execution context>
- **Decision:** <what was decided>
- **Rationale:** <why it was chosen>
- **Alternatives considered:** <what else was considered>
- **Trade-offs / Risks:** <accepted downsides or risks>
- **Affected scope:** <what this changed>
```

When prior decisions materially influenced the outcome, the log should also capture:

- `Prior related decisions reviewed`
- `Reusable prior / gut feeling`
- `Why prior was reused, refined, or overridden`

## Quick start

### 1. Open the workspace in an agent environment

The checked-in version of this repository is designed first for GitHub Copilot-style agent environments that understand:

- `.github/agents/*.agent.md`
- `.github/skills/*/SKILL.md`

If you are using another platform, treat those files as reference contracts for the roles and behaviors you want to recreate.

### 2. Start with `main-provenance`

For most tasks, users should invoke the provenance-aware coordinator rather than the helper agents directly.

Use `main-provenance` when:

- the task involves judgment
- prior decisions might matter
- you want both a deliverable and a structured audit trail

### 3. Give the agent a task with decision context

Good inputs include:

- the decision or task to perform
- the available options
- constraints and stakeholders
- output format or file requirements
- whether prior decisions should be considered

### 4. Review the outputs

After the run, check:

- the requested output artifact
- the appended entries in `decision-log.md`

If the task was a non-decision control, expect only the task artifact and no decision entry.

## Example prompts based on `test-prompts/`

The files in [`test-prompts/`](./test-prompts/) are useful as both demos and regression tests.

### 1. Product roadmap prioritization

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT1.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT1.md)

Use this when you want the agent to choose a primary and secondary initiative under competing business pressures.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT1.md.
```

Expected result:

- one result file with the final roadmap choice
- separate decision-log entries for the primary and secondary choices

### 2. Stale-prior handling

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT2-stale-prior.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT2-stale-prior.md)

Use this when previous decisions may no longer fit the current situation.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT2-stale-prior.md.
```

Expected result:

- a fresh decision under changed conditions
- provenance that explains why an earlier instinct was reused, refined, or overridden

### 3. Conflicting-prior handling

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT3-conflicting-priors.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT3-conflicting-priors.md)

Use this when prior history is inconsistent and the agent must not treat majority opinion as truth.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT3-conflicting-priors.md.
```

Expected result:

- a current decision made on present evidence
- a log entry that explains which prior reasoning still mattered and which did not

### 4. Multi-decision tasks

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT4-multi-decision.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT4-multi-decision.md)

Use this when a single task contains more than one explicit decision, such as rollout scope plus safety guardrail.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT4-multi-decision.md.
```

Expected result:

- one task artifact containing both final decisions
- multiple decision-log entries, one per meaningful decision

### 5. Escalation and gated release decisions

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT5-escalation.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT5-escalation.md)

Use this when the right answer may be a cautious release, explicit gating, or escalation instead of a simple yes/no decision.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT5-escalation.md.
```

Expected result:

- a release decision artifact
- a log entry that preserves the trade-offs and escalation logic

### 6. Negative-control non-decision task

File: [`test-prompts/DECISION_LOGGING_TEST_PROMPT6-negative-control.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT6-negative-control.md)

Use this to confirm that the workspace does not invent provenance for tasks that do not require meaningful judgment.

Example request:

```text
Use main-provenance and execute test-prompts/DECISION_LOGGING_TEST_PROMPT6-negative-control.md.
```

Expected result:

- a rewritten markdown artifact
- no new decision-log entry

### 7. Deterministic decision-logging threshold suite

Files:

- suite definition: [`threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md`](./threshold/DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md)
- prompts: [`test-prompts/DECISION_LOGGING_TEST_PROMPT7-threshold-high-score.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT7-threshold-high-score.md) through [`test-prompts/DECISION_LOGGING_TEST_PROMPT16-micro-decisions-control.md`](./test-prompts/DECISION_LOGGING_TEST_PROMPT16-micro-decisions-control.md)

Use this to validate that `decision-logging-threshold` behaves deterministically across positives, controls, override cases, split candidates, and reruns.

Saved artifacts:

- Mode A: [`decisions-logs-mode-a/`](./decisions-logs-mode-a/)
- Mode B: [`decisions-logs-mode-b/`](./decisions-logs-mode-b/)
- Mode C: [`decisions-logs-mode-c/`](./decisions-logs-mode-c/)

## Why these model recommendations are used

The recommended agent-to-model mapping is based on the behavior summarized in [`DESISION_PROVENANCE_EVOLUTION.md`](./DESISION_PROVENANCE_EVOLUTION.md), but the workspace intentionally keeps the explanation here practical rather than experimental.

### Why `Claude Opus 4.6` for `main-provenance`

This role needs strong judgment under ambiguity. The coordinator must:

- compare competing constraints
- review prior decisions without becoming overly anchored
- decide when a prior should be reused, refined, or overridden
- route work cleanly to the right helper agent

This is a reasoning-heavy coordination role, so the recommendation favors a model that is strong at nuanced judgment.

### Why `Claude Sonnet 4.6` for `user-task-executor`

This role is about execution quality and efficient task completion. It needs to:

- complete the task artifact well
- preserve enough reasoning detail for logging
- avoid overcomplicating the logging concern

This makes a strong execution-oriented model a good fit.

### Why `GPT-5.4` for `provenance-log`

This role needs strict schema discipline more than broad task creativity. The logger must:

- follow the required field format consistently
- keep one meaningful decision per entry
- preserve traceability fields when priors influenced the choice
- avoid inventing unsupported decisions

That makes a structure-focused model a strong choice for the logging role.

### If you want to swap models

You can change the `model:` field in the agent files, but keep the role split intact:

- use a high-judgment model for coordination
- use a strong delivery model for task execution
- use a highly schema-reliable model for logging

## Recommended usage patterns

### Use `main-provenance` by default

This is the normal user-facing path.

### Use `user-task-executor` directly only for focused testing

Direct use can be helpful if you want to test task execution without the full provenance loop, but it is not the normal end-user path.

### Use `provenance-log` directly only when you already have a decision summary

This is mainly useful for debugging or backfilling logs from a trusted execution summary.

## How to extend the workspace

Common extension points:

- edit `.github/agents/*.agent.md` to change responsibilities or models
- edit `.github/skills/*/SKILL.md` to tighten provenance rules
- add more prompts under `test-prompts/` for new decision types
- use `.github/hooks/HOOKS.md` for cross-cutting runtime guardrails
- point the workflow to a different decision log if you want a separate prior corpus

## Good prompting tips for this workspace

To get useful provenance, make the task explicit about:

- what decision must be made
- which options are in scope
- what constraints or risks matter
- whether prior decisions should influence the choice
- what final artifact should be produced

The clearer the task boundary, the better the execution summary and decision log will be.

## Summary

This workspace is a practical pattern for combining:

- decision-aware coordination
- reusable prior-decision memory
- clean task execution
- strict structured logging

If you want an agent workflow that not only produces outputs but also preserves why important choices were made, `main-provenance` plus the `gut-feeling` and `decision-log` skills is the intended path.
If you want deterministic log/no-log behavior rather than loose "meaningful-ish" intuition, keep `decision-logging-threshold` enabled in the coordinator + logger.
