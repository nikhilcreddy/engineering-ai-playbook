# Skill: Spring Boot

> Building robust, observable, testable Spring Boot microservices on Java 17.

## Best practices

- **Constructor injection only.** Make dependencies `private final`; never use field injection (`@Autowired` on fields).
- **Thin controllers, rich services.** Controllers map HTTP ↔ DTO; services hold business logic; repositories hold persistence.
- **DTOs at the boundary.** Never serialize JPA entities directly — avoids leaking schema and lazy-loading surprises.
- **Validate at the edge** with Jakarta Bean Validation (`@Valid`, `@NotNull`, `@Size`, custom validators).
- **Centralize error handling** with `@RestControllerAdvice` returning RFC 7807 `ProblemDetail`.
- **Externalize config** via `@ConfigurationProperties` (typed) over scattered `@Value`.
- **Explicit transactions.** `@Transactional` on service methods; keep them short; understand read-only vs. write.
- **Resilience** for outbound calls: Resilience4j circuit breaker + timeout + retry with backoff.
- **Profiles** for environment differences; never hardcode environment specifics.
- **Actuator + Micrometer** for health, metrics, and Prometheus scraping.

## Common mistakes

- ❌ Field injection — hides dependencies, breaks immutability, hard to test.
- ❌ Returning entities from controllers → `LazyInitializationException`, over-exposure.
- ❌ Business logic in controllers; fat controllers, untestable.
- ❌ `@Transactional` on private/self-invoked methods (proxy won't apply it).
- ❌ N+1 queries from lazy associations in loops — use fetch joins or `@EntityGraph`.
- ❌ Catching and rethrowing as generic `RuntimeException`, losing context.
- ❌ Blocking external calls with no timeout — one slow dependency stalls the pool.
- ❌ Putting secrets in `application.yml` committed to git.
- ❌ Using `RestTemplate` for new code — prefer `WebClient` or `RestClient`.

## Checklist

- [ ] All dependencies constructor-injected and `final`.
- [ ] Controllers return DTOs, not entities.
- [ ] Request bodies validated with `@Valid`.
- [ ] Global `@RestControllerAdvice` maps exceptions to `ProblemDetail`.
- [ ] `@Transactional` only on public service methods; read-only flagged.
- [ ] Outbound calls have timeouts + circuit breaker.
- [ ] No N+1 — verified with SQL logging in a test.
- [ ] Actuator health/metrics exposed; readiness/liveness mapped.
- [ ] Config via `@ConfigurationProperties`; secrets externalized.

## Example code

```java
@RestController
@RequestMapping("/api/v1/orders")
class OrderController {

    private final OrderService orderService;

    OrderController(OrderService orderService) {        // constructor injection
        this.orderService = orderService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    OrderResponse create(@Valid @RequestBody CreateOrderRequest request,
                         @RequestHeader("Idempotency-Key") String idempotencyKey) {
        return orderService.create(request, idempotencyKey);
    }

    @GetMapping("/{id}")
    OrderResponse getById(@PathVariable UUID id) {
        return orderService.getById(id);
    }
}
```

```java
@Service
class OrderService {

    private final OrderRepository orders;

    OrderService(OrderRepository orders) {
        this.orders = orders;
    }

    @Transactional
    OrderResponse create(CreateOrderRequest req, String idempotencyKey) {
        return orders.findByIdempotencyKey(idempotencyKey)
            .map(OrderResponse::from)                    // idempotent replay
            .orElseGet(() -> {
                var order = OrderEntity.from(req, idempotencyKey);
                return OrderResponse.from(orders.save(order));
            });
    }

    @Transactional(readOnly = true)
    OrderResponse getById(UUID id) {
        return orders.findById(id)
            .map(OrderResponse::from)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }
}
```

```java
// Global error handling with ProblemDetail (RFC 7807)
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    ProblemDetail handleNotFound(OrderNotFoundException ex) {
        var problem = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        problem.setTitle("Order not found");
        problem.setDetail(ex.getMessage());
        return problem;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        var problem = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        problem.setTitle("Validation failed");
        problem.setDetail(ex.getBindingResult().getAllErrors().size() + " field error(s)");
        return problem;
    }
}
```

```java
// Resilient outbound call
@Service
class PaymentClient {

    private final RestClient restClient;

    PaymentClient(RestClient.Builder builder) {
        this.restClient = builder.baseUrl("https://payments.internal").build();
    }

    @CircuitBreaker(name = "payments", fallbackMethod = "fallback")
    @Retry(name = "payments")
    @TimeLimiter(name = "payments")
    PaymentResult charge(ChargeRequest req) {
        return restClient.post().uri("/charges").body(req)
            .retrieve().body(PaymentResult.class);
    }

    PaymentResult fallback(ChargeRequest req, Throwable t) {
        return new PaymentResult.Error("payment service unavailable");
    }
}
```

## Recommended patterns

- **Hexagonal/ports-and-adapters** for non-trivial services: domain at the center, Spring at the edges.
- **`@ConfigurationProperties` records** for typed, immutable config.
- **`@EntityGraph` / fetch joins** to eliminate N+1.
- **Testcontainers** for integration tests against real PostgreSQL — see [skills/testing.md](testing.md).
- **Micrometer `@Timed` / counters** for business metrics — see [skills/observability.md](observability.md).
- Apply [standards/clean-code.md](../standards/clean-code.md) and [standards/security.md](../standards/security.md) throughout.
