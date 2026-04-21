# Decision Logging Threshold Test Results

## Executive Summary

This report evaluates whether the `decision-logging-threshold` workflow behaved correctly across the three execution modes defined in [DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md](DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md).

Top-line verdict:

- Mode A: pass
- Mode B: pass
- Mode C: partial pass

Overall conclusion:

- threshold classification is mostly valid
- control prompts remained suppressed
- override prompts logged correctly
- mixed-order rerun exposed inconsistent duplicate-handling / reconfirmation behavior

The saved artifacts also reveal two workflow-quality issues beyond the binary log/no-log gate:

- inconsistent prior-use behavior in clean-room and rerun conditions
- inconsistent duplicate / reconfirmation logging in Mode C

## Method And Evidence Scope

This analysis uses only the designated artifacts:

- [DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md](DECISION-LOGGING-THRESHOLD-TEST-SCENARIO.md)
- `test-result-mode-a/`
- `test-result-mode-b/`
- `test-result-mode-c/`

Prompt-to-result mapping is based on the prompt number in each filename. For Mode A, the split files `7.md`, `8.md`, `10.md`, `11.md`, `14.md`, and `15.md` are treated as the logged-decision artifacts. For Modes B and C, [test-result-mode-b/decision-log.md](test-result-mode-b/decision-log.md) and [test-result-mode-c/decision-log.md](test-result-mode-c/decision-log.md) are treated as the cumulative logs for those modes.

This report evaluates saved artifacts only. It can validate observed log/no-log outcomes and split behavior, but it cannot prove the exact internal threshold scoring path wherever persisted `threshold_check` data is absent.

## Per-Mode Findings

### Mode A: Clean-Room Run

Mode A meets the expected binary threshold behavior.

- `10/10` result files exist in `test-result-mode-a/`.
- Exactly `6` logged decision artifacts exist: prompts `7`, `8`, `10`, `11`, `14`, and `15`.
- No logged artifacts exist for control prompts `9`, `12`, `13`, and `16`.
- Prompt `15` logged only the substantive rollout-scope choice; there is no separate logged artifact for the memo-order decision.

Mode A therefore passes on the core threshold objective: the suite produced the expected six positives and zero control false positives.

Cautionary note:

- [test-result-mode-a/8.md](test-result-mode-a/8.md) references `decision_logging_test_result_7.md` as a reviewed prior. That weakens the purity of the clean-room condition, because the artifact suggests result-file reuse or cross-artifact influence even though Mode A is supposed to validate threshold behavior without prior-log contamination.

### Mode B: Prior-Rich Rerun

Mode B reran only the four control prompts: `9`, `12`, `13`, and `16`.

- `4/4` expected result files exist in `test-result-mode-b/`.
- [test-result-mode-b/decision-log.md](test-result-mode-b/decision-log.md) still contains exactly `6` entries.
- Those six entries match the carried-forward Mode A positives; no additional log entries were appended by the Mode B control rerun.

Mode B therefore passes. The threshold gate appears robust against false positives caused by prior-log presence on deterministic, retrieval-only, and micro-decision control prompts.

### Mode C: Mixed-Order Resilience Run

Mode C shows that order changes did not cause control prompts to start logging, but it also exposes inconsistent rerun behavior on positives.

- `10/10` result files exist in `test-result-mode-c/`.
- [test-result-mode-c/decision-log.md](test-result-mode-c/decision-log.md) contains `8` entries total.
- Control prompts `9`, `12`, `13`, and `16` still did not create false-positive log entries.
- Two additional entries were appended in Mode C:
  - prompt `15` reconfirmation
  - prompt `14` reconfirmation

Mode C is therefore a partial pass:

- pass on control suppression
- pass on order resilience for non-log cases
- issue on consistent rerun logging behavior

The key inconsistency is that prompts `14` and `15` were re-logged while similarly prior-backed prompts `7`, `8`, `10`, and `11` produced result artifacts but no new log entries. That makes the duplicate-suppression / reconfirmation policy look inconsistent rather than deterministic.

## Cross-Cutting Issues

1. **Threshold log/no-log behavior is mostly correct across all three modes.** The suite produced the expected six positives in Mode A, no new false positives in Mode B, and no control false positives in Mode C.
2. **Override handling works.** Prompts `10` and `11` logged as expected in the clean-room run and remained represented correctly in later cumulative logs.
3. **Split-before-log behavior works for prompt `15` in Mode A.** The rollout decision was logged, while the memo-order decision was not promoted into provenance.
4. **Clean-room contamination risk exists.** Prompt `8`'s logged artifact references `decision_logging_test_result_7.md` as a reviewed prior, which is not consistent with a strict clean-room interpretation.
5. **Mixed-order rerun shows inconsistent duplicate suppression / reconfirmation policy.** Mode C re-logged prompts `14` and `15`, but did not similarly re-log other prior-backed positives.
6. **Persisted log schema is inconsistent across entries and modes.** Some entries include detailed `threshold_check` data, others do not, and the cumulative logs contain formatting noise such as the stray `!` line visible in the combined Mode B and Mode C logs.

## Conclusions

The threshold skill is effective as a binary precision filter for this suite. False-positive control performance is strong, override cases are handled correctly, and the split-decision behavior for prompt `15` works on the saved evidence.

At the same time, the current provenance layer is not yet stable enough for fully reliable rerun semantics or audit-grade consistency. The two main weaknesses are not the threshold rule itself, but the surrounding workflow:

- prior-use boundaries are not consistently enforced
- duplicate / reconfirmation behavior is not deterministic across reruns

Recommended improvements:

- make duplicate / reconfirmation policy explicit and deterministic
- prevent result files from being treated as priors in clean-room mode
- persist structured `threshold_check` uniformly
- normalize decision-log schema and formatting across logger runs

## Appendix: Prompt Outcome Table

| Prompt | Expected log? | Mode A | Mode B | Mode C | Notes |
|---|---|---|---|---|---|
| `7` | yes | logged | not run | result only, no new Mode C entry | prior reused in Mode C |
| `8` | yes | logged | not run | result only, no new Mode C entry | clean-room prior contamination in Mode A |
| `9` | no | no log | no new entry | no log | correct control |
| `10` | yes | logged | not run | result only, no new Mode C entry | override case |
| `11` | yes | logged | not run | result only, no new Mode C entry | override case |
| `12` | no | no log | no new entry | no log | deterministic transform control |
| `13` | no | no log | no new entry | no log | retrieval-only control |
| `14` | yes | logged | not run | re-logged in Mode C | inconsistent rerun behavior |
| `15` | yes | logged | not run | re-logged in Mode C | split logic good, rerun behavior inconsistent |
| `16` | no | no log | no new entry | no log | micro-decision control |
