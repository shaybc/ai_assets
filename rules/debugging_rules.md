---
alwaysApply: false
name: Debugging rules
dcc_uri: dev/rules/debugging_rules
description: 'Minimal, high-signal rules for effective debugging: reproduce, isolate, test hypotheses, and verify fixes during debugging tasks.'
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - debug
---

# Debugging Rules

## 1) Reproduce and Pinpoint
- MUST start from the exact error/log/failed behavior and identify the failing component (file/module/config) using stack traces, line numbers, and error text.
- MUST define the shortest repro path (command/endpoint/steps) and the expected vs actual result.

## 2) One Hypothesis at a Time
- MUST propose a single primary root-cause hypothesis and one minimal fix.
- MUST avoid multiple unrelated changes (“shotgun debugging”).

## 3) Use Evidence, Not Assumptions
- MUST base conclusions on provided evidence (logs/trace/code).
- If a critical detail is unknown, MUST propose a quick check to confirm it (1–3 checks max).

## 4) Instrument to Reduce Uncertainty
- When the root cause isn’t certain, MUST add targeted instrumentation at:
  - inputs/outputs of the failing function
  - branch decisions (if/else)
  - before/after external calls (DB/HTTP/filesystem)
  - error handling paths
- MUST keep instrumentation minimal and reversible (temporary or behind a debug flag).

## 5) If the Fix Fails
- If the user reports “didn’t work”, MUST:
  - re-evaluate the hypothesis
  - request the next piece of evidence (full stack trace/log snippet around failure)
  - propose the next most likely hypothesis + a new minimal change
- MUST not repeat the same fix.

## 6) Verify the Fix
- MUST provide a verification plan (commands/steps) and success criteria.
- SHOULD add a regression test when feasible.

## 7) Stop Conditions
- If multiple iterations fail, MUST converge by isolating variables (disable features, narrow inputs, bisect changes) until a single cause remains.
