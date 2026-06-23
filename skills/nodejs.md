# Skill: Node.js

> Building robust, observable, testable backend services with Node.js + TypeScript.

## Best practices

- **TypeScript everywhere**, `strict` mode on — see [skills/typescript.md](typescript.md).
- **Use modern ESM and current LTS Node.** Prefer the built-in `node:` prefix for core modules.
- **Async/await over raw callbacks/promise chains.** Always handle rejections; never leave a floating promise.
- **Validate all input at the boundary** (HTTP body, query, params, env) with Zod; never trust `req.body`.
- **Layered structure:** route/controller → service → repository. Keep handlers thin.
- **Centralized error handling** middleware that maps errors to RFC 7807 `problem+json`.
- **Graceful shutdown:** handle `SIGTERM`, stop accepting connections, drain in-flight requests, close the DB pool.
- **Don't block the event loop.** Offload CPU-heavy work to worker threads or a queue.
- **Structured logging** (pino) with a correlation/trace id — see [standards/logging.md](../standards/logging.md).
- **Config from the environment**, validated at startup; fail fast if missing.
- **Parameterized SQL** (pg / Prisma / Drizzle) — never string-concatenate queries.

## Common mistakes

- ❌ Floating promises / unhandled rejections (missing `await`) → silent failures, crashes.
- ❌ Trusting `req.body`/`req.query`/`process.env` without validation.
- ❌ Blocking the event loop with sync crypto, big JSON, or CPU loops.
- ❌ Catching errors and swallowing them, or leaking stack traces to clients.
- ❌ No timeouts on outbound HTTP/DB calls → hung requests exhaust resources.
- ❌ String-built SQL → injection.
- ❌ Secrets in code or committed `.env`; logging secrets/PII.
- ❌ No graceful shutdown → dropped requests and connection leaks on deploy.

## Checklist

- [ ] `strict` TypeScript; no floating promises (lint enforced).
- [ ] Every request input validated with Zod at the boundary.
- [ ] Thin handlers; logic in services; persistence in repositories.
- [ ] Central error middleware → RFC 7807 responses; no stack traces to clients.
- [ ] Timeouts on all outbound calls; retries/circuit breaking where needed.
- [ ] Parameterized queries only.
- [ ] Graceful shutdown on `SIGTERM` (drain + close pool).
- [ ] Structured logs with correlation id; no secrets/PII.
- [ ] Config validated at startup.

## Example code

```ts
// Validated, thin Express handler with async error handling
import { Router } from 'express';
import { z } from 'zod';

const CreateOrder = z.object({
  customerId: z.string().uuid(),
  lines: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

export const orders = Router();

orders.post('/api/v1/orders', async (req, res, next) => {
  try {
    const input = CreateOrder.parse(req.body);          // validate at boundary
    const key = req.header('Idempotency-Key');
    if (!key) return res.status(400).type('application/problem+json')
      .json({ title: 'Missing Idempotency-Key', status: 400 });

    const order = await orderService.create(input, key); // idempotent
    res.status(201).json(order);
  } catch (err) {
    next(err);                                           // central error handler
  }
});
```

```ts
// Central error handler → RFC 7807, no stack traces leaked
import type { ErrorRequestHandler } from 'express';
import { ZodError } from 'zod';

export const errorHandler: ErrorRequestHandler = (err, _req, res, _next) => {
  if (err instanceof ZodError) {
    return res.status(400).type('application/problem+json').json({
      title: 'Validation failed', status: 400, code: 'VALIDATION',
      detail: err.issues.map(i => i.path.join('.')).join(', '),
    });
  }
  logger.error({ err }, 'unhandled_error');             // structured, full error
  res.status(500).type('application/problem+json')
    .json({ title: 'Internal Server Error', status: 500 });
};
```

```ts
// Graceful shutdown — drain in-flight, close the pool
const server = app.listen(port);

async function shutdown(signal: string) {
  logger.info({ signal }, 'shutting_down');
  server.close(async () => {            // stop accepting, let in-flight finish
    await pool.end();                   // close DB connections
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 10_000).unref();   // hard cap
}
process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

## Recommended patterns

- **Zod-validated config** loaded once at startup (`const env = EnvSchema.parse(process.env)`).
- **Repository pattern** over a typed query builder (Prisma/Drizzle) or `pg` with parameterized SQL.
- **pino** for fast structured JSON logging with request correlation ids.
- **OpenTelemetry** for traces/metrics — see [skills/observability.md](observability.md).
- **Testcontainers (Node)** for integration tests against real PostgreSQL — see [skills/testing.md](testing.md).
- Built by backend engineers; deployed on Kubernetes per [skills/kubernetes.md](kubernetes.md); secured per [standards/security.md](../standards/security.md).
