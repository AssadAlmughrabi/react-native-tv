# Debug Exercise: Remote Input Not Detected (TV)

Following the workflow: reproduce, classify, collect, compare, one change.

## 1. Reproduce

Write down:

- Steps ("Home, press right, nothing moves")
- TV model, OS version, release or debug build
- Pattern: all keys or some? One screen or all? Real remote or emulator?

## 2. Classify

An **input** problem. Two questions:

- Does the key event reach the app?
- If it does, does the code handle it?

## 3. Collect

Log raw key events before any filtering.

```js
// React Native TV
useTVEventHandler(evt => console.log(evt.eventType, evt.eventKeyAction));

// Web (LG, Samsung)
document.addEventListener('keydown', e => console.log(e.key, e.keyCode));
```

Also note which element has focus, and whether a modal or `TextInput` is open.

## 4. Compare

| What you see | Likely cause |
|---|---|
| No events at all | Nothing focused, overlay or `TextInput` capturing keys, handler not registered |
| Events arrive, nothing happens | Wrong key name or code, wrong `eventType`, stale closure, another handler consuming it |
| Some keys work, others don't | Platform key differences |
| Works on emulator, not on TV | Emulators send different keys |

Platform differences:

- **Android TV:** center button may be `select` or Enter. Back goes through `BackHandler`.
- **tvOS:** `select`, `menu`, `playPause`.
- **webOS / Tizen** (`web`): different key codes (Back is 461 on webOS, 10009 on Tizen). Tizen needs some keys registered. Magic Remote pointer mode sends mouse events. `verify`

## 5. One change

Fix the most likely cause only, retest on the real remote. If it doesn't help, revert and try the next one.

## Example

*OK does nothing on the Samsung TV, but works in the emulator.*

1. **Reproduce:** focus a card, press OK, no response, only on the TV.
2. **Classify:** input.
3. **Collect:** `keydown` log shows `keyCode 13`.
4. **Compare:** the emulator sent `Enter`, my handler checks `e.key === 'Enter'`, the TV reports something else.
5. **One change:** also check `keyCode === 13`.
