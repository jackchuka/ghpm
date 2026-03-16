---
name: ghpm-issue
description: "File a GitHub issue and add it to the project board. Works mid-session or standalone. Reads .ghpm/config.json for repo list and project."
argument-hint: "[title]"
allowed-tools: Bash(gh:*), Bash(git:*), Read, Grep, Glob
---

# ghpm-issue

> **PREREQUISITE:** Read `../ghpm-shared/SKILL.md` for prerequisites and error handling.

File a GitHub issue and add it to the project board in one step. Works mid-session (during `ghpm-work`) or standalone.

## Arguments

- `/ghpm-issue` — interactive: asks for title and body
- `/ghpm-issue <title>` — uses argument as issue title, asks for optional body

## Workflow

### 1. Load Config

Read `.ghpm/config.json`. If missing, tell the user to run `/ghpm-init` and stop.

Extract:
- `config.project.id`, `config.project.owner`, `config.project.number`, `config.project.title`
- `config.repos` — list of eligible repositories
- `config.workflow.columns` — for setting Status
- `config.workflow.field_id` — for the Status field mutation

### 2. Gather Info

- If argument provided, use as **title**. Ask for an optional body (user can skip).
- If no argument, ask for **title**, then ask for optional **body**.

### 3. Pick Repo

- If `config.repos` has **one entry** → use it.
- If `config.repos` is **empty or missing** → ask the user to specify a repo in `owner/repo` format.
- If `config.repos` has **multiple entries** → present a numbered list:
  ```
  Which repo?
  1. tailor-platform/erp-kit
  2. tailor-platform/frontend-packages
  ```

### 4. Infer Labels

Best-effort label inference from title and body keywords:

| Keywords | Label |
|----------|-------|
| `bug`, `fix`, `broken`, `crash`, `error` | `bug` |
| `feat`, `add`, `new`, `enhance`, `improve` | `enhancement` |
| `doc`, `docs`, `readme`, `documentation` | `documentation` |

If no keywords match, skip labels. Before using inferred labels, validate they exist on the target repo:

```bash
gh label list -R <repo> --json name --jq '.[].name'
```

Only pass labels that exist. Drop any that don't — `gh issue create --label` will fail if the label is missing from the repo.

### 5. Link to Active Session

Check if an active `ghpm-work` session matches the current branch:

```bash
branch=$(git branch --show-current 2>/dev/null)
if [ -n "$branch" ] && [ -d ".ghpm/sessions" ]; then
  for f in .ghpm/sessions/*.json; do
    [ -f "$f" ] || continue
    sb=$(grep -o '"branch": *"[^"]*"' "$f" | sed 's/"branch": *"//;s/"$//')
    if [ "$branch" = "$sb" ]; then
      num=$(basename "$f" .json)
      break
    fi
  done
fi
```

If a session is found, prepend to the issue body:
```
Related to <repo>#<num>

<original body>
```

### 6. Confirm Before Creating

> **Prompt** (`create_issue`): "Create \"<title>\" in <repo> and add to <project.title>? (y/n)"

Resolve via `prompts.ghpm-issue.create_issue` per `../ghpm-shared/references/prompts.md`.

### 7. Create Issue

```bash
gh issue create -R <repo> --title "<title>" --body "<body>" [--label <labels>]
```

Extract the issue URL from stdout (last line of output). Then fetch the node ID:

```bash
gh issue view <url> --json number,url,id --jq '{number: .number, url: .url, nodeId: .id}'
```

### 8. Add to Project

```bash
gh api graphql -f query='
mutation {
  addProjectV2ItemById(input: {
    projectId: "<config.project.id>"
    contentId: "<issue nodeId>"
  }) { item { id } }
}'
```

Extract the project item ID from the response for step 9.

**If this fails:** The issue still exists on GitHub. Report the URL and provide the manual fallback:
```
⚠ Could not add to project. Add manually:
  gh project item-add <config.project.number> --owner <config.project.owner> --url <issue-url>
```

### 9. Set Status to Planned

Find the "Planned" column in `config.workflow.columns` by name. If not found, use the first column.

```bash
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "<config.project.id>"
    itemId: "<project item ID from step 8>"
    fieldId: "<config.workflow.field_id>"
    value: { singleSelectOptionId: "<Planned column id>" }
  }) { projectV2Item { id } }
}'
```

If this fails, continue — the item is on the board, just without a status.

### 10. Output

```
Created #<num> "<title>" in <repo>
  Project: <config.project.title> (Planned)
  URL: <issue-url>
```

## Prompt Configuration

Per `../ghpm-shared/references/prompts.md`. Config lives at `prompts.ghpm-issue` in `.ghpm/config.json`.

Action keys: `create_issue`.

## Tips

- This skill does NOT create a work session. Use `/ghpm-work <number>` after filing to start working on the new issue.
- No cache load or stale session check — keeps invocation fast.

## See Also

- [ghpm-shared](../ghpm-shared/SKILL.md) — Prerequisites and error handling
- [ghpm-work](../ghpm-work/SKILL.md) — Start a work session on an issue
- [ghpm-suggest](../ghpm-suggest/SKILL.md) — Find what to work on next
