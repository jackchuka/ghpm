# ghpm

Agent skills for managing GitHub Projects v2.

Query project status, browse views, and get work suggestions — all through natural language. Works with any AI coding agent that supports skills (Claude Code, etc.).

## Prerequisites

- [`gh` CLI](https://cli.github.com/) installed and authenticated
- Token scopes: `read:project`, `project`
- A GitHub Projects v2 project (not classic projects)

## Setup

1. Clone or copy this repo into your project directory.
2. Run `ghpm-init` with your project URL:

```
ghpm-init https://github.com/orgs/<org>/projects/<number>
```

This generates a `.ghpm/` directory:

| File                           | Purpose                               |
| ------------------------------ | ------------------------------------- |
| `.ghpm/config.json`            | Project config (fields, views, repos) |
| `.ghpm/cache.json`             | Cached project items                  |
| `.ghpm/sessions/<number>.json` | Active work sessions (per issue)      |

The entire `.ghpm/` directory is gitignored.

## Skills

| Skill              | Description                                                               |
| ------------------ | ------------------------------------------------------------------------- |
| `ghpm-init`        | Initialize config from a GitHub Projects v2 project                       |
| `ghpm-status`      | Project health dashboard — workflow distribution, items needing attention |
| `ghpm-view <name>` | Query items by named view or ad-hoc filter                                |
| `ghpm-suggest`     | Context-aware work recommendations based on project state and session     |
| `ghpm-work <num>` | Start a work session — branch, status sync, decision capture, session journal |

## Usage

```
ghpm-status              # project overview
ghpm-status Backend      # scoped to Backend team
ghpm-view triage         # show Task Triage view
ghpm-view component:docs # ad-hoc filter
ghpm-suggest             # what should I work on next?
ghpm-suggest something small  # time-constrained suggestion
ghpm-work 772            # start session on issue #772
                         # creates branch, sets InProgress
                         # decisions auto-detected and posted
                         # session journal posted on wrap-up
```

## Conventions

The `conventions` block in `.ghpm/config.json` controls branch naming, status sync, and decision behavior using natural language rules:

```json
{
  "conventions": {
    "branch": "{type}/{issue-number}/{slug}. Detect type from issue labels...",
    "status_sync": "On start, set to InProgress if currently Planned or ReadyForDev...",
    "decisions": "When a design decision is detected, nudge before recording..."
  }
}
```

Edit these rules in plain English to customize behavior.

## Agent Integrations

ghpm skills work with any AI coding agent. For agents that support hooks or rules, `ghpm-init` auto-detects the environment and offers to install enhancements:

| Agent | Enhancement | What it does |
|-------|------------|-------------|
| Claude Code | `.claude/hooks.json` | Auto-injects session context, triggers wrap-up on exit |
| Others | Coming soon | Core skills work everywhere via explicit commands |

Without hooks, `ghpm-work` works via explicit triggers: `decide: <text>` for decisions, "wrap up" for session end.
