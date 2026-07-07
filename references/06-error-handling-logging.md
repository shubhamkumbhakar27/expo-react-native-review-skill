# Rule 7 — Error handling & logging

**Penalty −200/occurrence · Reward +50**

Every route/screen tree is wrapped in an **`ErrorBoundary`** so a render throw shows a fallback instead of a white screen. Errors are **caught, typed, and reported** to telemetry — never swallowed. No `console.log`/`console.error` in shipped code (use the app's logger/telemetry sink). No empty `catch {}`.

## Why it matters
An uncaught render error unmounts the whole tree to a blank screen. Empty catches hide failures; `console.*` leaks noise and PII to device logs and is invisible in production.

## Detection
- Route/stack with no `ErrorBoundary` wrapper.
- `catch {}` or `catch (e) {}` with no handling/reporting.
- `console.log` / `console.warn` / `console.error` in non-test source.
- `.catch(() => {})` on a promise.
- Errors shown via `Alert` from deep inside the transport layer (should surface as typed errors).

## Bad
```ts
try {
  await joinEvent(id);
} catch (e) {
  console.log(e); // swallowed + logged to device console
}
```

## Good
```tsx
// route
<ErrorBoundary fallback={<ErrorState onRetry={reset} />}>
  <EventDetailScreen />
</ErrorBoundary>;

// action
try {
  await joinEvent(id);
} catch (err) {
  logger.error("joinEvent failed", { id, err }); // typed + reported
  toast.error(messageFor(err));
}
```

## Scoring
- −200 per unbounded tree (no ErrorBoundary), empty catch, or `console.*` in source.
- +50 for ErrorBoundary coverage + telemetry reporting in the diff.

## Edge cases
- A dev-only `if (__DEV__) console.log` is tolerable but prefer the logger.
- React Query errors surface through the query's `error` state (rule 2), not thrown to the boundary.
