---
alwaysApply: true
name: Frontend Core Rules
dcc_uri: dev/rules/frontend_core_rules
description: Core frontend development rules for component quality, state, UI correctness, and accessibility. Technology-agnostic; platform-specific rules are layered on top.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - frontend
---

# Frontend Core Rules
These rules apply to all frontend projects (web, mobile, desktop). Technology-specific rules are defined separately and extend these.

## 1) Component Design
- MUST keep components small and single-purpose; split when a component handles more than one concern.
- MUST separate logic from presentation: derive display values at the boundary, not deep in the tree.
- MUST avoid hardcoding content strings, dimensions, or colors inline — use tokens/constants.
- MUST not leak internal implementation details through a component's public interface.

## 2) State Management
- MUST keep state as local as possible; lift only when genuinely shared.
- MUST not store derived values in state — compute them from source data.
- MUST treat async state explicitly: always represent loading, error, and empty as distinct states, not as implicit `null`/`undefined`.
- MUST avoid side effects inside render/compute paths.

## 3) UI Correctness & Edge Cases
- MUST handle empty, loading, and error states visibly — never render a blank or broken UI silently.
- MUST guard against rendering with missing/null data that would cause a crash.
- MUST validate and sanitize any data rendered from external sources.
- MUST not assume API response shapes — be defensive when mapping data to UI.

## 4) Accessibility (A11y)
- MUST ensure interactive elements are keyboard-reachable and operable.
- MUST provide meaningful labels for interactive and media elements (buttons, inputs, images).
- MUST not rely on color alone to convey meaning.
- MUST preserve focus management on route changes, modal open/close, and dynamic content updates.

## 5) Performance Hygiene
- MUST avoid re-renders or recomputations caused by unstable references (inline objects/functions/arrays created on every render).
- MUST lazy-load routes and heavy components; do not bundle everything upfront.
- MUST not block the main thread with synchronous heavy computation — defer or offload.
