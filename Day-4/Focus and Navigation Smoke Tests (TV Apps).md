# Focus and Navigation Smoke Tests (TV Apps)

TVs have no touch or mouse, only a remote (D-pad, OK, Back). If focus is lost, trapped, or invisible, the app is unusable. A smoke test is a quick pass to confirm the app is basically usable with a remote.

## Key sequences

Run the same presses every time so results are repeatable.

| Sequence | Verifies |
|---|---|
| Launch, no input | Initial focus is set |
| Right x N, then Left x N | Horizontal path is symmetric |
| Down x N, then Up x N | Vertical path works, scroll follows focus |
| Right at last item | Edge behavior (stay, wrap, next row) is as designed |
| OK on a focused item | Activation works |
| Back from detail screen | Previous screen returns with focus restored |
| Back on home | Exit confirmation or exit per platform rules |
| Mash Down quickly | No lag or dropped focus |
| Open modal, then Back | Modal closes, focus returns to trigger |

**Back key differs by platform:** Tizen = `10009`, webOS = `461`. Arrows are 37-40 and Enter is 13 on both.

## Manual focus walk

Step through each screen with the D-pad and watch where focus goes.

1. Note the first focused element.
2. Go Right to the end of the row, then Left back. Repeat with Down/Up.
3. Visit every interactive element at least once.
4. Try diagonal cases (Down from a far-right item into a shorter row).
5. Repeat after opening menus, dialogs, and error states.

**Look for:**

- Invisible focus (no visible style from 3 meters)
- Focus traps (carousels, modals, custom keyboards)
- Unreachable elements
- Lost focus after re-render or closing a dialog
- Illogical or asymmetric paths (A to B, but Left from B does not return to A)
- Focus on an off-screen element

## Tips

- Use a high-contrast focus style (outline or scale).
- Check `document.activeElement` to see where focus really is.
- Test on a real device when possible.
- On LG, test both the standard remote and Magic Remote pointer mode.
