# Two Families of Targets

One codebase, two very different ways to ship:

1. **Native React Native** — Android TV and Apple TV, bundled by **Metro**
2. **Web build** — LG (webOS) and Samsung (Tizen), bundled by **webpack**

---

## 1. Native: Android TV and Apple TV (Metro)

- **Runtime:** real native app using `react-native-tvos` (React Native fork with TV support).
- **Bundler:** Metro, same as mobile React Native.
- **JS engine:** Hermes, shipped inside the app, so you control the version.
- **Focus:** native focus engine of the OS. `Pressable` gets `focused`, `hasTVPreferredFocus`, `TVFocusGuideView`.
- **Output:** `.apk` / `.aab` for Android TV, `.ipa` for Apple TV.

```sh
npx react-native run-android   # Android TV emulator/device
npx react-native run-ios --simulator "Apple TV"
```

---

## 2. Web: LG and Samsung (webpack)

- **Runtime:** web app running in the TV's built-in browser engine (see [old-browser-engines.md](old-browser-engines.md)).
- **Bundler:** webpack + Babel, with `react-native` aliased to `react-native-web`.
- **JS engine:** the TV's old Chromium — you **don't** control it, so transpile and polyfill.
- **Focus:** no native focus engine. Use a spatial navigation library (example: `@noriginmedia/norigin-spatial-navigation`).
- **Keys:** handle remote key codes yourself (Back is `461` on webOS, `10009` on Tizen).
- **Output:** `.ipk` for LG, `.wgt` for Samsung.

```js
// webpack.config.js
module.exports = {
  resolve: {
    alias: { 'react-native$': 'react-native-web' },
    extensions: ['.web.tsx', '.web.ts', '.tsx', '.ts', '.js'],
  },
};
```

---

## 3. Sharing code between them

- Keep screens, logic, and data in shared files.
- Split only what differs with platform extensions:

```text
FocusableCard.native.tsx   // Metro picks this (Android TV, Apple TV)
FocusableCard.web.tsx      // webpack picks this (LG, Samsung)
```

- Typical split points: focus handling, key events, video player, storage.

---

## Summary

- Native family: Android TV + Apple TV, Metro, Hermes, native focus.
- Web family: LG + Samsung, webpack, old browser engine, custom focus and keys.
- Share most code, split focus, keys, and player with `.native` / `.web` files.
- Test each family on its own devices — passing on one says nothing about the other.
