# Debug Exercise: Focus Disappears or Jumps

- **Disappears:** nothing is highlighted and D-pad presses do nothing.
- **Jumps:** focus moves to an unexpected element, often the first one on the page.

## 1. Reproduce

Write down the exact key sequence, for example "open detail, press Back, focus is gone". Note whether it happens after a data load, after closing a modal, after scrolling, or only on a low-end device.

## 2. Watch where focus is

```js
document.addEventListener('focusin', e => console.log('focus ->', e.target));
```

If `document.activeElement` is `<body>`, focus was lost. If it is a different element than expected, focus jumped.

## 3. Common causes

| Symptom | Likely cause |
|---|---|
| Disappears | Element removed by a re-render, hidden, or focus set before it exists |
| Disappears | Modal or screen closed and nothing restored focus |
| Jumps | Re-render replaced elements and a default got focus |
| Jumps | Two key listeners handling the same press |
| Jumps | Unstable list keys or IDs, or autofocus code running again |

## 4. Fix

- Use stable keys so items are not recreated on every update.
- Save the focused item before navigating away and restore it on Back.
- Return focus to the trigger when a modal closes.
- Set focus only after the DOM is ready.
- If focus lands on `<body>`, move it to a sensible default.

## 5. Verify

Re-run the original key sequence, then the smoke tests (Back restores focus, rapid presses, low-end device).