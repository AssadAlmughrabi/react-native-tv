# Testing the Web Build

Test the LG/Samsung web build in two steps:

1. **Chrome on your desktop** — fast loop, keyboard as remote
2. **Real TV** — the truth: old engine, real remote, real speed

---

## 1. In Chrome: keyboard as remote

| Remote | Keyboard | `keyCode` |
| --- | --- | --- |
| Arrows | Arrow keys | 37–40 |
| OK | Enter | 13 |
| Back | Backspace | 8 |

Add Backspace to the key map **for dev only**, next to the real TV codes:

```ts
const BACK_KEYS = [461, 10009]; // LG, Samsung
if (__DEV__) BACK_KEYS.push(8); // Backspace in Chrome
```

- Don't map Backspace in production: it breaks text inputs (search).
- If Backspace is pressed inside a text input, let it delete text — don't treat it as Back.

How to test:

- Set the window to **1920×1080** (DevTools device toolbar → responsive, 1920×1080).
- **Don't touch the mouse.** Use only arrows, Enter, Backspace.
- Walk every screen: every direction from every element, open and close modals, go back from deep screens.
- Check: focus always visible, no dead ends, no traps, Back always goes one step back.
- Turn on Norigin `visualDebug: true` to see focus boxes.

What Chrome **can't** tell you: old engine bugs, real speed, real remote keys. Desktop Chrome is much newer than the TV engine.

---

## 2. On the real TV

### LG (webOS)

Enable **Developer Mode** on the TV (Developer Mode app), then with the webOS CLI:

```sh
ares-setup-device                      # add the TV (IP + passphrase)
ares-package ./dist                    # creates .ipk
ares-install --device tv app.ipk
ares-launch --device tv com.example.app
ares-inspect --device tv --app com.example.app --open   # DevTools
```

### Samsung (Tizen)

Enable **Developer Mode** on the TV (Apps → type `12345` → set your PC IP), then with Tizen CLI:

```sh
sdb connect <TV_IP>
tizen package -t wgt -s <cert-profile> -- ./dist   # creates .wgt
tizen install -n app.wgt -t <device-name>
tizen run -p <package-id> -t <device-name>
```

Debug with Chrome remote inspect from Tizen Studio (web inspector).

### What to check on the TV

- App loads at all (no blank screen from unsupported syntax).
- Real remote: arrows, OK, Back, media keys, key repeat when held.
- LG: Magic Remote pointer and hover.
- Speed: startup time, focus lag on long rails, video start.
- Text readable from the couch, content inside the safe area.

---

## Summary

- Chrome first: arrows, Enter, Backspace as the remote, 1920×1080, no mouse.
- Map Backspace as Back in dev only, and never inside text inputs.
- Then install on a real LG and Samsung TV with their CLIs, and debug with remote DevTools.
- The TV is the final test: old engine, real remote, real speed.
