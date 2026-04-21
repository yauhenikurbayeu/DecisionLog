You are choosing the default retry policy for an internal document-export notification service.

Options:
A) no retry
B) retry twice with a fixed 30-second delay
C) exponential backoff for up to 10 minutes
D) no automatic retry, operators must rerun manually

Context:

This service is internal-only.
The choice affects several worker steps and on-call behavior, but not the broader product roadmap.
It can be changed later, though doing so will require code changes, retesting, and some coordination with operations.
Some judgment is required because failure patterns are not yet well understood.
Each option has visible trade-offs between simplicity, queue load, operator burden, and failure recovery.
Whatever policy is chosen will likely remain in place for the rest of this quarter unless incidents force a change.

Tasks:

Select ONE option.

Final decision should be stored in ./decision_logging_test_result_8.md.

Add the Front Matter with the following fields to the top of the markdown file:
- model: <model name and version if applicable, otherwise "Not explicit">
- date: <current date in ISO format>
- type: decision-threshold-test

Add body sections to the markdown file for:
- Context
- Options Evaluated
- Final Decision
