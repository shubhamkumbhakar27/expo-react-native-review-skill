# Rule 13 — Feature-first module boundaries

**Penalty −100/occurrence · Reward +50**

The app is **feature-first**: code is grouped by domain (`events`, `messages`/`chat`, `profile`, `notifications`), not by technical type. A feature **owns** its screens, components, hooks, services, and types, and exposes a **public entrypoint** (a feature `index.ts`). Everything else about that feature is **internal** and must not be imported from outside. Only genuinely cross-cutting primitives (design system, platform/native wrappers, shared UI, query/http lib) live in shared layers and may be imported anywhere.

This complements Rule 3 (layer direction): Rule 3 governs *vertical* flow (`app → features → shared → lib`); this rule governs *horizontal* boundaries between sibling features.

## Why it matters
Grouping by type (`components/`, `hooks/`, `types/` buckets that mix every domain) scatters a single feature across the tree, so a change means editing ten unrelated folders and re-reading half the app to find what's related. Reaching into another feature's internals couples them, so one feature's refactor silently breaks another. Feature-first folders keep each domain cohesive, independently reviewable, and safe to move or delete.

## Detection
- New feature-specific code dropped into a generic type bucket (`components/organisms/<feature>-*`, `hooks/use-<feature>-*`, `types/<feature>.ts`) instead of a feature folder.
- A feature importing another feature's **internal** file: `import ... from "@/features/events/..."` from inside `messages` (reach through the feature's public `index.ts`, or lift the shared piece to `shared`).
- Generic, reusable UI/util placed **inside** a feature folder (belongs in `shared`/design).
- A feature's logic split across top-level `hooks/`, `services/`, and `types/` rather than colocated with the feature it serves.
- Deep imports that bypass the feature entrypoint (`.../events/components/internal/foo`) from another feature.

## Bad
```ts
// src/features/messages/hooks/use-channel-header.ts
// messages reaching into events' internals — sibling coupling
import { EventHero } from "@/features/events/components/event-detail/event-hero";
import { useEventHero } from "@/features/events/hooks/use-event-hero";
```

## Good
```ts
// events exposes a public surface
// src/features/events/index.ts
export { EventHero } from "./components/event-detail/event-hero";
export { useEventHero } from "./hooks/use-event-hero";

// messages consumes only the public entrypoint (or a shared primitive)
// src/features/messages/hooks/use-channel-header.ts
import { EventHero, useEventHero } from "@/features/events";
```

## Scoring
- −100 per feature-specific file placed in a generic type bucket, per cross-feature deep import, per reusable primitive buried inside a feature, or per feature whose code is scattered across unrelated top-level folders.
- +50 when a PR colocates a feature's screen/components/hooks/types under its feature folder and consumes siblings only through their public entrypoint.

## Edge cases
- Two features that legitimately share code: extract the shared piece to `shared`/design — don't import one feature from the other.
- Truly one-off UI used by a single feature stays inside that feature; only promote to `shared` on the second consumer.
- `app/**` route files map 1:1 to a feature and stay thin (Rule 3) — they compose the feature's public screen, not its internals.
