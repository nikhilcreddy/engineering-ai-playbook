# Prompt: Implement a Feature

> Parameterized template for implementing a backend feature end-to-end. Replace every `{{...}}`.

## Template

```
Act as agents/backend-engineer.md.

Use these skills:
- skills/spring-boot.md
- skills/java21.md
- skills/testing.md
{{ADDITIONAL_SKILLS e.g. skills/graphql.md, skills/observability.md}}

Enforce these standards as acceptance criteria:
- standards/clean-code.md
- standards/security.md
- standards/logging.md

## Feature
{{ONE_PARAGRAPH_DESCRIPTION_OF_THE_FEATURE}}

## Context
- Service: {{SERVICE_NAME}}
- Relevant existing code: {{FILES_OR_PACKAGES}}
- Data model: {{ENTITIES_TABLES}}
- Upstream/downstream dependencies: {{SERVICES_OR_APIS}}

## Requirements
- API: {{REST_OR_GRAPHQL + endpoint/operation}}
- Validation rules: {{RULES}}
- Idempotency: {{YES_NO + KEY_STRATEGY}}
- Persistence: PostgreSQL via {{JPA / jOOQ}}
- Events to publish/consume: {{EVENTS_OR_NONE}}
- Observability: metrics {{NAMES}}, key log events {{EVENTS}}

## Constraints
- Latency budget: {{e.g. p99 < 200ms}}
- Backward compatibility: {{REQUIREMENTS}}
- {{OTHER_CONSTRAINTS}}

## Deliverables
1. Implementation (controller/resolver, service, repository, DTOs).
2. Validation and error handling (RFC 7807 ProblemDetail).
3. Unit + slice + Testcontainers integration tests.
4. Metrics and structured logs.
5. A short summary of design decisions and any trade-offs.

Ask me clarifying questions only if a requirement is ambiguous enough to
change the design; otherwise proceed with sensible, documented assumptions.
```

## Tips

- Be explicit about **idempotency** and **failure behavior** — these drive most of the design.
- Name the **metrics and log events** you expect, so observability isn't skipped.
- Point at **existing code** so the implementation matches local conventions.
- For GraphQL features, add [skills/graphql.md](../skills/graphql.md) and specify batching needs.
