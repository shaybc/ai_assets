---
name: Node.js Rules
dcc_uri: dev/rules/nodejs_rules
description: Node.js development rules (Node 20+ LTS). Extends core and backend-core rules.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - backend
  - nodejs
---

# Node.js Rules (Node 20+ LTS)

These rules apply to all Node.js backend projects and extend the core and backend-core rules.

---

## Language & Style

- Use ES Modules (`import` / `export`) for all new projects. Use CommonJS (`require` / `module.exports`) only when the project or a dependency requires it — do not mix both in the same project.
- Use `async/await` for all asynchronous code. Do not introduce new callback-based APIs or raw `.then()/.catch()` chains unless integrating with a library that requires it.
- Prefer `const` over `let`; never use `var`.
- Use strict equality (`===`) everywhere. Never use `==`.
- Prefer plain functions and objects over classes unless a framework explicitly requires class syntax (e.g. NestJS decorators).
- Avoid deeply nested callbacks or promise chains — flatten with `async/await` and early returns.

---

## Error Handling

- Every `async` function that can fail must have explicit error handling. Never let a rejected promise go unhandled.
- Distinguish error types at the boundary: validation errors (400), not found (404), auth failures (401/403), and unexpected errors (500). Do not map everything to 500.
- Never expose internal error details, stack traces, or file paths in API responses. Log them server-side; return a safe message to the caller.
- Use a centralised error handling middleware in Express rather than try/catch-to-response in every route handler. Route handlers should throw or pass errors to `next(err)`.
- Do not swallow errors silently with empty `catch` blocks. At minimum, log and rethrow.

---

## Async & Concurrency

- Never use `await` inside a `for` loop when the operations are independent — use `Promise.all()` to run them concurrently.
- Never block the event loop with synchronous I/O (`fs.readFileSync`, `execSync`, etc.) in request handlers or hot paths. Use the async equivalents.
- Set explicit timeouts on all outbound HTTP calls. A call with no timeout can block a request handler indefinitely.
- Use Node's `AbortController` to cancel in-flight requests when the caller disconnects or a timeout is reached.

---

## Project Structure

- One responsibility per file. A file that exports a router, defines business logic, and talks to the database has too many responsibilities.
- Group by feature, not by layer. Prefer `features/users/` containing its router, service, and data access over top-level `routes/`, `controllers/`, `models/` folders that scale poorly.
- Keep framework-specific code (Express routers, middleware) separate from business logic. Business logic must be callable without an HTTP context — this is what makes it testable.
- Entry point (`app.js` / `index.js`) wires the application together — it must not contain business logic or route definitions.

---

## Configuration & Secrets

- All configuration comes from environment variables. Use a single centralised config module that reads from `process.env`, validates required values on startup, and fails fast if any are missing.
- Never access `process.env.VARIABLE` directly outside the config module — this scatters assumptions about env shape across the codebase.
- Never commit `.env` files. Commit a `.env.example` with all required keys and safe placeholder values.

---

## Dependencies

- Do not install a package for something Node's standard library handles well (`fs/promises`, `crypto`, `path`, `URL`). Check built-ins first.
- Do not introduce a new dependency without checking: is it actively maintained, does it have a reasonable install size, and does it have known vulnerabilities (`npm audit`)?
- Pin exact versions in `package.json` for application code. Use lockfile (`package-lock.json`) committed to version control — never delete or ignore it.
- Use `npm ci` in CI/CD pipelines, not `npm install`.
