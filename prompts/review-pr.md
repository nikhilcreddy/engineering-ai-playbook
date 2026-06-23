# Prompt: Review a Pull Request

> Template for a thorough, structured code review. Replace every `{{...}}`.

## Template

```
Act as agents/code-reviewer.md.

Review against:
- standards/code-review.md
- standards/clean-code.md
- standards/security.md
- standards/logging.md
- skills/testing.md
{{ADDITIONAL for the PR's stack:
   Java   -> skills/spring-boot.md, skills/java21.md
   Node   -> skills/nodejs.md, skills/typescript.md
   React  -> skills/react.md, skills/typescript.md
   plus skills/graphql.md or skills/api-design.md if relevant}}

## What changed
{{PR_TITLE_AND_DESCRIPTION}}

## Diff / files
{{PASTE_DIFF_OR_ATTACH_FILES e.g. #file:src/.../OrderService.java or #file:src/OrderView.tsx}}

## Areas I want scrutiny on
{{RISKY_AREAS_OR_QUESTIONS}}

## Please produce
1. A verdict: Mergeable / Changes requested / Needs discussion.
2. Findings grouped and labeled:
   - 🛑 Blocking (correctness/security/tests)
   - ⚠️ Should
   - 💡 Nit
   - ❓ Question
   For each: file + line reference, the issue, why it matters, and a concrete fix.
3. Specific check for:
   - Correctness & edge/failure cases (incl. idempotency, concurrency)
   - Security (OWASP Top 10, secrets, authz, injection; XSS & unsafe rendering on the frontend)
   - Test quality (behavior coverage, edge cases, determinism)
   - Observability (logs/metrics, no secrets in logs)
   - Backend performance: N+1 queries, blocking calls, missing timeouts
   - Frontend: type safety (no `any`), accessibility, unnecessary re-renders,
     loading/empty/error states, response validation at the boundary
4. A one-line summary of overall risk.

Be specific and actionable. Critique the code, not the author.
```

## Tips

- Attach the **actual diff or files** — reviews on summaries miss real bugs.
- Call out the **risky areas** so the reviewer focuses where it matters.
- For DB-heavy changes, explicitly ask to check for **N+1** and transaction boundaries.
- For React changes, ask to check **accessibility, re-renders, and XSS**.
- Pair with [examples/code-review-example.md](../examples/code-review-example.md) for the expected output shape.
