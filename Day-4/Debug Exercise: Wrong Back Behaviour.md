# Debug Exercise: Wrong Back Behaviour

Back is the most used key after the D-pad. Typical bugs:

- Back does nothing
- Back exits the app instead of going to the previous screen
- Back goes to the wrong screen
- Back returns but focus is lost
- Back closes the app from a modal instead of closing the modal

## 1. Reproduce

Write down the exact path, for example "Home, Detail, Player, press Back". Note which screen Back is pressed on and what you expected.

## 2. Check the key is received

On web-based TV apps, Back has a different key code per platform:

| Platform | Back key code |
|---|---|
| Samsung Tizen | `10009` |
| LG webOS | `461` |

```js
document.addEventListener('keydown', e => console.log('key', e.keyCode));
```

If nothing is logged, the key is not reaching the app (on Tizen, check the key is registered). If a code is logged, the problem is in your handler.

## 3. Common causes

| Symptom | Likely cause |
|---|---|
| Back does nothing | Handler uses the wrong key code for this platform |
| Back exits the app | Default behavior not prevented, or no screen history |
| Wrong screen | Navigation stack or history is out of order |
| Modal ignores Back | Modal does not handle Back or capture focus |
| Focus lost after Back | Previous focus was not saved and restored |
| Back fires twice | Two listeners handle the same press |

## 4. Fix

- Handle the correct Back code for each platform.
- Keep a clear navigation stack and pop one screen per press.
- Let modals and overlays handle Back first, then close.
- Save the focused item before leaving a screen and restore it on return.
- On the home screen, show exit confirmation or exit per platform guidelines.
- Make sure only one handler responds to each press.

## 5. Verify

Test Back from every screen type: home, list, detail, player, modal, and error screen. Then check that focus is restored each time.