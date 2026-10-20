# One Set of States

Every screen and rail can be in one of five states: **content**, **loading**, **empty**, **error**, **offline**. Build them **once** as shared components, and give every non-content state a **focusable action** — otherwise focus has nowhere to go and the remote stops working.

---

## 1. The states

| State | Shows | Focusable action |
| --- | --- | --- |
| **Loading** | Skeleton / spinner | After a timeout (~10 s): **Retry** |
| **Empty** | "Nothing here yet" | **Browse** / **Refresh** |
| **Error** | Short message + error code | **Retry** (+ Back works) |
| **Offline** | "No internet connection" | **Retry** (also auto-retry on reconnect) |

Why focusable: when content disappears, the focused card is gone. Without a focusable button, **focus is lost** and the user is stuck.

---

## 2. One shared component

```tsx
// StateView.tsx
type Kind = 'loading' | 'empty' | 'error' | 'offline';

const COPY: Record<Kind, { title: string; action: string }> = {
  loading: { title: 'Loading…', action: 'Retry' },
  empty: { title: 'Nothing here yet', action: 'Refresh' },
  error: { title: 'Something went wrong', action: 'Retry' },
  offline: { title: 'No internet connection', action: 'Retry' },
};

export function StateView({ kind, onAction, code }: { kind: Kind; onAction: () => void; code?: string }) {
  const [showAction, setShowAction] = useState(kind !== 'loading');

  useEffect(() => {
    if (kind !== 'loading') return setShowAction(true);
    const t = setTimeout(() => setShowAction(true), 10_000); // slow load: offer retry
    return () => clearTimeout(t);
  }, [kind]);

  return (
    <View style={styles.center}>
      {kind === 'loading' && <ActivityIndicator size="large" />}
      <Text style={styles.title}>{COPY[kind].title}</Text>
      {code && <Text style={styles.code}>Error {code}</Text>}
      {showAction && (
        <FocusableButton hasTVPreferredFocus title={COPY[kind].action} onPress={onAction} />
      )}
    </View>
  );
}
```

Usage — same pattern on every screen:

```tsx
if (offline) return <StateView kind="offline" onAction={refetch} />;
if (isLoading && !data) return <StateView kind="loading" onAction={refetch} />;
if (error) return <StateView kind="error" code={error.code} onAction={refetch} />;
if (!data.length) return <StateView kind="empty" onAction={refetch} />;
return <Content data={data} />;
```

---

## 3. Focus rules

- The action button gets **`hasTVPreferredFocus`**, so focus lands on it when the state appears.
- While loading with no button yet, the **menu stays focusable** — the user can always leave.
- When content arrives, focus moves to the **first item** (or the restored item id).
- **Per rail:** a failed rail shows a small inline Retry card in the rail; the rest of the screen still works.
- Back always works in every state.

---

## 4. Offline

- Native: `@react-native-community/netinfo`. Web: `navigator.onLine` + `online` / `offline` events.
- Show **offline** instead of a generic error when the network is down.
- **Auto-retry** when the connection comes back; keep the Retry button anyway.
- Keep showing **cached content** when available, with a small "Offline" banner, instead of a blank state.

---

## 5. Loading tips

- Prefer **skeletons** in the real layout over a full-screen spinner — less jump when content arrives.
- Keep the old content visible while refreshing.
- Skeletons are **not focusable**.

---

## Summary

- Five states: content, loading, empty, error, offline — one shared `StateView`.
- Every non-content state has a focusable action with `hasTVPreferredFocus`.
- Slow loading offers Retry after a timeout; the menu stays reachable.
- Offline: detect it, show cached content, auto-retry on reconnect.
