# Session Management

## Session Files

Each active work session is stored in `.ghpm/sessions/<issue-number>.json`. One file per issue. Multiple sessions can coexist (parallel Claude instances working on different issues).

## Schema

```json
{
  "item": {
    "id": "<project item node ID>",
    "number": 772,
    "title": "<issue title>",
    "repository": "<owner/repo>",
    "url": "<issue URL>"
  },
  "branch": "<branch name>",
  "started_at": "<ISO8601 timestamp with actual time, e.g. 2026-03-09T12:34:56Z>",
  "phase": "<setup|clarify|plan|implementation_plan|implement|pr|done>"
}
```

## Reading Session

1. Determine the current issue number from the git branch: extract the number segment from `{type}/{number}/{slug}`.
2. Read `.ghpm/sessions/<number>.json`.
3. If missing → no active session for this branch.

## Finding Active Session from Branch

```bash
branch=$(git branch --show-current 2>/dev/null)
num=$(echo "$branch" | sed -n 's|[^/]*/\([0-9]*\)/.*|\1|p')
```

If `$num` is empty, the current branch doesn't match a session branch pattern.

## Writing Session

Write the full JSON object to `.ghpm/sessions/<number>.json`. Create `.ghpm/sessions/` directory if it doesn't exist.

## Completing Session

On wrap-up, update `"phase": "done"`. Do NOT delete the session file — it serves as a record that work was done on this issue.

## Listing Active Sessions

```bash
ls .ghpm/sessions/*.json 2>/dev/null
```

## Detecting Stale Sessions

A session is considered stale if `started_at` is more than 24 hours ago and `phase` is NOT `"done"`. The shared startup sequence checks for stale sessions and prompts the user to wrap up or discard them. Sessions with `"phase": "done"` are never stale.
