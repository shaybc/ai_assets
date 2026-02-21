---
name: Spring Code Review
dcc_uri: dev/prompts/be_ms_codereview
description: Review Spring Boot microservice code against project-specific rules
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - codereview
  - backend
  - microservices
---

Review the selected Spring Boot code. Apply our core, backend-core, backend-java, and Spring-specific rules.

Check specifically for:

**Spring fundamentals**
- Field injection (`@Autowired` on fields) instead of constructor injection
- `@Value` scattered on individual fields instead of a typed `@ConfigurationProperties` class
- Business logic in a `@RestController` method
- `ApplicationContext.getBean()` outside bootstrap code
- `@ConfigurationProperties` class not validated with `@Validated`

**Web layer**
- Exception mapping in individual controllers instead of `@RestControllerAdvice`
- Error responses that leak stack traces, internal field names, or raw exception messages
- Missing Bean Validation on request body or path/query parameters
- Not using `ProblemDetail` (RFC 7807) as the error response shape

**Data & JPA**
- `spring.jpa.open-in-view` not explicitly set to `false`
- JPA entity returned directly from a service method (should map to DTO/record first)
- Repository called inside a loop (N+1)
- Fetch strategy not explicitly set (relying on JPA defaults)
- `@Transactional` on a repository method instead of a service method
- Missing `readOnly = true` on query-only transactions

**Resilience**
- Outbound HTTP call with no timeout configured
- Retry logic implemented manually instead of via Resilience4j
- Missing circuit breaker on calls to external services
- Trace ID not propagated through outbound calls

**Testing**
- `@SpringBootTest` used where `@WebMvcTest` or `@DataJpaTest` slice is appropriate
- In-memory H2 used for JPA integration tests instead of Testcontainers
- `@MockBean` used in a plain unit test that doesn't need a Spring context

Format: Critical → Important → Minor. Be direct.
