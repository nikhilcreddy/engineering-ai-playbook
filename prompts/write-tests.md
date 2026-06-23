# Prompt: Write Tests

> Template for adding meaningful tests to existing or new code. Replace every `{{...}}`.

## Template

```
Act as agents/backend-engineer.md.

Use:
- skills/testing.md
- skills/spring-boot.md
Enforce:
- standards/clean-code.md

## Code under test
{{ATTACH_FILES e.g. #file:src/.../OrderService.java}}

## What it should do
{{BEHAVIOR_SPEC / acceptance criteria}}

## Current test coverage
{{WHAT_EXISTS_OR_NONE}}

## Please produce
1. A short test plan: what behaviors and edge cases must be covered.
2. Tests at the right level:
   - Unit tests for business logic (no Spring context).
   - Slice tests (@WebMvcTest / @DataJpaTest / @GraphQlTest) where appropriate.
   - One integration test with Testcontainers (real PostgreSQL) for the
     critical path.
3. Coverage of edge/failure cases: nulls/empties, boundaries, validation
   errors, timeouts/exceptions, concurrency, and idempotency.
4. Deterministic tests: inject Clock, no Thread.sleep, no shared state.
5. Behavior-focused assertions (AssertJ), descriptive test names.

Do not test private methods or assert on logs. Test observable behavior.
Avoid over-mocking — prefer real collaborators where cheap.
```

## Tips

- Ask for a **test plan first** so coverage is intentional, not incidental.
- Demand **edge and failure cases** explicitly — happy-path-only is the common failure.
- For persistence code, insist on **Testcontainers** (not H2) to catch real SQL issues.
- Name tests by **behavior**: `create_returnsConflict_whenIdempotencyKeyReused`.
- See [skills/testing.md](../skills/testing.md) for patterns and examples.
