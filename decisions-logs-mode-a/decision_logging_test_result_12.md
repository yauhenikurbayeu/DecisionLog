---
model: Claude Sonnet 4.6
date: 2026-04-21
type: decision-threshold-test
---

## Original Data

```text
id,name,priority
1,OCR fix,high
2,Usage report,medium
3,SSO cleanup,low
```

## YAML Output

```yaml
tasks:
  - id: 1
    name: OCR fix
    priority: high
  - id: 2
    name: Usage report
    priority: medium
  - id: 3
    name: SSO cleanup
    priority: low
```
