# Skill: React

> Building maintainable, performant React applications with TypeScript.

## Best practices

- **Function components + hooks only.** No class components in new code.
- **Type every component's props** with an explicit `type`/`interface`; avoid `React.FC` (implicit `children`).
- **Colocate state where it's used; lift only when shared.** Avoid premature global state.
- **Server state ≠ UI state.** Use a data-fetching library (TanStack Query / RTK Query) for server data; `useState`/`useReducer` for local UI state.
- **Keep components small and presentational where possible**; push logic into hooks (`useOrders()`).
- **Derive, don't duplicate state.** Compute from props/state during render instead of syncing with `useEffect`.
- **`useEffect` is for synchronizing with external systems**, not for transforming data — most effects are a smell.
- **Stable keys** for lists (stable IDs, never array index for dynamic lists).
- **Accessibility first:** semantic HTML, labels, roles, keyboard support.
- **Code-split** routes and heavy components with `React.lazy`/dynamic import.

## Common mistakes

- ❌ Using `useEffect` to derive state that could be computed during render.
- ❌ Missing/incorrect `useEffect` dependency arrays → stale closures or infinite loops.
- ❌ Array index as `key` for reorderable/dynamic lists.
- ❌ Storing server data in `useState` and hand-rolling caching/loading/error.
- ❌ Prop drilling everywhere instead of composition or context for truly global concerns.
- ❌ Over-using `useMemo`/`useCallback` prematurely (measure first) — or omitting them where referential stability matters.
- ❌ Giant components mixing fetching, state, and markup.
- ❌ `dangerouslySetInnerHTML` with unsanitized input (XSS).

## Checklist

- [ ] Function components with explicitly typed props.
- [ ] Server state via a query library; UI state local.
- [ ] No effects used to derive state that render can compute.
- [ ] `useEffect` deps correct and exhaustive (lint enforced).
- [ ] Stable, unique list keys.
- [ ] Logic extracted into reusable, testable hooks.
- [ ] Accessible markup (semantic elements, labels, keyboard).
- [ ] No unsanitized HTML injection; user content escaped.

## Example code

```tsx
// Typed props, derived value, no needless effect
type Props = { firstName: string; lastName: string };

export function FullName({ firstName, lastName }: Props) {
  const fullName = `${firstName} ${lastName}`;   // derived during render
  return <span>{fullName}</span>;
}
```

```tsx
// Server state via TanStack Query — caching/loading/error handled for you
import { useQuery } from '@tanstack/react-query';
import { z } from 'zod';

const Order = z.object({ id: z.string(), status: z.string() });
type Order = z.infer<typeof Order>;

function useOrder(id: string) {
  return useQuery({
    queryKey: ['order', id],
    queryFn: async (): Promise<Order> => {
      const res = await fetch(`/api/v1/orders/${id}`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return Order.parse(await res.json());        // validate at the boundary
    },
  });
}

export function OrderView({ id }: { id: string }) {
  const { data, isLoading, isError, error } = useOrder(id);
  if (isLoading) return <p>Loading…</p>;
  if (isError) return <p role="alert">Error: {(error as Error).message}</p>;
  return <p>Order {data.id}: {data.status}</p>;
}
```

```tsx
// Custom hook encapsulating logic — easy to test in isolation
function useDebouncedValue<T>(value: T, delayMs: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setDebounced(value), delayMs);
    return () => clearTimeout(t);                  // cleanup avoids leaks
  }, [value, delayMs]);
  return debounced;
}
```

## Recommended patterns

- **Container/presentational split** via hooks: a `useX()` hook holds logic, a dumb component renders.
- **TanStack Query / RTK Query** for all server state (caching, retries, invalidation).
- **Suspense + error boundaries** for declarative loading/error states.
- **Zod at the fetch boundary** so untrusted JSON becomes typed, validated data — see [skills/typescript.md](typescript.md).
- **Component testing with RTL + Vitest**, end-to-end with Playwright — see [skills/testing.md](testing.md).
- Built by [agents/frontend-engineer.md](../agents/frontend-engineer.md); consumes APIs designed per [skills/api-design.md](api-design.md) and [skills/graphql.md](graphql.md).
