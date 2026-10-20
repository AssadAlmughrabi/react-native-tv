# Design at 1920×1080 and Scale

Design every screen on **one canvas: 1920×1080**. Then scale it to whatever the device reports. Check that it still looks right at **720p** and **4K**.

---

## 1. Why one canvas

- Designers and developers use the same numbers (a card is 320 px wide in Figma and in code).
- Layout stays the same on every TV — only the size changes.
- 1080p is the most common UI resolution on TVs, even on 4K panels.

---

## 2. What each platform reports

| Platform | Typical window size the app sees |
| --- | --- |
| Android TV (1080p or 720p device) | 960×540 dp |
| Apple TV (HD and 4K) | 1920×1080 points |
| LG / Samsung web | 1920×1080 CSS px (some older/cheap TVs: 1280×720) |

- A 4K TV usually draws the **UI at 1080p** and only the video at 4K.
- So "4K" mostly means: same layout, sharper images.

---

## 3. Scale from the canvas

```ts
// scale.ts
import { Dimensions } from 'react-native';

const DESIGN_WIDTH = 1920;
const { width } = Dimensions.get('window');

export const scale = (size: number) => (size * width) / DESIGN_WIDTH;
```

```tsx
const styles = StyleSheet.create({
  card: { width: scale(320), height: scale(180), borderRadius: scale(12) },
  title: { fontSize: scale(28) },
  screen: { paddingHorizontal: scale(96), paddingVertical: scale(54) }, // 5% safe area
});
```

- Use `scale()` for **every** size: width, height, margins, fonts, borders, radius.
- Build styles once at startup — TV screen size doesn't change while running.
- Round where needed (`Math.round`) to avoid blurry 1 px borders.

---

## 4. Check 720p and 4K

### 720p (1280×720)

- Text still readable? Smallest font should still be ≥ ~16 px after scaling.
- Thin borders and focus outlines don't disappear.
- No text clipping or wrapping into two lines.

### 4K (3840×2160)

- Images are sharp: load bigger image sizes when `PixelRatio.get()` or the window is large.
- Layout looks the same as 1080p, not tiny in a corner (happens when sizes are hard-coded).

### How to test

- **Android TV emulator:** create 720p and 1080p (and 4K) TV profiles.
- **Apple TV simulator:** Apple TV and Apple TV 4K.
- **Chrome (web build):** DevTools responsive mode at 1280×720, 1920×1080, 3840×2160.
- Then confirm on at least one real 720p and one real 4K TV.

---

## Summary

- Design once at 1920×1080; that's the shared canvas.
- Scale every size from the canvas with one `scale()` helper.
- 4K TVs usually render UI at 1080p — use sharper images, not a new layout.
- Check 720p (readability, thin lines) and 4K (image quality, no hard-coded sizes).
