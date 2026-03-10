# Agent Integrations

ghpm skills work with any AI coding agent that supports skill/instruction files. Some agents support hooks or rules that enhance the experience — these are optional.

## Core (all agents)

The skill files themselves are agent-agnostic. Decision capture and session wrap-up work via explicit triggers:

- **Decision:** user says `decide: <text>` or agent nudges and user confirms
- **Wrap-up:** user says "wrap up" or "done"

## Claude Code Integration

Claude Code supports hooks that inject session context automatically.

### Hook

Install to `.claude/settings.local.json` in the project root under the `"hooks"` key. This keeps hooks local (not committed to the repo) since session files in `.ghpm/` are also local. If the file already exists, merge the `"hooks"` key into the existing structure (preserve existing settings and hooks, add new ones).

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "branch=$(git branch --show-current 2>/dev/null); if [ -n \"$branch\" ] && [ -d \".ghpm/sessions\" ]; then for f in .ghpm/sessions/*.json; do [ -f \"$f\" ] || continue; sb=$(grep -o '\"branch\": *\"[^\"]*\"' \"$f\" | sed 's/\"branch\": *\"//;s/\"$//'); if [ \"$branch\" = \"$sb\" ]; then num=$(basename \"$f\" .json); title=$(grep -o '\"title\": *\"[^\"]*\"' \"$f\" | head -1 | sed 's/\"title\": *\"//;s/\"$//'); phase=$(grep -o '\"phase\": *\"[^\"]*\"' \"$f\" | head -1 | sed 's/\"phase\": *\"//;s/\"$//'); if [ \"$phase\" != \"done\" ]; then started=$(grep -o '\"started_at\": *\"[^\"]*\"' \"$f\" | sed 's/\"started_at\": *\"//;s/\"$//'); if [ -n \"$started\" ]; then started_epoch=$(date -jf '%Y-%m-%dT%H:%M:%SZ' \"$started\" '+%s' 2>/dev/null || date -d \"$started\" '+%s' 2>/dev/null); now_epoch=$(date '+%s'); if [ -n \"$started_epoch\" ] && [ $(( now_epoch - started_epoch )) -gt 86400 ]; then started_date=$(echo \"$started\" | cut -dT -f1); echo \"GHPM session #${num} \\\"${title}\\\" is stale (started ${started_date}). Wrap up with /ghpm-work ${num} or discard.\"; break; fi; fi; echo \"GHPM session #${num} \\\"${title}\\\" [${phase:-setup}]. Use /ghpm-work ${num} to resume.\"; fi; break; fi; done; fi"
          }
        ]
      }
    ]
  }
}
```

**UserPromptSubmit**: Injects active session context into every user message. Matches the current git branch against each session file's `branch` field — convention-independent, no regex parsing of branch names. Skips `done` sessions. Detects stale sessions (>24h old) and warns instead of showing the normal resume hint.

### Installation

`/ghpm-init` auto-detects Claude Code (checks for `.claude/` directory or `claude` in `$SHELL` / process tree) and offers to install hooks. If declined or not detected, hooks are skipped — the skill still works via explicit triggers.

## Future Integrations

Other agents can be supported by adding integration files here. The pattern:

1. Document the agent-specific enhancement (rules file, system prompt, etc.)
2. Add detection logic to `ghpm-init`
3. Keep it optional — core skill behavior never depends on agent-specific features
