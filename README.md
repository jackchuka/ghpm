# ghpm

Agent skills for managing GitHub Projects v2 — from triage to merged PR.

Works with any AI coding agent that supports skills (Claude Code, etc.).

## Install

```bash
npx skills add jackchuka/ghpm
```

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
    └── <number>.json        # work session per issue (kept after completion)
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

`ghpm-work` drives an issue from pickup to PR. Each phase prompts before taking action — configurable per action.

```
 1. Setup           assign, branch, status → InProgress
      │              → status_sync
 2. Clarify         evaluate issue, flesh out if vague
      │              → clarify_issue
 3. Plan            explore codebase, draft approach
      │              → post_plan
 4. Impl Plan       concrete steps, files, tests
      │              → post_impl_plan
 5. Implement       write code, commit, verify
      │
 6. Draft PR        create draft PR linking the issue
      │              → draft_pr
      │
      ╰── decisions detected and posted at every phase
                     → record_decision
```

- Sessions track phase in `.ghpm/sessions/<number>.json` — restarting resumes from where you left off.
- Decisions are posted as issue comments (single source of truth).
- Plans contain decisions — key design choices are extracted and recorded after each plan is posted.
- Completed sessions are marked `"phase": "done"`, not deleted.

## Configuration

### Conventions

The `conventions` block controls behavior with natural language rules:

```json
{
  "conventions": {
    "branch": "{type}/{issue-number}/{slug}. Detect type from issue labels...",
    "status_sync": "On start, set to InProgress if currently Planned or ReadyForDev...",
    "decisions": "When a design decision is detected, nudge before recording..."
  }
}
```

### Prompts

The `prompts` block controls which actions ask for confirmation, scoped per skill:

```json
{
  "prompts": {
    "default": "prompt",
    "ghpm-work": {
      "default": "auto",
      "draft_pr": "prompt"
    }
  }
}
```

| Value      | Behavior                       |
| ---------- | ------------------------------ |
| `"prompt"` | Ask the user before proceeding |
| `"auto"`   | Proceed without asking         |
| `"skip"`   | Skip the action entirely       |

Resolution: `prompts.<skill>.<action>` → `prompts.<skill>.default` → `prompts.default` → `"prompt"`. First match wins.

## Agent Integrations

`ghpm-init` auto-detects your agent and offers to install hooks:

| Agent       | Enhancement                           | What it does                                                   |
| ----------- | ------------------------------------- | -------------------------------------------------------------- |
| Claude Code | `.claude/settings.local.json` (`hooks` key) | Session context + phase on every turn, stale session detection |

Without hooks, everything works via explicit triggers: `decide: <text>` for decisions, "wrap up" for session end.
