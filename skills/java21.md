# Skill: Java 21

> Idiomatic, modern Java 21 (LTS) for enterprise backend services.

## Best practices

- **Use records** for immutable data carriers (DTOs, value objects, events). They give you `equals`/`hashCode`/`toString` for free.
- **Use sealed classes/interfaces** to model closed hierarchies (e.g. result types, domain states) and get exhaustive `switch`.
- **Pattern matching for `switch`** (finalized in 21) to branch on types with exhaustiveness checking — no `default` needed for sealed hierarchies.
- **Record patterns** (finalized in 21) to deconstruct records directly in `switch`/`instanceof`, including nested destructuring.
- **Pattern matching for `instanceof`** to remove cast noise.
- **Virtual threads** (finalized in 21) for high-throughput, blocking I/O workloads — write simple blocking code that scales like async.
- **Text blocks** (`"""`) for SQL, JSON, and multi-line strings.
- **Sequenced collections** (`SequencedCollection`/`SequencedMap`) for clear first/last access (`getFirst()`, `getLast()`, `reversed()`).
- **`var`** for local variables when the type is obvious from the right-hand side; not for public APIs.
- **`Optional`** as a return type for "maybe absent"; never as a field or method parameter.
- **Streams** for transformation pipelines, but prefer a plain loop when it's clearer or hot-path.
- **Immutability by default**: `final` fields, unmodifiable collections (`List.of`, `Map.of`, `.toList()`).
- **`java.time`** (`Instant`, `LocalDate`, `Duration`) — never `Date`/`Calendar`.

## Common mistakes

- ❌ Returning `null` instead of `Optional` or an empty collection.
- ❌ Using `Optional` for fields or parameters (it's a return-type tool).
- ❌ Mutable records by hiding mutable collections inside them (defensively copy).
- ❌ Overusing streams for side effects (`forEach` with mutation) — use a loop.
- ❌ Catching `Exception`/`Throwable` broadly and swallowing it.
- ❌ `==` for object/string equality; use `.equals()` / `Objects.equals()`.
- ❌ Ignoring `InterruptedException` — restore the interrupt flag.
- ❌ Floating-point (`double`) for money — use `BigDecimal`.
- ❌ Pooling virtual threads or pinning them with `synchronized` around blocking I/O — create one per task; prefer `ReentrantLock`.
- ❌ Adding a `default` branch to a `switch` over a sealed type — it defeats exhaustiveness checks for new subtypes.

## Checklist

- [ ] DTOs/value objects are `record`s where appropriate.
- [ ] Public methods never return `null` (use `Optional`/empty collections).
- [ ] Closed hierarchies use `sealed` + exhaustive `switch` (no `default`).
- [ ] Record/type patterns used to deconstruct instead of manual casting/getters.
- [ ] Blocking I/O tasks run on virtual threads; no virtual-thread pooling or pinning.
- [ ] No raw types; generics are fully parameterized.
- [ ] Collections returned from APIs are unmodifiable.
- [ ] `java.time` used for all date/time logic.
- [ ] Money uses `BigDecimal`.
- [ ] No broad `catch (Exception)` that hides failures.

## Example code

```java
// Sealed result type + records + exhaustive switch with record patterns
public sealed interface PaymentResult
        permits PaymentResult.Approved, PaymentResult.Declined, PaymentResult.Error {

    record Approved(String authCode) implements PaymentResult {}
    record Declined(String reason)   implements PaymentResult {}
    record Error(String message)     implements PaymentResult {}
}

String describe(PaymentResult result) {
    return switch (result) {                      // exhaustive — no default needed
        case PaymentResult.Approved(var authCode) -> "Approved: " + authCode;
        case PaymentResult.Declined(var reason)   -> "Declined: " + reason;
        case PaymentResult.Error(var message)     -> "Error: " + message;
    };
}
```

```java
// Nested record patterns + guarded patterns (Java 21)
record Address(String city, String country) {}
record Customer(String id, Address address) {}

String shippingZone(Object payload) {
    return switch (payload) {
        case Customer(var id, Address(var city, var country))
                when "US".equals(country)        -> "domestic:" + city;
        case Customer(var id, Address(var city, var country)) -> "intl:" + country;
        case null                                -> "unknown";
        default                                  -> "unsupported";
    };
}
```

```java
// Virtual threads (Java 21) — simple blocking code that scales
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<Quote>> futures = supplierIds.stream()
        .map(id -> executor.submit(() -> quoteClient.fetch(id)))  // blocking call per task
        .toList();
    for (var future : futures) {
        process(future.get());
    }
}   // one virtual thread per task; cheap to create by the thousands
```

```java
// Pattern matching for instanceof + text block + Optional
public Optional<String> extractCity(Object payload) {
    if (payload instanceof Address(var city, var country) && city != null) {
        return Optional.of(city);
    }
    return Optional.empty();
}

static final String FIND_ACTIVE = """
    SELECT id, email
    FROM   customer
    WHERE  status = 'ACTIVE'
    ORDER  BY created_at DESC
    """;
```

```java
// Immutable record with defensive copy
public record Order(String id, List<Line> lines) {
    public Order {                                 // compact constructor
        lines = List.copyOf(lines);                // defensive, unmodifiable
    }
}
```

## Recommended patterns

- **Value objects over primitives** — wrap `String customerId` in a `CustomerId` record to avoid mixing up identifiers.
- **Record patterns + sealed types** for exhaustive, destructuring `switch` over domain states — the compiler enforces handling every case.
- **Result/Either over exceptions** for expected business outcomes (sealed types); reserve exceptions for the exceptional.
- **Virtual threads** for I/O-bound fan-out (calling many services/DB queries) — keep code blocking and readable instead of reactive.
- **Static factory methods** (`Order.create(...)`) over telescoping constructors.
- See [skills/spring-boot.md](spring-boot.md) for how these map onto Spring components (incl. enabling virtual threads via `spring.threads.virtual.enabled`).
