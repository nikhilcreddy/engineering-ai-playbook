# Standard: Code Review

> How we review code: what to check, how to comment, and when to approve.

## Goals

1. Protect correctness, security, and long-term maintainability.
2. Share knowledge and raise the team's bar.
3. Keep delivery flowing — review fast, review small.

## Author responsibilities

- **Keep PRs small and focused** (ideally < ~400 lines of meaningful diff). Split large work.
- Write a clear description: _what_ changed, _why_, and _how it was tested_.
- Include tests for new behavior and bug fixes.
- Self-review the diff first; resolve obvious issues before requesting review.
- Call out risky areas and decisions you want scrutiny on.

## Reviewer responsibilities

- Understand the context before judging — read surrounding code.
- Review for, in priority order: **correctness → security → tests → design → readability**.
- Be specific and actionable; suggest the fix or link a standard/skill.
- Critique the code, not the author. Assume good intent.
- Respond promptly (same business day) to keep authors unblocked.

## What to check

- **Correctness:** logic, edge cases, concurrency, error/failure paths, idempotency.
- **Security:** OWASP Top 10, secrets, authz, input validation — see [standards/security.md](security.md).
- **Tests:** behavior covered (not just lines), edge cases, deterministic — see [skills/testing.md](../skills/testing.md).
- **Observability:** appropriate logs/metrics; no secrets in logs — see [standards/logging.md](logging.md).
- **Design:** fits architecture, no needless coupling/duplication.
- **Readability:** naming, function size, clarity — see [standards/clean-code.md](clean-code.md).
- **Performance:** N+1 queries, unbounded loops, missing pagination, blocking calls.

## Comment labels

| Label         | Meaning                                            |
| ------------- | -------------------------------------------------- |
| 🛑 **Blocking** | Must fix before merge (correctness/security/tests) |
| ⚠️ **Should**  | Strongly recommended; author's discretion          |
| 💡 **Nit**     | Minor/style; non-blocking                          |
| ❓ **Question** | Need clarification                                 |

## Approval criteria

Approve when the change:

- [ ] Is correct and handles edge/failure cases.
- [ ] Has no security regressions.
- [ ] Is covered by meaningful tests that pass.
- [ ] Is appropriately observable (logs/metrics, no secrets).
- [ ] Fits the architecture and meets clean-code standards.
- [ ] Has no unresolved 🛑 Blocking comments.

> Approve when it's **better than before and meets the bar** — not when it's perfect.

## Anti-patterns to avoid

- ❌ "LGTM" with no actual review on a non-trivial change.
- ❌ Blocking on pure style preferences (automate style with formatters/linters).
- ❌ Rewriting the author's design in comments instead of discussing.
- ❌ Giant PRs that can't be reviewed meaningfully — push back.

Applied through the lens of [agents/code-reviewer.md](../agents/code-reviewer.md); see [prompts/review-pr.md](../prompts/review-pr.md).
