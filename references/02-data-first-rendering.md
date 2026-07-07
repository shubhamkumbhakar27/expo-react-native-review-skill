# Rule 2 — Data-first rendering (no blank screens)

**Penalty −500/occurrence · Reward +50 · CRITICAL BLOCKER**

Every list/detail surface must resolve to one of **loading / empty / error+retry / ready**, and must render **data-first**: if cached data exists, show it — never gate it behind a spinner or drop it to an error screen on a failed refetch. A screen that can render an empty view with no loading/empty/error branch is the classic white-screen bug.

## Why it matters
Branching on `error` before `data`, or rendering a bare `<View/>` when a load settles with no items, produces blank screens with no recovery path — especially offline or when an SDK settles silently.

## Detection
- `if (error || !data) return <Error/>` — drops cached data to an error state. **Banned.**
- A `FlatList`/list with **no `ListEmptyComponent`**.
- A screen whose render returns nothing / an empty fragment on some state combination.
- `isLoading` gating the whole screen even when `data` is already cached.
- Missing retry affordance on the error branch.

## Bad
```tsx
// blanks the screen on any refetch error, even with cached data
if (isLoading) return <Spinner />;
if (error || !event) return <ErrorScreen />;
return <EventDetail event={event} />;
```

## Good
```tsx
// data-first: show data whenever it exists; skeleton only on cold load
const { data, status } = useApiQuery(...); // 'ready' | 'loading' | 'empty' | 'error'
if (status === "loading") return <Skeleton />;
if (status === "empty") return <EmptyState />;
if (status === "error" && !data) return <ErrorState onRetry={refetch} />;
return <EventDetail event={data} />; // stale data stays visible during refetch/error
```

## Scoring
- −500 for each surface that can blank or that branches on `error` before `data` (critical blocker).
- +50 for a screen that renders all four states through a shared `<ScreenState>` and keeps cached data visible.

## Edge cases
- Prefer a shared `<ScreenState>` component so the four branches are impossible to forget.
- Treat "offline" as *authenticated-but-offline*, not an error — render cache.
- Render precedence is **data → skeleton → error+retry**, never error before data.
