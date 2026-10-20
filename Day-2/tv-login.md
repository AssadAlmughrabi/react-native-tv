# Login on TV: QR Code or Code on a Phone

Typing an email and password with a remote takes **hundreds of presses**. On TV, the user signs in on their **phone** instead: scan a QR code, or open a short URL and type a short code. The TV just waits.

This is the **OAuth 2.0 Device Authorization Grant** (RFC 8628).

---

## 1. The flow

```text
TV                          Server                       Phone
│ 1. ask for a code ───────▶│
│◀── device_code, user_code,│
│    verification_uri, QR   │
│ 2. show QR + code         │                            │
│                           │◀── 3. open URL, log in, ───│
│                           │       enter/confirm code   │
│ 4. poll every N s ───────▶│                            │
│◀── tokens (when approved) │                            │
│ 5. signed in              │                            │
```

- **`user_code`**: short code shown on TV (example: `K7MP-4QXT`).
- **`verification_uri`**: short URL (example: `example.com/tv`).
- **`verification_uri_complete`**: same URL with the code inside — put this in the QR.
- **`device_code`**: secret, used only by the TV for polling. Never shown.

---

## 2. The screen

```text
┌───────────────────────────────────────────────┐
│  Sign in                                      │
│                                               │
│  ┌────────┐   1. Scan with your phone         │
│  │  QR    │   2. Or go to  example.com/tv     │
│  │        │      and enter:  K7MP-4QXT        │
│  └────────┘                                   │
│               Code expires in 9:41            │
│                                               │
│   [ Get a new code ]   [ Other sign-in ]      │
└───────────────────────────────────────────────┘
```

- QR big enough to scan from the couch (~300 px on the 1920 canvas).
- Code in **large** text, grouped (`K7MP-4QXT`), no confusing characters (`0/O`, `1/I/L`).
- Short URL, readable from 3 m.
- **Focusable buttons:** "Get a new code" and "Other sign-in" (email/password as last resort). One gets `hasTVPreferredFocus`.
- Show the expiry; get a new code automatically when it expires.

```tsx
import QRCode from 'react-native-qrcode-svg';

<QRCode value={verificationUriComplete} size={scale(300)} />
<Text style={styles.code}>{userCode}</Text>
```

---

## 3. Polling

```ts
async function poll(deviceCode: string, interval: number, signal: AbortSignal) {
  while (!signal.aborted) {
    await wait(interval * 1000);
    const res = await requestToken(deviceCode); // grant_type=urn:ietf:params:oauth:grant-type:device_code

    if (res.access_token) return res;            // approved
    if (res.error === 'authorization_pending') continue;
    if (res.error === 'slow_down') { interval += 5; continue; }
    throw res.error;                              // expired_token, access_denied
  }
}
```

- Respect the server's `interval`; add 5 s on `slow_down`.
- **Stop polling** when the user leaves the screen (abort on unmount).
- `expired_token` → show a new code. `access_denied` → show a message + "Try again".
- After success: store tokens in secure storage (Keychain / Keystore on native, platform storage on web TVs), then go to Home.

---

## 4. Security notes

- The QR contains only the **URL + user code**, never tokens or the `device_code`.
- Codes are short-lived (~10–15 min) and single-use.
- The phone page shows **what** is being signed in ("Living room TV") so the user can confirm.

---

## Summary

- No typing on TV: sign in on the phone with a QR code or a short URL + code.
- Use the OAuth Device Authorization Grant: show `user_code` + QR, poll for tokens.
- Big QR, big grouped code, short URL, focusable "new code" and "other sign-in" buttons.
- Poll at the server's interval, stop on leave, handle expiry, store tokens securely.
