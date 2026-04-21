# Decision Logging Threshold Test Scenario

## Purpose

This scenario is designed to validate the **validity** and **effectiveness** of the `decision-logging-threshold` skill as the deterministic major-decision gate in the current decision provenance workflow.

It is focused on a narrower question than the broader provenance suite:

- not "can the system reason well in general?"
- but "does the system correctly decide when a candidate decision should and should not be logged?"

## What this suite is meant to test

The suite is designed to verify all of the threshold contract's key behaviors:

1. **score-based positives**
   - a decision logs because it is genuinely major
   - no mandatory override is needed

2. **threshold-edge behavior**
   - a candidate at or near the threshold boundary is handled correctly

3. **mandatory overrides**
   - policy/compliance boundaries always log
   - human interaction boundaries always log

4. **explicit non-decisions**
   - deterministic transforms do not log
   - retrieval-only tasks do not log
   - repeated micro-decisions do not log

5. **candidate normalization**
   - bundled decisions are split before scoring
   - only the threshold-qualified candidate is logged

6. **scope-changing assumptions**
   - assumptions that materially change scope or risk are treated as loggable candidates

7. **robustness in a populated workspace**
   - the presence of earlier decision logs must not cause false positives on control prompts

## Core validity claims

The threshold skill should be considered valid only if all of the following are true:

- high-signal decisions are logged consistently
- explicit non-decisions are not logged
- override cases are logged even if their numeric score is modest
- split-decision prompts do not collapse into one vague entry
- local or presentation-only choices do not get promoted into provenance

## Suite files

This threshold-specific suite adds the following prompts:

- `DECISION_LOGGING_TEST_PROMPT7-threshold-high-score.md`
- `DECISION_LOGGING_TEST_PROMPT8-threshold-exact-five.md`
- `DECISION_LOGGING_TEST_PROMPT9-threshold-below-five-control.md`
- `DECISION_LOGGING_TEST_PROMPT10-policy-override-privacy.md`
- `DECISION_LOGGING_TEST_PROMPT11-human-escalation-override.md`
- `DECISION_LOGGING_TEST_PROMPT12-deterministic-transform-control.md`
- `DECISION_LOGGING_TEST_PROMPT13-retrieval-only-control.md`
- `DECISION_LOGGING_TEST_PROMPT14-assumption-scope-change.md`
- `DECISION_LOGGING_TEST_PROMPT15-split-mixed-threshold.md`
- `DECISION_LOGGING_TEST_PROMPT16-micro-decisions-control.md`

The earlier prompts can still be used as companion regressions:

- `DECISION_LOGGING_TEST_PROMPT4-multi-decision.md`
- `DECISION_LOGGING_TEST_PROMPT5-escalation.md`
- `DECISION_LOGGING_TEST_PROMPT6-negative-control.md`

But the new threshold suite is intended to be self-sufficient.

## Execution modes

### Mode A: Clean-room run

Run prompts `7` through `16` in a clean environment where no prior decision log is readable.

This mode validates the threshold logic itself without prior-history contamination.

### Mode B: Prior-rich rerun

After Mode A has produced a populated decision log, rerun these prompts:

- `DECISION_LOGGING_TEST_PROMPT9-threshold-below-five-control.md`
- `DECISION_LOGGING_TEST_PROMPT12-deterministic-transform-control.md`
- `DECISION_LOGGING_TEST_PROMPT13-retrieval-only-control.md`
- `DECISION_LOGGING_TEST_PROMPT16-micro-decisions-control.md`

Expected result:

- they should still produce `0` new decision-log entries

This tests whether the threshold gate remains driven by the current candidate rather than by the mere existence of earlier provenance.

### Mode C: Mixed-order resilience run

Run the full suite again in a different order.

Recommended shuffled order:

`10 -> 7 -> 12 -> 15 -> 8 -> 13 -> 11 -> 16 -> 14 -> 9`

Expected result:

- threshold outcomes should stay materially the same
- no-ordering effect should turn a control prompt into a logged decision

## Expected outcomes by prompt

| Prompt | Main behavior under test | Expected candidate handling | Expected decision-log entries |
|---|---|---|---:|
| `7` | strong score-based positive | `1` candidate, score-based log, no override | `1` |
| `8` | exact-threshold or near-threshold inclusive positive | `1` candidate, should log at the boundary, no override | `1` |
| `9` | below-threshold local choice | `1` candidate, should not log | `0` |
| `10` | policy/compliance override | `1` candidate, must log via override | `1` |
| `11` | human escalation override | `1` candidate, must log via override | `1` |
| `12` | deterministic transform control | explicit non-decision, should not log | `0` |
| `13` | retrieval-only control | explicit non-decision, should not log | `0` |
| `14` | scope-changing assumption | `1` candidate, should log | `1` |
| `15` | split mixed prompt | `2` candidates, only the substantive one should log | `1` |
| `16` | repeated micro-decisions | several local choices, none should log | `0` |

Expected clean-room total:

- explicit result files: `10`
- total decision-log entries: `6`

## Intended threshold interpretation by prompt

These are the intended classifications. Exact numeric dimension values may vary slightly, but the final gate should not.

### Prompt 7

- intended outcome: `should_log = true`
- intended mechanism: score-based major decision
- intended override: none

### Prompt 8

- intended outcome: `should_log = true`
- intended mechanism: boundary / exact-threshold positive
- intended override: none

### Prompt 9

- intended outcome: `should_log = false`
- intended mechanism: low score and low/no usefulness
- intended override: none

### Prompt 10

- intended outcome: `should_log = true`
- intended mechanism: `policy_compliance_boundary`
- intended override: yes

### Prompt 11

- intended outcome: `should_log = true`
- intended mechanism: `human_interaction_boundary`
- intended override: yes

### Prompt 12

- intended outcome: `should_log = false`
- intended mechanism: explicit non-decision, deterministic transform

### Prompt 13

- intended outcome: `should_log = false`
- intended mechanism: explicit non-decision, retrieval-only

### Prompt 14

- intended outcome: `should_log = true`
- intended mechanism: scope-changing assumption accepted as a real candidate decision

### Prompt 15

- intended outcome:
  - rollout decision -> `should_log = true`
  - memo section ordering decision -> `should_log = false`

### Prompt 16

- intended outcome: `should_log = false`
- intended mechanism: repeated micro-decisions inside an already chosen strategy

## Evaluation rubric

### Minimum per-prompt checks

For every prompt, verify:

1. the explicit task result file was produced
2. the number of decision-log entries matches the expected count
3. any logged entry matches the right candidate rather than a nearby but wrong one
4. no control prompt produces a fake provenance record

### Additional checks where threshold traces are available

If the system exposes `threshold_check` in coordinator traces, logger traces, or future persisted schema, verify:

- `threshold_policy_ref` is present
- `override_applied` is correct
- `should_log` matches the expected prompt outcome
- split candidates have separate `threshold_check` results
- controls return `should_log: false`

### Pass criteria

Treat the suite as a strong pass only if all of the following are true:

- `6/6` expected positive entries are present
- `0/4` control prompts create false-positive log entries
- `1/1` mixed split prompt logs only the substantive decision
- both override prompts log successfully

Treat any of these as a serious threshold failure:

- a deterministic transform is logged
- a retrieval-only prompt is logged
- an override prompt does not log
- the split prompt logs both candidates together or logs only the presentation-only choice

## What this suite measures well

- threshold correctness
- false-positive control
- false-negative control on must-log cases
- split-before-score behavior
- override handling
- prior-rich robustness on control prompts

## What this suite does not fully measure

- long-horizon prior reuse quality
- downstream provenance schema quality beyond threshold correctness
- actor-level attribution such as `decision_made_by` or `logged_by`
- exact dimension-level calibration unless `threshold_check` is persisted or otherwise exposed

## Current limitation

The current workflow now applies the threshold operationally, but the saved decision-log artifacts do not yet persist the full structured `threshold_check` object.

That means this suite can already validate:

- whether a prompt did or did not create a decision-log entry
- whether split behavior was correct

But it cannot yet fully validate from saved artifacts alone:

- the exact numeric dimension breakdown used by the threshold skill

Once `threshold_check` is persisted, this same suite can be upgraded from binary log/no-log validation to full field-level threshold validation.
