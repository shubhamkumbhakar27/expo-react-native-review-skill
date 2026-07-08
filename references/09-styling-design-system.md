# Rule 9 — Styling & design tokens

**Penalty −50/occurrence · Reward +50**

Styling goes through **NativeWind `className` + design tokens**, not raw magic values. No hardcoded hex/spacing that duplicates a token, no inline `style={{ ... }}` for values that a token/utility already expresses. The brand accent is used **sparingly** (1–2 focal points per screen); semantic green/red are reserved for their specific purpose (e.g. delta badges) and never decorative. Sans-serif only — no serif fonts.

## Why it matters
Duplicated magic values drift from the system, break theming/dark-mode, and erode visual consistency. Overusing the accent color destroys hierarchy.

## Detection
- Inline `style={{ color: "#FF7A49" }}` / `padding: 14` duplicating a token.
- Hex literals in components that match a defined token (should reference the token).
- Accent color applied to many elements on one screen (not sparingly).
- Semantic positive/negative colors used for decoration/category coloring.
- Ad-hoc font sizes outside the type scale; any serif font.

## Bad
```tsx
<View style={{ backgroundColor: "#FF7A49", padding: 14, borderRadius: 14 }}>
  <Text style={{ color: "#15803d" }}>Featured</Text>   {/* semantic green misused */}
</View>
```

## Good
```tsx
<View className="bg-accent p-3.5 rounded-lg">        {/* tokens via NativeWind */}
  <Text className="text-foreground font-semibold">Featured</Text>
</View>
```

## Scoring
- −50 per magic value duplicating a token, misused semantic color, or off-scale type.
- +50 when the diff replaces magic values with tokens / consolidates styles into the design layer.

## Edge cases
- Genuinely one-off dynamic values (computed at runtime) may use `style`; static values must use tokens.
- If the project has a `src/design` UI layer, prefer its primitives over raw `react-native` styling.
