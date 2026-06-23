# Skill: Java 17

> Idiomatic, modern Java 17 for enterprise backend services.

## Best practices

- **Use records** for immutable data carriers (DTOs, value objects, events). They give you `equals`/`hashCode`/`toString` for free.
- **Use sealed classes/interfaces** to model closed hierarchies (e.g. result types, domain states) and get exhaustive `switch`.
- **Pattern matching for `instanceof`** to remove cast noise.
- **Switch expressions** (arrow form) for exhaustive, value-returning branching.
- **Text blocks** (`"""`) for SQL, JSON, and multi-line strings.
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

## Checklist

- [ ] DTOs/value objects are `record`s where appropriate.
- [ ] Public methods never return `null` (use `Optional`/empty collections).
- [ ] Closed hierarchies use `sealed` + exhaustive `switch`.
- [ ] No raw types; generics are fully parameterized.
- [ ] Collections returned from APIs are unmodifiable.
- [ ] `java.time` used for all date/time logic.
- [ ] Money uses `BigDecimal`.
- [ ] No broad `catch (Exception)` that hides failures.

## Example code

```java
// Sealed result type + records + exhaustive switch
public sealed interface PaymentResult
        permits PaymentResult.Approved, PaymentResult.Declined, PaymentResult.Error {

    record Approved(String authCode) implements PaymentResult {}
    record Declined(String reason)   implements PaymentResult {}
    record Error(String message)     implements PaymentResult {}
}

String describe(PaymentResult result) {
    return switch (result) {                      // exhaustive — no default needed
        case PaymentResult.Approved a -> "Approved: " + a.authCode();
        case PaymentResult.Declined d -> "Declined: " + d.reason();
        case PaymentResult.Error e    -> "Error: " + e.message();
    };
}
```

```java
// Pattern matching for instanceof + text block + Optional
public Optional<String> extractCity(Object payload) {
    if (payload instanceof Address addr && addr.city() != null) {
        return Optional.of(addr.city());
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
- **Result/Either over exceptions** for expected business outcomes (sealed types); reserve exceptions for the exceptional.
- **Builder via records + `with`-style copy methods** for objects with many optional fields.
- **Static factory methods** (`Order.create(...)`) over telescoping constructors.
- See [skills/spring-boot.md](spring-boot.md) for how these map onto Spring components.
