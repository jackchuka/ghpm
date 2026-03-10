# Phase 1: Setup

## Steps

1. **Project config**: Follow startup sequence in `../../ghpm-shared/SKILL.md`.

2. **Check for existing session**: Read `.ghpm/sessions/<issue-number>.json` per `../../ghpm-shared/references/session.md`. If exists:
   - Read `phase` from the session file. If `phase` is `"done"`, tell the user this session is already complete and stop.
   - If `phase` is missing, treat as `"setup"` — all setup steps already happened, so resume from Clarify.
   - Check out the existing branch.
   - Resume from the recorded phase per the resume table in SKILL.md.
   - Briefly show what's already done (branch, status, etc.) before continuing.

3. **Load cache** per `../../ghpm-shared/references/cache.md`.

4. **Find the item**: Search cached items where `content.number` matches. If not found, refresh cache and retry. If still missing, stop.

5. **Identify repo**: Extract `content.repository`. Check `git remote get-url origin`. Warn on mismatch but continue.

6. **Get GitHub user**: `gh api user --jq '.login'`

7. **Self-assign** if not in cached `assignees`:
   ```bash
   gh issue edit <number> -R <content.repository> --add-assignee <login>
   ```
   Update cached `assignees` in `.ghpm/cache.json`.

8. **Create branch**:
   - Read `conventions.branch` from config (default: `{type}/{issue-number}/{slug}`).
   - Auto-detect type from labels/title: bug→`fix`, enhancement/feature→`feat`, documentation→`docs`, test→`test`, refactor→`refactor`, fallback→`chore`.
   - Slug: lowercase title, non-alphanumeric→hyphens, collapse, max 5 words.
   ```bash
   git checkout -b <type>/<issue-number>/<slug>
   ```
   If exists, check it out instead.

9. **Sync status**: Per `conventions.status_sync`. If currently Planned/ReadyForDev, update to InProgress:
   ```bash
   gh api graphql -f query='
   mutation {
     updateProjectV2ItemFieldValue(input: {
       projectId: "<project.id>"
       itemId: "<item.id>"
       fieldId: "<workflow.field_id>"
       value: { singleSelectOptionId: "<InProgress column id>" }
     }) { projectV2Item { id } }
   }'
   ```
   > **Prompt** (`status_sync`): "Update status to InProgress? (y/n)"

   Update cached `status` in `.ghpm/cache.json`.

10. **Write session file** to `.ghpm/sessions/<issue-number>.json` with `"phase": "setup"`.

11. **Print**:
    ```
    Session started for #<num> "<title>"
      Branch: <branch>
      Status: <old> → <new>
      Issue:  <url>
    ```
