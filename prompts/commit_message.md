---
name: Commit
dcc_uri: dev/prompts/commit_message
version: '1.0.0'
schema: ''
description: Generate a conventional commit message from the current change
dcc_definition_type: prompt
dcc_tags:
  - dev
invokable: true
---

Write a conventional commit message for the selected diff or change description.

Rules:
- Format: `type(scope): short imperative summary` — max 72 chars
- Types: feat | fix | refactor | test | chore | docs | perf | ci
- Scope is the module, component, or feature area affected (be specific, not generic like "app")
- If the change is breaking, append `!` after the type/scope and add a `BREAKING CHANGE:` footer
- Body is optional — include it only if the *why* is not obvious from the summary
- Do not describe *what* the code does line by line — describe *why* the change was made

Output the commit message only, no explanation.
