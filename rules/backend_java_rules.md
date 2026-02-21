---
alwaysApply: false
name: Backend Java Rules
dcc_uri: dev/rules/backend_java_rules
description: Java-specific backend development rules (Java 21+). Extends core and backend-core rules. Framework-specific rules are layered on top.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - backend
  - java
---

# Backend Java Rules (Java 21+)

These rules apply to all Java backend projects and extend the core and backend-core rules.

---

## Language & Style

- Prefer `record` for immutable data carriers. Avoid writing POJOs with manual getters/setters/equals/hashCode for new code.
- Use sealed classes/interfaces to model closed, mutually exclusive domains (e.g. command types, result types). Pair with exhaustive `switch` expressions.
- Prefer `switch` expressions over `switch` statements and chains of `if/else if`. Use exhaustive pattern matching — the compiler is your safety net.
- Use `var` for local variables only when the type is obvious from the right-hand side. Never use `var` for fields or method signatures.
- Prefer immutable collections (`List.of()`, `Map.of()`, `Set.of()`). Only reach for mutable collections when you explicitly need mutation.
- Never return `null` from a method that might reasonably have no value — return `Optional<T>`. Never pass `null` as a method argument; use overloads or `Optional` at boundaries.
- Use `Optional` only as a return type. Do not use it as a field type, constructor parameter, or collection element.
- Avoid checked exceptions for conditions the caller cannot meaningfully recover from. Prefer unchecked exceptions, and document them in Javadoc.

---

## Concurrency

- Prefer `java.util.concurrent` abstractions (`ExecutorService`, `CompletableFuture`, structured concurrency via `StructuredTaskScope`) over raw `Thread` creation.
- Use virtual threads (Project Loom) for I/O-bound work. Do not block virtual threads with `synchronized` on contended monitors — use `ReentrantLock` or redesign to avoid shared mutable state.
- Treat `CompletableFuture` chains as pipelines: use `thenApply` for transforms, `thenCompose` for dependent async calls, `exceptionally`/`handle` for error recovery. Never call `.get()` on a `CompletableFuture` inside another async context without understanding the blocking implications.
- Any class accessed from multiple threads must be explicitly documented as thread-safe or not. Do not leave it implicit.
- Prefer immutability and message-passing over shared mutable state. If shared state is unavoidable, keep the critical section as small as possible.

---

## Streams & Functional Style

- Use the Stream API for data transformations and aggregations. Prefer a clear chain of `filter → map → collect` over imperative loops when it improves readability.
- Do not use parallel streams without a measured justification. The overhead and thread-safety implications are rarely worth it for typical service workloads.
- Keep lambdas short. If a lambda body exceeds 2–3 lines, extract it as a named private method and use a method reference.
- Avoid stateful lambdas and side effects inside stream operations. Streams should be pure transformations.

---

## Error Handling

- Define a small, consistent exception hierarchy per module. Don't throw `RuntimeException` or `Exception` directly — give callers something meaningful to catch or log.
- Always include the causing exception when wrapping: `new ServiceException("context", cause)`. Never swallow the original cause.
- Use `try-with-resources` for anything that implements `AutoCloseable`. Never close resources manually in a `finally` block.
- Do not use exceptions for flow control. Exceptions are for exceptional conditions, not expected branching.

---

## Code Organisation

- One top-level public type per file.
- Order class members consistently: constants → fields → constructors → public methods → package/protected methods → private methods. Use `// region` or well-placed blank lines for visual separation in longer classes.
- Keep classes small. A class that needs more than one screen to read probably has more than one responsibility.
- Favour composition over inheritance. Extend only when there is a genuine "is-a" relationship; otherwise delegate.
- Write Javadoc on every public API type and method. Document the contract (preconditions, postconditions, what exceptions mean), not the implementation.

---

## Testing

- Write unit tests for all logic that can be tested without I/O. Pure functions and domain objects should have near-complete coverage.
- Use integration tests to verify persistence, serialisation, and external protocol boundaries. Keep them separate from unit tests so fast feedback is always available.
- Avoid `static` state and singletons in test code — they cause test ordering dependencies. Each test must be able to run in isolation.
- Name tests to describe behaviour, not implementation: `shouldReturnEmptyListWhenNoResultsFound` over `testGetResults`.
