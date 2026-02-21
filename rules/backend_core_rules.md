---
alwaysApply: false
name: Backend Core Rules
dcc_uri: dev/rules/backend_core_rules
description: Technology-agnostic backend development rules. Extends core rules. Language and framework-specific rules are layered on top.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - backend
---

# Backend Core Rules

These rules apply to all backend projects regardless of language or framework. Technology-specific rules are defined separately and extend these.

---

## API Design

- Design APIs around resources and intentions, not internal implementation or database shape.
- Be consistent: same naming conventions, same error envelope, same pagination shape across all endpoints in a service.
- Never leak internal details in responses: no stack traces, no internal IDs or table names, no raw ORM objects.
- Version APIs from day one. Adding a version prefix costs nothing; removing a broken unversioned API costs a lot.
- Treat every public API as a contract. Breaking changes require a migration path, deprecation notice, and a timeline.
- Validate all input at the boundary. Never trust caller-supplied data deeper than the entry point.

---

## Data & Persistence

- Keep business logic out of the database. Stored procedures and triggers are hard to version, test, and migrate.
- Write every schema change as a versioned, incremental migration. Never mutate a schema manually in any environment.
- Prefer additive migrations (add column, add table); avoid destructive changes in the same migration that runs alongside a deployment.
- Design for the read pattern, not just the write pattern. Know your query access patterns before choosing a data model.
- Never store secrets, PII, or sensitive data in plaintext. Encrypt at rest or delegate to a dedicated vault.

---

## Error Handling & Resilience

- Every external call (database, HTTP, queue, cache) can fail. Handle it explicitly — timeouts, retries with backoff, and circuit breakers where appropriate.
- Distinguish between retryable errors (transient network, rate limit) and non-retryable errors (validation, not found). Don't retry blindly.
- Fail fast on startup if required config or dependencies are missing. A service that starts silently broken is harder to debug than one that refuses to start.
- Design for partial failure: a downstream service being unavailable should degrade gracefully, not take down the whole system.

---

## Security

- Authenticate every request that touches non-public data. Never rely on network topology alone ("it's internal") as a security boundary.
- Apply least-privilege to every service account, DB user, and API token.
- Sanitize and parameterize all external inputs used in queries, commands, or templates. No string concatenation for SQL, shell, or HTML.
- Do not log secrets, tokens, passwords, or raw PII. Mask or omit sensitive fields before they reach any log sink.
- Set explicit timeouts on all inbound and outbound connections.

---

## Observability

- Every service must expose: a health/readiness endpoint, structured logs with a correlation/request ID, and basic error rate and latency metrics.
- Log at boundaries: incoming requests, outgoing calls, and significant state transitions. Not inside loops.
- Distinguish log levels deliberately: `ERROR` means human action required, `WARN` means degraded but continuing, `INFO` means normal significant events, `DEBUG` for development only.
- Trace IDs must flow through the entire call chain — from the inbound request through every downstream call, queue message, and async job.

---

## Async & Background Work

- Treat background jobs and queue consumers as first-class code: they need the same error handling, logging, and tests as synchronous paths.
- Make background jobs idempotent wherever possible. Queues deliver at-least-once; your consumer must handle duplicates safely.
- Never do unbounded work in a single job run. Paginate, batch, or checkpoint so that a large dataset doesn't cause timeouts or memory exhaustion.
- Dead-letter queues are not optional. Every queue consumer must have a defined failure path.
