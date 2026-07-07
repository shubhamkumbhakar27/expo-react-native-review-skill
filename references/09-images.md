# Rule 9 — Remote images via `expo-image`

**Penalty −50/occurrence · Reward +50**

Remote/network images use **`expo-image`** with an explicit `contentFit`, a `transition`, and — inside lists — a `recyclingKey`. React Native's core `<Image>` with a `{ uri }` source is banned for remote images (no disk cache, worse memory behavior, flicker on recycle). Provide a `placeholder`/fallback and fixed dimensions to avoid layout shift.

## Why it matters
Core `<Image>` re-downloads on every mount, flickers when list cells recycle, and lacks a memory/disk cache — causing jank and data waste on remote content.

## Detection
- `import { Image } from "react-native"` (or `@/design`) used with `source={{ uri }}`.
- `expo-image` used in a list without `recyclingKey`.
- Missing `contentFit` (defaults surprise; be explicit).
- Remote image with no width/height → layout shift.

## Bad
```tsx
import { Image } from "react-native";
<Image source={{ uri: user.avatarUrl }} style={{ width: 40, height: 40 }} />;
```

## Good
```tsx
import { Image } from "expo-image";
<Image
  source={user.avatarUrl}
  contentFit="cover"
  transition={150}
  recyclingKey={user.id}          // stable identity in recycled cells
  placeholder={blurhash}
  style={{ width: 40, height: 40 }}
/>;
```

## Scoring
- −50 per remote image using core `<Image>`, or `expo-image` in a list without `recyclingKey`.
- +50 when the diff standardizes remote images on `expo-image` with cache + recyclingKey.

## Edge cases
- Local static assets (`require(...)`) may use either; the rule targets **remote** images.
- SVGs use `react-native-svg`, not `expo-image`.
