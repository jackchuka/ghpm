# Output Formatting Conventions

## Item References

- Single item: `#<num> "<title>"`
- With status: `#<num> "<title>" (<status>)`
- With status and size: `#<num> "<title>" (<status>, <size>)`
- With assignee: `#<num> <title> (@user)`

## Counts

- Inline counts in headings: `## <Column Name> (N)`
- Summary counts: `<N> items`

## Section Headers

Use `##` for top-level groupings (workflow columns, iteration groups).

## Tables

Markdown tables for table/roadmap layouts:

```
| # | Title | Assignee | Status |
|---|-------|----------|--------|
| 1 | ...   | ...      | ...    |
```

## Board Layout

Grouped lists by workflow column:

```
## <Column> (N)
- #<num> <Title>
- #<num> <Title> (@user)
```

## Truncation

If more than 30 items, show first 30 and note: "Showing 30 of <total>. Narrow your filter to see more."

## Skill Cross-References

When suggesting the user run another skill, use the `/ghpm-<name>` format:

- "Run `/ghpm-status` for project overview."
- "Use `/ghpm-view <name>` to drill into a view."
- "Run `/ghpm-init` to set up project tracking."
