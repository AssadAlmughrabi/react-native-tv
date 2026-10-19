# Key Matrix per Platform and Brand

Every platform sends different key codes for the same remote button. Some keys never reach your app because the OS keeps them. Build one key map, and normalize everything to your own action names.

---

## 1. Key codes per platform

| Action | Android TV (`KeyEvent`) | Apple TV (`TVEventHandler`) | LG webOS (`keyCode`) | Samsung Tizen (`keyCode`) |
| --- | --- | --- | --- | --- |
| Up / Down | 19 / 20 | `up` / `down` | 38 / 40 | 38 / 40 |
| Left / Right | 21 / 22 | `left` / `right` | 37 / 39 | 37 / 39 |
| OK | 23 (also 66) | `select` | 13 | 13 |
| Back | 4 | `menu` | 461 | 10009 |
| Play/Pause | 85 | `playPause` | 415 / 19 | 10252 (or 415 / 19) |
| Rewind / FF | 89 / 90 | `rewind` / `fastForward` | 412 / 417 | 412 / 417 |
| Color keys | — (brand dependent) | — | 403–406 | 403–406 |
| Numbers | 7–16 | — | 48–57 | 48–57 |

Notes:

- **Native (Android TV, Apple TV):** `react-native-tvos` maps keys to event names (`up`, `select`, `playPause` …) via `useTVEventHandler`. Back on Android goes through `BackHandler`.
- **Web (LG, Samsung):** read `event.keyCode` on `keydown`.
- **Samsung:** only arrows, OK, and Back come by default. Other keys must be registered:

```js
tizen.tvinputdevice.registerKey('MediaPlayPause');
tizen.tvinputdevice.registerKey('ColorF0Red');
```

---

## 2. OS-reserved keys

These keys are handled by the OS. Your app can't use them (or must not block them):

| Platform | Reserved |
| --- | --- |
| Android TV | Home, Power, Volume, Mute, Input, Voice/Assistant |
| Apple TV | TV/Home, Siri, Volume. `menu` on the root screen must exit to Home |
| LG webOS | Home, Power, Volume, Input, voice, Magic Remote pointer mode |
| Samsung Tizen | Home, Power, Volume, Source, Exit (`10182`) closes the app |

Apple TV: to handle `menu` yourself on inner screens, enable it, and disable it again on the root screen so the app exits:

```ts
import { TVEventControl } from 'react-native';
TVEventControl.enableTVMenuKey();  // inner screens
TVEventControl.disableTVMenuKey(); // root screen
```

---

## 3. Brand differences (Android TV family)

Same OS, different remotes:

- **Amazon Fire TV:** has Menu key (`82`) and media keys. Alexa and Home are reserved.
- **Google TV / Chromecast:** small remote, no media keys or number keys — don't depend on them.
- **Operator boxes / TV makers (Sony, TCL, Philips…):** may add color keys, number keys, or custom app buttons with non-standard codes.

Rule: assume only **arrows + OK + Back** exist everywhere. Every other key is a shortcut, never the only way.

---

## 4. TV way: one normalized key map

```ts
type Action = 'up' | 'down' | 'left' | 'right' | 'select' | 'back' | 'playPause';

const WEB_KEYS: Record<number, Action> = {
  38: 'up', 40: 'down', 37: 'left', 39: 'right', 13: 'select',
  461: 'back',   // LG
  10009: 'back', // Samsung
  415: 'playPause', 19: 'playPause', 10252: 'playPause',
};
```

- App code listens to `Action`, never to raw codes.
- Log unknown key codes on real devices to grow the map.

---

## Summary

- Same button, different code on each platform and brand.
- Some keys (Home, Volume, Power, voice) belong to the OS — never rely on them.
- Only arrows, OK, and Back are guaranteed. Everything else is optional.
- Map raw codes to your own actions in one place, and test with each real remote.
