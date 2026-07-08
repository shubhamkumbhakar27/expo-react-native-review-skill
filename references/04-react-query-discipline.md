# Rule 4 — React Query key + cache discipline

**Penalty −100/occurrence · Reward +50**

Query keys come from a **typed factory** (`lib/query/keys.ts`), never inline arrays. Cache timing comes from **named tiers** (`reference` ~1h, `detail` ~15m, `list` ~5m), not inline `staleTime` literals scattered across hooks. Mutations **invalidate** the relevant keys. One coherent refetch policy (`refetchOnReconnect: true`, `refetchOnWindowFocus: false`).

## Why it matters
Inline key arrays cause silent cache mismatches (a write invalidates a differently-shaped key and never updates the read). Scattered `staleTime` literals make caching behavior unpredictable and un-tunable.

## Detection
- `useQuery({ queryKey: ["events", id], ... })` — inline array literal.
- `getKeys(...props: unknown[])` style ad-hoc key builders.
- `staleTime: 1000 * 60 * 5` literals inside hooks.
- Mutations that don't `invalidateQueries` (or that `await` a refetch that can be paused offline).

## Bad
```ts
useQuery({ queryKey: ["event", id], staleTime: 900000, queryFn: () => getEvent(id) });
// elsewhere a mutation invalidates ["events", id] — different shape, silent miss
```

## Good
```ts
import { keys } from "@/lib/query/keys";
import { tier } from "@/lib/query/cache-policy";
useQuery({ queryKey: keys.event.detail(id), ...tier.detail, queryFn: () => getEvent(id) });
// mutation: queryClient.invalidateQueries({ queryKey: keys.event.detail(id) });
```

## Scoring
- −100 per inline key array, ad-hoc key builder, inline `staleTime`, or non-invalidating mutation.
- +50 for typed keys + tiered cache policy + correct invalidation in the diff.

## Edge cases
- Prefetch on navigation intent (`prefetchQuery` on row tap) is a bonus (+50), not required.
- Don't `await` refetches that may be paused offline — fire and forget.
