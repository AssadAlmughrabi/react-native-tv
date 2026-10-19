# Focus Problems and Recovery

Four focus bugs make a TV app feel broken. The user can't tap their way out, so every one of them is serious:

1. **Dead ends** — pressing a direction does nothing
2. **Traps** — focus gets in and can't get out
3. **Unexpected jumps** — focus lands somewhere surprising
4. **Lost focus** — nothing is focused at all

---

## 1. Dead ends

**What:** user presses Down (or Right…) and nothing happens, though there is content in that direction.

**Why:** gap in the layout, short row under a long one, or target is not focusable.

**Fix:**

- Fill gaps with `TVFocusGuideView` + `destinations`.
- Make sure everything interactive is focusable.
- At the true edge of the screen, doing nothing is fine — but only there.

---

## 2. Traps

**What:** focus enters an area (row, menu, player controls) and can't leave except with Back.

**Why:** wrong `nextFocus*` links, `trapFocus*` used outside a modal, or a guide that redirects back to itself.

**Fix:**

- Use `trapFocus*` **only** on modals and dialogs.
- Every area needs a way out in each direction that makes sense.
- Back must always work, even inside a trap-like area.

---

## 3. Unexpected jumps

**What:** Up from row 3 lands on the far left of row 2, or Left jumps into a different section.

**Why:** the engine picks the "nearest" element by position, not by meaning.

**Fix:**

- Wrap rows and menus in `TVFocusGuideView autoFocus` so focus returns to the last item.
- Keep layouts aligned (same card widths per row, clean columns).
- Fix single wrong links with `nextFocus*`.

---

## 4. Lost focus and recovery

**What:** no element is focused. Remote presses do nothing, or focus reappears in a random place.

**Why:** the focused element was **unmounted**:

- list refreshed and items re-rendered with new keys
- loading spinner replaced the content
- modal closed and the element that opened it is gone
- item deleted from a list

**Prevent:**

- Use stable `key`s (item id, never array index or random values).
- Keep content mounted while loading; show the spinner on top.
- When a modal closes, return focus to the element that opened it.

**Recover:** keep a fallback target and move focus to it when the focused item disappears.

```tsx
const [focusTarget, setFocusTarget] = useState<string | null>(null);

function removeItem(id: string) {
  const index = items.findIndex((i) => i.id === id);
  const next = items[index + 1] ?? items[index - 1]; // neighbour
  setItems(items.filter((i) => i.id !== id));
  setFocusTarget(next?.id ?? 'menu');               // fallback: menu
}

{items.map((item) => (
  <Card key={item.id} item={item} hasTVPreferredFocus={item.id === focusTarget} />
))}
```

- Fallback order: neighbour item → first item of the section → main menu.
- On the web build (LG, Samsung), the spatial navigation library usually has a `setFocus(key)` call for the same job.

---

## How to find them

- Walk every screen with the **real remote**: press each direction from each element.
- Test while data **loads, refreshes, and changes**, not only on a static screen.
- Log the focused element on device (`onFocus`) to see where focus really goes.

---

## Summary

- Dead ends: fill gaps with focus guides.
- Traps: `trapFocus*` only for modals, and Back always works.
- Unexpected jumps: `autoFocus` rows and aligned layouts.
- Lost focus: stable keys, keep content mounted, and always have a fallback target.
