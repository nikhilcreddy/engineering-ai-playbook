# Getting Started with the Engineering AI Playbook

> How to put the playbook to work in your day-to-day engineering with AI tools.

## The mental model

Every effective AI request combines up to four building blocks from this repo:

```
Persona (agents/)  +  Knowledge (skills/)  +  Task (prompts/)  +  Rules (standards/)
```

- **Persona** — _who_ the AI acts as.
- **Knowledge** — _what_ it should know for this task.
- **Task** — _the structured request_ to perform.
- **Rules** — _the acceptance criteria_ that define "done."

## A 60-second workflow

1. **Pick a persona** that matches the job:
   - Build a feature → [agents/backend-engineer.md](../agents/backend-engineer.md)
   - Design a system/API → [agents/software-architect.md](../agents/software-architect.md)
   - Review code → [agents/code-reviewer.md](../agents/code-reviewer.md)
   - Reliability/incident → [agents/sre-engineer.md](../agents/sre-engineer.md)
   - Security review → [agents/security-engineer.md](../agents/security-engineer.md)

2. **Attach relevant skills** (1–3) from [skills/](../skills/).

3. **Start from a prompt template** in [prompts/](../prompts/) and fill in the placeholders.

4. **Name the standards** that must be satisfied from [standards/](../standards/).

## Using it with GitHub Copilot

In Copilot Chat, attach files as context and reference them:

```
#file:agents/backend-engineer.md #file:skills/spring-boot.md
Implement a paginated GET /api/v1/customers endpoint. Must satisfy
#file:standards/clean-code.md and #file:standards/security.md.
```

To enforce standards repo-wide, reference them from your service's
`.github/copilot-instructions.md` (see the [README](../README.md#how-to-use-with-github-copilot)).

## Using it with AI agents

Give the agent the persona first, then skills, then the filled-in prompt template, and
list the standards as acceptance criteria. See full walkthroughs in [examples/](../examples/).

## Where to go next

| I want to…                     | Start here                                                        |
| ------------------------------ | ----------------------------------------------------------------- |
| Implement a feature            | [prompts/implement-feature.md](../prompts/implement-feature.md)   |
| Review a PR                    | [prompts/review-pr.md](../prompts/review-pr.md)                   |
| Design an API                  | [prompts/design-api.md](../prompts/design-api.md)                 |
| Debug a production issue       | [prompts/debug-production-issue.md](../prompts/debug-production-issue.md) |
| See it all combined            | [examples/](../examples/)                                         |
| Record a decision              | [templates/adr-template.md](../templates/adr-template.md)         |

## Contributing

This playbook improves through pull requests like any other code. See the
[contribution guidelines](../README.md#contribution-guidelines).
