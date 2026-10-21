# Feature Detection, Not Device Checks

### 1. Test the feature, not the device

Don't ask "is this a Samsung TV?". Ask "does this TV support what my code needs?".

```js
// Don't
if (navigator.userAgent.includes('Tizen')) { ... }

// Do
if ('IntersectionObserver' in window) { ... } else { /* fallback */ }
```

### 2. Why device checks fail

- User-agent strings can be wrong or change.
- Many TV models and firmware versions exist, so you can't list them all.
- A feature check also works on TVs you never tested.

### 3. Old TVs are the problem

LG and Samsung TVs often run an old browser that never updates. Newer APIs may be missing, so detection plus a fallback keeps the app working.

### 4. Polyfills fill the gaps

A polyfill adds a missing deature so you can still write modern code.

- Use `core-js` instead of writing your own.
- Load a polyfill only when the feature is missing.
- Syntax like `?.` needs Babel, not a polyfill.
- Set `browserslist` to the oldest TV you support.
