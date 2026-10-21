# Startup on Slow TVs

On a weak TV, startup can take 5–10 s. Users stare at a splash screen with a remote that does nothing. Three levers: **smaller bundle**, **lazy loading**, **deferred work**.

Goal: **first focusable Home screen as fast as possible**; everything else later.

---

## 1. Bundle size

Every byte of JS must be loaded and parsed before the first render.

- **Native (Hermes):** bundle is precompiled to bytecode — parsing is cheap, but size still costs load time and memory.
- **Web (LG, Samsung):** the old engine parses plain JS on a slow CPU — **bundle size matters most here**. Transpiling for old engines also makes the bundle bigger.

Cut it:

- Find big dependencies: `webpack-bundle-analyzer` (web), `source-map-explorer` (both).
- Replace heavy libs: full `lodash` → single functions, `moment` → `date-fns` / `Intl`.
- Ship only the **polyfills** your oldest target needs.
- Remove dev-only code from production builds.

---

## 2. Lazy loading

Don't load screens the user doesn't see at startup: Player, Search, Settings, Account.

```tsx
const PlayerScreen = lazy(() => import('./screens/PlayerScreen'));

<Suspense fallback={<StateView kind="loading" onAction={retry} />}>
  <PlayerScreen />
</Suspense>
```

- **Web (webpack):** `import()` splits code into separate chunks, loaded on demand.
- **Native (Metro):** no real code splitting; turn on **inline requires** so modules run only when first used:

```js
// metro.config.js
transformer: {
  getTransformOptions: async () => ({ transform: { inlineRequires: true } }),
},
```

---

## 3. Deferred work

At startup, do only what's needed to show Home. Push the rest after the first render.

| Startup (now) | After first render (later) |
| --- | --- |
| Read saved session/token | Analytics and crash reporting init |
| Load cached Home data | Remote config, feature flags |
| Render first rails | Rails below the screen |
| Set initial focus | Prefetch images, EPG, recommendations |

```ts
useEffect(() => {
  const t = setTimeout(() => {
    initAnalytics();
    prefetchMoreRails();
  }, 0); // after first frame; or requestIdleCallback where available
  return () => clearTimeout(t);
}, []);
```

- Show **cached Home** immediately, refresh from the network in the background.
- Keep the native **splash** until the first real frame — no blank screen in between.

---

## 4. Measure

- Time from launch to **first focusable element** — on the weakest device, cold start.
- Compare before and after every change; watch it in every release.

---

## Summary

- Target: Home focusable ASAP; everything else later.
- Small bundle: analyse, replace heavy libs, minimal polyfills — critical on old web engines.
- Lazy load non-startup screens: `import()` on web, inline requires on Metro.
- Defer analytics, config, prefetch, and below-the-fold rails until after first render.
- Measure cold start on the weakest TV.
