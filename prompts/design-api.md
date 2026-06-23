# Prompt: Design an API

> Template for designing a REST or GraphQL API contract. Replace every `{{...}}`.

## Template

```
Act as agents/software-architect.md.

Use:
- skills/api-design.md
- {{skills/graphql.md OR REST focus}}
- {{implementation skill: skills/spring-boot.md (Java) OR skills/nodejs.md (Node/TS)}}
- {{if there's a web client: skills/react.md + skills/typescript.md}}
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
- Implementation stack: {{Java + Spring Boot | Node.js + TypeScript}}
- Consumers: {{internal services | React/TypeScript web app | mobile | public}}
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
4. Validation approach for the chosen stack (Bean Validation / Zod).
5. Security: authz model (object-level), scopes, rate limiting.
6. Consumer guidance: how a React client should validate responses at the boundary.
7. Evolution strategy: how consumers stay unbroken as it grows.
8. Open questions / decisions needing product input.

Design the contract first; do not write implementation unless I ask.
```

## Tips

- State **who consumes** the API — internal service, React web app, or public client changes the design.
- Decide **REST vs GraphQL** based on access patterns; ask the architect to recommend if unsure.
- Always specify **pagination, idempotency, and error format** up front.
- The contract is **stack-agnostic** — validation maps to Bean Validation (Java) or Zod (Node/TS); see [skills/api-design.md](../skills/api-design.md).
- See [examples/api-design-example.md](../examples/api-design-example.md).
