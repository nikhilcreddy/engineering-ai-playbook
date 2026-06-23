# Skill: TypeScript

> Idiomatic, type-safe TypeScript for frontend (React) and backend (Node.js) services.

## Best practices

- **`strict` mode on, always.** Enable `strict`, `noUncheckedIndexedAccess`, and `noImplicitOverride` in `tsconfig.json`. The compiler is your first test suite.
- **Model the domain with types.** Use discriminated unions, `readonly`, and literal types so illegal states won't compile.
- **Prefer `type` for unions/composition, `interface` for object shapes** that may be extended/implemented.
- **Parse, don't assume, at boundaries.** Validate external input (HTTP, env, JSON) with a runtime validator (Zod) and infer the static type from the schema.
- **Avoid `any`.** Use `unknown` at boundaries and narrow; reserve `any` for genuinely untyped escape hatches with a comment.
- **Discriminated unions over boolean flags** for state (`{ status: 'loading' } | { status: 'success'; data: T }`).
- **`as const`** for literal tuples/objects to get precise types.
- **No non-null assertions (`!`) by habit** — narrow with guards instead.
- **Derive types, don't duplicate** — `ReturnType`, `Parameters`, `Pick`, `Omit`, `z.infer`.

## Common mistakes

- ❌ `any` leaking through the codebase, disabling type safety silently.
- ❌ Type assertions (`as Foo`) to silence the compiler instead of fixing the type.
- ❌ Trusting `JSON.parse`/`req.body`/`process.env` as typed without runtime validation.
- ❌ `enum` where a union of string literals is simpler and tree-shakeable.
- ❌ Overusing non-null assertions (`!`) and optional chaining to paper over modeling gaps.
- ❌ Mutating shared objects; not using `readonly`/`Readonly<T>`.
- ❌ `==` instead of `===`.
- ❌ Floating-point for money — use integer minor units or a decimal library.

## Checklist

- [ ] `strict` (and `noUncheckedIndexedAccess`) enabled.
- [ ] No `any` except documented escape hatches; `unknown` at boundaries.
- [ ] External input validated at runtime (Zod) and types inferred from schemas.
- [ ] State modeled as discriminated unions, not flag soup.
- [ ] No assertion-based silencing of real type errors.
- [ ] Public function signatures fully typed (params + return).
- [ ] Immutability via `readonly` where it matters.

## Example code

```ts
// Discriminated union for async state — illegal states unrepresentable
type RemoteData<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function render<T>(state: RemoteData<T>): string {
  switch (state.status) {
    case 'idle':    return 'Idle';
    case 'loading': return 'Loading…';
    case 'success': return `Loaded ${JSON.stringify(state.data)}`;
    case 'error':   return `Error: ${state.error}`;
    // no default needed — exhaustive; new cases become compile errors
  }
}
```

```ts
// Parse, don't assume — Zod validates at the boundary and infers the type
import { z } from 'zod';

const CreateOrder = z.object({
  customerId: z.string().uuid(),
  lines: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

type CreateOrder = z.infer<typeof CreateOrder>;   // single source of truth

export function parseCreateOrder(input: unknown): CreateOrder {
  return CreateOrder.parse(input);                // throws on invalid input
}
```

```ts
// Result type instead of throwing for expected outcomes
type Result<T, E = string> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function divide(a: number, b: number): Result<number> {
  return b === 0 ? { ok: false, error: 'divide by zero' } : { ok: true, value: a / b };
}
```

## Recommended patterns

- **Schema-first with Zod** → infer types so runtime and compile-time stay in sync.
- **Branded types** for IDs (`type CustomerId = string & { readonly brand: unique symbol }`) to prevent mixing identifiers.
- **`Result`/discriminated unions** for expected failures; exceptions for the truly exceptional.
- **Shared types package** in a monorepo so frontend and Node backend share contracts.
- Apply [standards/clean-code.md](../standards/clean-code.md) and [standards/security.md](../standards/security.md). Used by [skills/react.md](react.md) and [skills/nodejs.md](nodejs.md).
