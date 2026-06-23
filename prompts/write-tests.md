# Prompt: Write Tests

> Template for adding meaningful tests to existing or new code. Replace every `{{...}}`.
> Works for **Java**, **Node.js**, and **React** — pick your stack below.

## Pick your stack

| Stack                | Persona                       | Tooling                                                    |
| -------------------- | ----------------------------- | ---------------------------------------------------------- |
| Java / Spring Boot   | `agents/backend-engineer.md`  | JUnit 5, Mockito, slice tests, Testcontainers              |
| Node.js / TypeScript | `agents/backend-engineer.md`  | Vitest/Jest, MSW, Testcontainers (Node)                    |
| React / TypeScript   | `agents/frontend-engineer.md` | Vitest + React Testing Library, MSW, Playwright (e2e)      |

## Template

```
Act as {{PERSONA — backend-engineer or frontend-engineer}}.

Use:
- skills/testing.md
- {{skills/spring-boot.md | skills/nodejs.md + skills/typescript.md | skills/react.md + skills/typescript.md}}
Enforce:
- standards/clean-code.md

## Code under test
{{ATTACH_FILES e.g. #file:src/.../OrderService.java or #file:src/OrderView.tsx}}

## What it should do
{{BEHAVIOR_SPEC / acceptance criteria}}

## Current test coverage
{{WHAT_EXISTS_OR_NONE}}

## Please produce
1. A short test plan: what behaviors and edge cases must be covered.
2. Tests at the right level:
   - Java: unit (no Spring context), slice (@WebMvcTest / @DataJpaTest /
     @GraphQlTest), one Testcontainers integration test for the critical path.
   - Node: unit (Vitest/Jest), HTTP mocked with MSW, one Testcontainers
     integration test against real PostgreSQL.
   - React: component tests with RTL (query by role/text), MSW for API calls,
     one Playwright e2e for the critical journey.
3. Coverage of edge/failure cases: nulls/empties, boundaries, validation
   errors, timeouts/exceptions, concurrency/idempotency (backend);
   loading/empty/error states (frontend).
4. Deterministic tests: inject Clock / fake timers; no sleeps; no shared state.
5. Behavior-focused assertions and descriptive test names.

Do not test private methods or implementation details. Test observable behavior.
Avoid over-mocking — prefer real collaborators where cheap; use Testcontainers
(not in-memory fakes) for DB code and MSW (not raw fetch mocks) for HTTP.
```

## Tips

- Ask for a **test plan first** so coverage is intentional, not incidental.
- Demand **edge and failure cases** explicitly — happy-path-only is the common failure.
- Backend persistence: insist on **Testcontainers** (not H2/in-memory) to catch real SQL issues.
- Frontend: **query by role/label/text** (RTL), not test ids; mock HTTP with **MSW**.
- Name tests by **behavior**: `create_returnsConflict_whenIdempotencyKeyReused`.
- See [skills/testing.md](../skills/testing.md) for patterns and examples.
