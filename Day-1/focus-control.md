# nextFocus* and TVFocusGuideView

The native focus engine picks the "nearest" element in the pressed direction. That works for clean grids, but fails on uneven layouts, gaps, and side menus. Two tools to take control:

1. **`nextFocus*`** — point one element to another, per direction
2. **`TVFocusGuideView`** — a focus area that redirects, remembers, or traps focus

---

## 1. nextFocus*

Props: `nextFocusUp`, `nextFocusDown`, `nextFocusLeft`, `nextFocusRight`. Each takes a **node handle** of the target.

```tsx
import { useRef, useState, useEffect } from 'react';
import { Pressable, Text, findNodeHandle } from 'react-native';

export function Toolbar() {
  const playRef = useRef(null);
  const [playHandle, setPlayHandle] = useState<number | null>(null);

  useEffect(() => {
    setPlayHandle(findNodeHandle(playRef.current));
  }, []);

  return (
    <>
      <Pressable ref={playRef}><Text>Play</Text></Pressable>
      <Pressable nextFocusUp={playHandle ?? undefined}>
        <Text>Settings</Text>
      </Pressable>
    </>
  );
}
```

- Good for **one-off fixes**: "from here, Up must go to Play".
- Handle exists only after mount, so set it in `useEffect`.
- Mainly an Android API — check tvOS support in your `react-native-tvos` version.
- Many manual links get hard to maintain. For areas, use `TVFocusGuideView`.

---

## 2. TVFocusGuideView

A `View` that controls focus entering it. Works on Android TV and Apple TV.

### Redirect: `destinations`

When focus enters the guide, send it to a chosen element.

```tsx
import { TVFocusGuideView } from 'react-native';

<TVFocusGuideView destinations={[playRef.current]}>
  {/* empty space or short row: focus lands on Play */}
</TVFocusGuideView>
```

`playRef.current` is `null` on first render — keep the ref in state after mount (like the handle above) so the guide gets the real element.

Use it to fill **gaps**: a short row under a long one, so Down never gets "stuck".

### Remember: `autoFocus`

When focus comes back into the guide, it goes to the **last focused child** (or the first one).

```tsx
<TVFocusGuideView autoFocus style={styles.row}>
  {items.map((item) => <Card key={item.id} item={item} />)}
</TVFocusGuideView>
```

Use it on **every row and side menu**: Up from row 3 back to row 2 returns to the same card, not a random one.

### Trap: `trapFocus*`

Stop focus from leaving in a direction: `trapFocusUp`, `trapFocusDown`, `trapFocusLeft`, `trapFocusRight`.

```tsx
<TVFocusGuideView trapFocusUp trapFocusDown trapFocusLeft trapFocusRight>
  <ConfirmDialog />
</TVFocusGuideView>
```

Use it for **modals and dialogs**, so focus can't escape behind them.

---

## 3. When to use which

| Problem | Tool |
| --- | --- |
| One element goes to the wrong place | `nextFocus*` |
| Gap / short row / uneven layout | `TVFocusGuideView` + `destinations` |
| Row or menu should remember position | `TVFocusGuideView` + `autoFocus` |
| Modal must keep focus inside | `TVFocusGuideView` + `trapFocus*` |

---

## Summary

- The focus engine guesses by position; uneven layouts break the guess.
- `nextFocus*` fixes single links, using a node handle set after mount.
- `TVFocusGuideView` handles areas: redirect with `destinations`, remember with `autoFocus`, block with `trapFocus*`.
- Wrap each row and side menu in an `autoFocus` guide by default.
