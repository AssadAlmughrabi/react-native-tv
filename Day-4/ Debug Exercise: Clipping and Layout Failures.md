# Debug Exercise: Clipping and Layout Failures

Typical bugs: focused (scaled) items cut off at the edges, text overflow, content too close to the screen edge, layout breaking at 720p or 4K.

## 1. Reproduce

Note the screen, the resolution, and the trigger, for example "focus the last card in a row at 720p".

## 2. Inspect

In the remote debugger, select the broken element and check its size, any parent with `overflow: hidden` or fixed width and height, and the `transform: scale()` applied on focus.

## 3. Common causes

| Symptom | Likely cause |
|---|---|
| Focused item clipped | Parent has `overflow: hidden` and no room for the scaled size |
| Text cut off | Fixed width or height, no wrapping or ellipsis |
| Too close to edge | No safe-area margin |
| Breaks at 720p or 4K | Fixed pixel values built only for 1080p |

## 4. Fix

- Add padding so scaled focus has space.
- Avoid `overflow: hidden` on parents of focusable items.
- Use relative units (`%`, `rem`, `vw`) instead of fixed pixels.
- Let text wrap or truncate with an ellipsis.
- Keep a safe-area margin around the screen edges.

## 5. Verify

Re-test at 720p, 1080p, and 4K. Focus the first, middle, and last item in every row.