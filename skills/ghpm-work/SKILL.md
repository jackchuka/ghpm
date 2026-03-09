---
name: ghpm-work
description: "End-to-end work session on a GitHub Project item. Setup → Clarify → Plan → Implement → PR, with decisions captured throughout."
allowed-tools: Bash(gh:*), Bash(git:*), Read, Write, Edit, Grep, Glob
---

# ghpm-work

> **PREREQUISITE:** Read `../ghpm-shared/SKILL.md` for prerequisites and error handling.

End-to-end workflow for a GitHub Project item. `/ghpm-work <issue-number>` drives the full lifecycle.

If no argument, ask for the issue number.

## Flow

```
Setup → Clarify → Plan → Implementation Plan → Implement → Draft PR
                    ↕ decisions captured at every phase ↕
```

| Phase | Reference | What happens |
|-------|-----------|-------------|
| 1. Setup | `references/setup.md` | Resolve item, assign, branch, status sync, write session |
| 2. Clarify | `references/clarify.md` | Evaluate issue clarity, flesh out if needed, gather context |
| 3. Plan | `references/plan.md` | Explore codebase, draft approach, post as issue comment |
| 4. Impl Plan | `references/implementation-plan.md` | Concrete steps, files, tests — post as issue comment |
| 5. Implement | `references/implement.md` | Execute plan, write code, commit, verify |
| 6. Draft PR | `references/draft-pr.md` | Create draft PR with summary |

**Decisions**: See `references/decisions.md`. Capture at ANY phase — not just coding. Watch every interaction for choices worth recording.

**Wrap up**: See `references/wrap-up.md`. Triggered by "wrap up", "done", or agent hooks.

## Session Resume

Sessions track `phase` in `.ghpm/sessions/<number>.json` (schema in `../ghpm-shared/references/session.md`). On resume, continue from last recorded phase:

| `phase` | Resume from |
|---------|-------------|
| `setup` | Clarify |
| `clarify` | Plan |
| `plan` | Implementation Plan |
| `implementation_plan` | Implement |
| `implement` | Draft PR |
| `pr` | Print summary |

On resume, briefly show what's already done before continuing.

## Rules

- Each phase updates `"phase"` in the session file before moving on.
- The user can say "skip" to skip any phase.
- Multiple sessions can run in parallel on different branches.
- Restarting Claude on the same branch auto-resumes via session detection.

## See Also

- [ghpm-shared](../ghpm-shared/SKILL.md) — Prerequisites and error handling
- [ghpm-status](../ghpm-status/SKILL.md) — Project health dashboard
- [ghpm-suggest](../ghpm-suggest/SKILL.md) — Find what to work on next
- [ghpm-view](../ghpm-view/SKILL.md) — Query project items
