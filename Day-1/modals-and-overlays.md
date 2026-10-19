# Modals and Overlays

A modal on TV must do three things:

1. **Take focus** when it opens
2. **Trap focus** inside while it is open
3. **Restore focus** to the element that opened it when it closes

If any of these fail, focus is lost behind the overlay, or the user lands somewhere random.

---

## 1. Take focus on open

Give one button in the modal `hasTVPreferredFocus`.

- Normal dialog: focus the main action (OK, Continue).
- Destructive dialog (Delete, Sign out): focus the **safe** action (Cancel).

```tsx
<Pressable hasTVPreferredFocus onPress={onCancel}>
  <Text>Cancel</Text>
</Pressable>
```

---

## 2. Trap focus inside

- **React Native `Modal`:** opens its own window, so focus usually stays inside. Still test it on each platform.
- **Overlays** (absolute `View` on top of the screen): focus can escape to the content behind. Trap it:

```tsx
import { TVFocusGuideView } from 'react-native';

<TVFocusGuideView
  style={styles.overlay}
  trapFocusUp
  trapFocusDown
  trapFocusLeft
  trapFocusRight
>
  <ConfirmButtons />
</TVFocusGuideView>
```

- Back must **close the modal**, not the screen behind it:

```tsx
<Modal visible={open} onRequestClose={close}>{/* Android Back */}</Modal>
```

For overlays, use `BackHandler` and return `true` while open.

---

## 3. Restore focus on close

Save the **id** of the element that opened the modal, then give it focus again on close.

```tsx
const [modalOpen, setModalOpen] = useState(false);
const [openerId, setOpenerId] = useState<string | null>(null);

function openModal(id: string) {
  setOpenerId(id);
  setModalOpen(true);
}

function closeModal() {
  setModalOpen(false); // openerId stays, so the opener gets focus back
}

{items.map((item) => (
  <Card
    key={item.id}
    item={item}
    onPress={() => openModal(item.id)}
    hasTVPreferredFocus={!modalOpen && item.id === openerId}
  />
))}
```

- If the opener is gone (example: item was deleted), fall back to its neighbour, then to the first item.
- Don't unmount the screen behind the modal, or the opener can't get focus back.

---

## Summary

- Open: focus one button — the safe one for destructive actions.
- While open: trap focus with `TVFocusGuideView trapFocus*`; Back closes the modal.
- Close: restore focus to the opener by id, with a fallback if it's gone.
- Keep the screen behind mounted so focus has a place to return.
