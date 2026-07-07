# Rule 4 — No work in `renderItem` / list performance

**Penalty −200/occurrence · Reward +50**

Lists render through a shared `<List>` (or a `FlatList` with windowing defaults) using a **memoized row** and a **stable `keyExtractor`**. No allocation, `.reverse()`, `.map()`, `.filter()`, `new Intl.*`, or new closures created **inside `renderItem`** — precompute in a `useMemo`. Every list has a `ListEmptyComponent` (see rule 2).

## Why it matters
Work inside `renderItem` runs per row, per render. Reversing/mapping the whole dataset per row is O(n²) and janks scrolling; unstable `renderItem`/`keyExtractor` identity defeats cell memoization and re-renders every row.

## Detection
- `renderItem={({ item }) => { const x = [...data].reverse(); ... }}` — allocation/reverse per row.
- `new Intl.DateTimeFormat(...)` or regex literals built inside a row component.
- `renderItem` referencing the full `data` array (breaks stable identity).
- Row component not wrapped in `React.memo` while props are stable.
- Missing `keyExtractor` or using array index as key.
- Missing windowing props on large lists (`windowSize`, `maxToRenderPerBatch`, `initialNumToRender`).

## Bad
```tsx
renderItem={({ item, index }) => {
  const newestFirst = [...messages].reverse();      // O(n) per row → O(n²)
  const prev = newestFirst[index + 1];
  return <MessageBubble message={item} prev={prev} />; // non-memoized
}}
```

## Good
```tsx
const items = useMemo(() => withNeighbors(messages), [messages]); // precompute prev/next once
const renderItem = useCallback(({ item }) => <MessageBubbleMemo {...item} />, []);
<List data={items} renderItem={renderItem} keyExtractor={keyById} />;
```

## Scoring
- −200 per `renderItem` doing per-row allocation/iteration, or per unstable/non-memoized row.
- +50 for precomputed neighbors + memoized row + shared `<List>` windowing.

## Edge cases
- Hoist `Intl.*` formatters and regexes to module scope.
- Sendbird/SDK objects mutate in place — use `extraData` with a signature string to force targeted repaints.
