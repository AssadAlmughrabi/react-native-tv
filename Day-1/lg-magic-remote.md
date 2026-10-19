# LG Magic Remote: Pointer and Hover

LG's Magic Remote is a **motion pointer**: the user waves it and a cursor moves on screen, like a mouse. It also has a D-pad, OK, and a scroll wheel. So on LG, your app must handle **both** D-pad focus and pointer hover/click — often switching between them in seconds.

---

## 1. How it behaves

- Moving the remote shows the **cursor**. It hides after a few seconds without movement, or when the user presses an arrow key.
- The pointer sends normal web events: `mousemove`, `mouseover`, `mouseout`, `click`.
- The scroll wheel sends `wheel` events.
- webOS tells you when the cursor shows or hides with the `cursorStateChange` event:

```js
document.addEventListener('cursorStateChange', (e) => {
  const pointerMode = e.detail.visibility; // true = cursor visible
});
```

---

## 2. Problems if you ignore it

- **Two highlights:** D-pad focus on one card, hover on another. User doesn't know what OK will press.
- **Hover does nothing:** pointer over a card, but the focus engine still thinks another card is focused.
- **Unreachable content:** pointer can't "swipe" a rail, so items off-screen can't be reached.
- **Tiny targets:** pointer aim from the couch is shaky.

---

## 3. TV way

### One state: hover = focus

When the pointer enters an element, **move focus to it**. Then there is only one highlight, and OK / click / D-pad all continue from the same place.

```tsx
// FocusableCard.web.tsx
const { ref, focused, focusSelf } = useFocusable({
  focusKey: `card-${item.id}`,
  onEnterPress: onPress,
});

return (
  <View
    ref={ref}
    onMouseEnter={focusSelf} // pointer hover -> focus
    onClick={onPress}        // pointer click -> same action as OK
    style={[styles.card, focused && styles.cardFocused]}
  >
    <Text>{item.title}</Text>
  </View>
);
```

- No separate `:hover` style — only the `focused` style.
- When the pointer leaves, keep focus where it is, so the D-pad continues from there.

### Make everything reachable by pointer

- Show **left/right arrow buttons** on rails while the cursor is visible, to scroll pages.
- Support the **wheel** for vertical scrolling.
- Make sure buttons that only appear on focus also appear on hover.

### Pointer-friendly sizes

- Big hit areas; no tiny icons placed close together.
- No important action only reachable by long press or hover-only menus.

---

## Summary

- LG Magic Remote is a pointer, so hover and click happen on LG.
- Keep one state: hover moves focus, click does the same as OK.
- Rails need visible arrow buttons and wheel support for pointer users.
- Test on LG with both the D-pad and the pointer, switching between them.
