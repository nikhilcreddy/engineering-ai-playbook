# Skill: Testing

> A pragmatic testing strategy for Spring Boot microservices.

## Best practices

- **Test behavior, not implementation.** Assert observable outcomes, not internal calls (mostly).
- **Follow the test pyramid:** many fast unit tests, fewer slice/integration tests, very few end-to-end.
- **Arrange–Act–Assert** structure; one logical assertion focus per test.
- **Name tests by behavior:** `create_returnsConflict_whenIdempotencyKeyReused`.
- **Use Testcontainers** for integration tests against real PostgreSQL/Kafka — not H2.
- **Slice tests** (`@WebMvcTest`, `@DataJpaTest`, `@GraphQlTest`) to test one layer fast.
- **Deterministic tests:** no real time, randomness, or network; inject `Clock`, fix seeds.
- **Test the edges:** null/empty, boundaries, concurrency, failure paths, idempotency.
- **Keep tests independent** — no shared mutable state, no ordering dependencies.

## Common mistakes

- ❌ Mocking everything, so tests pass while the system is broken (over-mocking).
- ❌ Asserting on logs or private methods.
- ❌ Using H2 to emulate PostgreSQL — different SQL/behavior hides bugs.
- ❌ Flaky tests from `Thread.sleep`, real clocks, or shared state — quarantine and fix, don't ignore.
- ❌ Testing only the happy path.
- ❌ Giant integration tests that boot the whole app for trivial logic.
- ❌ Chasing line coverage instead of meaningful behavior coverage.

## Checklist

- [ ] Unit tests cover business logic and edge cases.
- [ ] Slice tests cover web/persistence/GraphQL layers.
- [ ] At least one integration test with Testcontainers for the real DB.
- [ ] Failure paths (timeouts, exceptions, validation) are tested.
- [ ] Idempotency and concurrency tested for mutating endpoints.
- [ ] Tests are deterministic (injected `Clock`, no sleeps).
- [ ] No test depends on another's execution order.

## Example code

```java
// Unit test — pure logic, no Spring context
class OrderPricingTest {

    @Test
    void total_includesTaxAndShipping() {
        var pricing = new OrderPricing(new BigDecimal("0.10"));   // 10% tax
        var total = pricing.total(new BigDecimal("100.00"), new BigDecimal("5.00"));
        assertThat(total).isEqualByComparingTo("115.00");
    }
}
```

```java
// Slice test — web layer only
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired MockMvc mvc;
    @MockBean OrderService orderService;

    @Test
    void create_returns400_whenBodyInvalid() throws Exception {
        mvc.perform(post("/api/v1/orders")
                .header("Idempotency-Key", "k1")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{}"))
            .andExpect(status().isBadRequest());
    }
}
```

```java
// Integration test — real PostgreSQL via Testcontainers
@SpringBootTest
@Testcontainers
class OrderIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url", postgres::getJdbcUrl);
        r.add("spring.datasource.username", postgres::getUsername);
        r.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired OrderService orderService;

    @Test
    void create_isIdempotent_forSameKey() {
        var req = new CreateOrderRequest(/* ... */);
        var first = orderService.create(req, "key-1");
        var second = orderService.create(req, "key-1");
        assertThat(second.id()).isEqualTo(first.id());   // no duplicate
    }
}
```

## Recommended patterns

- **Inject `Clock`** so time-based logic is testable and deterministic.
- **Object Mother / test data builders** for readable fixtures.
- **`@DataJpaTest` + Testcontainers** to verify queries (catch N+1 with SQL logging).
- **Contract tests** (e.g. Spring Cloud Contract) between services to prevent integration drift.
- **Mutation testing** (PIT) periodically to validate test effectiveness.
- Used heavily by [agents/backend-engineer.md](../agents/backend-engineer.md) and [agents/code-reviewer.md](../agents/code-reviewer.md).
