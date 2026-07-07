# Rule 3 — Feature-first architecture, layer direction & thin routes

**Penalty −200/occurrence · Reward +50**

The app is **feature-first**: code is organized by domain, dependencies flow in one direction, routes are thin, and infrastructure has no UI. A feature **owns** its screens, components, hooks, data-fetch usage, and types, and exposes a **public entrypoint** (`src/features/<feature>/index.ts`). Everything else about that feature is internal.

## Canonical structure
```
app/
  _layout.tsx          # composition root only: <AppProviders> + <Stack>
  routes/*             # expo-router routes. Each route imports ONE page from
                       #   src/features/<feature> and renders it. Nothing else (SRP).
src/
  features/*           # one folder per feature (auth, events, chat, profile),
                       #   bundling its screens, components, hooks, types, and
                       #   data-fetch usage. Exposes a public index.ts.
  shared/*             # cross-feature, domain-free UI + primitives
    design/ atoms/ molecules/ organisms/ components/
  lib/*                # infrastructure, NO UI
    endpoints/         # API paths + methods
    api/               # fetch client (timeout/abort), errors, query-string
    data-fetch/        # functions calling api.method(endpoints.path), used by hooks
    hooks/             # generic/shared hooks
    sendbird/          # instance, connection state, shared collection primitive
    storage/           # persistence primitives
  providers/*          # app-providers.tsx (single composition root) + providers
```

## Layer direction
`app/routes → src/features → src/shared → src/lib` (with `src/lib/sendbird` and `src/lib/storage` as infra consumed downward, never importing upward). Lower layers never import from higher ones, and **features never import other features** — share code via `src/shared`/`src/lib`, or consume a sibling only through its public `index.ts`.

## Why it matters
Grouping by technical type scatters one feature across the tree, so a change means editing many unrelated folders. Logic in routes and cross-feature imports create god-files and tangled coupling that block reuse, testing, and safe refactors. Feature-first folders with one-directional imports keep each domain cohesive and independently reviewable.

## Detection
- A file in `app/routes/**` doing anything beyond importing + rendering a feature page: `useQuery`, `fetch`, `useState`/effects, animation math, platform branching, or large JSX trees. Routes are single-responsibility — pick a page, render it.
- Feature-specific code placed in `src/shared/**` (or any generic bucket) instead of its `src/features/<feature>/` folder.
- One feature importing another feature's internals: a deep `@/features/<other>/...` import (allowed only through the sibling's public `index.ts` — prefer lifting shared code to `src/shared`).
- `src/shared/**` or `src/lib/**` importing from `src/features/**` or `app/**` (upward import).
- Data-fetch / endpoint / api-client logic living inside a feature or component instead of `src/lib`.
- A file over ~300 lines (split it), or "export everything" barrels (`export *`).

## Bad
```tsx
// app/routes/events/[id].tsx — route doing data work
export default function EventRoute() {
  const { id } = useLocalSearchParams();
  const { data, isLoading } = useQuery(...);   // data-fetching in a route
  return isLoading ? <Spinner /> : <EventDetail event={data} />;
}
```

## Good
```tsx
// app/routes/events/[id].tsx — thin route, single responsibility
import { EventDetailPage } from "@/features/events";
export default function EventRoute() {
  return <EventDetailPage />;
}

// src/features/events/index.ts — feature public surface
export { EventDetailPage } from "./screens/event-detail-page";
```

## Scoring
- −200 per route holding logic, per cross-feature/upward import, per feature-specific file in `src/shared`, per data/api logic outside `src/lib`, or per file >~300 lines.
- +50 when a PR keeps routes thin and colocates a feature's code under `src/features/<feature>` behind a public entrypoint.

## Edge cases
- `app/_layout.tsx` composing providers + `<Stack>` is fine — that's the composition root, not logic.
- Shared UI primitives (`src/shared`, `src/shared/design`) may be imported anywhere; that is not an upward import.
- Two features needing the same code → lift it to `src/shared`/`src/lib`; don't import feature-to-feature.
