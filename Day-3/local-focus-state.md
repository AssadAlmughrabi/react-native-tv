# Keep Focus State Local

Focus changes on **every D-pad press** — many times per second when a key is held. If the focused item lives in a **top-level context or store**, every press re-renders the whole screen. On a weak TV that means focus lag.

Rule: **only the item that gains focus and the item that loses focus should re-render.**

---

## 1. The problem

```tsx
// ❌ focused id in top-level context
const FocusContext = createContext({ focusedId: null, setFocusedId: (id: string) => {} });

function Card({ item }) {
  const { focusedId, setFocusedId } = useContext(FocusContext);
  const focused = focusedId === item.id;
  return (
    <Pressable onFocus={() => setFocusedId(item.id)} style={[styles.card, focused && styles.focused]}>
      …
    </Pressable>
  );
}
```

- Every press → context value changes → **every card on screen** re-renders (hundreds on a Home screen).
- Hold an arrow key → dozens of full re-renders per second.

---

## 2. Keep it local

The focus system already knows which element is focused. Read it **where it is used**.

```tsx
// ✅ native: Pressable gives `focused` locally
const Card = memo(function Card({ item, onPress }) {
  return (
    <Pressable onPress={onPress} style={({ focused }) => [styles.card, focused && styles.focused]}>
      …
    </Pressable>
  );
});
```

```tsx
// ✅ web: Norigin gives `focused` locally
const Card = memo(function Card({ item, onPress }) {
  const { ref, focused } = useFocusable({ focusKey: `card-${item.id}`, onEnterPress: onPress });
  return <View ref={ref} style={[styles.card, focused && styles.focused]}>…</View>;
});
```

- Wrap cards in `memo`, and pass **stable** props (`useCallback`, no inline objects).
- Now one press = two re-renders (old card, new card).

---

## 3. When other parts need the focused item

Some UI really depends on focus: the **hero/background preview**, a **details panel**, or **remember last focus**.

### Remember focus: write without re-rendering

Save the id to a ref or module store — nothing subscribes, nothing re-renders.

```tsx
<Card onFocus={() => rememberFocus('home', rail.id, item.id)} />
```

### Hero preview: narrow subscription + debounce

Only the hero subscribes, and only after focus settles.

```ts
// previewStore.ts (zustand)
export const usePreview = create<{ item: Item | null; set: (i: Item) => void }>((set) => ({
  item: null,
  set: (item) => set({ item }),
}));
```

```tsx
// Card
const setPreview = usePreview((s) => s.set);        // selector: never re-renders on change
const preview = useDebounced(setPreview, 300);
<Pressable onFocus={() => preview(item)} />

// Hero — the only subscriber
const item = usePreview((s) => s.item);
```

- Selectors keep subscribers narrow: cards read the setter only, the hero reads the item.
- Debounce so fast scrolling doesn't redraw the hero for every card passed.

---

## 4. How to check

- React DevTools → Profiler → **"Highlight updates when components render"**, then press arrows: only two cards should flash.
- Hold an arrow on a long rail on the **weakest TV**: focus must keep up with the key.

---

## Summary

- Focus changes on every press — don't put it in top-level context or a broad store.
- Read `focused` locally (`Pressable` / `useFocusable`) and `memo` the cards.
- Save last focus to a ref/module store, without re-renders.
- Things that need the focused item (hero) subscribe narrowly and debounce.
