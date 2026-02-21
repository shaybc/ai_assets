---
name: Fix a Bug
dcc_uri: dev/prompts/fix_a_bug
version: '1.0.0'
schema: ''
description: Fix an error or bug and explain the root cause
dcc_definition_type: prompt
dcc_tags:
  - dev
invokable: true
---

Fix the issue described below (or paste the error/stacktrace).

{{{ input }}}

Rules:
- Fix the root cause, not just the symptom
- Do not change unrelated code
- If the fix requires a behaviour change that affects callers, say so explicitly
- After the fix, provide a 2-line explanation: what was wrong and why this fix is correct
- If a regression test should be added, show it
