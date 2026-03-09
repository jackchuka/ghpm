# Decision Capture

Decisions can happen at ANY phase. Watch throughout the entire workflow.

## What counts as a decision

Any choice that shapes how the system works and would be useful to remember:

- **Explicit choices**: "Let's go with X", "I chose X because Y"
- **Behavioral/API design**: flag semantics, error handling, defaults, destructive vs safe
- **Architecture**: patterns, data models, directory structure, naming conventions
- **Trade-offs**: accepting limitations, scoping down, simplicity over completeness
- **Planning choices**: choosing an approach over alternatives, scoping out items
- **Selecting between alternatives** after discussion, even briefly

## When detected

Nudge:
```
Decision detected. Record to #<num>?
  > "<proposed summary>"
(y/n)
```

User can also trigger: `decide: <text>` or just `decide`.

## Recording

1. Post issue comment first (durable store):
   ```bash
   gh issue comment <number> -R <content.repository> --body "**Decision:** <text>"
   ```
2. On success → append to session `decisions` per `../../ghpm-shared/references/session.md`.
3. On failure → warn, offer retry. Do NOT write to session file.
4. Confirm: `Recorded to #<num>.`

If declined, continue without recording.
