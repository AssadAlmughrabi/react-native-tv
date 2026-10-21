# Component Tests for Focus and Key-Handling Logic (TV)

## Main ideas

### 1. Why

Deploying to a TV for every check is slow. Test your focus and key decisions on your laptop.

### 2. What you can test

- **Can:** your logic (which item gets focus, what OK or Back does, key mapping).
- **Can't:** the native focus engine or real remote behaviour. Use a real device for that.

### 3. Keep logic in plain functions

```js
export function nextIndex(current, key, length) {
  if (key === 'right') return Math.min(current + 1, length - 1);
  if (key === 'left') return Math.max(current - 1, 0);
  return current;
}

test('right stops at the last item', () => {
  expect(nextIndex(4, 'right', 5)).toBe(4);
});
```

### 4. Simulate events in

Fire focus and key events (`fireEvent`) and check what renders or gets called. Mock `useTVEventHandler` to send keys by hand. `verify`

### 5. What to test

- Focus at rail edges and between rails
- Restoring the remembered focus
- Key mapping per platform
- Back order: modal, then screen, then exit
- Empty rails don't crash
