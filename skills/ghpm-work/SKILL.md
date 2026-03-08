---
name: ghpm-work
description: "Start a work session on a GitHub Project item. Creates branch, syncs status, and auto-captures decisions and session journal."
allowed-tools: Bash(gh:*), Bash(git:*), Read, Write, Grep, Glob
---

# ghpm-work

> **PREREQUISITE:** Read `../ghpm-shared/SKILL.md` for prerequisites and error handling.

Start a work session on a project item. One command handles everything: branch creation, status sync, decision capture, and session journaling.

## Arguments

- `/ghpm-work <issue-number>` — start a session for the given issue

If no argument, ask the user for the issue number.

## Workflow

### Phase 1: Resolve Item

1. **Project config** (required): Follow the startup sequence in `../ghpm-shared/SKILL.md`.

2. **Check for existing session on this issue**: Read `.ghpm/sessions/<issue-number>.json` per `../ghpm-shared/references/session.md`. If it exists, resume:
   ```
   Resuming session for #<num> "<title>" on branch <branch>
   ```
   Check out the existing branch and skip to Phase 4 (step 12).

3. **Load cache** per `../ghpm-shared/references/cache.md`.

4. **Find the item**: Search cached items where `content.number` matches the given issue number. If not found, refresh cache and retry. If still not found, tell the user the issue isn't in this project and stop.

5. **Identify the repo**: Extract `content.repository` from the matched item (format: `owner/repo`). Verify the current working directory is a git repo for that repository by checking `git remote get-url origin`. If mismatched, warn the user but continue.

### Phase 2: Create Branch

6. **Read conventions**: Read `conventions.branch` from `.ghpm/config.json`. If `conventions` is missing, use defaults: `{type}/{issue-number}/{slug}`.

7. **Auto-detect type** from the item's labels and title. Interpret the `conventions.branch` rule. Default mapping:
   - Label contains "bug" → `fix`
   - Label contains "enhancement" or "feature" → `feat`
   - Title or label contains "refactor" → `refactor`
   - Label contains "documentation" → `docs`
   - Label contains "test" → `test`
   - Fallback → `chore`

8. **Generate slug**: Take the issue title, lowercase, replace non-alphanumeric characters with hyphens, collapse multiple hyphens, trim to max 5 words, strip trailing hyphens.

9. **Create and checkout branch**:
   ```bash
   git checkout -b <type>/<issue-number>/<slug>
   ```
   If the branch already exists, check it out instead:
   ```bash
   git checkout <type>/<issue-number>/<slug>
   ```

### Phase 3: Sync Status

10. **Read conventions**: Read `conventions.status_sync` from `.ghpm/config.json`. Interpret the rule to decide whether to update status.

11. **Update status** if the rule allows it. Use GraphQL to update the project item's status field:
    ```bash
    gh api graphql -f query='
    mutation {
      updateProjectV2ItemFieldValue(input: {
        projectId: "<project.id>"
        itemId: "<item.id>"
        fieldId: "<workflow.field_id>"
        value: { singleSelectOptionId: "<target_column_id>" }
      }) {
        projectV2Item { id }
      }
    }'
    ```
    Find the target column ID from `workflow.columns` in config (e.g., "InProgress" column).

    > **Write operation**: Confirm with the user before executing, per `../ghpm-shared/SKILL.md` write operations guidance.

### Phase 4: Start Session

12. **Get GitHub user**: `gh api user --jq '.login'`

13. **Write session file** to `.ghpm/sessions/<issue-number>.json` per `../ghpm-shared/references/session.md`:
    ```json
    {
      "item": {
        "id": "<item.id>",
        "number": <issue-number>,
        "title": "<item.title>",
        "repository": "<owner/repo>",
        "url": "<issue URL>"
      },
      "branch": "<branch-name>",
      "started_at": "<ISO8601 timestamp>",
      "decisions": []
    }
    ```

14. **Print summary**:
    ```
    Started session for #<num> "<title>"

      Branch:  <type>/<issue-number>/<slug>
      Status:  <old> → <new>
      Issue:   <url>

    I'll nudge you when I detect a design decision.
    Say "wrap up" when you're done.
    ```

## During Session: Decision Detection

If agent-specific hooks are installed (see `../ghpm-shared/references/integrations.md`), session context is injected into every message automatically — this ensures decision detection survives context compression. Without hooks, the skill still works via explicit triggers below.

Watch the conversation for design decision moments. Signals include:

- "Let's go with X"
- "I chose X because Y"
- "We'll use X instead of Y"
- "The approach is X"
- Selecting between alternatives after discussion

When detected, nudge:

```
This sounds like a design decision. Record it to #<num>?
  > "<proposed decision summary>"
(y/n)
```

The user can also trigger explicitly: `decide: <text>` or just `decide` (then prompt for text).

If confirmed:
1. Post issue comment (durable store first):
   ```bash
   gh issue comment <number> -R <repo> --body "**Decision:** <text>"
   ```
2. If POST succeeds → append to session decisions per `../ghpm-shared/references/session.md`.
3. If POST fails → warn user, offer retry. Do NOT write to session file.
4. Briefly confirm: `Recorded to #<num>.`

If declined, continue without recording.

## Session End: Wrap Up

Triggered when the user says "wrap up", "done", "that's it", or similar. Also triggered automatically by agent hooks if installed (see `../ghpm-shared/references/integrations.md`).

### Wrap Sequence

1. **Read session file** from `.ghpm/sessions/<number>.json`.

2. **Gather git context**:
   ```bash
   git log --oneline $(git merge-base main HEAD)..HEAD
   git diff --stat $(git merge-base main HEAD)..HEAD
   ```

3. **Check PR status**:
   ```bash
   gh pr list --head <branch> -R <repo> --json number,merged,url --limit 1
   ```

4. **Compose session journal**:
   ```
   **Session journal** — <date>

   **Changes:**
   - <N> commits, <M> files changed
   - <bullet summary of key commits>

   **Decisions made:**
   - <decision 1>
   - <decision 2>
   (or "None")

   **Status:** <current status> (PR: <merged|open|not created>)
   ```

5. **Post journal as issue comment**:
   ```bash
   gh issue comment <number> -R <repo> --body "<journal>"
   ```

6. **Apply status sync**: Read `conventions.status_sync`. If a PR is merged and the rule says to transition to Done, update status using the same GraphQL mutation from Phase 3.

7. **Clear session**: Delete `.ghpm/sessions/<number>.json`.

8. **Print summary**:
   ```
   Session ended for #<num> "<title>"

     Commits: <N>
     Decisions: <N> recorded
     Journal: posted to <issue-url>
     Status: <old> → <new> (or unchanged)
   ```

## Tips

- Multiple sessions can run in parallel (separate Claude instances on different branches).
- If you restart Claude on the same branch, `/ghpm-work <num>` detects the existing session and resumes.
- Decision detection is a nudge, not enforcement. You can always decline.

## See Also

- [ghpm-shared](../ghpm-shared/SKILL.md) — Prerequisites and error handling
- [ghpm-status](../ghpm-status/SKILL.md) — Project health dashboard
- [ghpm-suggest](../ghpm-suggest/SKILL.md) — Find what to work on next
- [ghpm-view](../ghpm-view/SKILL.md) — Query project items
