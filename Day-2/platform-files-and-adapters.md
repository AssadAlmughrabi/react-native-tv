# Platform Files and Adapters

Six targets, one codebase. Platform differences live in **platform files** and **adapters** — never as `if (platform)` checks inside feature code.

---

## 1. Platform files: most specific wins

The bundler picks the **most specific file** that exists for the target.

### Web (webpack)

One webpack build per brand, each with its own extension order:

```js
// webpack.config.js
const target = process.env.TARGET; // 'tizen' | 'webos'

module.exports = {
  resolve: {
    alias: { 'react-native$': 'react-native-web' },
    extensions: [
      `.${target}.web.tsx`, `.${target}.web.ts`, // 1. brand + web
      '.web.tsx', '.web.ts',                     // 2. any web
      '.tsx', '.ts', '.js',                      // 3. shared
    ],
  },
};
```

So for the Samsung build:

```text
exitApp.tizen.web.ts   ← picked first (Samsung only)
exitApp.web.ts         ← used if no brand file (LG, Samsung)
exitApp.ts             ← shared fallback
```

**`.tizen.web.tsx` beats `.web.tsx` beats `.tsx`** — same for `.webos.web.tsx` on LG.

### Native (Metro)

Same idea, built in: `.android.tsx` / `.ios.tsx` beat `.native.tsx` beat `.tsx`.

```text
exitApp.android.ts     ← Android TV
exitApp.ios.ts         ← Apple TV
exitApp.native.ts      ← both native
```

---

## 2. Adapters

An **adapter** is one small module that hides a platform difference behind a **shared interface**. Feature code imports the adapter, never the platform API.

```text
src/adapters/exitApp/
  index.ts              // shared interface + re-export
  exitApp.native.ts     // BackHandler.exitApp()
  exitApp.tizen.web.ts  // tizen.application.getCurrentApplication().exit()
  exitApp.webos.web.ts  // webOS.platformBack()
```

```ts
// exitApp.tizen.web.ts
export function exitApp() {
  tizen.application.getCurrentApplication().exit();
}
```

```ts
// exitApp.native.ts
import { BackHandler } from 'react-native';
export function exitApp() {
  BackHandler.exitApp();
}
```

Typical adapters:

| Adapter | Why it differs |
| --- | --- |
| `keys` | Back is 4 / `menu` / 461 / 10009 |
| `exitApp` | Each OS closes apps differently |
| `storage` | AsyncStorage vs `localStorage` / platform storage |
| `player` | Native video vs AVPlay (Samsung) / HTML5 video |
| `deviceInfo` | Model, OS version, engine |
| `FocusableCard` | Native focus vs Norigin |

---

## 3. Keep platform checks out of feature code

Bad — feature code knows every platform:

```tsx
// HomeScreen.tsx  ❌
if (Platform.OS === 'web' && isTizen) {
  tizen.application.getCurrentApplication().exit();
} else if (Platform.OS === 'web') {
  webOS.platformBack();
} else {
  BackHandler.exitApp();
}
```

Good — feature code calls the adapter:

```tsx
// HomeScreen.tsx  ✅
import { exitApp } from '@/adapters/exitApp';

exitApp();
```

Rules:

- `Platform.OS`, `isTizen`, `isWebOS`, `tizen.*`, `webOS.*` appear **only** inside adapters and platform files.
- Feature code (screens, rails, business logic) is the same for all six targets.
- Adding a new platform = adding adapter files, not editing features.
- Enforce with lint: block `tizen`, `webOS`, and `Platform.OS` imports/usage outside `src/adapters/`.

---

## Summary

- The bundler picks the most specific file: `.tizen.web.tsx` > `.web.tsx` > `.tsx` (and `.android/.ios` > `.native` > `.tsx` on Metro).
- One webpack build per brand, each with its own extension order.
- Adapters hide each platform difference behind one shared interface.
- Feature code never checks the platform — it only imports adapters.
