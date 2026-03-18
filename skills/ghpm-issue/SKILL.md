---
name: ghpm-issue
description: "File a GitHub issue and add it to the project board. Works mid-session or standalone. Reads .ghpm/config.json for repo list and project."
argument-hint: "[title]"
allowed-tools: Bash(gh:*), Bash(git:*), Read, Grep, Glob
compatibility: "Requires gh CLI authenticated with read:project and project scopes"
metadata:
  author: jackchuka
  scope: generic
  confirms:
    - create GitHub issues
---

# ghpm-issue

> **PREREQUISITE:** Read `../ghpm-shared/SKILL.md` for prerequisites and error handling.

File a GitHub issue and add it to the project board in one step. Works mid-session (during `ghpm-work`) or standalone.

## Arguments

- `/ghpm-issue` — interactive: asks for title and body
- `/ghpm-issue <title>` — uses argument as issue title, asks for optional body

## Workflow

### Phase 1: Setup

1. **Startup sequence**: Follow the startup sequence in `../ghpm-shared/SKILL.md`. This loads config, refreshes cache if stale, and checks for stale sessions.

   Extract from config:
   - `config.project` — id, owner, number, title
   - `config.repos` — eligible repositories
   - `config.workflow` — columns, field_id
   - `config.fields` — project fields with name, id, type, options
   - `config.conventions` — natural language rules (if present)

### Phase 2: Gather Info

2. **Title and body**:
   - If argument provided, use as **title**. Ask for an optional body (user can skip).
   - If no argument, ask for **title**, then ask for optional **body**.

3. **Pick repo**:
   - If `config.repos` has **one entry** → use it.
   - If `config.repos` is **empty or missing** → ask the user to specify a repo in `owner/repo` format.
   - If `config.repos` has **multiple entries** → present a numbered list and ask.

4. **Link to active session**: Find active session per `../ghpm-shared/references/session.md` (match current branch against session files). If found, prepend to the issue body:
   ```
   Related to <repo>#<num>

   <original body>
   ```

### Phase 3: Infer Metadata

All inference is best-effort. Skip silently when no confident match exists.

5. **Infer labels**: Fetch the target repo's actual labels:

   ```bash
   gh label list -R <repo> --json name,description --jq '.[] | "\(.name)\t\(.description)"'
   ```

   Match the issue title and body against the fetched label names and descriptions (case-insensitive substring/keyword matching). Only use labels that exist on the repo — never assume a label exists.

   If no labels match confidently, skip. Don't force a match.

6. **Infer initial status**: Use `config.workflow.columns` to determine the initial status:
   - If `config.conventions.status_sync` is present, interpret it for new issue context (e.g., if it describes transitions from Planned, that implies new issues start at Planned or earlier).
   - Otherwise, use the first column in `config.workflow.columns` as the default.
   - Never hardcode a column name — always resolve from config.

7. **Infer project fields**: Iterate over `config.fields` entries where `type` is `"single_select"` and `options` is non-empty. For each field, infer the best option from the issue context (title, body, labels, repo name):
   - Compare each `options[].name` (case-insensitive) against the issue context for a substring or keyword match.
   - If exactly one option matches confidently, use it. If ambiguous or no match, skip silently.
   - When in doubt, skip rather than guess.

### Phase 4: Confirm and Create

8. **Confirm**:

   > **Prompt** (`create_issue`): Show the resolved issue details — title, repo, labels (if any), initial status, inferred fields — and ask: "Create this issue and add to <project.title>?"

   Resolve via `prompts.ghpm-issue.create_issue` per `../ghpm-shared/references/prompts.md`.

9. **Create issue**:

   ```bash
   gh issue create -R <repo> --title "<title>" --body "<body>" [--label <labels>]
   ```

   Extract the issue URL from stdout. Then fetch the node ID:

   ```bash
   gh issue view <url> --json number,url,id --jq '{number: .number, url: .url, nodeId: .id}'
   ```

10. **Add to project**:

    ```bash
    gh api graphql -f query='
    mutation {
      addProjectV2ItemById(input: {
        projectId: "<config.project.id>"
        contentId: "<issue nodeId>"
      }) { item { id } }
    }'
    ```

    Extract the project item ID for field mutations.

    **If this fails:** The issue still exists on GitHub. Report the URL and provide the manual fallback:
    ```
    Could not add to project. Add manually:
      gh project item-add <config.project.number> --owner <config.project.owner> --url <issue-url>
    ```

11. **Set status**: Using the status resolved in step 6:

    ```bash
    gh api graphql -f query='
    mutation {
      updateProjectV2ItemFieldValue(input: {
        projectId: "<config.project.id>"
        itemId: "<project item ID>"
        fieldId: "<config.workflow.field_id>"
        value: { singleSelectOptionId: "<resolved status column id>" }
      }) { projectV2Item { id } }
    }'
    ```

    If this fails, continue — the item is on the board, just without a status.

12. **Set project fields**: For each field inferred in step 7, set via GraphQL (run mutations in parallel where possible):

    ```bash
    gh api graphql -f query='
    mutation {
      updateProjectV2ItemFieldValue(input: {
        projectId: "<config.project.id>"
        itemId: "<project item ID>"
        fieldId: "<field.id>"
        value: { singleSelectOptionId: "<matched option.id>" }
      }) { projectV2Item { id } }
    }'
    ```

    If a mutation fails, continue with remaining fields.

### Phase 5: Output

13. Format per `../ghpm-shared/references/format.md`:

    ```
    Created #<num> "<title>" in <repo>
      Project: <config.project.title> (<resolved status>)
      <field.name>: <option.name>  (for each field set in step 12)
      URL: <issue-url>
    ```

## Prompt Configuration

Per `../ghpm-shared/references/prompts.md`. Config lives at `prompts.ghpm-issue` in `.ghpm/config.json`.

Action keys: `create_issue`.

## Tips

- This skill does NOT create a work session. Use `/ghpm-work <number>` after filing to start working on the new issue.

## See Also

- [ghpm-shared](../ghpm-shared/SKILL.md) — Prerequisites and error handling
- [ghpm-work](../ghpm-work/SKILL.md) — Start a work session on an issue
- [ghpm-suggest](../ghpm-suggest/SKILL.md) — Find what to work on next
