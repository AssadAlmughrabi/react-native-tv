# Old Engines: No Flex Gap

`gap`, `rowGap`, and `columnGap` work in native React Native (0.71+), but on the **web build** `react-native-web` turns them into CSS `gap` on a flex container. Flexbox `gap` needs **Chromium 84+**. Older TVs silently ignore it — items touch each other, and layouts look broken.

---

## 1. Who is affected

| Platform | No flex `gap` on |
| --- | --- |
| LG webOS | webOS 6.x (2021, Chromium 79) and older |
| Samsung Tizen | Tizen 6.0 (2021, Chromium 76) and older |
| Android TV / Apple TV | Not affected (Yoga supports `gap`) |

See [old-browser-engines.md](../Day-1/old-browser-engines.md) for the full engine list.

- No error, no warning — the property is just ignored.
- Desktop Chrome supports it, so you won't see the bug until you test on an old TV.

---

## 2. Rule

**Don't use `gap` in shared code.** Use margins instead — they work everywhere.

---

## 3. Fallbacks

### Margin on each item (except the last)

```tsx
<View style={{ flexDirection: 'row' }}>
  {items.map((item, i) => (
    <Card
      key={item.id}
      item={item}
      style={i < items.length - 1 && { marginRight: scale(24) }}
    />
  ))}
</View>
```

### FlatList: `ItemSeparatorComponent`

```tsx
<FlatList
  horizontal
  data={items}
  renderItem={({ item }) => <Card item={item} />}
  ItemSeparatorComponent={() => <View style={{ width: scale(24) }} />}
/>
```

### A small `Stack` helper

One component, used everywhere instead of `gap`:

```tsx
// Stack.tsx
import { Children, Fragment } from 'react';
import { View, ViewStyle } from 'react-native';

type Props = { direction?: 'row' | 'column'; space: number; style?: ViewStyle; children: React.ReactNode };

export function Stack({ direction = 'column', space, style, children }: Props) {
  const items = Children.toArray(children);
  const spacer = direction === 'row' ? { width: space } : { height: space };

  return (
    <View style={[{ flexDirection: direction }, style]}>
      {items.map((child, i) => (
        <Fragment key={i}>
          {i > 0 && <View style={spacer} />}
          {child}
        </Fragment>
      ))}
    </View>
  );
}
```

```tsx
<Stack direction="row" space={scale(24)}>
  <Button title="Play" />
  <Button title="Trailer" />
  <Button title="My List" />
</Stack>
```

### Wrapping grids

For `flexWrap: 'wrap'`, use margin on every item and a **negative margin** on the container to cancel the outer edges:

```tsx
<View style={{ flexDirection: 'row', flexWrap: 'wrap', margin: -scale(12) }}>
  {items.map((item) => (
    <Card key={item.id} item={item} style={{ margin: scale(12) }} />
  ))}
</View>
```

---

## 4. Catch it early

- Add a lint rule or code review check: no `gap` / `rowGap` / `columnGap` in styles.
- Test layouts on the oldest LG and Samsung TV you support.

---

## Summary

- Flex `gap` needs Chromium 84; LG webOS ≤ 6 and Samsung Tizen ≤ 6.0 ignore it silently.
- Native Android TV and Apple TV are fine — the problem is the web build.
- Use margins, `ItemSeparatorComponent`, or a shared `Stack` helper instead of `gap`.
- Block `gap` with a lint rule and test on the oldest TVs.
