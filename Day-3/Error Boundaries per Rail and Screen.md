# Error Boundaries per Rail and Screen (TV)

### 1. Why it 

On TV the user only has a remote. If the app crashes to a blank screen, they are stuck.

### 2. What an error boundary does

It catches a crash in part of the UI and shows a small fallback instead of crashing the whole app.

### 3. One per rail

Each rail loads its own data, and bad data can crash it. A boundary keeps the other rails working.

### 4. One per screen

If something else breaks, the user sees an error screen with Retry instead of leaving the app.

```jsx
<ErrorBoundary fallback={<ScreenError />}>
  <ErrorBoundary fallback={<RailError />}>
    <Rail data={trending} />
  </ErrorBoundary>
</ErrorBoundary>
```

### 5. Fallbacks must be focusable

Add a focusable Retry button, or the remote has nothing to land on.

### 6. Limits

Boundaries catch render errors only. Handle network errors with try/catch.
