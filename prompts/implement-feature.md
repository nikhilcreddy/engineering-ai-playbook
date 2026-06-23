# Prompt: Implement a Feature

> Parameterized template for implementing a feature end-to-end. Replace every `{{...}}`.
> Works for **Java/Spring Boot**, **Node.js/TypeScript**, and **React/TypeScript** — pick your stack below.

## Pick your stack

| Stack                 | Persona                              | Core skills                                                        |
| --------------------- | ------------------------------------ | ----------------------------------------------------------------- |
| Java / Spring Boot    | `agents/backend-engineer.md`         | `skills/spring-boot.md`, `skills/java21.md`, `skills/testing.md`   |
| Node.js / TypeScript  | `agents/backend-engineer.md`         | `skills/nodejs.md`, `skills/typescript.md`, `skills/testing.md`    |
| React / TypeScript    | `agents/frontend-engineer.md`        | `skills/react.md`, `skills/typescript.md`, `skills/testing.md`     |

## Template

```
Act as {{PERSONA — backend-engineer or frontend-engineer}}.

Use these skills (pick for your stack):
- {{skills/spring-boot.md + skills/java21.md  |  skills/nodejs.md + skills/typescript.md  |  skills/react.md + skills/typescript.md}}
- skills/testing.md
{{ADDITIONAL_SKILLS e.g. skills/graphql.md, skills/api-design.md, skills/observability.md}}

Enforce these standards as acceptance criteria:
- standards/clean-code.md
- standards/security.md
- standards/logging.md   (backend)

## Feature
{{ONE_PARAGRAPH_DESCRIPTION_OF_THE_FEATURE}}

## Context
- Service / app: {{NAME}}
- Stack: {{Java+Spring | Node+TS | React+TS}}
- Relevant existing code: {{FILES_OR_MODULES}}
- Data model / state: {{ENTITIES_TABLES or component/server state}}
- Upstream/downstream dependencies: {{SERVICES_OR_APIS}}

## Requirements
- API / interface: {{REST_OR_GRAPHQL endpoint  |  component + props/routes}}
- Validation rules: {{RULES — Bean Validation / Zod}}
- Idempotency (backend): {{YES_NO + KEY_STRATEGY}}
- Persistence (backend): PostgreSQL via {{JPA | Prisma/Drizzle/pg}}
- Server-state handling (frontend): {{TanStack Query / RTK Query}}
- Events to publish/consume: {{EVENTS_OR_NONE}}
- Observability: metrics {{NAMES}}, key log/error events {{EVENTS}}

## Constraints
- Performance budget: {{p99 < 200ms  |  Core Web Vitals targets}}
- Accessibility (frontend): {{WCAG requirements}}
- Backward compatibility: {{REQUIREMENTS}}
- {{OTHER_CONSTRAINTS}}

## Deliverables
1. Implementation:
   - Backend: controller/route/resolver, service, repository, DTOs.
   - Frontend: component(s), hook(s), typed API client.
2. Validation and error handling (RFC 7807 problem+json on the backend;
   loading/empty/error states on the frontend).
3. Tests: unit + integration (Testcontainers on the backend) /
   component (RTL) + e2e (Playwright) on the frontend.
4. Observability: metrics & structured logs (backend) / error + web-vitals
   reporting (frontend).
5. A short summary of design decisions and any trade-offs.

Ask me clarifying questions only if a requirement is ambiguous enough to
change the design; otherwise proceed with sensible, documented assumptions.
```

## Tips

- Be explicit about **idempotency** and **failure behavior** (backend) — these drive most of the design.
- For frontend, always specify **loading/empty/error states** and **accessibility** expectations.
- Name the **metrics and log/error events** you expect, so observability isn't skipped.
- Point at **existing code** so the implementation matches local conventions.
- For GraphQL features, add [skills/graphql.md](../skills/graphql.md) and specify batching needs.
