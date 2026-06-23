# Standard: Architecture

> Architectural guardrails for microservices on Java 17 + Spring Boot.

## Principles

1. **Boundaries follow business capabilities and data ownership** — not org charts or convenience.
2. **One service owns its data.** No shared databases across service boundaries.
3. **Design for failure as the default.** Every remote call can time out, retry, or duplicate.
4. **Evolutionary architecture.** Optimize for change; avoid irreversible early decisions.
5. **Explicit over implicit.** Latency budgets, SLOs, and consistency models are written down.
6. **Boring beats clever on core paths.** Reserve novelty for true differentiators.

## Service design

- Each service is independently deployable, owns its schema, and exposes a versioned contract.
- Keep services **cohesive** (one capability) and **loosely coupled** (stable contracts, no shared internals).
- Stateless application tier; state lives in PostgreSQL/queues. Enables horizontal scaling on Kubernetes.
- Use a layered/hexagonal internal structure: domain core isolated from framework and I/O.

## Communication

| Need                              | Use                                  |
| --------------------------------- | ------------------------------------ |
| Synchronous query, need an answer | REST or GraphQL                      |
| State propagation / decoupling    | Events (async messaging)             |
| Aggregating many sources for UI   | GraphQL gateway / BFF                |

- Prefer **asynchronous events** for cross-service state changes to reduce coupling and improve resilience.
- Define **idempotency** for all consumers; events can be redelivered.
- No synchronous call chains more than a couple hops deep on a user-facing path.

## Data & consistency

- **Strong consistency within a service**; **eventual consistency across services**.
- Use the **Saga pattern** (with compensating actions) for multi-service transactions — no distributed 2PC.
- **Outbox pattern** for reliable event publishing alongside DB writes.
- Each service migrates its own schema (e.g. Flyway/Liquibase).

## Resilience

- Timeouts on every outbound call; retries with exponential backoff **and jitter**; circuit breakers.
- Bulkheads to isolate failures; graceful degradation over hard failure.
- Design the **blast radius** of every dependency failure.

## Cross-cutting requirements (non-negotiable)

- **Observability** built in from day one — see [skills/observability.md](../skills/observability.md).
- **Security** by default — see [standards/security.md](security.md).
- **Operability** on Kubernetes with GitOps (ArgoCD) — see [skills/kubernetes.md](../skills/kubernetes.md).

## Decision records (ADRs)

Every significant decision is captured as an ADR with:

- **Context** — forces and constraints.
- **Options considered** — with trade-offs.
- **Decision** — what we chose.
- **Consequences** — positive and negative, including what becomes harder.

## Checklist for a new service / major change

- [ ] Clear capability boundary and owned data.
- [ ] Versioned, backward-compatible contract designed first.
- [ ] Sync vs. async integration chosen deliberately.
- [ ] Consistency model defined (saga/outbox where needed).
- [ ] Resilience: timeouts, retries+jitter, circuit breakers, degradation.
- [ ] Observability, security, and K8s operability addressed.
- [ ] ADR written with options and consequences.

Owned by [agents/software-architect.md](../agents/software-architect.md); see [skills/api-design.md](../skills/api-design.md).
