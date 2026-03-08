---
name: ghpm-status
description: "GitHub Project health dashboard. Shows workflow distribution, items needing attention, and available views."
allowed-tools: Bash(gh:*), Read, Grep
---

# ghpm-status

> **PREREQUISITE:** Read `../ghpm-shared/SKILL.md` for prerequisites and error handling.

Project health dashboard. Answers "how's the project doing?" at a glance.

## Arguments

- `/ghpm-status` — full dashboard for the project
- `/ghpm-status <scope>` — scoped to a team or component name (matched against field options in `.ghpm/config.json`)

## Workflow

### Phase 1: Load Config and Cache

Follow the startup sequence in `../ghpm-shared/SKILL.md` and load cache per `../ghpm-shared/references/cache.md`.

### Phase 2: Aggregate

3. Count items by workflow column (using the `workflow.field` name from config to find the matching field on each item). Include a count for items with no status.

4. If a scope argument was provided, filter items first:
   - Match the argument against all field option names in `.ghpm/config.json` (case-insensitive).
   - Filter items to only those matching the scope before aggregating.

5. Identify items needing attention:
   - **Untriaged**: items with no status, no component, or no assigned team (check which fields exist in config — not all projects have these fields).
   - **No assignee**: items in non-Done columns with empty assignees.
   - **Active InProgress**: items in InProgress status (or equivalent active column) — count them as a workload indicator. No date info is available, so don't claim staleness.

6. List available views with their layout type and filter text. Do NOT attempt to count items per view here — filter parsing is handled by `/ghpm-view`. Just show the view name, layout, and raw filter so the user knows what's available.

For key normalization when matching fields, see `../ghpm-shared/references/cache.md`.

### Phase 3: Format Output

7. Format per conventions in `../ghpm-shared/references/format.md`:

```
<Project Title> — <total> items

Workflow:
  <Column1>:    <count>
  <Column2>:    <count>
  ...
  (no status):  <count>

Attention:
  <N> items untriaged (no status/component/team)
  <N> items in progress with no assignee
  [Only show categories where count > 0]

Views:
  <view name>  (<layout>)  filter: <filter text or "none">
  ...

Use /ghpm-view <name> to drill into a view.
```

If scoped, prefix with the scope name:

```
<Project Title> — <scope> — <filtered count> of <total> items
...
```

## Tips

- Read-only — never modifies the project or its items.

## See Also

- [ghpm-shared](../ghpm-shared/SKILL.md) — Prerequisites and error handling
- [ghpm-view](../ghpm-view/SKILL.md) — Drill into a specific view
- [ghpm-suggest](../ghpm-suggest/SKILL.md) — Get work recommendations
