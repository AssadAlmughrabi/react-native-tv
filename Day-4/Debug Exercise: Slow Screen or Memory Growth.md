# Debug Exercise: Slow Screen or Memory Growth

Typical bugs: slow screen load, stuttering scroll, laggy focus, or the app getting slower or crashing the longer it runs.

## 1. Reproduce

Note the screen, the device (especially a low-end TV), and the trigger, for example "open and close the player 20 times".

## 2. Measure

In the remote debugger, record a performance trace to find long tasks and dropped frames. Take a heap snapshot before and after repeating the action; memory that keeps rising means a leak.

## 3. Common causes

| Symptom | Likely cause |
|---|---|
| Slow load | Too much data or too many requests at once |
| Stuttering scroll | Too many DOM nodes or heavy re-renders |
| Memory growth | Listeners, timers, or players not cleaned up on leaving a screen |
| Crash on low-end device | Images too large for device memory |

## 4. Fix

- Load and render only what is visible.
- Keep key and focus handlers light.
- Remove listeners, clear timers, and destroy players when leaving a screen.
- Resize images to the size they are shown at.

## 5. Verify

Repeat the action many times on a low-end device. Memory should level off and scrolling should stay smooth.