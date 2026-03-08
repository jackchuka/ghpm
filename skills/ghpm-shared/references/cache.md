# Cache Management

## Loading Cache

1. Read `.ghpm/cache.json` from the current directory.
2. Check `fetched_at` against `ttl_minutes` to determine freshness.
   - **Fresh** (within TTL): use cached items directly.
   - **Stale or missing**: automatically re-fetch (see below) — no user prompt needed.
   - **Re-fetch fails**: if a stale cache exists, use it as fallback and note the age inline (e.g., "Using cached data from 2 hours ago"). If no cache exists at all, stop with an error.

## Re-fetching

```bash
gh project item-list <number> --owner <owner> --format json --limit 1000
```

> **Note:** `--limit` caps the total items returned with no cursor pagination. Projects exceeding 1000 items will have silently truncated data.

Write the result to `.ghpm/cache.json`:

```json
{
  "version": 1,
  "fetched_at": "<ISO8601 timestamp>",
  "ttl_minutes": 30,
  "items": [<all fetched items>]
}
```

## Key Normalization

Cache item field keys use spaces and mixed case (e.g., `"Assigned Team"`). View filter strings use lowercase hyphens (e.g., `assigned-team`). When matching:

1. Normalize both sides: lowercase, replace hyphens with spaces.
2. Compare normalized strings.

Example: filter `assigned-team:Backend` → normalize to `assigned team` → matches cache key `"Assigned Team"`.
