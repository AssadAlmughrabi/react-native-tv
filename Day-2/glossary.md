# Glossary: Screen, Route, Overlay, Component

Four words used everywhere in this course. Same meaning on every platform.

---

## Screen

A **full-page view** the user navigates to: Home, Details, Search, Player, Settings.

- Fills the whole TV.
- Has its own **initial focus** and **remembered focus**.
- Back from a screen goes to the previous screen.

---

## Route

The **name + params** that identify a screen in navigation.

```ts
{ name: 'Details', params: { id: 'show-123' } }
```

- A screen is *what you see*; a route is *how you get there*.
- The same screen can be opened by many routes (Details for different ids).
- The **route stack** (Home → Details → Player) decides what Back does.
- Restore focus is saved per route, so each Details page remembers its own position.

---

## Overlay

UI shown **on top of** a screen, without leaving it: modal, dialog, side menu, player controls, toast.

- The screen behind stays mounted.
- Interactive overlays **trap focus** and get **Back first**.
- Closing it **restores focus** to the element that opened it.
- Not part of the route stack (unless you choose to make it a route).

---

## Component

A **reusable piece of UI** used inside screens and overlays: `Card`, `Rail`, `Button`, `FocusableCard`, `Stack`.

- Has no route, no own Back handling.
- Gets data and callbacks through props.
- May have `.native.tsx` / `.web.tsx` versions for focus differences.

---

## How they fit together

```text
Route stack:  Home  →  Details(id: show-123)
                          │
Screen:               DetailsScreen
                          ├── Component: Hero
                          ├── Component: Rail → Card, Card, Card
                          └── Overlay:  ConfirmDialog (focus trapped, Back closes it)
```

---

## Summary

- **Screen:** full-page view with its own focus.
- **Route:** name + params that open a screen; the stack decides Back.
- **Overlay:** on top of a screen; traps focus, takes Back first, restores focus on close.
- **Component:** reusable UI inside screens and overlays; no route, no Back.
