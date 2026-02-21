---
name: Spring Endpoint
dcc_uri: dev/prompts/be_ms_create_endpoint
description: Scaffold a Spring Boot REST endpoint with all required layers
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - backend
  - microservices
---

Scaffold a complete Spring Boot REST endpoint for the following:

{{{ input }}}

Generate these layers in order:

**1. Request/Response DTOs** (as Java `record`)
- Request record with Bean Validation annotations (`@NotNull`, `@NotBlank`, etc.)
- Response record mapping only what the caller needs (no entity fields leaked)

**2. Controller** (`@RestController`)
- Thin: validate input (via `@Valid`), call service, return `ResponseEntity` only if status code control is needed
- No business logic
- No exception handling — that belongs in `@RestControllerAdvice`

**3. Service**
- `@Transactional` at the correct boundary
- Maps entity → response DTO before returning — entity must not escape this layer
- Throws a meaningful domain exception for not-found / conflict cases

**4. Exception handler stub** (if a new error case is introduced)
- `@RestControllerAdvice` method returning `ProblemDetail`

**5. Test stubs**
- `@WebMvcTest` for the controller: happy path + validation failure + not-found
- Service unit test: happy path + not-found

State any assumptions about the entity model or security context.
