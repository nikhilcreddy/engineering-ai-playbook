# Prompt: Design an API

> Template for designing a REST or GraphQL API contract. Replace every `{{...}}`.

## Template

```
Act as agents/software-architect.md.

Use:
- skills/api-design.md
- {{skills/graphql.md OR REST focus}}
Enforce:
- standards/architecture.md
- standards/security.md

## Goal
{{WHAT_THE_API_ENABLES + WHO_CONSUMES_IT}}

## Domain
- Core resources/concepts: {{RESOURCES}}
- Key operations: {{LIST}}
- Data ownership / source services: {{SERVICES}}
- Expected scale: {{RPS, DATA_VOLUME, GROWTH}}

## Constraints
- Style: {{REST / GraphQL / both}} and why if unsure (ask me to decide)
- Consistency: {{STRONG / EVENTUAL}}
- AuthN/AuthZ: {{SCHEME + SCOPES}}
- Backward compatibility / versioning needs: {{...}}
- Non-functional: {{latency budget, rate limits, pagination expectations}}

## Please produce
1. A recommendation on style (REST vs GraphQL) with trade-offs if not fixed.
2. The contract:
   - REST: OpenAPI excerpt (paths, methods, status codes, schemas, errors).
   - GraphQL: SDL (types, queries, mutations, connections, error/union types).
3. Conventions: naming, versioning, pagination, idempotency, RFC 7807 errors.
4. Security: authz model (object-level), scopes, rate limiting.
5. Evolution strategy: how consumers stay unbroken as it grows.
6. Open questions / decisions needing product input.

Design the contract first; do not write implementation unless I ask.
```

## Tips

- State **who consumes** the API — internal service vs. public client changes the design.
- Decide **REST vs GraphQL** based on access patterns; ask the architect to recommend if unsure.
- Always specify **pagination, idempotency, and error format** up front.
- See [examples/api-design-example.md](../examples/api-design-example.md).
