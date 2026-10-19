# Emulator vs Real Device

Emulators and simulators run on your fast dev machine. They are great for building layouts and flows, but they hide most of the problems real TVs have.

---

## 1. What an emulator is good for

- Fast iteration on UI, layout, and navigation.
- Checking focus order with keyboard arrows.
- Debugging JS with full dev tools.
- Testing many OS versions and screen sizes quickly.

---

## 2. What an emulator cannot tell you

| Area | Emulator | Real device |
| --- | --- | --- |
| CPU speed | Uses your laptop CPU — fast | Slow TV chip, focus lag shows up |
| RAM | Plenty of memory | 1–2 GB shared, app can be killed |
| Video playback | Software decoding, DRM often missing | Hardware decoder, DRM, codec limits |
| Browser engine (LG/Samsung) | Simulators may not match the real engine | Exact old Chromium of that model year |
| Remote | Keyboard arrows | Real remote: key repeat, delays, extra keys (Back, Home, color keys) |
| Display | Your monitor, sharp and close | Real TV: overscan, color, viewing from 3 m |
| Network | Fast office Wi-Fi/Ethernet | Weak home Wi-Fi |
| Startup time | Fast disk | Slow storage, cold start much longer |
| Long use | Short test sessions | Memory leaks after hours of watching |

---

## 3. TV way

- Build on the emulator, **verify on real devices**.
- Keep at least one **low-end** device per family (example: cheap Android TV stick, oldest LG/Samsung you support).
- Test with the **real remote** from the couch, not from your desk.
- Measure on device: startup time, focus lag, memory, video start time.

---

## Summary

- Emulators are for fast building, not for final answers.
- They hide slow CPUs, low RAM, real video/DRM, old engines, and real remote behavior.
- Always test on real, low-end TVs before calling a feature done.
