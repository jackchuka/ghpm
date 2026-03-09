# Phase 4: Implementation Plan

## Steps

1. **Draft an implementation plan** — concrete, ordered steps:
   - Specific files to create/modify/delete
   - Key code changes per file
   - Test strategy (what to test, how)
   - Order of operations (dependencies between steps)

   Show to user for feedback. Iterate until approved.

2. **Post implementation plan as issue comment**:
   ```bash
   gh issue comment <number> -R <content.repository> --body "$(cat <<'EOF'
   **Implementation plan**

   <implementation plan content>
   EOF
   )"
   ```

3. Update session: `"phase": "implementation_plan"`.
