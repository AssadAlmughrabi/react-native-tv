# Focus Basics

On TV, focus is the user's position. Three things to get right:

1. **Focusable vs non-focusable** — what the D-pad can land on
2. **`onFocus` / `onBlur`** — react when focus arrives or leaves
3. **`hasTVPreferredFocus`** — where focus starts

---

## 1. Focusable vs non-focusable

| Focusable by default | Not focusable by default |
| --- | --- |
| `Pressable` | `View` |
| `TouchableOpacity`, `TouchableHighlight` | `Text` |
| `TextInput` | `Image` |
| `Button` | `ScrollView` content (only its focusable children) |

- Only make **interactive** things focusable. Focus on a plain label is a dead stop.
- To skip a touchable from focus:

```tsx
<Pressable focusable={false}>{/* decorative */}</Pressable>
```

- If nothing on screen is focusable, the remote does nothing — a "frozen" screen.

---

## 2. onFocus / onBlur

Fire when focus lands on or leaves an element.

```tsx
import { Pressable, Text } from 'react-native';

export function Card({ item, onPreview, hasTVPreferredFocus = false }) {
  return (
    <Pressable
      hasTVPreferredFocus={hasTVPreferredFocus}
      onFocus={() => onPreview(item)}  // update hero/background
      onBlur={() => {}}                // stop preview, etc.
      style={({ focused }) => [styles.card, focused && styles.cardFocused]}
    >
      <Text style={styles.title}>{item.title}</Text>
    </Pressable>
  );
}
```

- For **styling**, prefer the `focused` value in `style={({ focused }) => …}` — no state needed.
- Use `onFocus` for **side effects**: update a preview, scroll a row, load details.
- Keep it fast. Heavy work in `onFocus` makes focus lag on slow TVs. Debounce expensive actions (example: load trailer after ~300 ms of focus).

---

## 3. Initial focus with hasTVPreferredFocus

Tells the TV which element gets focus when the screen appears.

```tsx
<Pressable hasTVPreferredFocus onPress={play}>
  <Text>Play</Text>
</Pressable>
```

- Put it on the **most likely action** (Play, Continue, first item of first row).
- Use it on **one element per screen**. Several conflict, and the result is unpredictable.
- It applies when the element mounts. If content loads later, render the preferred element only after data is ready, or focus may land somewhere else.
- When returning with Back, restore the last focused item instead of resetting to the top.

```tsx
{items.map((item, i) => (
  <Card key={item.id} item={item} hasTVPreferredFocus={i === lastFocusedIndex} />
))}
```

---

## Summary

- Only interactive elements should be focusable; a screen needs at least one.
- Style focus with `focused`, use `onFocus` / `onBlur` for light side effects.
- `hasTVPreferredFocus` on one element per screen, placed on the most likely action.
- Restore last focus when coming back — never make the user start over.
