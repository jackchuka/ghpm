---
name: ghpm-shared
description: "Shared reference for all ghpm skills — prerequisites, config format, startup sequence, and error handling."
allowed-tools: Bash(gh:*), Read, Grep
---

# ghpm — Shared Reference

## Prerequisites

- `gh` CLI installed and authenticated
- Token scopes: `read:project`, `project`
- GitHub Projects v2 (not classic projects)

## Config Files

All ghpm files live under `.ghpm/`:

| File                          | Purpose                               |
| ----------------------------- | ------------------------------------- |
| `.ghpm/config.json`           | Project config (fields, views, repos) |
| `.ghpm/cache.json`            | Cached project items                  |
| `.ghpm/sessions/<number>.json`| Active work session per issue         |

The entire `.ghpm/` directory is gitignored (project-specific).

If `.ghpm/config.json` is missing, tell the user to run `/ghpm-init` first and stop.

## Startup Sequence

Every skill that reads project data should follow this sequence:

1. Read `.ghpm/config.json` from the current directory. If missing, stop with init guidance (above).
2. Load cache following `references/cache.md`. If `gh` commands fail, follow the error handling guidance there — offer stale cache as fallback when available.

## Error Handling

- **Auth expired** (`gh auth status` fails): tell the user to run `gh auth login` and stop.
- **Rate limited** (HTTP 403 with rate limit message): tell the user to wait and retry later.
- **Network error**: if cache exists (even stale), offer to use stale cache as fallback.
- **Permission denied** (HTTP 403/404 on project): tell the user to check token scopes (`read:project`, `project`).

## Write Operations

> [!CAUTION]
> Skills that modify project items (move status, assign, create, delete) are **write** commands. Always confirm with the user before executing.

## Agent Integrations

Optional hooks/rules enhance `ghpm-work` sessions (auto session context, wrap-up on exit). See `references/integrations.md` for details. Installed by `/ghpm-init` when an agent is detected. Without hooks, all skills work via explicit commands.

## Skills

| Skill                                      | Purpose                                   |
| ------------------------------------------ | ----------------------------------------- |
| [`ghpm-init`](../ghpm-init/SKILL.md)       | Initialize config from GitHub Projects v2 |
| [`ghpm-status`](../ghpm-status/SKILL.md)   | Project health dashboard                  |
| [`ghpm-view`](../ghpm-view/SKILL.md)       | Query items by view or ad-hoc filter      |
| [`ghpm-suggest`](../ghpm-suggest/SKILL.md) | Context-aware work recommendations        |
| [`ghpm-work`](../ghpm-work/SKILL.md)       | Start a work session on an issue      |
