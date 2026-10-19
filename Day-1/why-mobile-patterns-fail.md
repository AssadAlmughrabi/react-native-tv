# Why Mobile Patterns Fail on TV

Mobile patterns assume a finger on glass. TV has a remote with a D-pad, OK, and Back. Anything that needs touch, swiping, or precision breaks.

---

## 1. Swipe-to-open drawers

- **Problem:** there is no swipe on a remote. A hidden drawer is invisible and unreachable.
- **TV way:** a **visible side menu** that expands when focus moves left onto it. ✅

```tsx
<View style={{ flexDirection: 'row' }}>
  <SideMenu />   {/* always visible, focus Left to reach it */}
  <Content />
</View>
```

---

## 2. Pull-to-refresh

- **Problem:** no pull gesture. Users can't trigger it.
- **TV way:** refresh automatically (on screen focus, on app resume, or on a timer). Add a visible "Refresh" button only if really needed.

---

## 3. Small targets

- **Problem:** small icons and links are hard to see from 3 m and hard to show focus on.
- **TV way:** big cards and buttons, clear focus style, enough spacing so focus movement is obvious.

---

## 4. Gestures

- **Problem:** swipe, long-press, pinch, double-tap — none map to a D-pad (long-press is unreliable across remotes).
- **TV way:** every action reachable by arrows + OK. Carousels move with Left/Right, not swipe. Secondary actions go in a visible button or menu.

---

## Summary

- No touch on TV — if it needs a finger, it fails.
- Hidden UI (drawers, gestures) becomes unreachable; make it visible and focusable.
- Replace pull-to-refresh with automatic refresh.
- Big targets, clear focus, every action reachable with D-pad + OK.
