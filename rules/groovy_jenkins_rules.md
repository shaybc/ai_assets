---
name: Groovy Jenkins Pipeline
dcc_uri: dev/rules/groovy_jenkins_rules
description: Groovy development rules for Jenkins pipeline developers. Extends core rules.
version: 1.0.0
dcc_definition_type: rule
dcc_tags:
  - dev
  - devops
  - groovy
  - jenkins
  - ci
---
# Groovy Jenkins Pipeline Rules

These rules apply to all Jenkins pipeline development (Declarative and Scripted) and shared library Groovy code. They extend the core rules.

---

## Pipeline Structure

- Prefer Declarative pipelines over Scripted pipelines for all new work. Declarative syntax is validated at parse time, produces better error messages, and is easier to read and maintain.
- Use Scripted pipelines (`node {}` / `script {}`) only when Declarative cannot express the required logic — and only in the smallest possible `script {}` block inside an otherwise Declarative pipeline.
- Never put business logic directly in a `Jenkinsfile`. Extract reusable logic into a shared library. A `Jenkinsfile` should read like a high-level description of the pipeline, not an implementation.
- Keep `Jenkinsfile` at the root of the repository it describes. Do not centralise all Jenkinsfiles in a separate repo — this breaks the principle that pipeline definition and application code evolve together.

---

## Shared Libraries

- All shared, reusable pipeline logic must live in a shared library under `vars/` (global pipeline steps) or `src/` (Groovy classes with full package structure).
- Functions in `vars/` must be named in camelCase and match their filename exactly (`deployService.groovy` exports `deployService()`).
- `src/` classes must follow standard Java/Groovy package conventions and be fully unit-testable without a Jenkins instance.
- Never put credentials, environment-specific values, or hardcoded URLs inside a shared library. Accept them as parameters or read them from the environment.
- Document every public `vars/` step with a corresponding `vars/stepName.txt` file describing its parameters, return value, and usage example. This is Jenkins' native documentation mechanism.

---

## Groovy Code Quality

- Avoid dynamic typing where the type is known. Use explicit types (`String`, `List<String>`, `Map<String, Object>`) rather than `def` everywhere — it makes intent clear and catches bugs earlier.
- Use `def` only for local variables where the type is obvious from the right-hand side, or where dynamic dispatch is genuinely needed.
- Prefer named parameters (maps) over positional parameters for any function with more than two arguments. Jenkins pipeline steps follow this convention; your shared library should too.
- Never use `@NonCPS` as a shortcut for avoiding serialisation issues. Understand why the issue exists first — `@NonCPS` methods cannot use Jenkins pipeline steps, cannot be resumed after a restart, and silently break continuability if misapplied.
- Avoid Groovy's `metaClass`, runtime method injection, and dynamic dispatch tricks. They make code impossible to reason about and break static analysis.

---

## Security & Credentials

- Never hardcode credentials, tokens, secrets, or environment-specific URLs in a `Jenkinsfile` or shared library. Always use the Jenkins Credentials store and bind them with `withCredentials`.
- Use the most restrictive credential type available: `usernamePassword` over `string` where both are available, `sshUserPrivateKey` over a stored password for SSH operations.
- Never echo or print credential values in logs, even in debug steps. `withCredentials` masks values automatically only if they are used via the bound variable — constructing strings from them manually may bypass masking.
- Apply least privilege to Jenkins agents and service accounts used in pipelines. A pipeline that deploys to production should not run on the same agent class or with the same credentials as a pipeline that only runs tests.

---

## Reliability & Correctness

- Always set explicit timeouts on stages and the overall pipeline. A hung build that never fails is worse than one that fails fast.
- Wrap every external call (shell command, HTTP request, deployment step) in error handling. Do not assume a `sh` step failing will cleanly fail the build — check exit codes explicitly or use `returnStatus: true` when you need conditional logic.
- Use `try / catch / finally` in Scripted blocks to ensure cleanup steps (workspace cleanup, resource release, notification) always run regardless of failure. In Declarative pipelines, use `post { always {} }`.
- Never use `sleep` as a synchronisation mechanism. Use polling with a timeout (`waitUntil`, `timeout + retry`) or a proper webhook/event trigger instead.
- Make pipelines idempotent where possible. Running the same pipeline twice on the same commit should produce the same result and not leave orphaned resources.

---

## Agents & Resources

- Always declare an explicit `agent` label. Never use `agent any` in production pipelines — it offers no guarantee about the environment, installed tools, or available resources.
- Clean the workspace at the start or end of a build on persistent agents. Stale artifacts from previous builds cause subtle, hard-to-reproduce failures.
- Avoid storing large files or build artifacts on the Jenkins controller. Use `stash` / `unstash` for inter-stage file passing and an external artifact store (Nexus, Artifactory, S3) for anything that needs to persist beyond the build.
- Request only the resources a stage actually needs. If only one stage requires a Docker socket or privileged access, scope the agent label to that stage, not the whole pipeline.

---

## Testing Shared Libraries

- Shared library `src/` classes must have unit tests. Use the `jenkins-pipeline-unit` framework to test pipeline logic without a live Jenkins instance.
- Test the failure paths: verify that your error handling, retry logic, and `post` conditions behave correctly when a step fails.
- Do not test Jenkins built-in steps — test your code's logic around them. Mock `sh`, `echo`, and other Jenkins steps in unit tests.
