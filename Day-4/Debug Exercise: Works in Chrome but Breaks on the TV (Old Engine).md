# Debug Exercise: Works in Chrome but Breaks on the TV (Old Engine)

TV web engines are often several years behind desktop Chrome. Code that runs fine in a modern browser can fail on the TV with a blank screen, missing styles, or a broken feature.

## 1. Reproduce

Confirm it works in desktop Chrome and fails on the TV. Note the TV model and its Chromium version, since older models can be several versions behind.

## 2. Find the error

Connect the remote debugger to the TV and check the console. A syntax error usually means the engine cannot parse newer JavaScript. A "not a function" or "undefined" error usually means a missing API.

## 3. Common causes

| Symptom | Likely cause |
|---|---|
| Blank screen, syntax error | Newer JS syntax not supported (for example `?.`, `??`) |
| "Not a function" error | Missing API or method (for example `Array.at`, `structuredClone`) |
| Layout or style broken | CSS feature not supported (for example `gap` in flexbox, `aspect-ratio`) |
| Works in dev, fails in build | Build output targets modern browsers |

## 4. Fix

- Set the build target to match the TV engine (Babel or TypeScript target, browserslist).
- Add polyfills for missing APIs.
- Add CSS fallbacks or prefixes, or avoid the unsupported feature.
- Check support on caniuse or MDN against the TV's Chromium version.

## 5. Verify

Run the production build on the oldest TV you support, not only on desktop Chrome or a new TV.