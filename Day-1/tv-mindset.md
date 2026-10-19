# The TV Mindset

Building for TV is not "mobile on a big screen". Three ideas to carry into every screen:

1. **Living room** — who is watching, and how
2. **Ten-foot UI** — what can be seen from the couch
3. **Press cost** — every D-pad press is friction

---

## 1. Living room

- **Distance:** the user sits ~3 m (10 ft) away.
- **Input:** a remote with D-pad, OK, and Back. No touch, no mouse, hard typing.
- **Shared screen:** family watches together — keep flows short and simple.
- **Lean-back:** users want to *watch*, not *operate*. Get them to content fast.
- **Back is sacred:** always one step back, predictably.

---

## 2. Ten-foot UI

- **Text:** body ≥ 24px, titles 32–48px.
- **Contrast:** high, no thin fonts.
- **Safe area:** keep content ~5% inside screen edges.
- **Layout:** simple rows and grids, fewer and bigger items.
- **Focus is the cursor:** the focused element must always be clearly visible.

```tsx
<Pressable
  hasTVPreferredFocus // where focus lands first
  style={({ focused }) => [styles.button, focused && styles.focused]}
>
  <Text style={styles.label}>Play</Text>
</Pressable>
```

---

## 3. Press cost

Count every press from where focus lands to the goal (arrow = 1, OK = 1, Back = 1), then cut.

**Example: "Play next episode of my show"**

- **Before — 11 presses:** down to "My List" (3) → right to show (4) → OK (1) → pick episode (2) → OK play (1)
- **After — 1 press:** "Continue Watching" row on top, initial focus on latest show, OK plays next episode directly.

**Ways to cut presses:**

- Smart initial focus on the most likely action.
- Most-used items first.
- Restore last focus when coming Back.
- Avoid typing (voice, suggestions, QR login).
- Flat over deep — fewer nested screens.

---

## Summary

- TV users sit far away, use a simple remote, and want to relax.
- Design big, high-contrast, and always show clear focus.
- Measure flows in D-pad presses — fewer presses = better experience.
