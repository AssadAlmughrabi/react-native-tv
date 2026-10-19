# Old Browser Engines on LG and Samsung TVs

LG (webOS) and Samsung (Tizen) TV apps are web apps. With React Native they run through `react-native-web`, inside the TV's built-in browser engine. That engine is fixed to the TV's model year and **never gets upgraded**. A 2019 TV runs a 2017-era Chromium forever.

---

## 1. Engine version per target

### LG webOS

| webOS TV | Year | Engine |
| --- | --- | --- |
| 26 | 2026 | Chromium 132 |
| 25 | 2025 | Chromium 120 |
| 24 | 2024 | Chromium 108 |
| 23 | 2023 | Chromium 94 |
| 22 | 2022 | Chromium 87 |
| 6.x | 2021 | Chromium 79 |
| 5.x | 2020 | Chromium 68 |
| 4.x | 2018–2019 | Chromium 53 |
| 3.x | 2016–2017 | Chromium 38 |
| 2.x | 2015 | WebKit 538.2 |
| 1.x | 2014 | WebKit 537.41 |

### Samsung Tizen

| Tizen | Year | Engine |
| --- | --- | --- |
| 10.0 | 2026 | Chromium 130 |
| 9.0 | 2025 | Chromium 120 |
| 8.0 | 2024 | Chromium 108 |
| 7.0 | 2023 | Chromium 94 |
| 6.5 | 2022 | Chromium 85 |
| 6.0 | 2021 | Chromium 76 |
| 5.5 | 2020 | Chromium 69 |
| 5.0 | 2019 | Chromium 63 |
| 4.0 | 2018 | Chromium 56 |
| 3.0 | 2017 | Chromium 47 |
| 2.4 | 2016 | WebKit |

Sources: [LG webOS TV web engine](https://webostv.developer.lge.com/develop/specifications/web-api-and-web-engine), [Samsung web engine specifications](https://developer.samsung.com/smarttv/develop/specifications/web-engine-specifications.html)

---

## 2. Why it matters

The oldest engine you support decides which JS and CSS you can use. Examples:

| Feature | Needs | Breaks on |
| --- | --- | --- |
| Arrow functions, `let`/`const` | Chromium 49 | webOS 3.x, Tizen 3.0 |
| `async` / `await` | Chromium 55 | webOS 4.x |
| Optional chaining `?.` | Chromium 80 | webOS 6.x and older, Tizen 6.0 and older |
| Flexbox `gap` | Chromium 84 | webOS 6.x and older, Tizen 6.0 and older |

If the engine can't parse your bundle, you get a **blank screen**, often with no visible error.

---

## 3. TV way

- **Pick a floor:** decide the oldest model year you support (example: 2018+ means Chromium 53 on LG, 56 on Samsung).
- **Transpile to that floor** with Babel + `browserslist`:

```json
"browserslist": ["chrome 53"]
```

- **Polyfill** missing APIs (`core-js`), and check that your libraries are transpiled too.
- **Check the engine on a device:**

```js
console.log(navigator.userAgent); // shows Chrome/xx version
```

- **Test on real old TVs** or the official emulators/simulators, not only desktop Chrome.

---

## Summary

- LG and Samsung TVs run a fixed browser engine tied to their model year.
- Find the engine version of each target, and choose the oldest one as your floor.
- Transpile and polyfill to that floor. Modern syntax can break the whole app.
- Always test on the oldest TV you support.
