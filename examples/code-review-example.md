# Example: Performing a Code Review

> A worked example of using the code-reviewer agent to produce a structured, actionable review.

## Scenario

A teammate opens a PR adding a customer lookup endpoint. We review it with [agents/code-reviewer.md](../agents/code-reviewer.md) using [prompts/review-pr.md](../prompts/review-pr.md).

## The code under review

```java
@RestController
public class CustomerController {

    @Autowired
    private CustomerRepository repo;          // field injection

    @GetMapping("/customers")
    public List<CustomerEntity> all(@RequestParam String name) {
        String sql = "SELECT * FROM customer WHERE name = '" + name + "'";   // string SQL
        return repo.runNative(sql);
    }

    @GetMapping("/customers/{id}")
    public CustomerEntity get(@PathVariable Long id) {
        return repo.findById(id).get();        // unguarded Optional.get()
    }
}
```

## The review output

**Verdict:** 🛑 Changes requested — security and correctness issues block merge.

### Findings

- 🛑 **Blocking — SQL injection** (`all`, line 8). User input is concatenated into SQL. Per [standards/security.md](../standards/security.md), use a parameterized query:
  ```java
  @Query("SELECT c FROM CustomerEntity c WHERE c.name = :name")
  List<CustomerEntity> findByName(@Param("name") String name);
  ```

- 🛑 **Blocking — entity exposed at API boundary** (`all`, `get`). Returning `CustomerEntity` leaks the schema and risks `LazyInitializationException`. Map to a `CustomerResponse` DTO ([skills/spring-boot.md](../skills/spring-boot.md)).

- 🛑 **Blocking — unguarded `Optional.get()`** (`get`, line 14). Throws `NoSuchElementException` → 500 for a missing customer. Return 404:
  ```java
  return repo.findById(id).map(CustomerResponse::from)
      .orElseThrow(() -> new CustomerNotFoundException(id));
  ```

- ⚠️ **Should — field injection** (line 5). Use constructor injection with a `final` field per [standards/clean-code.md](../standards/clean-code.md).

- ⚠️ **Should — unbounded list** (`all`). No pagination; large result sets will hurt latency/memory. Add pagination per [skills/api-design.md](../skills/api-design.md).

- ⚠️ **Should — missing tests.** No tests for not-found, injection-safe lookup, or pagination ([skills/testing.md](../skills/testing.md)).

- 💡 **Nit — versioning.** Path should be `/api/v1/customers` to match our convention.

- ❓ **Question.** Should `name` be an exact match or a search? That changes the contract and indexing.

**Overall risk:** High until the three blocking items are fixed; the change is small and easy to correct.

## Why this is a good review

- **Prioritized**: security and correctness first; style last.
- **Actionable**: every blocking comment includes the fix and links the standard.
- **Specific**: file/line references, not vague disapproval.
- **Constructive**: critiques the code, teaches via links, and ends with a clear verdict.

See [standards/code-review.md](../standards/code-review.md) for the label definitions used above.
