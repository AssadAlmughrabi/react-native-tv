# Platform.isTV, useTVEventHandler, BackHandler

Three core APIs from `react-native-tvos` for the native family (Android TV, Apple TV):

1. **`Platform.isTV`** — am I running on a TV?
2. **`useTVEventHandler`** — listen to remote keys
3. **`BackHandler`** — handle the Back button

---

## 1. Platform.isTV

`true` on Android TV and Apple TV, `false` on phones and tablets.

```tsx
import { Platform } from 'react-native';

const isTV = Platform.isTV;
const isAppleTV = Platform.isTV && Platform.OS === 'ios';
const isAndroidTV = Platform.isTV && Platform.OS === 'android';
```

- Apple TV reports `Platform.OS === 'ios'`, so always combine with `isTV`.
- Use it to switch layouts, sizes, and focus behavior between mobile and TV.
- **Web build (LG, Samsung):** `react-native-web` does not set it to `true`. Use your own flag (example: a webpack `DefinePlugin` constant).

---

## 2. useTVEventHandler

Hook that receives remote events while the component is mounted.

```tsx
import { useTVEventHandler } from 'react-native';

export function Player() {
  useTVEventHandler((evt) => {
    switch (evt.eventType) {
      case 'playPause':
        togglePlay();
        break;
      case 'rewind':
        seekBy(-10);
        break;
      case 'fastForward':
        seekBy(10);
        break;
    }
  });

  return <Video />;
}
```

Common `eventType` values: `up`, `down`, `left`, `right`, `select`, `longSelect`, `playPause`, `rewind`, `fastForward`, `menu`.

- Don't use it to move focus — the native focus engine already does that. Use it for **extra actions** (media keys, shortcuts).
- On Android, `evt.eventKeyAction` tells key down (`0`) vs key up (`1`). Act on one, or you run the action twice.
- Keep the handler light — it runs on every key press.

---

## 3. BackHandler

Handle the Back button: go back one step, close a modal, or let the app exit.

```tsx
import { useEffect } from 'react';
import { BackHandler } from 'react-native';

export function useBack(onBack: () => boolean) {
  useEffect(() => {
    const sub = BackHandler.addEventListener('hardwareBackPress', onBack);
    return () => sub.remove();
  }, [onBack]);
}

// usage: return true = handled, false = let the system handle it (exit)
useBack(() => {
  if (menuOpen) {
    closeMenu();
    return true;
  }
  return false;
});
```

- **Android TV:** the remote Back key fires `hardwareBackPress`.
- **Apple TV:** the `menu` key fires it only after `TVEventControl.enableTVMenuKey()`. Disable it on the root screen so the app can exit to Home.
- **Web build:** `BackHandler` does nothing. Listen to `keydown` for `461` (LG) and `10009` (Samsung).
- Always remove the listener on unmount, or old screens keep catching Back.

---

## Summary

- `Platform.isTV` detects TV; combine with `Platform.OS` to tell Android TV from Apple TV.
- `useTVEventHandler` is for extra remote keys, not for moving focus.
- `BackHandler` returns `true` to handle Back, `false` to let the system exit.
- None of these work on the LG/Samsung web build — handle `keydown` there.
