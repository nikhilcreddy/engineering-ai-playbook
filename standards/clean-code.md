# Standard: Clean Code

> How we write readable, maintainable Java in this organization.

## Principles

1. **Code is read far more than it's written.** Optimize for the next engineer.
2. **Make intent obvious.** Names and structure should explain the "what"; comments explain the "why."
3. **Small, cohesive units.** Functions do one thing; classes have one reason to change.
4. **Least surprise.** Follow framework idioms and team conventions over personal style.
5. **Delete code freely.** Dead code is a liability; version control remembers it.

## Naming

- Reveal intent: `findActiveCustomers()`, not `getData()`.
- Booleans read as predicates: `isActive`, `hasShipped`, `canRetry`.
- Avoid abbreviations and Hungarian notation; avoid noise words (`Manager`, `Data`, `Info`) unless meaningful.
- Use domain language consistently (ubiquitous language).

## Functions

- Aim for **< 20 lines**; extract when a block needs a comment to explain it.
- **One level of abstraction** per function.
- **≤ 3 parameters**; group related params into a record/value object.
- **No boolean flag params** that switch behavior — split into two methods.
- **Return early** to avoid deep nesting; prefer guard clauses.
- **No side effects hidden** behind innocent names.

## Classes & structure

- One public responsibility per class; keep them small.
- Prefer composition over inheritance.
- Keep controllers thin, services focused, repositories for persistence only.
- Package by **feature/domain**, not by layer-only.

## Error handling

- Don't swallow exceptions; either handle meaningfully or propagate with context.
- Use specific exception types; avoid throwing/catching raw `Exception`.
- Validate inputs at boundaries; trust invariants internally.
- Never use exceptions for normal control flow.

## Comments

- Explain **why**, not **what** the code already says.
- Delete commented-out code.
- Keep TODOs actionable and tracked (`// TODO(JIRA-123): ...`).

## ✅ / ❌ Examples

```java
// ❌ Unclear name, flag param, deep nesting
public List<O> get(List<O> os, boolean f) {
    List<O> r = new ArrayList<>();
    for (O o : os) {
        if (o != null) {
            if (f ? o.isPaid() : o.isOpen()) {
                r.add(o);
            }
        }
    }
    return r;
}
```

```java
// ✅ Intent-revealing, no flags, guard clause, stream
public List<Order> paidOrders(List<Order> orders) {
    return orders.stream()
        .filter(Objects::nonNull)
        .filter(Order::isPaid)
        .toList();
}
```

## Checklist

- [ ] Names reveal intent; no noise words or abbreviations.
- [ ] Functions are small and do one thing.
- [ ] No boolean flag parameters.
- [ ] Guard clauses instead of deep nesting.
- [ ] No swallowed exceptions; errors carry context.
- [ ] No dead/commented-out code.
- [ ] Comments explain "why," not "what."

Used as a review lens by [agents/code-reviewer.md](../agents/code-reviewer.md); complements [skills/java21.md](../skills/java21.md).
