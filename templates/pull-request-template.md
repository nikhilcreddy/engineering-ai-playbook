# Pull Request

> Copy into your PR description. Keep PRs small and focused (see [standards/code-review.md](../standards/code-review.md)).

## What & why

{{One or two sentences: what does this change do and why does it matter?}}

- **Related issue:** {{JIRA-123 / #issue}}
- **Type:** {{feature | bugfix | refactor | chore | docs}}

## Changes

- {{bullet list of the key changes}}

## How it was tested

- {{unit / slice / integration tests added or updated}}
- {{manual verification steps, if any}}

## Risk & rollout

- **Risk level:** {{low | medium | high}}
- **Backward compatible:** {{yes | no — details}}
- **Rollback plan:** {{how to revert safely}}
- **Observability:** {{metrics/logs added so we can verify in prod}}

## Author checklist

- [ ] Scoped and reasonably small.
- [ ] Tests cover new behavior and edge/failure cases ([skills/testing.md](../skills/testing.md)).
- [ ] No secrets in code, config, or logs ([standards/security.md](../standards/security.md)).
- [ ] Structured logging, no sensitive data ([standards/logging.md](../standards/logging.md)).
- [ ] Meets clean-code standards ([standards/clean-code.md](../standards/clean-code.md)).
- [ ] Self-reviewed the diff.

## Reviewer focus

{{Point reviewers at the risky areas or decisions you want scrutiny on.}}

---

Reviewed using [agents/code-reviewer.md](../agents/code-reviewer.md) and [prompts/review-pr.md](../prompts/review-pr.md).
