# App Suspend and Resume on TV

TV users leave apps without closing them: press **Home**, switch **input** (HDMI), open another app, or turn the TV **off** (many TVs only go to standby). Your app is **suspended**, then later **resumed** — minutes or days later — or silently **killed** to free memory.

---

## 1. How each platform tells you

| Platform | Suspend / resume signal |
| --- | --- |
| Android TV, Apple TV | `AppState`: `active` ↔ `background` (`inactive` on Apple TV briefly) |
| LG webOS | `visibilitychange` (`document.hidden`); `webOSRelaunch` when launched again while running |
| Samsung Tizen | `visibilitychange` (`document.hidden`) |

One adapter hides the difference:

```ts
// useAppVisible.native.ts
import { AppState } from 'react-native';

export function onAppVisibleChange(cb: (visible: boolean) => void) {
  const sub = AppState.addEventListener('change', (s) => cb(s === 'active'));
  return () => sub.remove();
}
```

```ts
// useAppVisible.web.ts
export function onAppVisibleChange(cb: (visible: boolean) => void) {
  const handler = () => cb(!document.hidden);
  document.addEventListener('visibilitychange', handler);
  return () => document.removeEventListener('visibilitychange', handler);
}
```

---

## 2. On suspend

Do it fast — the OS may give you very little time.

- **Pause playback** and save the position (Samsung AVPlay also needs `suspend()`).
- **Stop** timers, polling, trailers, animations.
- **Save state:** current route, focused item id, playback position — to storage, in case the app is killed.
- Don't start network requests that must finish.

---

## 3. On resume

- **Check the session:** refresh the token if it expired while away.
- **Refresh stale data** if away longer than a few minutes (Home rails, live channel guide). Show cached content first.
- **Check network** — Wi-Fi may have reconnected or changed.
- **Player:** restore the stream (Samsung AVPlay `restore()`), show it **paused** at the saved position, or resume — choose one behavior and keep it consistent. Live streams: jump to live.
- **Restore focus:** make sure something is focused; focus can be lost after resume. Use the saved focused item id.

```ts
useEffect(() =>
  onAppVisibleChange((visible) => {
    if (!visible) {
      player.pause();
      saveSession({ route, focusedId, position: player.position });
    } else {
      refreshIfStale();
      restoreFocus(focusedId);
    }
  }), []);
```

---

## 4. Killed in the background

Low-RAM TVs often kill suspended apps. Next launch is a **cold start**.

- On launch, read the saved state: go back to the last screen and focused item, offer "Continue watching" at the saved position.
- Don't restore into the **player** directly — land on Details/Home with a resume option.
- Ignore saved state older than a set time (example: 24 h).

---

## 5. How to test

- Press Home, wait, reopen — short and long (> 30 min) gaps.
- Switch HDMI input and back.
- Turn the TV off (standby) and on.
- Open several heavy apps in between to force a kill.
- Do all of it **during playback** and on screens with overlays open.

---

## Summary

- TV apps are suspended often: Home, input switch, standby; and may be killed silently.
- One adapter: `AppState` on native, `visibilitychange` on LG/Samsung.
- Suspend: pause, stop work, save route + focus + position quickly.
- Resume: refresh session and stale data, restore player and focus.
- Killed: cold start back to the last screen with a resume option, not straight into playback.
