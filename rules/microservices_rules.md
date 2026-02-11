---
alwaysApply: false
name: Java Spring Microservice
dcc_uri: dev/rules/microservices_rules
description: >-
  Java + Spring Boot microservice rules (Spring Boot 3.x, Java 21+). Extends
  core, backend-core, and backend-java rules.
version: '1.1'
dcc_tags:
  - dev
  - backend
  - java
  - spring
  - microservices
---
# Backend Java + Spring Microservice Rules (Spring Boot 3.x / Java 21+)

These rules apply to Java microservices built with Spring Boot and extend the core, backend-core, and backend-java rules.

---

## Spring Fundamentals

- Prefer constructor injection over field injection (`@Autowired` on fields). Constructor injection makes dependencies explicit, supports immutability, and is trivially testable without a Spring context.
- Declare beans in `@Configuration` classes or via component scanning on well-defined packages. Avoid scattering `@Bean` definitions across unrelated classes.
- Never use `ApplicationContext.getBean()` outside of bootstrap/infrastructure code. If you need to pull a bean manually, it is a signal the design needs revisiting.
- Prefer `@ConfigurationProperties` over `@Value` for any group of related configuration. Bind to a typed, validated record or class with `@Validated`.
- Mark `@ConfigurationProperties` classes as `record` or with `final` fields where possible — config should be immutable after startup.
- Fail fast: use Bean Validation (`@NotNull`, `@NotBlank`, etc.) on `@ConfigurationProperties` so the service refuses to start with missing or invalid config rather than failing at runtime.

---

## Web Layer

- Keep `@RestController` classes thin: validate input, delegate to a service, map the result to a response. No business logic in controllers.
- Use `@ControllerAdvice` / `@RestControllerAdvice` with `@ExceptionHandler` for centralised error mapping. Do not catch and map exceptions in individual controllers.
- Return consistent error response envelopes from all endpoints. Include a machine-readable error code, a human message, and the correlation/trace ID.
- Use Spring's `ProblemDetail` (RFC 7807) as the standard error response shape. It is built into Spring 6 / Boot 3 and avoids inventing a custom envelope.
- Validate all request bodies and path/query parameters with Bean Validation at the controller boundary. Do not propagate unvalidated input into the service layer.
- Use `ResponseEntity<T>` when you need explicit control over status codes and headers. Avoid it when the default `@ResponseStatus` or implicit 200 is sufficient — it adds noise.

---

## Service & Domain Layer

- The service layer owns business rules and transaction boundaries. It must not know about HTTP, serialisation formats, or response shapes.
- Annotate transaction boundaries explicitly with `@Transactional`. Do not rely on implicit transactionality or assume it propagates correctly without checking.
- Keep `@Transactional` on the service layer, not the repository. Transactions should span a unit of business work, not a single DB call.
- Avoid `@Transactional(readOnly = true)` as a performance afterthought — set it intentionally for read-only operations from the start, as it prevents accidental writes and allows DB routing optimisations.
- Do not let JPA/Hibernate entities escape the service layer. Map to DTOs or records before returning from a service method. Leaking entities into the web layer causes lazy-load exceptions and tight coupling.

---

## Data & JPA

- Define fetch strategies explicitly. The default `EAGER` on `@ManyToOne` and `LAZY` on `@OneToMany` are often wrong for the use case — choose deliberately.
- Prefer `@Query` with JPQL or native SQL for anything beyond the simplest derived queries. Derived method names become unreadable quickly and hide what SQL is generated.
- Use projections (interfaces or records) for read-only queries that don't need a full entity. Loading a full entity graph to read two fields is wasteful.
- Never call a repository in a loop. Batch or use a `IN` query. N+1 is the single most common JPA performance mistake and it's always avoidable.
- Enable `spring.jpa.open-in-view=false`. The open-session-in-view anti-pattern silently triggers lazy loads during serialisation and hides missing fetch joins.

---

## Resilience & Inter-Service Communication

- All outbound HTTP calls to other services must have an explicit connection timeout and read timeout configured. No call should be able to block indefinitely.
- Use a resilience library (Resilience4j) for retry, circuit breaker, and rate limiter patterns on outbound calls. Do not implement retry logic manually.
- Propagate trace IDs across all outbound calls. Use Spring's Micrometer Tracing / Brave instrumentation — do not manually set headers.
- Treat other services as unreliable. Design the feature to degrade gracefully when a downstream call fails, rather than propagating the failure upward.
- Prefer asynchronous messaging (events via a broker) over synchronous HTTP for cross-service workflows that do not require an immediate response. Synchronous chains create tight coupling and cascading failure risk.

---

## Configuration & Profiles

- Use Spring profiles (`dev`, `test`, `prod`) to separate environment-specific config. Never use `if (profile.equals("prod"))` checks in application code.
- Keep secrets out of `application.yml` and version control entirely. Inject via environment variables or a secrets manager; reference them with `${ENV_VAR}` placeholders.
- Provide a fully runnable `application-local.yml` with safe defaults for local development. Every developer should be able to run the service with `./gradlew bootRun` or `mvn spring-boot:run` without manual setup.

---

## Observability

- Expose `/actuator/health` (liveness + readiness probes), `/actuator/info`, and `/actuator/prometheus` (or equivalent metrics endpoint) in every service.
- Use structured (JSON) logging in non-local environments. Include `traceId`, `spanId`, `service`, and `environment` in every log line via MDC or Micrometer Tracing auto-instrumentation.
- Add custom metrics (`Counter`, `Timer`, `Gauge`) via Micrometer for any business-critical operation: order placed, payment processed, message consumed. Infrastructure metrics alone are not enough.

---

## Testing

- Use `@SpringBootTest` sparingly — only for full integration tests that verify the wired-up application. It is slow and should not be the default test style.
- Prefer `@WebMvcTest` for controller layer tests and `@DataJpaTest` for repository tests. These slice contexts start fast and test the right boundaries.
- Use `@MockBean` only when you need to replace a Spring-managed bean in a slice test. In unit tests, plain Mockito mocks without a Spring context are faster and simpler.
- Use Testcontainers for integration tests that touch a real database, broker, or cache. Do not test JPA behaviour against an in-memory H2 database if production uses Postgres — the SQL dialects diverge in ways that matter.
- Test the failure paths: verify that your `@ExceptionHandler` mappings, circuit breaker fallbacks, and retry logic behave correctly under failure conditions.
