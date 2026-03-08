# Agent Integrations

ghpm skills work with any AI coding agent that supports skill/instruction files. Some agents support hooks or rules that enhance the experience — these are optional.

## Core (all agents)

The skill files themselves are agent-agnostic. Decision capture and session wrap-up work via explicit triggers:

- **Decision:** user says `decide: <text>` or agent nudges and user confirms
- **Wrap-up:** user says "wrap up" or "done"

## Claude Code Integration

Claude Code supports hooks that inject session context automatically.

### Hooks

Install to `.claude/hooks.json` in the project root. If the file already exists, merge the hooks into the existing structure.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "branch=$(git branch --show-current 2>/dev/null); num=$(echo \"$branch\" | sed -n 's|[^/]*/\\([0-9]*\\)/.*|\\1|p'); if [ -n \"$num\" ] && [ -f \".ghpm/sessions/${num}.json\" ]; then title=$(grep -o '\"title\": *\"[^\"]*\"' \".ghpm/sessions/${num}.json\" | head -1 | sed 's/\"title\": *\"//;s/\"$//'); echo \"GHPM session active: #${num} \\\"${title}\\\". Watch for design decisions and nudge to record them.\"; fi"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "branch=$(git branch --show-current 2>/dev/null); num=$(echo \"$branch\" | sed -n 's|[^/]*/\\([0-9]*\\)/.*|\\1|p'); if [ -n \"$num\" ] && [ -f \".ghpm/sessions/${num}.json\" ]; then echo \"GHPM_WRAP: Session #${num} is active. Run the wrap-up sequence from ghpm-work before ending.\"; fi"
          }
        ]
      }
    ]
  }
}
```

**UserPromptSubmit**: Injects active session context into every user message. This ensures decision detection survives context compression — the agent is reminded of the active session on every turn.

**Stop**: Triggers wrap-up when the session ends, prompting the agent to post the session journal before exiting.

### Installation

`/ghpm-init` auto-detects Claude Code (checks for `.claude/` directory or `claude` in `$SHELL` / process tree) and offers to install hooks. If declined or not detected, hooks are skipped — the skill still works via explicit triggers.

## Future Integrations

Other agents can be supported by adding integration files here. The pattern:

1. Document the agent-specific enhancement (rules file, system prompt, etc.)
2. Add detection logic to `ghpm-init`
3. Keep it optional — core skill behavior never depends on agent-specific features
