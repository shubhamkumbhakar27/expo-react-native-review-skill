# Rule 3 — Layer direction & thin routes

**Penalty −200/occurrence · Reward +50**

Dependencies flow one direction: `app (routes) → features → shared → lib`. Lower layers never import upward; features never import other features. Route files (`app/**`) are **thin** — a screen re-export or a few lines of composition, with **no** data-fetching, animation math, or platform branching. Files stay **under ~300 lines**.

## Why it matters
Business logic in route files and cross-feature imports create god-files and tangled coupling that block reuse, testing, and safe refactors.

## Detection
- `app/**/*.tsx` route file containing `useQuery`, `fetch`, `AbortController`, animation math, or large JSX trees.
- A file over ~300 lines (split it).
- `import ... from "@/features/<other-feature>/..."` inside a feature (cross-feature import).
- `lib/` or `shared/` importing from `features/` or `app/` (upward import).
- "Export everything" barrels (`export *`) or partial barrels.

## Bad
```tsx
// app/home/messages/index.tsx — 240 lines of hooks, list logic, handlers
export default function MessagesScreen() {
  const { channels, loading, error } = useChannels();
  /* ...200+ lines... */
}
```

## Good
```tsx
// app/home/messages/index.tsx — thin route
import { MessagesPage } from "@/components/pages/messages-page";
export default function MessagesRoute() {
  return <MessagesPage />;
}
```

## Scoring
- −200 per route file holding logic, per cross-feature/upward import, or per file >~300 lines.
- +50 when a PR extracts a route's logic into a screen/page component or feature hook.

## Edge cases
- A tiny route that composes 2–3 providers is fine; the line is "no data-fetching / no branching logic."
- Shared UI primitives may be imported anywhere; that is not an upward import.
