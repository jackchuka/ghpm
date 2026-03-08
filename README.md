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

This generates two files:

| File               | Purpose                               | Git                           |
| ------------------ | ------------------------------------- | ----------------------------- |
| `.ghpm.json`       | Project config (fields, views, repos) | Gitignored (project-specific) |
| `.ghpm-cache.json` | Cached project items                  | Gitignored                    |

See `.ghpm.json.example` for the config format.

## Skills

| Skill              | Description                                                               |
| ------------------ | ------------------------------------------------------------------------- |
| `ghpm-init`        | Initialize config from a GitHub Projects v2 project                       |
| `ghpm-status`      | Project health dashboard — workflow distribution, items needing attention |
| `ghpm-view <name>` | Query items by named view or ad-hoc filter                                |
| `ghpm-suggest`     | Context-aware work recommendations based on project state and session     |

## Usage

```
ghpm-status              # project overview
ghpm-status Backend      # scoped to Backend team
ghpm-view triage         # show Task Triage view
ghpm-view component:docs # ad-hoc filter
ghpm-suggest             # what should I work on next?
ghpm-suggest something small  # time-constrained suggestion
```
