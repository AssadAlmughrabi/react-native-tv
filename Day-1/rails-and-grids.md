# Rails and Grids: Remember and Restore Focus

Most TV screens are **rails** (horizontal rows of cards) stacked vertically, or **grids**. Users scroll deep, open an item, press Back — and expect to land on the **same card**. Starting over from the top is a big press cost.

Two jobs:

1. **Remember** the last focused item (per rail and per screen)
2. **Restore** it by **item id** when returning

---

## 1. Remember by id, not by index

- **Id:** stays correct when data refreshes, reorders, or items are added.
- **Index:** points to a different item as soon as the list changes.

Store per screen: which rail, and which item in each rail.

```ts
// focusMemory.ts — simple in-memory store
type ScreenMemory = { railId: string | null; items: Record<string, string> };
const memory: Record<string, ScreenMemory> = {};

export function rememberFocus(screen: string, railId: string, itemId: string) {
  const m = (memory[screen] ??= { railId: null, items: {} });
  m.railId = railId;
  m.items[railId] = itemId;
}

export function getFocus(screen: string) {
  return memory[screen] ?? { railId: null, items: {} };
}
```

Call it from `onFocus`:

```tsx
<Card
  item={item}
  onFocus={() => rememberFocus('home', rail.id, item.id)}
/>
```

- Each rail keeps its own last item, so moving Up/Down between rails also returns to the right card.

---

## 2. Restore when returning

```tsx
function Rail({ rail, screen }) {
  const { railId, items } = getFocus(screen);
  const savedId = items[rail.id];
  const savedIndex = Math.max(0, rail.items.findIndex((i) => i.id === savedId));

  return (
    <FlatList
      horizontal
      data={rail.items}
      keyExtractor={(item) => item.id}
      initialScrollIndex={savedIndex}
      getItemLayout={(_, index) => ({ length: CARD_W, offset: CARD_W * index, index })}
      renderItem={({ item }) => (
        <Card
          item={item}
          hasTVPreferredFocus={rail.id === railId && item.id === savedId}
          onFocus={() => rememberFocus(screen, rail.id, item.id)}
        />
      )}
    />
  );
}
```

Key points:

- **Virtualized lists:** a card far down the list may not be rendered. Scroll to it first (`initialScrollIndex` + `getItemLayout`), or focus can't land on it.
- Do the same for the **vertical** list of rails, so the saved rail is on screen.
- Only **one** card gets `hasTVPreferredFocus` — the saved card in the saved rail.
- Wrap each rail in `TVFocusGuideView autoFocus` so moving between rails also keeps position while the user is on the screen.

---

## 3. Fallback when the id is gone

The item may be removed after a refresh. Fall back in order:

1. Saved item id
2. Item at the same index (clamped to list length)
3. First item of the saved rail
4. First item of the first rail

Never leave the screen with nothing focused.

---

## Summary

- Rails and grids must remember the last focused item, per rail and per screen.
- Save by **item id**, not index — ids survive data changes.
- On return: scroll the item into view, then give it `hasTVPreferredFocus`.
- Always have a fallback, so focus is never lost.
