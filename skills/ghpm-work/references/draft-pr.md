# Phase 6: Draft PR

> **FIRST**: Update `.ghpm/sessions/<number>.json` → `"phase": "pr"` before doing anything else.

## Steps

1. **Create a draft PR**:
   ```bash
   gh pr create --draft \
     --title "<issue title>" \
     --body "$(cat <<'EOF'
   Closes <content.repository>#<number>

   ## Summary
   <brief description of changes>

   ## Changes
   <bullet list of key changes>

   ## Test plan
   <how this was tested>
   EOF
   )" \
     -R <current repo> --head <branch>
   ```
   - Use the current repo (from `git remote get-url origin`), not the planning repo.
   - Use `Closes <content.repository>#<number>` (fully qualified) for cross-repo linking.

2. **Print final summary**:
   ```
   Work complete for #<num> "<title>"

     Branch:    <branch>
     PR:        <PR url>
     Commits:   <N>
     Issue:     <url>
   ```
