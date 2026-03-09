# ghpm

Agent skills for managing GitHub Projects v2 — from triage to merged PR.

Works with any AI coding agent that supports skills (Claude Code, etc.).

## Prerequisites

- [`gh` CLI](https://cli.github.com/) installed and authenticated
- Token scopes: `read:project`, `project`
- A GitHub Projects v2 project (not classic projects)

## Setup

```
ghpm-init https://github.com/orgs/<org>/projects/<number>
```

This creates a `.ghpm/` directory (gitignored):

```
.ghpm/
├── config.json              # project config (fields, views, conventions)
├── cache.json               # cached project items
└── sessions/
    └── <number>.json        # active work session per issue
```

## Skills

```
ghpm-init <url>              # initialize project config
ghpm-status [team]           # project health dashboard
ghpm-view <name|filter>      # query items by view or ad-hoc filter
ghpm-suggest [constraint]    # what should I work on next?
ghpm-work <issue-number>     # full work lifecycle (see below)
```

## ghpm-work: Full Lifecycle

`ghpm-work` drives an issue from pickup to PR:

```
 1. Setup           assign, branch, status → InProgress
      │
 2. Clarify         evaluate issue, flesh out if vague
      │
 3. Plan            explore codebase, draft approach → post to issue
      │
 4. Impl Plan       concrete steps, files, tests → post to issue
      │
 5. Implement       write code, commit, verify
      │
 6. Draft PR        create draft PR linking the issue
      │
      ╰── decisions captured and posted at every phase
```

Sessions are saved to `.ghpm/sessions/<number>.json` with the current phase. Restarting Claude resumes from where you left off.

## Conventions

The `conventions` block in `.ghpm/config.json` controls behavior with natural language rules:

```json
{
  "conventions": {
    "branch": "{type}/{issue-number}/{slug}. Detect type from issue labels...",
    "status_sync": "On start, set to InProgress if currently Planned or ReadyForDev...",
    "decisions": "When a design decision is detected, nudge before recording..."
  }
}
```

## Agent Integrations

`ghpm-init` auto-detects your agent and offers to install hooks:

| Agent | Enhancement | What it does |
|-------|------------|-------------|
| Claude Code | `.claude/hooks.json` | Session context on every turn, wrap-up on exit |

Without hooks, everything works via explicit triggers: `decide: <text>` for decisions, "wrap up" for session end.
