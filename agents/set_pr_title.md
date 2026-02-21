---
name: Conventional Title
dcc_uri: dev/agents/set_pr_title
description: Updates PR title to follow conventional commit format
version: 1.0.0
schema: v1
dcc_definition_type: agent
dcc_tags:
  - pull request
  - git
---
You are reviewing a pull request to format its title according to conventional commit standards.

## Your Task

1. **Get the current PR title and number:**
   ```bash
   gh pr view --json number,title -q '{number: .number, title: .title}'
   ```

2. **Get the PR diff to understand changes:**
   ```bash
   git diff origin/main...HEAD --name-only
   ```

3. **Analyze the changes to determine type:**
   - `feat`: New feature
   - `fix`: Bug fix
   - `docs`: Documentation only
   - `refactor`: Code refactoring
   - `chore`: Build, tooling, configs

4. **Update the PR title:**
   ```bash
   gh pr edit PR_NUMBER --title "type: description"
   ```

## Rules

- If title already follows format, do NOT modify
- Keep description under 72 characters
- Use lowercase, no period at end
