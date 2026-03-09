# Phase 2: Clarify

> **FIRST**: Update `.ghpm/sessions/<number>.json` → `"phase": "clarify"` before doing anything else.

## Steps

1. **Fetch the full issue**:
   ```bash
   gh issue view <number> -R <content.repository> --json title,body,labels,comments
   ```

2. **Evaluate clarity**. Is the issue actionable?
   - Enough context to understand what needs to change?
   - Clear scope (what's in, what's out)?
   - Written in shorthand or another language that needs expansion?

   If clear and actionable, say so and move on.

   If it needs work, show what's unclear and offer to improve:
   ```
   Issue #<num> needs clarification:
   - <what's missing or vague>

   Want me to flesh it out? (y/n)
   ```
   If yes, draft an improved body preserving original intent. Add structure (context, scope, action items). Show draft, confirm, then:
   ```bash
   gh issue edit <number> -R <content.repository> --body "<improved body>"
   ```

3. **Ask the user** if they have additional context or constraints not captured in the issue.
