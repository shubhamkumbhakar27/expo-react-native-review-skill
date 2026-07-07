# Rule 6 — Server state in React Query only

**Penalty −100/occurrence · Reward +50**

Server state lives in **React Query only**. Never mirror fetched data into `useState`/`useReducer`/Context. Local UI state (open/closed, input text, selection) uses `useState`. Cross-tree app concerns (auth, network, alerts) use Context, kept minimal and logic-free. Derive values with `useMemo` — don't store derived copies.

## Why it matters
Copying server data into local state creates two sources of truth that drift: the cache updates but the mirror doesn't, producing stale UI, extra renders, and subtle bugs after refetch/invalidation.

## Detection
- `const [items, setItems] = useState([]); useEffect(() => setItems(data), [data]);` — mirroring.
- Storing query results in Context to "share" them (import the hook instead).
- `useState` holding a value that is purely derived from props/query data.
- Manual refetch-and-setState instead of query invalidation.

## Bad
```tsx
const { data } = useEvents();
const [events, setEvents] = useState<Event[]>([]);
useEffect(() => { if (data) setEvents(data); }, [data]); // second source of truth
```

## Good
```tsx
const { data: events = [] } = useEvents();          // single source of truth
const sorted = useMemo(() => [...events].sort(byDate), [events]); // derive, don't store
```

## Scoring
- −100 per server-data mirror into local state/Context, or per stored derived copy.
- +50 for consuming the query directly and deriving with `useMemo`.

## Edge cases
- Optimistic UI belongs in the mutation's `onMutate`/cache update, not a separate `useState`.
- Form draft state (before submit) is legitimate local state.
