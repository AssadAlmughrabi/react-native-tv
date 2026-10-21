# React Compiler

A **Babel plugin** that adds memoization at build time. You write plain code; it does the `useMemo` / `useCallback` / `memo` work for you.

```js
// babel.config.js — used by Metro and the webpack web build
module.exports = {
  presets: ['module:@react-native/babel-preset'],
  plugins: ['babel-plugin-react-compiler'], // must run first
};
```

---

## What it memoises automatically

- Inline objects and styles: `style={{ … }}`
- Inline callbacks: `onPress={() => open(item)}`
- Derived values: `items.filter(…)`
- Child JSX — a child **skips re-render** when its props didn't change

On TV: cards no longer re-render just because the parent did.

---

## What it does NOT fix

- **Context changes** still re-render every consumer — keep focus state local.
- **Store subscribers** still re-render — keep selectors narrow.
- Big lists, images, and non-React code aren't faster.

---

## Gotchas

- **Rules of React required.** No mutating props/state, no reading `ref.current` during render, no conditional hooks. Otherwise the compiler **silently skips** the component.

```tsx
items.sort(byDate);                    // ❌ mutates a prop
const sorted = [...items].sort(byDate); // ✅
```

- **Find skipped components:** `eslint-plugin-react-hooks` lint rules, and the **"Memo ✨"** badge in React DevTools.
- **Mutable libraries** (change objects in place) can leave the UI stale. Opt out with `'use no memo'` at the top of the component.
- **Effects may run less often** — values are now stable between renders.

---

## Summary

- Auto-memoises values, callbacks, and JSX — fewer card re-renders on weak TVs.
- Doesn't fix context-wide re-renders: keep focus local.
- Follow the Rules of React, check the "Memo ✨" badge, use `'use no memo'` when needed.
