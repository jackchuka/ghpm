# Session Wrap Up

Triggered by "wrap up", "done", or agent hooks (see `../../ghpm-shared/references/integrations.md`).

## Steps

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

   **Status:** <current status> (PR: <merged|open|draft|not created>)
   ```

5. **Post journal as issue comment**:
   ```bash
   gh issue comment <number> -R <repo> --body "<journal>"
   ```

6. **Status sync**: If PR is merged and `conventions.status_sync` says to transition to Done, update status.

7. **Clear session**: Delete `.ghpm/sessions/<number>.json`.

8. **Print**:
   ```
   Session ended for #<num> "<title>"
     Commits: <N>
     Decisions: <N> recorded
     Journal: posted to <issue-url>
     Status: <old> → <new> (or unchanged)
   ```
