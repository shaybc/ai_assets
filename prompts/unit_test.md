---
name: Unit test
dcc_uri: dev/prompts/unit_test
version: '0.3'
schema: ''
description: Generate unit tests for the selected code
dcc_definition_type: prompt
dcc_tags:
  - backend
  - spring
  - microservices
  - docker containers
  - frontend
  - angular
invokable: true
prompt: |-
  Write a complete unit test suite for the selected code.

  Rules:
  - Use JUnit 5 for Java, pytest for Python, Jest for JS/TS.
  - Cover edge cases and failure paths.
  - Include all imports and setup.
  - Output complete, ready-to-run test files (no placeholders).
---

