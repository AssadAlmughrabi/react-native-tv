# Keep Mounted vs Unmount

When the user leaves a screen, you either **keep it mounted** (hidden, still in memory) or **unmount** it (destroyed, rebuilt on return). On TV this is a trade-off between **memory** and **focus**.

---

## 1. The trade-off

| | Keep mounted | Unmount |
| --- | --- | --- |
| Back to the screen | Instant | Rebuild + reload data |
| Focus on return | Still there, scroll position kept | Must restore by id + scroll |
| Memory | Grows with every screen kept | Freed |
| Background work | Timers, videos, re-renders keep running | Stops |
| Focus leaks | Hidden screen may still take focus | None |

On weak TVs (1–2 GB RAM) keeping everything mounted gets the app killed. Unmounting everything makes Back slow and loses focus.

---

## 2. Keep mounted: when and how

Keep mounted:

- **Home** — the most common Back target.
- The **screen under an overlay** (modal, player controls) — the opener must still exist to restore focus.
- The **previous screen** in a short stack (Home → Details).

Do it safely:

- **Pause work when hidden:** stop videos, trailers, timers, polling.
- **Freeze re-renders:** with `react-native-screens`, `freezeOnBlur: true` stops hidden screens from re-rendering.
- **No focus leaks:** hidden screens must not be focusable. Check that focus can't move into a screen behind the current one (especially on web with Norigin — pause or remove its focusables).

```tsx
import { useIsFocused } from '@react-navigation/native';

function Hero({ trailerUrl }) {
  const isFocused = useIsFocused();             // screen visible?
  return <Video source={{ uri: trailerUrl }} paused={!isFocused} />;
}
```

---

## 3. Unmount: when and how

Unmount:

- **Player** — heavy (decoder, buffers). Always unmount when leaving.
- **Deep stacks** — Details → Details → Details: keep only the last few.
- **Rarely revisited screens** — Settings, Account, Search results.
- Any screen with big images or long lists when memory is low.

Make return feel instant anyway:

- **Restore focus by item id**, and scroll it into view.
- **Cache data** (in memory or query cache) so the screen renders without waiting for the network.
- Show the cached content first, refresh in the background.

---

## 4. Rules of thumb

- Keep mounted: Home + the screen directly behind the current one + anything under an overlay.
- Unmount: player, deep history, rarely used screens.
- Whatever stays mounted: paused, frozen, not focusable.
- Whatever is unmounted: focus restored by id, data from cache.
- Measure memory on the weakest device while navigating deep and back many times.

---

## Summary

- Keeping mounted = fast Back and kept focus, but costs memory and can leak focus.
- Unmounting = saves memory, but you must restore focus by id and cache data.
- Keep Home and the screen under overlays mounted; unmount the player and deep history.
- Mounted-but-hidden screens must pause work, stop re-rendering, and never take focus.
