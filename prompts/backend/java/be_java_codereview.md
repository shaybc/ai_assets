---
name: Java Code Review
dcc_uri: dev/prompts/be_java_codereview
description: Review Java code against project-specific rules
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - codereview
---

Review the selected Java code. Apply our core, backend-core, and Java-specific rules.

Check specifically for:

**Modern Java correctness**
- POJOs with manual getters/setters/equals/hashCode that should be `record`
- `class` used to model closed domains that should be `sealed interface` + `record` implementations
- Force-unwrapped nulls or missing null checks where `Optional` is the right return type
- `Optional` misused as a field type, collection element, or method parameter
- Checked exceptions thrown for conditions the caller cannot recover from
- `switch` statements / `if-else if` chains that should be `switch` expressions

**Concurrency**
- Raw `Thread` creation instead of `ExecutorService` or virtual threads
- `synchronized` blocks on virtual threads (contention risk with Loom)
- `.get()` called on a `CompletableFuture` inside an async context
- Missing error recovery in `CompletableFuture` chains (`exceptionally` / `handle`)
- Wrong flattening operator for the async intent

**Code quality**
- Mutable collections returned from public APIs where immutable is correct
- Null returned where `Optional` is appropriate
- Exception wrapping without preserving the original cause
- Resources not closed via `try-with-resources`
- Stateful lambdas or side effects inside Stream operations

**Testing**
- Static state or singletons that create test ordering dependencies
- Tests named after implementation rather than behaviour

Format: Critical → Important → Minor. Be direct.
