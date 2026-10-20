# Navigation on TV

Main navigation on TV is a **visible menu you reach with the D-pad**. Two patterns work well: a **sidebar** or **top tabs**. Mobile swipe patterns don't work at all.

---

## 1. Sidebar (left menu)

Used by Android TV / Google TV apps and many streaming apps.

- **Collapsed:** a thin column of icons, always visible.
- **Expanded:** opens with labels when focus moves onto it (Left from the content edge).
- **Leaving:** Right goes back into the content, to the last focused item.
- **Selected** item (current section) looks different from the **focused** item.
- Good for **many sections** (5–8): Home, Search, Movies, Series, Live, Settings.

```tsx
<View style={{ flex: 1, flexDirection: 'row' }}>
  <TVFocusGuideView autoFocus>
    <Sidebar expanded={menuFocused} />   {/* icons, labels when expanded */}
  </TVFocusGuideView>
  <TVFocusGuideView autoFocus style={{ flex: 1 }}>
    <CurrentScreen />
  </TVFocusGuideView>
</View>
```

- RTL: the sidebar moves to the right, opened with Right.

---

## 2. Top tabs

Used by Apple TV apps and many web TV apps.

- A row of tabs at the top: Home, Movies, Series, Search.
- **Up** from the first row of content reaches the tabs.
- Moving between tabs switches the section (on focus, or on OK — pick one, be consistent).
- Tabs can **hide** when the user scrolls down and come back when they go up.
- Good for **few sections** (3–6) with short names.

---

## 3. Swipe gestures don't work

| Mobile pattern | Why it fails on TV | Use instead |
| --- | --- | --- |
| Swipe-to-open drawer | No swipe on a remote; hidden menu can't be found | Visible sidebar |
| Swipeable tab pager | Left/Right is needed for moving inside content | Top tabs, change on focus/OK |
| Bottom tab bar | Far from content; a mobile habit; overlaps video | Sidebar or top tabs |
| Pull-to-refresh, swipe-to-delete | No gestures | Auto refresh, visible buttons |

With React Navigation: disable swipe on drawers and pagers (`swipeEnabled: false`), and render your own focusable tab bar / sidebar instead of the default touch UI.

---

## 4. Back with main navigation

A predictable order:

1. Inside content → Back goes to the **top of the screen** / first rail.
2. At the top → Back moves focus to the **menu** (sidebar or tabs).
3. On the menu, not on Home → Back goes to **Home**.
4. On Home in the menu → Back **exits** (or asks to confirm).

---

## Summary

- TV main navigation = a visible, focusable menu: sidebar or top tabs.
- Sidebar for many sections, expands on focus; top tabs for few sections, reached with Up.
- Swipe drawers, swipe pagers, and bottom tabs don't work on a remote.
- Back follows a fixed path: content → top → menu → Home → exit.
