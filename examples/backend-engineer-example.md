# Example: Using the Backend Engineer Agent

> A worked example of combining a persona + skills + a prompt template to implement a feature.

## Scenario

Add a `POST /api/v1/orders` endpoint that creates an order, persists it to PostgreSQL, and is safe to retry.

## Step 1 — Assemble the request

We combine:

- **Persona:** [agents/backend-engineer.md](../agents/backend-engineer.md)
- **Skills:** [skills/spring-boot.md](../skills/spring-boot.md), [skills/testing.md](../skills/testing.md)
- **Template:** [prompts/implement-feature.md](../prompts/implement-feature.md)
- **Standards (acceptance criteria):** [standards/clean-code.md](../standards/clean-code.md), [standards/security.md](../standards/security.md)

## Step 2 — The filled-in prompt

```
Act as agents/backend-engineer.md.
Use skills/spring-boot.md and skills/testing.md.
Enforce standards/clean-code.md and standards/security.md.

## Feature
Create an order for an authenticated customer and persist it.

## Requirements
- API: POST /api/v1/orders
- Validation: at least one line item; quantity >= 1; known productId.
- Idempotency: YES — via Idempotency-Key header; replays return the original order.
- Persistence: PostgreSQL via JPA.
- Observability: counter orders.created; INFO log "order_created orderId=...".

## Deliverables
Implementation, validation + ProblemDetail errors, unit + Testcontainers tests,
metrics + logs, and a short note on design decisions.
```

## Step 3 — What good output looks like

```java
@RestController
@RequestMapping("/api/v1/orders")
class OrderController {

    private final OrderService orderService;

    OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    OrderResponse create(@Valid @RequestBody CreateOrderRequest request,
                         @RequestHeader("Idempotency-Key") String key) {
        return orderService.create(request, key);
    }
}
```

```java
@Service
class OrderService {

    private final OrderRepository orders;
    private final Counter created;

    OrderService(OrderRepository orders, MeterRegistry registry) {
        this.orders = orders;
        this.created = Counter.builder("orders.created").register(registry);
    }

    @Transactional
    OrderResponse create(CreateOrderRequest req, String idempotencyKey) {
        return orders.findByIdempotencyKey(idempotencyKey)
            .map(OrderResponse::from)                       // idempotent replay
            .orElseGet(() -> {
                var saved = orders.save(OrderEntity.from(req, idempotencyKey));
                created.increment();
                log.info("order_created orderId={}", saved.getId());
                return OrderResponse.from(saved);
            });
    }
}
```

```java
@SpringBootTest
@Testcontainers
class OrderIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url", postgres::getJdbcUrl);
        r.add("spring.datasource.username", postgres::getUsername);
        r.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired OrderService service;

    @Test
    void create_isIdempotent_forSameKey() {
        var req = aValidRequest();
        assertThat(service.create(req, "k1").id())
            .isEqualTo(service.create(req, "k1").id());     // no duplicate
    }
}
```

## Why this works

- The **persona** enforced constructor injection, thin controller, DTOs, and observability.
- The **skills** supplied idiomatic Spring + Testcontainers patterns.
- The **standards** became acceptance criteria, so security (validation) and clean-code were not optional.
- The **idempotency requirement** drove the core design — exactly what we want surfaced early.
