# How Focus Works on Web

On Android TV and Apple TV, the **OS** moves focus. On LG and Samsung (web build via `react-native-web`), there is **no native focus engine**: the browser only knows Tab order, not D-pad. A **spatial navigation library** does the job instead. In `tv-app` that library is **Norigin** (`@noriginmedia/norigin-spatial-navigation`).

---

## 1. How Norigin works

- Every focusable element registers itself with a **focus key**.
- The library listens to arrow keys and picks the next element by **position on screen**.
- Parents group children (rails, menus) and can remember the last focused child.
- Focus is **library state**, not real DOM focus — style from the `focused` value it returns.

Setup once at app start:

```ts
import { init } from '@noriginmedia/norigin-spatial-navigation';

init({
  debug: false,
  visualDebug: false, // true draws focus boxes — handy on device
});
```

---

## 2. A focusable element

```tsx
// FocusableCard.web.tsx
import { useFocusable } from '@noriginmedia/norigin-spatial-navigation';
import { View, Text } from 'react-native';

export function FocusableCard({ item, onPress, onFocus }) {
  const { ref, focused } = useFocusable({
    focusKey: `card-${item.id}`,
    onEnterPress: onPress,
    onFocus,
  });

  return (
    <View ref={ref} style={[styles.card, focused && styles.cardFocused]}>
      <Text style={styles.title}>{item.title}</Text>
    </View>
  );
}
```

A rail that groups cards and remembers the last one:

```tsx
import { useFocusable, FocusContext } from '@noriginmedia/norigin-spatial-navigation';

export function Rail({ rail }) {
  const { ref, focusKey } = useFocusable({
    focusKey: `rail-${rail.id}`,
    saveLastFocusedChild: true,
    trackChildren: true,
  });

  return (
    <FocusContext.Provider value={focusKey}>
      <View ref={ref} style={styles.row}>
        {rail.items.map((item) => <FocusableCard key={item.id} item={item} />)}
      </View>
    </FocusContext.Provider>
  );
}
```

---

## 3. RN focus APIs mapped to Norigin

| Native (`react-native-tvos`) | Web (Norigin) |
| --- | --- |
| Focusable `Pressable` | `useFocusable()` + `ref` on the element |
| `style={({ focused }) => …}` | `focused` returned by `useFocusable` |
| `onFocus` / `onBlur` | `onFocus` / `onBlur` options |
| `onPress` (OK) | `onEnterPress` |
| `hasTVPreferredFocus` | `focusSelf()` in `useEffect`, or `setFocus(key)` |
| `TVFocusGuideView autoFocus` | parent with `saveLastFocusedChild` + `trackChildren` |
| `TVFocusGuideView destinations` | `preferredChildFocusKey`, or `setFocus(key)` |
| `trapFocus*` | `isFocusBoundary` + `focusBoundaryDirections` |
| `nextFocus*` | `onArrowPress` → `setFocus(target)` and return `false` |
| `useTVEventHandler` | `keydown` listener (media keys, color keys) |
| `BackHandler` | `keydown` for `461` (LG) / `10009` (Samsung) → same `handleBack()` |

Initial focus example:

```tsx
const { ref, focusSelf } = useFocusable({ focusKey: 'play' });
useEffect(() => { focusSelf(); }, [focusSelf]);
```

Restore focus by id (rails, modals) is the same idea as native — save the id, then:

```ts
import { setFocus } from '@noriginmedia/norigin-spatial-navigation';
setFocus(`card-${savedId}`);
```

---

## 4. Sharing code

Same component name, two files — the bundler picks one:

```text
FocusableCard.native.tsx   // Pressable + hasTVPreferredFocus (Metro)
FocusableCard.web.tsx      // useFocusable (webpack)
```

Screens use `<FocusableCard />` and never know which focus system runs underneath.

---

## Summary

- Web TVs have no native D-pad focus; Norigin provides it in `tv-app`.
- Every focusable registers with a focus key; parents group and remember children.
- Each native focus API has a Norigin equivalent — keep the mapping table at hand.
- Hide the difference behind `.native` / `.web` components so screens stay shared.
