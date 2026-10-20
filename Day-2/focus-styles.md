# Focus Styles

The focus style is the user's cursor. It must be **obvious from the couch**, **fast** on weak TVs, and **never cut off**.

---

## 1. Border, scale, shadow

Combine two or more — one signal alone is easy to miss.

| Style | Good for | Watch out |
| --- | --- | --- |
| **Border / outline** | Clear edge on any background | Keep the border always there (transparent when not focused) so layout doesn't jump |
| **Scale** (1.05–1.1) | Strong "lifted" feeling on cards | Needs room around the card; animate on native thread |
| **Shadow / glow** | Depth on dark backgrounds | Expensive on weak TVs; keep it small or skip on low-end |
| Brightness / color | Buttons, menu items | Never color alone — color-blind users and TV color modes |

```tsx
<Pressable
  style={({ focused }) => [
    styles.card,
    focused && styles.cardFocused,
  ]}
>
  <View style={styles.imageWrap}>
    <Image source={{ uri: item.image }} style={styles.image} />
  </View>
</Pressable>

const styles = StyleSheet.create({
  card: {
    borderWidth: scale(4),
    borderColor: 'transparent',  // always present, no layout jump
    borderRadius: scale(12),
  },
  cardFocused: {
    borderColor: '#FFFFFF',
    transform: [{ scale: 1.08 }],
    zIndex: 1,                    // above neighbours
    elevation: 8,                 // Android shadow + draw order
  },
  imageWrap: { borderRadius: scale(8), overflow: 'hidden' }, // clip the image, not the card
  image: { width: '100%', aspectRatio: 16 / 9 },
});
```

- Animate scale with `Animated` + `useNativeDriver: true` (~150–200 ms) for smooth growth.
- One focus style for the whole app — users learn it once.

---

## 2. Focused vs selected

Two different states, two different looks:

| | Focused | Selected |
| --- | --- | --- |
| Meaning | Where the cursor **is now** | What is **chosen / active** |
| How many | Exactly **one** on screen | Can be one or many |
| Example | Card the D-pad is on | Active tab, current season, checked option |
| Style | Strong: border + scale | Quiet: underline, filled background, check icon |

- A selected item can **also** be focused — that combination must look different from both.
- Never use the same style for both, or users can't tell where they are.

```tsx
style={({ focused }) => [
  styles.tab,
  selected && styles.tabSelected, // underline
  focused && styles.tabFocused,   // border + scale
]}
```

---

## 3. Don't clip the scale with overflow

A scaled card grows **outside** its box. If a parent has `overflow: 'hidden'`, the edges get cut.

Common causes:

- `overflow: 'hidden'` on the **card itself** (used for rounded image corners).
- A rail container with `overflow: 'hidden'` or no space above/below.
- Horizontal `FlatList` / `ScrollView` clipping content at its own bounds.
- Neighbour cards drawn on top of the focused one.

Fixes:

- Put `overflow: 'hidden'` on an **inner** wrapper around the image, not on the focusable card.
- Give the rail **padding** at least as big as the growth (card 180 px × 8% ≈ 15 px total, ~8 px each side; add room for border and shadow, so ~20 px).
- Raise the focused card with `zIndex` (and `elevation` on Android).

```tsx
<FlatList
  horizontal
  data={rail.items}
  contentContainerStyle={{ paddingVertical: scale(20), paddingHorizontal: SAFE.horizontal }}
  renderItem={({ item }) => <Card item={item} />}
/>
```

- Check the **first and last** card of each rail, and the top and bottom rows — they clip first.

---

## Summary

- Combine border + scale (+ light shadow); keep the border always present to avoid jumps.
- Focused = where the cursor is (one); selected = what is chosen (many). Style them differently.
- Never put `overflow: 'hidden'` on the scaled element; clip an inner wrapper instead.
- Give rails padding for the scale, and raise the focused card with `zIndex`/`elevation`.
