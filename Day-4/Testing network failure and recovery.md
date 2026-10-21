# Testing Network Failure and Recovery

TVs are often on weak Wi-Fi, and connections drop mid-use. Check that the app handles the network going away and coming back without freezing or leaving the user stuck.

## What to test

| Scenario | What should happen |
|---|---|
| Launch with no network | Clear offline message, not a blank or endless loading screen |
| Network drops while browsing | Screen stays usable; failed requests show an error, not a crash |
| Network drops during playback | Buffering or error state, no silent freeze |
| Network returns | App recovers automatically or offers Retry |
| Slow connection or failed request | Loading state, sensible timeout, useful error message |

## Focus during failures

- Focus lands on **Retry** when an error appears.
- Error dialogs can be dismissed with the D-pad and Back.
- After recovery, focus returns to a sensible element.

## How to simulate it

- Turn off Wi-Fi or unplug Ethernet on the TV.
- Use offline mode or throttling in the emulator or remote debugger.
- Toggle the network during launch, scrolling, and playback.

## Tips

- Use `navigator.onLine` and the `online` / `offline` events as hints, and confirm with a real request.
- Set request timeouts so the app never waits forever.
- Make Retry safe to press repeatedly.