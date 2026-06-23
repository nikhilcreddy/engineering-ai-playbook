# Prompt: Debug a Production Issue

> Template for structured, data-driven incident diagnosis. Replace every `{{...}}`.

## Template

```
Act as agents/sre-engineer.md.

Use:
- skills/observability.md
- skills/kubernetes.md
{{ADDITIONAL e.g. skills/spring-boot.md}}

## Symptom
{{WHAT_USERS_OR_ALERTS_REPORT e.g. "p99 latency on POST /orders spiked to 4s; 503s rising"}}

## When / scope
- Started: {{TIMESTAMP}}
- Affected: {{SERVICE(S) / endpoints / % of traffic / regions}}
- Recent changes: {{DEPLOYS, CONFIG, TRAFFIC, DEPENDENCY_CHANGES}}

## Signals available
- Metrics (RED/USE): {{WHAT_YOU_SEE_OR_LINK}}
- Logs: {{RELEVANT_LINES_OR_PATTERNS}}
- Traces: {{SLOW_SPANS_IF_KNOWN}}
- Dashboards/alerts: {{LINKS}}

## Please do
1. Triage: assess severity and blast radius; is the error budget burning?
2. Mitigation first: propose the fastest safe action to stop user impact
   (rollback, scale, feature-flag, circuit-break, drain).
3. Hypotheses: rank likely root causes given the signals; state what evidence
   would confirm/refute each.
4. Diagnosis plan: exact metrics/queries/traces to inspect next, in order.
5. Root cause (once identified) and a durable fix.
6. Follow-ups: what to add (alert, dashboard, test) so we detect this faster.

Stabilize first, root-cause second. Be concrete about the commands/queries.
```

## Tips

- Lead with **mitigation** — restore service before chasing root cause.
- Always include **recent changes**; most incidents correlate with a deploy/config change.
- Provide **real signals** (metrics/logs/traces); diagnosis quality tracks data quality.
- Capture follow-ups for a **blameless postmortem** — fix the system, not the person.
- See [examples/debug-microservice-example.md](../examples/debug-microservice-example.md).
