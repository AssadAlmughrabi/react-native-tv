# Troubleshooting Workflow (TV)

**Reproduce, classify, collect, compare, one change.** A repeatable routine that stops guessing. It matters on TV because debugging is slow: you deploy to a device and use a remote.

## 1. Reproduce

Make the bug happen on purpose and write down:

- The exact steps ("Home, right 5 times, OK, Back")
- Device, OS version, build type (debug or release)
- How often it happens

If you can't reproduce it, you can't tell if a fix worked.

## 2. Classify

Decide the type of problem before touching code:

- Input, focus, Back behaviour, layout, performance, or old engine (`web`)

Each type has its own first checks, so you don't search in the wrong place.

## 3. Collect

Gather evidence instead of guessing:

- Logs (`adb logcat`, Xcode console, remote DevTools on web)
- Temporary `onFocus`, `onBlur`, and key event logs
- Screenshots or a recording
- A profiler capture for slow screens
- App, RN, and OS versions

## 4. Compare

Find what differs between working and broken:

- Another device or OS version
- The last good build (`git bisect`)
- Chrome vs the real TV
- Debug vs release build

## 5. One change

Change one thing, retest, note the result.

- Several changes at once means you don't know what fixed or broke it.
- Revert changes that didn't help.

## Example

*Focus jumps to the first card after coming back to Home.*

1. **Reproduce:** 3rd rail, open a title, Back. Happens every time on the Android TV stick.
2. **Classify:** focus.
3. **Collect:** logs show the focused card was unmounted while away.
4. **Compare:** only happens on long virtualised rails.
5. **One change:** restore the saved focused index with `hasTVPreferredFocus`.

## Common mistakes

- Fixing before reproducing
- Testing only on a simulator or in Chrome
- Changing several things at once
