# Skill: API Design

> Designing consistent, evolvable REST and GraphQL APIs for microservices.

## Best practices

- **Design the contract first.** Agree on the API (OpenAPI/SDL) before writing implementation.
- **Model resources and capabilities**, not internal tables. The API is a product for its consumers.
- **Consistent conventions:** plural nouns (`/orders`), nested resources sparingly, kebab/lower-case paths.
- **Correct HTTP semantics:** `GET` safe, `PUT`/`DELETE` idempotent, `POST` for creation; meaningful status codes.
- **Version explicitly** (`/api/v1/...`) and evolve additively; never break existing consumers.
- **Standard error format:** RFC 7807 `application/problem+json` with stable error codes.
- **Pagination, filtering, sorting** for collections; cursor-based for large/volatile sets.
- **Idempotency keys** for unsafe-but-retryable operations (payments, order creation).
- **Pagination & rate limits** to protect the service; document them.
- **Security in the contract:** auth scheme, scopes, and per-operation authorization documented.

## Common mistakes

- ❌ Verbs in URLs (`/createOrder`) instead of resources + HTTP methods.
- ❌ Returning `200 OK` for errors with an error body.
- ❌ Breaking changes without a new version (removing/renaming fields, tightening types).
- ❌ Inconsistent naming/casing across endpoints.
- ❌ Unbounded list endpoints (no pagination) → memory and latency blowups.
- ❌ Leaking internal models, enums, or DB ids that constrain future change.
- ❌ Non-idempotent create endpoints that duplicate on retry.
- ❌ Ad-hoc, inconsistent error shapes.

## Checklist

- [ ] Contract (OpenAPI/SDL) written and reviewed before build.
- [ ] Resource-oriented paths; correct HTTP methods and status codes.
- [ ] Versioned and backward-compatible.
- [ ] RFC 7807 error format with stable codes.
- [ ] Collections paginated, filterable, sortable.
- [ ] Idempotency supported for retryable mutations.
- [ ] AuthN/AuthZ and scopes documented per operation.
- [ ] Rate limits defined and documented.
- [ ] Examples included for every operation.

## Example code

```yaml
# OpenAPI excerpt — resource-oriented, versioned, problem+json errors
paths:
  /api/v1/orders:
    post:
      summary: Create an order
      parameters:
        - in: header
          name: Idempotency-Key
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/CreateOrderRequest' }
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema: { $ref: '#/components/schemas/OrderResponse' }
        '400':
          description: Validation error
          content:
            application/problem+json:
              schema: { $ref: '#/components/schemas/Problem' }
    get:
      summary: List orders (cursor paginated)
      parameters:
        - { in: query, name: first, schema: { type: integer, default: 20 } }
        - { in: query, name: after, schema: { type: string } }
      responses:
        '200': { description: OK }
```

```json
// RFC 7807 error response
{
  "type": "https://errors.internal/orders/validation",
  "title": "Validation failed",
  "status": 400,
  "code": "ORDER_VALIDATION",
  "detail": "field 'quantity' must be >= 1",
  "instance": "/api/v1/orders"
}
```

## Recommended patterns

- **API style guide** enforced in CI (spectral lint for OpenAPI, GraphQL schema lint).
- **Consumer-driven contract tests** to catch breaking changes before release.
- **Backward-compatible evolution:** add fields, deprecate (don't delete), default new fields.
- **Choose the right tool:** REST for resource CRUD and caching; GraphQL for client-driven aggregation — see [skills/graphql.md](graphql.md).
- Designed by [agents/software-architect.md](../agents/software-architect.md), implemented per [skills/spring-boot.md](spring-boot.md) (Java) or [skills/nodejs.md](nodejs.md) (Node/TypeScript), consumed by clients per [skills/react.md](react.md), and secured per [standards/security.md](../standards/security.md).

## Stack notes

These principles are **language-agnostic** — the contract is the product, regardless of who implements it.

| Concern              | Java / Spring Boot                        | Node.js / TypeScript                      |
| -------------------- | ----------------------------------------- | ----------------------------------------- |
| Contract definition  | OpenAPI (springdoc), GraphQL SDL          | OpenAPI (zod-to-openapi/tsoa), GraphQL SDL |
| Input validation     | Jakarta Bean Validation (`@Valid`)        | Zod at the request boundary               |
| Error format         | `ProblemDetail` (RFC 7807)                | `application/problem+json` payload        |
| Typed client         | Generated client / `RestClient`           | Generated types / typed `fetch` + Zod     |

```ts
// Node/TypeScript — same contract, validated with Zod, RFC 7807 errors
import { z } from 'zod';

const CreateOrderRequest = z.object({
  customerId: z.string().uuid(),
  lines: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

// 201 -> OrderResponse | 400 -> application/problem+json | 409 -> idempotency conflict
```

Frontend consumers (React) should **validate responses at the boundary** and treat the API contract as untrusted input — see [skills/react.md](react.md) and [skills/typescript.md](typescript.md).
