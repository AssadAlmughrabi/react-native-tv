# Safe Areas, Overscan, TV-Safe Margins

Many TVs **crop the edges** of the picture (overscan). Anything placed at the very edge may be cut off on some sets. Keep important content inside a **safe area**.

---

## 1. Overscan

- Old habit from CRT TVs: the TV zooms slightly and hides the outer few percent.
- Many modern TVs still do it by default, depending on model and picture settings.
- You can't detect it from the app — assume it can happen.

---

## 2. TV-safe margins

Common rule: keep content **5% inside** each edge.

| Canvas | Horizontal | Vertical |
| --- | --- | --- |
| 1920×1080 (our canvas) | 96 px | 54 px |
| Android TV (960×540 dp) | 48 dp | 27 dp |
| Apple TV (tvOS guideline) | 80 pt | 60 pt |

Pick one value for the app (example: 96 / 54 on the 1920 canvas, via `scale()`) and use it everywhere.

```ts
// safeArea.ts
import { scale } from './scale';

export const SAFE = {
  horizontal: scale(96),
  vertical: scale(54),
};
```

```tsx
<View style={{ flex: 1, paddingHorizontal: SAFE.horizontal, paddingVertical: SAFE.vertical }}>
  {/* screen content */}
</View>
```

---

## 3. What goes where

| Inside the safe area | Can go full-bleed (to the edge) |
| --- | --- |
| Text, titles, logos | Background images and colors |
| Buttons, menus, focusable cards | Hero artwork, video |
| Player controls, subtitles | Gradients and decoration |
| Toasts, error messages | Rails scrolling off the right edge |

- **Rails:** the first card starts inside the safe area; cards can run off the right edge to show "more content this way".
- **Focus growth:** a focused card that scales up 10% needs room — don't let it grow outside the safe area or under another row.
- **Side menu:** starts inside the left margin, even when collapsed.

---

## 4. How to check

- Turn on a **debug overlay** that draws the safe area lines on screen.
- Check every screen at 720p, 1080p, and 4K.
- On a real TV, switch picture size between "16:9" / "Just Scan" / overscan modes and confirm nothing important is cut.

```tsx
{__DEV__ && (
  <View
    pointerEvents="none"
    style={[StyleSheet.absoluteFill, {
      margin: 0,
      borderColor: 'red',
      borderLeftWidth: SAFE.horizontal, borderRightWidth: SAFE.horizontal,
      borderTopWidth: SAFE.vertical, borderBottomWidth: SAFE.vertical,
      opacity: 0.2,
    }]}
  />
)}
```

---

## Summary

- TVs may crop the edges (overscan); you can't detect it.
- Keep all text, focusable elements, and controls ~5% inside each edge (96 / 54 px on 1920×1080).
- Backgrounds, artwork, and video can go full-bleed.
- Use one shared `SAFE` value and a debug overlay to check every screen.
