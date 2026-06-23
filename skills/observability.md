# Skill: Observability

> Metrics, logs, and traces that make microservices operable in production.

## Best practices

- **Three pillars, used deliberately:** metrics (aggregate health/alerting), logs (discrete events/context), traces (request flow across services).
- **Instrument with Micrometer** → expose Prometheus metrics via Actuator (`/actuator/prometheus`).
- **Follow RED** for request-driven services: **R**ate, **E**rrors, **D**uration. Follow **USE** (Utilization, Saturation, Errors) for resources.
- **Structured (JSON) logs** with a correlation/trace ID on every line — see [standards/logging.md](../standards/logging.md).
- **Propagate trace context** across service calls (W3C `traceparent`) via Micrometer Tracing / OpenTelemetry.
- **Alert on SLOs/symptoms**, not raw causes. Use multi-window burn-rate alerts.
- **Dashboards tell a story:** top row = user-facing SLOs, then dependencies, then resources.
- **Label cardinality discipline** — never put unbounded values (user IDs, emails) in metric labels.

## Common mistakes

- ❌ Logging instead of metrics for things you need to alert/aggregate on.
- ❌ High-cardinality metric labels (user id, request id) → Prometheus blowup.
- ❌ No correlation/trace ID, so logs can't be tied to a request across services.
- ❌ Alerting on CPU instead of user-facing symptoms → noisy, non-actionable pages.
- ❌ Measuring averages instead of percentiles (p95/p99) for latency.
- ❌ Traces that stop at the service boundary (context not propagated).
- ❌ Dashboards full of graphs no one can act on.

## Checklist

- [ ] `/actuator/prometheus` exposed and scraped.
- [ ] RED metrics present for every endpoint (rate, errors, p95/p99 duration).
- [ ] USE metrics for DB pools, thread pools, queues.
- [ ] Every log line carries `traceId`/`spanId` and key business identifiers.
- [ ] Trace context propagated on all outbound calls.
- [ ] SLOs defined; burn-rate alerts wired (not raw-resource alerts).
- [ ] Metric labels are bounded/low-cardinality.
- [ ] Each alert links to a runbook.

## Example code

```java
// Business + latency metrics with Micrometer
@Service
class OrderService {

    private final Counter ordersCreated;
    private final Timer createTimer;

    OrderService(MeterRegistry registry) {
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total orders created")
            .register(registry);
        this.createTimer = Timer.builder("orders.create.duration")
            .publishPercentiles(0.95, 0.99)
            .register(registry);
    }

    OrderResponse create(CreateOrderRequest req) {
        return createTimer.record(() -> {
            var saved = doCreate(req);
            ordersCreated.increment();
            return saved;
        });
    }
}
```

```yaml
# application.yml — observability wiring
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true
  tracing:
    sampling:
      probability: 0.1            # sample 10% in high-traffic prod
```

```yaml
# Prometheus alert — SLO burn rate (fast + slow window)
groups:
  - name: orders-slo
    rules:
      - alert: OrdersHighErrorBurnRate
        expr: |
          (
            sum(rate(http_server_requests_seconds_count{job="orders",status=~"5.."}[5m]))
            / sum(rate(http_server_requests_seconds_count{job="orders"}[5m]))
          ) > 0.02
        for: 5m
        labels: { severity: page }
        annotations:
          summary: "Orders API error budget burning fast"
          runbook: "https://runbooks/orders/high-error-rate"
```

## Recommended patterns

- **RED dashboard per service** in Grafana, with SLO panels on top.
- **Exemplars** linking metrics → traces for fast drill-down.
- **OpenTelemetry** as the vendor-neutral instrumentation layer.
- **Correlation ID filter** that sets `traceId` into MDC for all logs.
- Owned primarily by [agents/sre-engineer.md](../agents/sre-engineer.md); informs [skills/kubernetes.md](kubernetes.md) probes and rollout health checks.
