# Text Entry on TV

### 1. The problem

Typing with a D-pad is slow: move focus, press OK, repeat for every letter.

### 2. Avoid typing where you can

Use pickers, recent searches, and suggestions. Ask for text only when needed (search, login, profile name).

### 3. On-screen keyboard (baseline)

- **System keyboard:** `TextInput` opens the platform keyboard. Easy and familiar.
- **Custom keyboard:** a grid of focusable keys. More control, more work.
- Show live results while typing so users can stop early.

### 4. QR code / phone input

Show a QR code. The user types on their phone and the text appears on the TV. Best for logins and long passwords. Offer a short code as a fallback.

### 5. Voice (advanced)

Fastest for search, but depends on platform and remote support `verify`. Treat it as an extra, not a replacement.

### 6. Always keep a keyboard path

QR and voice can fail, and focus must enter and leave the input cleanly.

| Situation | Best option |
|---|---|
| Search | Keyboard with suggestions, plus voice |
| Login / password | QR or phone input |
| Short input (name, PIN) | On-screen keyboard |