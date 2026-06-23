# Standard: Security

> Secure-by-default requirements for backend services. Aligned to the OWASP Top 10.

## Principles

1. **Secure by default; insecure by documented exception.**
2. **Never trust input** — validate and encode at every boundary (allow-list first).
3. **Least privilege** for users, services, tokens, DB accounts, and network paths.
4. **Defense in depth** — assume any single control can fail.
5. **Fail closed** — deny on auth/validation errors; never fall through to allow.
6. **No secrets in code, logs, images, or config repos. Ever.**

## OWASP Top 10 — required controls

| Risk                          | Required control                                                                 |
| ----------------------------- | -------------------------------------------------------------------------------- |
| Broken Access Control         | Enforce object-level authz on every request; deny by default                     |
| Cryptographic Failures        | TLS in transit; encrypt sensitive data at rest; strong, current algorithms       |
| Injection                     | Parameterized queries only; never concatenate SQL/JPQL; validate + encode output |
| Insecure Design               | Threat-model significant changes; abuse cases in acceptance criteria             |
| Security Misconfiguration     | Harden defaults; disable debug/actuator-sensitive endpoints in prod              |
| Vulnerable Components         | SCA in CI; no known-critical CVEs; patch promptly                                |
| Identification/Auth Failures  | Validate JWT signature/issuer/audience/expiry; strong session handling           |
| Software/Data Integrity       | Signed artifacts; constrain deserialization; verify supply chain                 |
| Logging/Monitoring Failures   | Log security events; alert on auth anomalies; never log secrets                  |
| SSRF                          | Allow-list outbound URLs; block internal metadata endpoints                      |

## Authentication & authorization

- Use OAuth2/OIDC; validate JWTs fully (signature, `iss`, `aud`, `exp`, `nbf`).
- Enforce **object-level authorization** — verify the caller may act on _this_ resource, not just that they're logged in.
- Method/endpoint authorization via `@PreAuthorize` / security config; deny by default.
- Short-lived tokens; rotate and revoke; never log tokens.

## Input handling

- Validate with Jakarta Bean Validation at the controller boundary.
- Parameterized queries (JPA/`PreparedStatement`) — **no string-built SQL/JPQL**.
- Constrain file uploads (type, size) and outbound URL fetches (SSRF allow-list).
- Avoid native Java serialization; constrain Jackson polymorphic typing.

## Secrets & config

- Secrets via Kubernetes Secrets / vault, injected at runtime; never in `application.yml` or images.
- Separate credentials per environment; least-privilege DB users.
- Disable verbose error/debug output and sensitive actuator endpoints in production.

## ✅ / ❌ Examples

```java
// ❌ SQL injection
var sql = "SELECT * FROM orders WHERE customer_id = '" + customerId + "'";
```

```java
// ✅ Parameterized query
@Query("SELECT o FROM OrderEntity o WHERE o.customerId = :customerId")
List<OrderEntity> findByCustomer(@Param("customerId") UUID customerId);
```

```java
// ✅ Object-level authorization
@PreAuthorize("#order.customerId == authentication.principal.customerId")
public OrderResponse view(OrderEntity order) { ... }
```

## Checklist

- [ ] All queries parameterized; no string-built SQL/JPQL.
- [ ] JWTs fully validated (signature, iss, aud, exp).
- [ ] Object-level authorization enforced; deny by default.
- [ ] Input validated/allow-listed at boundaries.
- [ ] No secrets in code/config/images/logs.
- [ ] TLS in transit; sensitive data encrypted at rest.
- [ ] SCA passes (no known-critical CVEs).
- [ ] SSRF allow-list for outbound calls; metadata endpoints blocked.
- [ ] Sensitive actuator/debug endpoints disabled in prod.

Owned by [agents/security-engineer.md](../agents/security-engineer.md); enforced in review by [agents/code-reviewer.md](../agents/code-reviewer.md).
