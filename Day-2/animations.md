# Animations on TV

TV hardware is weak, and the web build has no native animation thread. Three rules:

1. **Animate only `transform` and `opacity`**
2. **No native driver on web** — plan for it
3. **Nothing may stay invisible** if an animation fails on an old engine

---

## 1. Use transform and opacity

`transform` (scale, translate) and `opacity` are cheap: the GPU moves pixels, no layout is recalculated.

| Animate | Avoid animating |
| --- | --- |
| `transform: scale` (focus growth) | `width`, `height` |
| `transform: translateX/Y` (slide in, rail scroll) | `top`, `left`, `margin`, `padding` |
| `opacity` (fade in/out) | `borderWidth`, `fontSize` |

- Layout properties re-layout the whole row or screen on every frame — visible stutter on TVs.
- Keep durations short: 150–250 ms for focus, ≤ 400 ms for screen transitions.
- Few animations at once. A rail of 20 cards all animating is too much.

---

## 2. Native driver: native yes, web no

**Native (Android TV, Apple TV):** `useNativeDriver: true` runs the animation on the UI thread, so it stays smooth even when JS is busy. It only supports `transform` and `opacity` — one more reason to use them.

```tsx
Animated.timing(scaleAnim, {
  toValue: focused ? 1.08 : 1,
  duration: 150,
  useNativeDriver: true,
}).start();
```

**Web (LG, Samsung):** `react-native-web` has no native driver. `Animated` runs in **JS**, updating styles each frame. If JS is busy (loading data, rendering a rail), the animation stutters.

On web, prefer **CSS transitions** — the browser runs them, not your JS:

```tsx
// FocusableCard.web.tsx
<View
  style={[
    styles.card,
    { transitionProperty: 'transform', transitionDuration: '150ms' },
    focused && { transform: [{ scale: 1.08 }] },
  ]}
/>
```

- Same look, split into `.native.tsx` (Animated) and `.web.tsx` (CSS transition).
- On the weakest TVs, consider no animation at all for focus — instant feedback beats a laggy one.

---

## 3. Nothing stays invisible on old engines

Common pattern: start at `opacity: 0`, animate to `1`. If the animation never runs or never finishes, the content **stays invisible** — a blank screen with no error.

Why it fails on old TVs:

- An unsupported API or CSS property in the animation path (example: `IntersectionObserver` for "fade in when visible", newer CSS like `aspect-ratio` or the separate `translate` property).
- A JS error before `.start()` is called.
- JS too busy, so the callback runs very late.

Fixes:

- **Visible by default:** the resting style must be visible. Animate *from* hidden only when you are sure the animation will run.
- **Safety timeout:** force the final state if the animation didn't finish.

```tsx
useEffect(() => {
  const anim = Animated.timing(opacity, { toValue: 1, duration: 250, useNativeDriver: true });
  anim.start();
  const t = setTimeout(() => opacity.setValue(1), 1000); // safety net
  return () => { anim.stop(); clearTimeout(t); };
}, []);
```

- **Fallback for missing APIs:** if `IntersectionObserver` is missing, show everything immediately.
- **Test with animations off:** content must look correct with no animation at all.
- **Test on the oldest LG/Samsung TV** you support — desktop Chrome hides all of these problems.

---

## Summary

- Animate only `transform` and `opacity`; keep it short and few at once.
- Native: `useNativeDriver: true`. Web: no native driver — use CSS transitions or skip animation.
- Resting state must be visible; add a safety timeout for fade-ins.
- Test on old engines: a failed animation must never leave content invisible.
