# Right-to-Left Layouts and Focus Direction

In RTL languages (Arabic), the layout is mirrored: rails start on the **right**, and "next" is to the **left**. The D-pad keys stay physical — Left is still left — so anything that means "next / previous" must flip.

---

## 1. What flips, what doesn't

| Flips in RTL | Stays the same |
| --- | --- |
| Row order (`flexDirection: 'row'` starts on the right) | Up / Down |
| Rails: first item on the right, scroll toward the left | Video playback controls and progress bar (stay LTR) |
| "Next" = **Left**, "Previous" = **Right** | Numbers, clocks, phone numbers |
| Side menu on the right | Logos, media icons (play, pause) |
| Arrow/chevron icons, slide animations | |

---

## 2. Native (Android TV, Apple TV)

```ts
import { I18nManager } from 'react-native';

I18nManager.allowRTL(true);
I18nManager.forceRTL(isArabic); // needs an app restart to apply
const isRTL = I18nManager.isRTL;
```

- Layout flips automatically: `row` starts on the right.
- The **native focus engine is spatial**: pressing Left goes to the element on the left of the screen. So plain focus movement just works.
- Use `start` / `end` styles, not `left` / `right`: `marginStart`, `paddingEnd`, `textAlign: 'auto'`.

---

## 3. Web (LG, Samsung)

- Set direction on the document: `<html dir="rtl" lang="ar">`.
- Norigin finds the next element by **screen position**, so arrows follow the mirrored layout too.
- Check scroll: horizontal scroll position in RTL behaves differently between browser engines — test rail scrolling on old TVs.

---

## 4. What breaks: your own direction logic

Anything you wrote that assumes **Right = next** breaks in RTL:

- `nextFocusRight` / `nextFocusLeft` links
- Norigin `onArrowPress` handlers
- Custom carousels (`index + 1` on Right)
- Paging buttons and "scroll to next page" on rails
- `translateX` slide animations (sign must flip)

Fix: map physical keys to **logical** directions in one place.

```ts
type Logical = 'next' | 'prev';

export function toLogical(key: 'left' | 'right', isRTL: boolean): Logical {
  if (key === 'right') return isRTL ? 'prev' : 'next';
  return isRTL ? 'next' : 'prev';
}

// carousel
onArrowPress: (dir) => {
  if (dir === 'left' || dir === 'right') {
    const move = toLogical(dir, isRTL);
    setIndex((i) => (move === 'next' ? i + 1 : i - 1));
    return false; // we handled it
  }
  return true;
},
```

- **Exception — player:** seek stays physical: Right = forward, Left = back, same as the LTR progress bar.

---

## 5. How to test RTL

- Initial focus lands on the **rightmost** first item.
- Left moves forward through rails; Right moves back.
- Side menu opens from the right with Right key.
- Restore focus by id still works after mirroring.
- Player: Right = fast-forward, progress bar left-to-right.

---

## Summary

- RTL mirrors the layout; D-pad keys stay physical.
- Spatial focus (native and Norigin) follows the mirrored layout automatically.
- Your own "Right = next" logic must flip — map keys to logical next/prev in one helper.
- Video controls and seeking stay LTR.
- Use `start` / `end` styles and test every screen in Arabic.
