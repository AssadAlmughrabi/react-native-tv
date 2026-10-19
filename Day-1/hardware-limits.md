# Hardware Limits on TV

A big screen does not mean a powerful device. Many TVs and sticks have phone-level hardware from years ago. Build for the weakest device you support.

---

## 1. Slow CPUs

- **Problem:** cheap TV chips are slow. Heavy JS work makes focus lag, and that feels broken when the user presses a key.
- **TV way:**
  - Keep JS work small on each focus change and each render.
  - Use `useNativeDriver: true` for animations so they don't run on the JS thread.
  - Use `React.memo` / `useCallback` on rows and cards to avoid extra re-renders.
  - Use Hermes (React Native's default JS engine) for faster startup.

```tsx
Animated.timing(scale, {
  toValue: 1.1,
  duration: 150,
  useNativeDriver: true, // runs on the native side, not JS
}).start();
```

---

## 2. Low RAM

- **Problem:** many devices have 1–2 GB of RAM shared with the OS and other apps. Too much memory means stutters, or the OS kills the app.
- **TV way:**
  - Use `FlatList` with windowing (`initialNumToRender`, `windowSize`), not `ScrollView` with every item.
  - Load images at the size they are shown, not full 4K posters.
  - Cache images, but with limits.
  - Unmount or pause screens and videos that are not visible.

---

## 3. Old OS versions

- **Problem:** TVs get few updates. Many users run old Android TV / Fire OS / tvOS versions, with older WebViews and missing APIs.
- **TV way:**
  - Check your `minSdkVersion` / tvOS deployment target against real devices.
  - Avoid new platform APIs without a fallback.
  - Check that native libraries support TV and old versions.

---

## Summary

- TV hardware is often weak: slow CPU, little RAM, old OS.
- Keep focus changes fast, and run animations on the native thread.
- Virtualize lists, use smaller images, and free memory you don't need.
- Test on the cheapest, oldest device you support, not just the emulator.
