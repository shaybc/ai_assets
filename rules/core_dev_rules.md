---
alwaysApply: true
name: Dev Core Rules
dcc_uri: dev/rules/dev-core-rules
description: 'Core development rules for quality, maintainability, testing, and delivery.'
version: '1.2'
dcc_tags:
  - dev
---
# Core development Rules
These rules apply to all projects (web/mobile/backend/devops etc).

## 1) Small, Focused Changes
- Prefer small, correct, maintainable changes.
- MUST keep PRs/changesets focused on one goal.
- MUST avoid unrelated refactors in the same change.
- MUST delete dead code and unused exports touched by the change.

## 2) Readability & Maintainability
- MUST use clear names; avoid cleverness.
- MUST keep functions/modules small and single-purpose.
- MUST avoid magic numbers/strings (use constants/enums).
- MUST add comments only for “why”, not “what”.

## 3) Error Handling
- MUST not swallow errors silently.
- MUST return/throw consistent error types within a module.
- MUST keep public APIs stable; breaking changes require migration notes.

## 4) Testing Discipline
- MUST add/update tests for new behavior.
- MUST add a regression test for every bug fix (when feasible).

## 5) Determinism & Reproducibility
- MUST ensure builds are reproducible (lockfiles committed where relevant).
- MUST avoid reliance on local machine state.
- MUST document required env/config in README or docs.

## 6) Performance Hygiene
- MUST avoid obvious O(n²) or unbounded loops on user-sized inputs.
- MUST avoid expensive work in hot paths without caching/limits.
- MUST measure before “optimizing” (add a benchmark or profiling note when relevant).

## 7) Dependency Hygiene
- MUST remove unused dependencies.
- MUST not introduce large dependencies without justification.
- MUST pin/lock dependency versions per ecosystem norms.

## 8) Logging & Debuggability
- SHOULD add basic metrics/tracing hooks for critical paths (latency, errors, queue depth, retries).
- MUST keep logs actionable (what happened + context).
- SHOULD include correlation/request IDs where applicable.
- MUST not spam logs in tight loops.

## 9) Documentation & Developer Experience
- MUST update docs when behavior/config changes.
- MUST keep examples and scripts runnable.
- MUST include a clear “how to run/test locally” path.
- Optional document changes in changelog or release notes.
- SHOULD include rollback notes when change is risky.

## 10) Configuration
- MUST keep config keys centralized (single source of truth).
- MUST include defaults only when safe; otherwise fail fast.
- MUST document config schema (names, types, examples).

## 11) Migrations and Backwards Compatibility
- MUST treat schema/config/protocol changes as migrations (forward + rollback/compat plan).
- MUST avoid breaking changes without a migration path.

## 12) Stability & Contracts
- MUST define and keep stable contracts between components (API schemas, events, interfaces).
- MUST treat contract changes as breaking unless explicitly versioned or migrated.
- MUST update all consumers when modifying shared interfaces.
- MUST keep backward compatibility during transitions (deprecate before removal).
- MUST document contract expectations (inputs, outputs, invariants).
- MUST avoid implicit behavior changes in shared modules.

## 13) Output Expectation
- Prefer minimal diffs.
- If multiple approaches exist: pick one, state tradeoffs briefly.
- Provide a short test plan (commands + what to verify).
