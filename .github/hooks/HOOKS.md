# Hooks Specification
This document shows how hooks support the inheritance-like agent model in this workspace.

Use hooks for cross-cutting concerns that should apply to every agent session without repeating them in each `.agent.md` file.

- Validate the task envelope before tools run
- Log session start and end events
- Capture failures in a consistent way
- Enforce lightweight guardrails around delegation and tool usage

## Reference

- [GitHub Hooks Configuration](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/hooks-configuration)
- https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/agents/coding-agent/about-hooks

## Example `hooks.json`

```json
{
  "sessionStart": [
    {
      "type": "command",
      "bash": "echo \"session_start agent=$COPILOT_AGENT_NAME\"",
      "cwd": ".",
      "timeoutSec": 10
    }
  ],
  "preToolUse": [
    {
      "type": "command",
      "bash": "test -n \"$COPILOT_AGENT_NAME\"",
      "cwd": ".",
      "timeoutSec": 10
    }
  ],
  "postToolUse": [
    {
      "type": "command",
      "bash": "echo \"post_tool_use agent=$COPILOT_AGENT_NAME\"",
      "cwd": ".",
      "timeoutSec": 15
    }
  ],
  "sessionEnd": [
    {
      "type": "command",
      "bash": "echo \"session_end agent=$COPILOT_AGENT_NAME\"",
      "cwd": ".",
      "timeoutSec": 10
    }
  ],
  "errorOccurred": [
    {
      "type": "command",
      "bash": "echo \"error agent=$COPILOT_AGENT_NAME\"",
      "cwd": ".",
      "timeoutSec": 10
    }
  ]
}
```

## How hooks fit the inheritance model

- Global instructions define the contract.
- Skills define reusable role behavior.
- Agent files define local specialization.
- Hooks enforce shared runtime policy outside the agent prompt itself.

This makes hooks the equivalent of infrastructure-level inheritance for validation, auditing, and safety.
