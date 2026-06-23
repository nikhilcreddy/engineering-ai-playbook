---
name: java-spring-backend
description: 'Build backend services in Java with Spring/Spring Boot and JUnit tests using solid engineering practices. Use when writing or reviewing Java backend code, designing REST controllers/services/repositories, structuring Spring Boot apps, dependency injection, exception handling, or writing JUnit/Mockito unit and integration tests.'
---

# Java Spring Boot Backend Development

## When to Use

- Writing or reviewing Java backend code (services, controllers, repositories).
- Designing or modifying Spring / Spring Boot applications.
- Writing JUnit 5 / Mockito unit or integration tests.
- Refactoring backend code for testability, layering, or clarity.

## Architecture & Layering

Keep a clear separation of concerns:

- **Controller** — HTTP only. Validate input, map DTOs, delegate to services. No business logic.
- **Service** — business logic. Stateless, transactional boundaries (`@Transactional`).
- **Repository** — data access only (Spring Data JPA interfaces).
- **DTO vs Entity** — never expose JPA entities directly over the API; map to DTOs/records.

Use packages by feature (e.g. `com.app.order`) rather than by layer when the project grows.

## Spring Conventions

- **Constructor injection only** — `final` fields, no `@Autowired` on fields. Enables easy testing and immutability.
- Use `@RestController`, `@Service`, `@Repository`, `@Configuration` deliberately.
- Externalize config via `application.yml` and `@ConfigurationProperties` (typed) over scattered `@Value`.
- Prefer `record` for immutable DTOs and request/response models.
- Use Bean Validation (`@Valid`, `@NotNull`, `@Size`, etc.) at the controller boundary.

## Error Handling

- Centralize with `@RestControllerAdvice` + `@ExceptionHandler`; return a consistent error body.
- Throw meaningful domain exceptions; never swallow exceptions or `catch (Exception)` blindly.
- Map errors to correct HTTP status codes (400 validation, 404 not found, 409 conflict, 500 unexpected).

## REST API Guidelines

- Plural nouns for resources (`/api/orders`), proper verbs (GET/POST/PUT/PATCH/DELETE).
- Return `ResponseEntity<T>` with explicit status; use 201 + `Location` on create.
- Paginate list endpoints (`Pageable`). Never return unbounded collections.

## Persistence

- Use Spring Data JPA repositories; derive queries or `@Query` for complex cases.
- Avoid N+1: use fetch joins or entity graphs intentionally.
- Manage transactions at the service layer, not the repository.
- Use Flyway/Liquibase for schema migrations; don't rely on `ddl-auto` in production.

## Testing (JUnit 5 + Mockito)

Aim for fast, deterministic, isolated tests covering happy paths **and** edge/failure cases.

- **Unit tests** — `@ExtendWith(MockitoExtension.class)`, mock collaborators with `@Mock`,
  inject with `@InjectMocks`. No Spring context. Test one class.
- **Web slice** — `@WebMvcTest(Controller.class)` + `MockMvc` to test controllers in isolation.
- **Data slice** — `@DataJpaTest` for repositories against an in-memory/Testcontainers DB.
- **Integration** — `@SpringBootTest` only when you genuinely need the full context.
- Use **AssertJ** (`assertThat(...)`) for readable assertions.
- Name tests by behavior: `methodName_givenCondition_expectedResult`.
- Follow **Arrange-Act-Assert**; one logical assertion focus per test.
- Verify interactions with `verify(...)` where behavior (not just state) matters.
- Prefer **Testcontainers** over H2 when DB fidelity matters.

## General Quality Bar

- Small, single-responsibility methods; meaningful names; no dead code.
- Validate inputs at boundaries; never trust external data (OWASP).
- No secrets in code or config committed to git; use environment/secret managers.
- Log meaningfully (no sensitive data); use SLF4J, not `System.out`.
- Keep diffs minimal and follow the existing project's conventions first.

## Quick Checklist Before Done

1. New behavior has JUnit tests (happy + edge + failure).
2. Constructor injection, DTOs at the boundary, no entity leakage.
3. Errors handled centrally with correct status codes.
4. Inputs validated; no secrets committed.
5. Build and tests pass (`./mvnw test` or `./gradlew test`).
