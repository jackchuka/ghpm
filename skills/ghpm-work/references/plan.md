# Phase 3: Plan

> **FIRST**: Update `.ghpm/sessions/<number>.json` → `"phase": "plan"` before doing anything else.

## Steps

1. **Explore the codebase** to understand the current state relevant to the issue. Read key files, search for related code, understand existing patterns.

2. **Draft a plan** — high-level approach:
   - What needs to change and why
   - Key components/files affected
   - Approach and alternatives considered
   - Risks or open questions

   Show to user for feedback. Iterate until approved.

3. **Post plan as issue comment**:
   ```bash
   gh issue comment <number> -R <content.repository> --body "$(cat <<'EOF'
   **Plan**

   <plan content>
   EOF
   )"
   ```

4. **Extract decisions from the plan**. Review the approved plan and identify distinct design choices (see `decisions.md` — "Plans contain decisions"). Nudge for each one.

