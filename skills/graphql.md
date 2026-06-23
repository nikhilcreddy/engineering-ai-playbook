# Skill: GraphQL

> Designing and implementing GraphQL APIs with Spring for GraphQL.

## Best practices

- **Schema-first.** Design the SDL (`.graphqls`) as the contract; generate/wire resolvers to it.
- **Model the domain, not the database.** Types reflect business concepts and client needs.
- **Use connections for pagination** (Relay-style `edges`/`pageInfo`) for lists that can grow.
- **Batch with DataLoaders** to eliminate N+1 across resolvers.
- **Nullability is a contract.** Mark non-null (`!`) deliberately; a non-null field that errors nulls its whole parent.
- **Errors as data where appropriate.** Use union/result types for expected failures; reserve GraphQL errors for exceptional ones.
- **Depth & complexity limits** to prevent abusive queries.
- **Persisted queries / allow-lists** for public-facing APIs.
- **Version by evolution**, not URL — add fields, deprecate with `@deprecated`, avoid breaking changes.

## Common mistakes

- ❌ N+1 resolver calls (fetching each child per parent) — must use DataLoader.
- ❌ Exposing database columns 1:1 as the schema.
- ❌ Over-using non-null (`!`) and then nulling parents on partial failure.
- ❌ No query depth/complexity limits → DoS via deeply nested queries.
- ❌ Returning generic errors with stack traces / internal details.
- ❌ Mutations that aren't idempotent or don't return the mutated entity.
- ❌ Treating GraphQL like REST (one resolver doing everything, no batching).

## Checklist

- [ ] Schema is the source of truth and reviewed like an API contract.
- [ ] List fields use cursor-based connections.
- [ ] DataLoaders batch all child/association fetches (no N+1).
- [ ] Nullability chosen intentionally per field.
- [ ] Query depth and complexity limits configured.
- [ ] Expected business errors modeled as result/union types.
- [ ] Mutations return the affected entity and are idempotent where possible.
- [ ] Field-level authorization enforced.

## Example code

```graphql
# schema.graphqls
type Query {
  order(id: ID!): Order
  orders(first: Int!, after: String): OrderConnection!
}

type Mutation {
  createOrder(input: CreateOrderInput!): CreateOrderPayload!
}

type Order {
  id: ID!
  status: OrderStatus!
  customer: Customer!          # resolved via DataLoader
  lines: [OrderLine!]!
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}

union CreateOrderPayload = CreateOrderSuccess | ValidationError
```

```java
@Controller
class OrderGraphQlController {

    private final OrderService orderService;

    OrderGraphQlController(OrderService orderService) {
        this.orderService = orderService;
    }

    @QueryMapping
    Order order(@Argument UUID id) {
        return orderService.getById(id);
    }

    @MutationMapping
    CreateOrderPayload createOrder(@Argument @Valid CreateOrderInput input) {
        return orderService.create(input);
    }

    // Batched association resolution — avoids N+1
    @BatchMapping(typeName = "Order", field = "customer")
    Map<Order, Customer> customer(List<Order> orders) {
        var ids = orders.stream().map(Order::customerId).toList();
        var byId = customerService.findAllById(ids);
        return orders.stream()
            .collect(Collectors.toMap(o -> o, o -> byId.get(o.customerId())));
    }
}
```

```yaml
# application.yml — guardrails
spring:
  graphql:
    schema:
      introspection:
        enabled: false      # disable in production
# Add a complexity/depth instrumentation bean to cap abusive queries.
```

## Recommended patterns

- **DataLoader per request** for batching + caching within a single request.
- **Result/union payload types** for mutations so clients handle validation/business errors as data.
- **Schema linting in CI** (e.g. breaking-change detection) to protect consumers.
- **Pair with REST** where appropriate — GraphQL for aggregation/read flexibility, REST for simple resource CRUD. See [skills/api-design.md](api-design.md).
- Enforce [standards/security.md](../standards/security.md) for authz and query-cost limits.

## Server: Node.js (Apollo Server)

The same schema-first principles apply on Node. Define the SDL, resolve with batching via DataLoader, and validate inputs.

```ts
import { ApolloServer } from '@apollo/server';
import DataLoader from 'dataloader';

const typeDefs = /* GraphQL */ `
  type Query { order(id: ID!): Order }
  type Order { id: ID!, status: String!, customer: Customer! }
  type Customer { id: ID!, name: String! }
`;

const resolvers = {
  Query: {
    order: (_: unknown, { id }: { id: string }, ctx: Context) => ctx.orders.byId(id),
  },
  Order: {
    // batched — avoids N+1 across the request
    customer: (order: Order, _args: unknown, ctx: Context) =>
      ctx.loaders.customer.load(order.customerId),
  },
};

// One DataLoader per request (batching + caching scoped to the request)
function createLoaders(db: Db) {
  return {
    customer: new DataLoader<string, Customer>(ids => db.customers.byIds(ids)),
  };
}

const server = new ApolloServer({ typeDefs, resolvers });
```

Guardrails on Node: add **depth/complexity limits** (e.g. `graphql-depth-limit`, `graphql-query-complexity`), disable introspection in production, and enforce field-level authorization in resolvers/context.

## Client: React (Apollo Client / urql)

```tsx
import { useQuery, gql } from '@apollo/client';

const ORDER = gql`
  query Order($id: ID!) {
    order(id: $id) { id status customer { name } }
  }
`;

export function OrderView({ id }: { id: string }) {
  const { data, loading, error } = useQuery(ORDER, { variables: { id } });
  if (loading) return <p>Loading…</p>;
  if (error) return <p role="alert">Error: {error.message}</p>;
  return <p>Order {data.order.id}: {data.order.status}</p>;
}
```

Prefer **typed operations** (GraphQL Code Generator) so queries/mutations are fully type-checked against the schema — see [skills/typescript.md](typescript.md) and [skills/react.md](react.md).

## Stack notes

| Concern        | Java / Spring for GraphQL          | Node.js / TypeScript                       |
| -------------- | ---------------------------------- | ------------------------------------------ |
| Server         | `@Controller` + `@QueryMapping`    | Apollo Server / GraphQL Yoga               |
| Batching       | `@BatchMapping`                    | DataLoader (per request)                   |
| Input validation | Jakarta Bean Validation          | Zod inside resolvers                        |
| Client         | —                                  | Apollo Client / urql + GraphQL Code Generator |
