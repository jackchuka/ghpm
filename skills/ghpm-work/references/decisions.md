# Decision Capture

Decisions can happen at ANY phase. Watch throughout the entire workflow.

## What counts as a decision

Any choice that shapes how the system works and would be useful to remember. **If you had to choose between options (even implicitly), it's a decision.**

- **Explicit choices**: "Let's go with X", "I chose X because Y"
- **Behavioral/API design**: flag semantics, error handling, defaults, destructive vs safe
- **Architecture**: patterns, data models, directory structure, naming conventions
- **Trade-offs**: accepting limitations, scoping down, simplicity over completeness
- **Planning choices**: choosing an approach over alternatives, scoping out items
- **Rules and policies**: defining when something should/shouldn't happen
- **Selecting between alternatives** after discussion, even briefly

## Plans contain decisions

When a plan or implementation plan is approved, extract the key design choices from it and record them as individual decisions. A plan is a collection of decisions packaged together — don't let them pass unrecorded.

Examples of decisions hiding inside plans:
- A rule about when to apply a behavior vs not → policy decision
- Choosing a naming convention or prefix scheme → naming decision
- Defining what a flag or option does → API design decision
- Picking one data model over another → architecture decision

**After posting a plan comment, review it and nudge for each distinct design choice found.**

## When detected

> **Prompt** (`record_decision`): "Decision detected. Record to #\<num\>?\n  > \"\<proposed summary\>\"\n(y/n)"

User can also trigger: `decide: <text>` or just `decide`.

## Recording

Post as issue comment (the single source of truth):
```bash
gh issue comment <number> -R <content.repository> --body "**Decision:** <text>"
```
On failure → warn, offer retry. On success → confirm: `Recorded to #<num>.`

If declined, continue without recording.
