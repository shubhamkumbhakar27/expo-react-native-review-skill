# Rule 12 — Accessibility

**Penalty −50/occurrence · Reward +50**

Interactive elements are reachable and labeled for screen readers. Icon-only buttons have an `accessibilityLabel`; pressables declare `accessibilityRole` and, where relevant, `accessibilityState`. Text meets **WCAG AA** contrast (4.5:1 body, 3:1 large text/UI). Touch targets are ≥44×44. Decorative images/icons are hidden from the a11y tree.

## Why it matters
Icon-only controls with no label are invisible to VoiceOver/TalkBack users; low-contrast text is unreadable; tiny targets are unusable. These are correctness bugs, not polish.

## Detection
- `<Pressable onPress={...}><Icon/></Pressable>` with no `accessibilityLabel`.
- Touchables missing `accessibilityRole="button"`.
- Toggles/tabs without `accessibilityState={{ selected }}`.
- Muted text colors on tinted backgrounds below AA contrast.
- Tap target smaller than 44×44 with no `hitSlop`.

## Bad
```tsx
<Pressable onPress={close}>
  <CloseIcon />                    {/* announced as "button", no name */}
</Pressable>
```

## Good
```tsx
<Pressable
  onPress={close}
  accessibilityRole="button"
  accessibilityLabel="Close"
  hitSlop={8}
>
  <CloseIcon accessibilityElementsHidden importantForAccessibility="no" />
</Pressable>
```

## Scoring
- −50 per unlabeled icon-only control, missing role/state, sub-AA contrast, or undersized target.
- +50 when the diff adds labels/roles/contrast fixes across the touched surface.

## Edge cases
- Group related text so the reader announces a card once, not field-by-field, via `accessible` on the wrapper.
- Dynamic content changes should use `accessibilityLiveRegion`/announcements where important.
