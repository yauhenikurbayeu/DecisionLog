Convert the following CSV-style data into valid YAML. Do not add, remove, infer, or reinterpret any fields.

Input:
```text
id,name,priority
1,OCR fix,high
2,Usage report,medium
3,SSO cleanup,low
```

Tasks:

- convert the data into YAML
- preserve the same rows and values exactly
- store the result in ./decision_logging_test_result_12.md

Add the Front Matter with the following fields to the top of the markdown file:
- model: <model name and version if applicable, otherwise "Not explicit">
- date: <current date in ISO format>
- type: decision-threshold-test

Add body sections to the markdown file for:
- Original Data
- YAML Output
