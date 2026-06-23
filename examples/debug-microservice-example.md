# Example: Debugging a Microservice in Production

> A worked example of using the SRE agent to diagnose and mitigate a live incident.

## Scenario

PagerDuty fires: **"Orders API error budget burning fast."** Users report failing checkouts. We use [agents/sre-engineer.md](../agents/sre-engineer.md) with [prompts/debug-production-issue.md](../prompts/debug-production-issue.md), [skills/observability.md](../skills/observability.md), and [skills/kubernetes.md](../skills/kubernetes.md).

## Step 1 — Triage

- **Symptom:** `POST /api/v1/orders` returning 503s; p99 latency 200ms → 4s.
- **Scope:** ~30% of order creations failing; single region; started 10 minutes ago.
- **Recent change:** Payments service deployed 12 minutes ago.
- **Severity:** SEV-2, customer-facing revenue impact, error budget burning. Page on-call lead.

## Step 2 — Mitigate first

Timing lines up with the Payments deploy. **Fastest safe action: roll back Payments** via ArgoCD while we diagnose, rather than chasing root cause live.

```bash
# Roll back to the previously synced healthy revision (GitOps)
argocd app history payments
argocd app rollback payments <previous-healthy-revision>
```

If rollback were slow, the secondary mitigation is to **open the circuit** to Payments so Orders degrades gracefully (queues the payment) instead of timing out — see the Resilience4j fallback in [skills/spring-boot.md](../skills/spring-boot.md).

## Step 3 — Confirm with data (RED + traces)

```promql
# Error rate on Orders (RED: Errors)
sum(rate(http_server_requests_seconds_count{job="orders",status=~"5.."}[5m]))
  / sum(rate(http_server_requests_seconds_count{job="orders"}[5m]))

# Duration p99 (RED: Duration) — confirm latency source
histogram_quantile(0.99,
  sum(rate(http_server_requests_seconds_bucket{job="orders"}[5m])) by (le, uri))
```

A trace for a failing request shows the span `POST orders → PaymentClient.charge` taking ~3s then timing out → confirms the dependency, not Orders itself.

## Step 4 — Root cause

The new Payments build introduced a slow DB query (missing index) under load. Orders' synchronous call to Payments hit its 3s timeout, returning 503. Because the circuit breaker threshold was too high, it never opened, so failures passed through to users.

## Step 5 — Durable fixes & follow-ups

- **Payments:** add the missing index; re-deploy with load test in the pipeline.
- **Orders resilience:** lower circuit-breaker error threshold and add a fallback so a slow Payments degrades gracefully ([standards/architecture.md](../standards/architecture.md)).
- **Detection:** add a burn-rate alert on the **Orders → Payments** dependency latency, with a runbook link ([skills/observability.md](../skills/observability.md)).
- **Process:** blameless postmortem; add a pre-deploy load/query check.

## Why this works

The SRE persona **mitigated before root-causing**, used **RED metrics + traces** to localize the failure to a dependency, and turned the incident into **durable detection and resilience improvements** — not just a one-off fix.
