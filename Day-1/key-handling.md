# Key Handling: Repeats, Debounce, Long Press, Back

Remote input is noisy. Keys repeat when held, some events fire twice, and many parts of the app want the Back key. Rules:

1. **Repeated keys** — expect them, keep them cheap
2. **Debouncing** — delay heavy work until focus settles
3. **Long presses** — nice extra, never the only way
4. **No double actions** — one press = one action
5. **One owner of Back** — a single place decides what Back does

---

## 1. Repeated keys

Holding an arrow sends the key again and again (many per second). Focus races across a rail.

- Let focus move freely — don't block repeats.
- Keep `onFocus` **light**: it runs for every card passed.
- Web build: check `event.repeat` to tell held keys from new presses.

```js
window.addEventListener('keydown', (e) => {
  if (e.repeat && e.keyCode === 13) return; // ignore held OK
});
```

---

## 2. Debouncing

Run heavy work (load details, start trailer, change background) only after focus **stops** on an item.

```tsx
function useDebounced<T>(fn: (v: T) => void, ms = 300) {
  const timer = useRef<ReturnType<typeof setTimeout>>();
  useEffect(() => () => clearTimeout(timer.current), []);
  return (v: T) => {
    clearTimeout(timer.current);
    timer.current = setTimeout(() => fn(v), ms);
  };
}

const preview = useDebounced(loadPreview);
<Card onFocus={() => preview(item)} />
```

- Focus style changes instantly; only the side effect waits.
- For held Rewind / Fast-forward, **throttle** seeks (example: one seek every 200 ms), and seek further the longer it's held.

---

## 3. Long presses

- Native: `onLongPress` on `Pressable`, or `longSelect` in `useTVEventHandler`.
- Not every remote sends it reliably, and users don't discover it.
- Use it only as a **shortcut**. The same action must also exist as a visible button or menu item.

---

## 4. No double actions

Common causes:

- Handling OK in **both** `onPress` and `useTVEventHandler` (`select`).
- Acting on **key down and key up** (Android `eventKeyAction`).
- User presses OK twice while the next screen is loading.

Fixes:

- Handle each key in **one** place.
- Act on key down only.
- Guard actions that navigate or call the network:

```tsx
const busy = useRef(false);

async function onPlay() {
  if (busy.current) return;
  busy.current = true;
  try {
    await startPlayback();
  } finally {
    busy.current = false;
  }
}
```

---

## 5. One owner of Back

If the player, a modal, a menu, and the screen all listen to Back, they fight: two things close, or nothing does.

Use **one** Back handler with a stack. The top layer handles Back first:

```ts
// backStack.ts
type Handler = () => boolean; // true = handled
const stack: Handler[] = [];

export function pushBack(h: Handler) {
  stack.push(h);
  return () => stack.splice(stack.indexOf(h), 1);
}

export function handleBack(): boolean {
  for (let i = stack.length - 1; i >= 0; i--) {
    if (stack[i]()) return true;
  }
  return false; // nothing handled: navigate back or exit
}
```

```tsx
// App root: the only BackHandler listener
useEffect(() => {
  const sub = BackHandler.addEventListener('hardwareBackPress', handleBack);
  return () => sub.remove();
}, []);

// Modal: register while open
useEffect(() => (open ? pushBack(() => (close(), true)) : undefined), [open]);
```

- Order: modal → overlay/menu → player controls → screen → navigation → exit.
- Web build: the same `handleBack()` is called from the single `keydown` listener (`461` LG, `10009` Samsung).

---

## Summary

- Held keys repeat: keep focus fast and `onFocus` light.
- Debounce heavy side effects; throttle held seeks.
- Long press is only a shortcut, never the only way.
- One press = one action: one handler per key, key down only, guard async actions.
- Back has one owner: a single handler with a stack, top layer first.
