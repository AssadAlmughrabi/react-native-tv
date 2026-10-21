# Virtualised Rails and Focus

`FlatList` only renders items **near the visible area** and unmounts the rest to save memory. On TV that collides with focus: **an unmounted item can't keep focus, and can't receive it.**

---

## What goes wrong

- **Lost focus:** the focused card scrolls out of the render window and unmounts → nothing is focused.
- **Dead end at the window edge:** holding Right moves faster than new cards render → no card to the right yet, focus stops or jumps.
- **Forgotten position:** a rail far down the screen unmounts entirely; coming back, it starts at the first card.
- **`removeClippedSubviews`** detaches off-screen views on Android → focus jumps or disappears.
- **Web (Norigin):** an unmounted card unregisters its focus key → focus falls back somewhere random.

---

## Fixes

- **Big enough window:** render the visible cards plus a few on each side.
- **Fixed sizes:** `getItemLayout`, so scrolling to an index is exact and cheap.
- **No clipped subviews** on TV rails.
- **Scroll the focused card into view** on focus, so it never leaves the window.
- **Remember focus by id outside the card** (store or ref), not in the card's own state — it dies when the card unmounts.

```tsx
<FlatList
  ref={listRef}
  horizontal
  data={items}
  keyExtractor={(i) => i.id}
  initialNumToRender={8}
  maxToRenderPerBatch={6}
  windowSize={5}
  removeClippedSubviews={false}
  getItemLayout={(_, index) => ({ length: CARD_W, offset: CARD_W * index, index })}
  renderItem={({ item, index }) => (
    <Card
      item={item}
      onFocus={() => {
        rememberFocus(railId, item.id);
        listRef.current?.scrollToIndex({ index, viewPosition: 0.1, animated: true });
      }}
    />
  )}
/>
```

- Restore on return: scroll to the saved index first (`initialScrollIndex`), then give that card `hasTVPreferredFocus`.
- Keep a fallback (first card) when the saved id is gone.

---

## Summary

- Virtualisation unmounts off-screen cards; unmounted cards can't hold or take focus.
- Render a few extra cards, use `getItemLayout`, turn off `removeClippedSubviews`.
- Scroll the focused card into view on focus.
- Store last focus by id outside the card, and restore by scrolling first, focusing second.
