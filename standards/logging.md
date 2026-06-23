# Standard: Logging

> Structured, correlated, safe logging for production microservices.

## Principles

1. **Logs are for humans debugging incidents** — make them searchable and contextual.
2. **Structured over free-text.** Emit JSON with consistent fields, not interpolated prose.
3. **Correlate everything.** Every log line carries a trace/correlation id.
4. **Never log secrets or PII.** Logs are widely accessible; treat them as untrusted storage.
5. **Log the "why," metric the "how many."** Use metrics for aggregation/alerting (see [skills/observability.md](../skills/observability.md)).

## Levels — when to use

| Level   | Use for                                                                 |
| ------- | ----------------------------------------------------------------------- |
| `ERROR` | A request/operation failed and needs attention; include cause + context |
| `WARN`  | Recoverable/abnormal condition (retry, fallback, degraded mode)         |
| `INFO`  | Significant business events (order created, payment captured)           |
| `DEBUG` | Developer detail for diagnosing in non-prod (off by default in prod)    |
| `TRACE` | Very fine-grained; rarely enabled                                       |

## Rules

- **Structured fields**, not string concatenation: log key/value pairs.
- **Always include**: `traceId`, `spanId`, service name, and relevant business ids (e.g. `orderId`).
- **Never log**: passwords, tokens, full card numbers, secrets, full PII. Mask/redact (`****1234`).
- **Log exceptions with the throwable**, not `e.getMessage()` only — preserve the stack trace.
- **No logging in tight loops** / hot paths at INFO+.
- **One event, one log line.** Don't split a single event across multiple lines.
- **Put trace id in MDC** so every line in the request is correlated automatically.

## ✅ / ❌ Examples

```java
// ❌ Unstructured, leaks PII, loses stack trace
log.info("User " + email + " with card " + cardNumber + " failed: " + e.getMessage());
```

```java
// ✅ Structured, correlated, redacted, full stack trace
log.error("payment_failed orderId={} customerId={} reason={}",
          orderId, customerId, result.reason(), exception);
// email/card omitted; traceId/spanId added automatically via MDC
```

```java
// Correlation id into MDC (e.g. in a filter)
MDC.put("traceId", traceId);
try {
    chain.doFilter(request, response);
} finally {
    MDC.clear();
}
```

```yaml
# logback / application.yml — JSON output with MDC fields
logging:
  pattern:
    level: "%5p [traceId=%X{traceId} spanId=%X{spanId}]"
# Prefer a JSON encoder (logstash-logback-encoder) in production.
```

## Checklist

- [ ] Logs are structured (JSON) in production.
- [ ] Every line carries `traceId`/`spanId` via MDC.
- [ ] No secrets or PII in any log statement.
- [ ] Exceptions logged with the throwable (stack trace preserved).
- [ ] Correct level chosen (no INFO spam, no swallowed ERRORs).
- [ ] Business-significant events logged at INFO.
- [ ] No logging in hot loops.

Enforced by [agents/code-reviewer.md](../agents/code-reviewer.md) and [agents/security-engineer.md](../agents/security-engineer.md).
