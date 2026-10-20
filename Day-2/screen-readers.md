# Screen Readers on TV and Focus Order

On TV, screen readers **follow D-pad focus**: when focus moves, the reader speaks the focused element. So good focus order and good labels are the whole job.

| Platform | Screen reader |
| --- | --- |
| Android TV | TalkBack |
| Apple TV | VoiceOver |
| LG webOS | Audio Guidance |
| Samsung Tizen | Voice Guide |

---

## 1. Native: TalkBack and VoiceOver

Both read the focused element's **label**, **role**, and **state**.

```tsx
<Pressable
  accessibilityRole="button"
  accessibilityLabel={`${item.title}, ${item.year}, ${item.duration}`}
  accessibilityHint="Opens details"
  accessibilityState={{ selected: isSelected }}
  onPress={open}
>
  <Image source={{ uri: item.image }} accessible={false} />
  <Text>{item.title}</Text>
</Pressable>
```

- **One focusable = one sentence.** Combine title, year, duration into one label on the card.
- Don't make inner `Text` / `Image` separately focusable or readable.
- Decorative images: `accessible={false}` (iOS/tvOS) and `importantForAccessibility="no"` (Android).
- Announce screen changes:

```ts
import { AccessibilityInfo } from 'react-native';
AccessibilityInfo.announceForAccessibility('Movie details: The Crown');
```

---

## 2. Web: LG and Samsung

`react-native-web` maps the same props to HTML: `accessibilityLabel` → `aria-label`, `accessibilityRole` → `role`.

**The catch:** a spatial navigation library (Norigin) keeps focus in its **own state**, not real DOM focus. The TV screen reader only follows real DOM focus, so it says **nothing** when the user moves.

Fixes:

- **Move real DOM focus** with the library focus:

```tsx
const { ref, focused } = useFocusable({ focusKey: `card-${item.id}` });

useEffect(() => {
  if (focused) ref.current?.focus();
}, [focused]);

<View ref={ref} focusable accessibilityLabel={label} /* tabIndex 0 on web */ />
```

- Or use an **`aria-live` region** that receives the focused item's label.
- Each brand also has its own TTS/accessibility APIs — check LG and Samsung docs for your target versions.

---

## 3. Focus order

The screen reader reads in **focus order**, so focus order must make sense when heard, not only seen.

- **Logical order:** top to bottom, left to right — same as reading order.
- **Context when entering a group:** label rails so the reader says the row name ("Continue Watching") when focus enters it.
- **Position info** helps: "3 of 12".
- **No hidden focus targets:** off-screen or invisible elements must not take focus.
- **Initial focus** (`hasTVPreferredFocus`) on screen open is the first thing spoken — make it meaningful.
- **Modals:** trap focus inside, so the reader doesn't wander behind them.

---

## 4. Adapt when a screen reader is on

```ts
const [readerOn, setReaderOn] = useState(false);

useEffect(() => {
  AccessibilityInfo.isScreenReaderEnabled().then(setReaderOn);
  const sub = AccessibilityInfo.addEventListener('screenReaderChanged', setReaderOn);
  return () => sub.remove();
}, []);
```

- Don't auto-play trailers on focus — they talk over the reader.
- Don't auto-hide player controls or toasts too fast.
- Avoid moving focus without user action.

---

## Summary

- TV screen readers speak whatever has D-pad focus.
- One focusable = one clear label (title, year, duration); hide decorative parts.
- Web build: sync real DOM focus with Norigin, or the screen reader stays silent.
- Focus order = reading order: logical, grouped with row labels, no hidden targets.
- When a reader is on: no auto-play, slower auto-hide, no surprise focus jumps.
