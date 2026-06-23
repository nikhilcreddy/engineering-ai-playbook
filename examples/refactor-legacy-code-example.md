# Example: Refactoring Legacy Code

> A worked example of safely refactoring a legacy Spring service using the playbook.

## Scenario

A legacy `OrderService` mixes HTTP concerns, business logic, and persistence in one fat method. It has no tests, an N+1 query, and swallows exceptions. We refactor it with [agents/backend-engineer.md](../agents/backend-engineer.md), guided by [skills/spring-boot.md](../skills/spring-boot.md), [skills/java21.md](../skills/java21.md), and [standards/clean-code.md](../standards/clean-code.md).

## The legacy code

```java
@RestController
public class OrderService {

    @Autowired EntityManager em;

    @PostMapping("/orders")
    public Object process(@RequestBody Map<String, Object> body) {
        try {
            Order o = new Order();
            o.setCustomerId(Long.valueOf(body.get("customerId").toString()));
            em.persist(o);
            List<Order> all = em.createQuery("from Order").getResultList();
            for (Order x : all) {
                x.getCustomer().getName();          // N+1: lazy load per row
            }
            return o;
        } catch (Exception e) {
            return null;                            // swallowed error
        }
    }
}
```

## Step 1 — Characterize before changing

Refactoring without tests is gambling. First add a **characterization test** (current behavior) using Testcontainers so we can refactor safely ([skills/testing.md](../skills/testing.md)).

```java
@SpringBootTest @Testcontainers
class OrderCreationCharacterizationTest {
    @Container static PostgreSQLContainer<?> db = new PostgreSQLContainer<>("postgres:16-alpine");
    // ...wiring...
    @Test void create_persistsOrder_forValidCustomer() { /* lock in behavior */ }
}
```

## Step 2 — Separate the layers

Split the one class into controller → service → repository, each with one responsibility.

```java
@RestController
@RequestMapping("/api/v1/orders")
class OrderController {
    private final OrderCreator orderCreator;
    OrderController(OrderCreator orderCreator) { this.orderCreator = orderCreator; }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    OrderResponse create(@Valid @RequestBody CreateOrderRequest request) {
        return orderCreator.create(request);
    }
}
```

```java
@Service
class OrderCreator {
    private final OrderRepository orders;
    OrderCreator(OrderRepository orders) { this.orders = orders; }

    @Transactional
    OrderResponse create(CreateOrderRequest req) {
        var saved = orders.save(OrderEntity.from(req));   // validated DTO, no Map
        return OrderResponse.from(saved);                 // DTO out, not entity
    }
}
```

## Step 3 — Fix the specific defects

- **Typed input:** `Map<String,Object>` → validated `CreateOrderRequest` record ([skills/java21.md](../skills/java21.md)).
- **No entity exposure:** return `OrderResponse` DTO.
- **N+1 removed:** the gratuitous "load all orders" loop is deleted; where such a join is genuinely needed, use `@EntityGraph`/fetch join.
- **No swallowed errors:** remove the `catch (Exception) → null`; let a global `@RestControllerAdvice` map failures to RFC 7807 `ProblemDetail`.
- **Constructor injection** + `final` fields throughout.

## Step 4 — Verify

Run the characterization test (still green) plus new unit/slice tests for validation and not-found. Enable SQL logging in the integration test to **prove the N+1 is gone**.

## Refactoring principles used

1. **Tests first** — characterize legacy behavior before touching it.
2. **Small, behavior-preserving steps** — separate layers, then fix defects, committing between.
3. **One concern per change** — don't add features while refactoring.
4. **Leave it cleaner** — typed inputs, DTO boundaries, real error handling, no N+1.

The result is reviewable with [agents/code-reviewer.md](../agents/code-reviewer.md) and meets [standards/clean-code.md](../standards/clean-code.md) and [standards/security.md](../standards/security.md).
