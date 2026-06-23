# Engineering AI Playbook

A curated, version-controlled source of **AI context, agent personas, engineering standards, prompt templates, and technical skills** for use with GitHub Copilot, coding agents, and future AI tooling.

The goal is simple: give every AI assistant the same high-quality, opinionated context that a senior engineer on our team would bring — so generated code, reviews, and designs match how we actually build and operate **enterprise Java backend systems**.

---

## Why this repository exists

AI tools are only as good as the context they're given. Without shared context, every developer gets inconsistent output, different conventions, and one-off prompts that live in chat history and disappear.

This playbook makes that context:

- **Reusable** — write it once, reference it everywhere.
- **Reviewable** — standards and personas evolve through pull requests.
- **Portable** — the same Markdown works across Copilot, agents, and other LLM tools.
- **Authoritative** — a single source of truth for how we engineer software.

---

## Primary technology stack

This playbook covers full-stack enterprise development across two backend stacks and a TypeScript frontend:

| Layer            | Technology                                      |
| ---------------- | ----------------------------------------------- |
| Backend (JVM)    | Java 21, Spring Boot                            |
| Backend (Node)   | Node.js, TypeScript                             |
| Frontend         | TypeScript, React                               |
| APIs             | GraphQL, REST                                   |
| Data             | PostgreSQL                                       |
| Orchestration    | Kubernetes                                       |
| Delivery         | ArgoCD (GitOps)                                  |
| Observability    | Prometheus, Grafana, OpenTelemetry              |
| Architecture     | Microservices                                    |

> The standards, prompts, and shared skills (api-design, graphql, testing, observability) are **stack-agnostic** and call out per-stack specifics where they differ.

---

## Repository structure

```
engineering-ai-playbook/
├── README.md          # You are here
├── docs/              # Deep-dive guides and onboarding material
├── agents/            # Reusable AI agent personas (roles + behavior)
├── skills/            # Focused technical know-how (one topic per file)
├── prompts/           # Parameterized prompt templates for common tasks
├── standards/         # Engineering standards the team holds itself to
├── templates/         # Boilerplate (PRs, ADRs, service scaffolds)
└── examples/          # End-to-end walkthroughs of the playbook in action
```

### What goes where

- **`agents/`** — _Who_ the AI should act as. Personas like `backend-engineer`, `frontend-engineer`, `code-reviewer`, `sre-engineer`. Each defines a role, responsibilities, decision-making principles, and example prompts.
- **`skills/`** — _What_ the AI should know. Deep, focused technical references (`java21`, `spring-boot`, `nodejs`, `typescript`, `react`, `graphql`, `kubernetes`) with best practices, anti-patterns, checklists, and code.
- **`standards/`** — _The rules_ we hold ourselves to (`clean-code`, `logging`, `security`).
- **`prompts/`** — _Ready-to-use_ task templates (`implement-feature`, `review-pr`, `design-api`).
- **`examples/`** — _Worked scenarios_ showing agents + skills + standards combined.

---

## How to use with GitHub Copilot

GitHub Copilot can consume this content in several ways.

### 1. Repository custom instructions

Copy or symlink the standards you want enforced into `.github/copilot-instructions.md` in your service repo, or reference them:

```markdown
<!-- .github/copilot-instructions.md -->
Follow the engineering standards defined in the Engineering AI Playbook:
- standards/clean-code.md
- standards/logging.md
- standards/security.md
Our stack is Java 21 + Spring Boot (or Node.js + TypeScript). Prefer the patterns in skills/spring-boot.md or skills/nodejs.md.
```

### 2. Copilot Chat with attached context

In Copilot Chat, attach the relevant skill or standard as context and ask:

```
#file:skills/spring-boot.md Implement a paginated REST endpoint for the
Customer resource following these patterns and standards/clean-code.md.
```

### 3. Prompt files

Drop the contents of any file in `prompts/` into Copilot Chat (or a `.prompt.md` file) and fill in the placeholders.

---

## How to use with AI agents

Coding agents (autonomous or semi-autonomous) work best when given a **persona + skills + a task template**.

A typical agent invocation combines three pieces from this repo:

```
Persona:   agents/backend-engineer.md
Skills:    skills/spring-boot.md, skills/graphql.md, skills/testing.md
Task:      prompts/implement-feature.md (filled in)
```

Recommended pattern:

1. **Load the persona** so the agent adopts the right role and decision-making principles.
2. **Attach the relevant skills** so it has deep technical knowledge for the task.
3. **Apply a prompt template** to structure the actual request.
4. **Enforce standards** by referencing the relevant `standards/` files as acceptance criteria.

See [`examples/`](examples/) for full walkthroughs.

---

## How to reference agents and skills

Use stable, relative paths so references survive across tools:

| Reference type | Syntax                                      | Example                              |
| -------------- | ------------------------------------------- | ------------------------------------ |
| Agent          | `agents/<name>.md`                          | `agents/code-reviewer.md`            |
| Skill          | `skills/<name>.md`                          | `skills/kubernetes.md`               |
| Standard       | `standards/<name>.md`                       | `standards/security.md`             |
| Prompt         | `prompts/<name>.md`                         | `prompts/design-api.md`              |
| Copilot Chat   | `#file:skills/<name>.md`                    | `#file:skills/observability.md`      |

A good request names **one agent**, **one or more skills**, and **the standards** that define "done":

> Act as `agents/software-architect.md`. Using `skills/api-design.md` and
> `skills/graphql.md`, design the schema for an Orders service. The design must
> satisfy `standards/architecture.md` and `standards/security.md`.

---

## Contribution guidelines

This playbook is a living asset. Improve it the same way we improve production code.

### Principles

1. **Practical over theoretical.** Every file should help a senior engineer ship safer, faster. Include real code, real checklists, real trade-offs.
2. **One concern per file.** Keep skills and standards focused. Split rather than bloat.
3. **Show, don't just tell.** Pair every rule with a ✅ good / ❌ bad example where possible.
4. **Stack-aware.** Default to the stack above (Java 21 + Spring Boot, Node.js + TypeScript, React) unless a file is intentionally generic.

### Workflow

1. Create a branch: `git checkout -b add/skill-redis`.
2. Add or edit Markdown following the existing structure and front-matter conventions.
3. Keep examples runnable and idiomatic.
4. Open a pull request describing _what changed_ and _why it matters_.
5. At least one reviewer approves using `agents/code-reviewer.md` as a lens.

### File conventions

- Filenames are lowercase, hyphenated, `.md`.
- Each file starts with an H1 title and a one-line purpose statement.
- Prefer tables and checklists over long prose.
- Link related files instead of duplicating content.

### Review checklist for new content

- [ ] Scoped to a single concern.
- [ ] Includes best practices **and** common mistakes.
- [ ] Includes at least one concrete code example.
- [ ] Aligns with the standards in `standards/`.
- [ ] Cross-links related agents/skills/standards.

---

## License

Internal engineering asset. See your organization's policy before sharing externally.
