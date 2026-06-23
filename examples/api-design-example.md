# Example: Designing an API

> A worked example of using the architect agent to design an API contract before implementation.

## Scenario

Design a customer-facing API to browse and create orders. Consumers are a web app and a mobile app with different data needs.

We use [agents/software-architect.md](../agents/software-architect.md), [prompts/design-api.md](../prompts/design-api.md), and [skills/api-design.md](../skills/api-design.md) + [skills/graphql.md](../skills/graphql.md).

## Step 1 — Style recommendation

**Recommendation: GraphQL for reads, REST for the create mutation-style action.**

Trade-offs:

- Web and mobile need **different field sets** of the same order/customer graph → GraphQL avoids over/under-fetching and multiple REST round-trips.
- Order **creation** is a simple, cacheable-by-idempotency, well-bounded action → a REST `POST` (or a single GraphQL mutation) is fine; we keep it REST for idempotency-key ergonomics and HTTP semantics.

## Step 2 — GraphQL read contract

```graphql
type Query {
  order(id: ID!): Order
  orders(first: Int!, after: String): OrderConnection!
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  customer: Customer!          # batched via DataLoader (no N+1)
  lines: [OrderLine!]!
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}

enum OrderStatus { PENDING PAID SHIPPED CANCELLED }
```

Conventions applied:

- **Cursor pagination** (`first`/`after`, `pageInfo`) for the volatile orders list.
- **DataLoader** batches `customer` resolution.
- **Nullability** chosen deliberately (`status!`, `total!`; `customer!` only because it always exists).
- Query **depth/complexity limits** configured to prevent abusive queries.

## Step 3 — REST create contract (OpenAPI excerpt)

```yaml
paths:
  /api/v1/orders:
    post:
      summary: Create an order
      parameters:
        - { in: header, name: Idempotency-Key, required: true, schema: { type: string } }
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/CreateOrderRequest' }
      responses:
        '201': { description: Created }
        '400':
          description: Validation error
          content:
            application/problem+json:
              schema: { $ref: '#/components/schemas/Problem' }
        '409': { description: Idempotency-Key conflict }
```

Conventions applied: versioned path, **idempotency key**, **RFC 7807** errors, correct status codes.

## Step 4 — Security & evolution

- **AuthZ:** OAuth2 scopes (`orders:read`, `orders:write`) plus **object-level** checks — a customer may only read their own orders ([standards/security.md](../standards/security.md)).
- **Rate limits:** documented per client; GraphQL query cost capped.
- **Evolution:** add fields and `@deprecated` rather than breaking; REST stays additive within `v1`. Breaking changes → `v2` + schema-lint in CI ([standards/architecture.md](../standards/architecture.md)).

## Open questions for product

- Should cancelled orders be visible in the default `orders` list or filtered out?
- Is partial-shipment status needed in v1?

## Why this works

The architect persona designed the **contract first**, chose styles by **access pattern**, and baked in pagination, idempotency, errors, authz, and an evolution strategy — before any implementation. The backend engineer can now implement against a stable contract.
