# Filter Syntax

## Supported Tokens

Applied left to right; all must match for an item to be included.

| Syntax | Meaning |
|--------|---------|
| `field:value` | Item's field matches value (case-insensitive) |
| `field:val1,val2` | Item's field matches any listed value |
| `-field:value` | Item's field does NOT match value |
| `no:field` | Item's field is empty/null |
| `no:field1,field2` | Item is missing ANY of the listed fields |
| `-no:field` | Item's field has a value |
| `is:open` | Item's status is not the last workflow column (assumed Done) |

## Unsupported Tokens

Skip these and note them in output:

- Temporal filters (`<@current`, `>@today`)
- Any other syntax not listed above

When a filter token is skipped, append: `Note: filter token "<token>" was skipped (unsupported syntax). Results may include extra items.`

## Key Normalization

See `cache.md` for how to normalize field keys when matching filter tokens against cached item data.
