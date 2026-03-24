---
name: reviewing-pull-requests
description: >
  Perform systematic pull request reviews covering correctness, security,
  performance, and maintainability. Use when reviewing PRs, providing
  code feedback, or conducting pre-merge checks.
---

# Reviewing Pull Requests

Conduct a thorough, structured review of a pull request to catch issues before they reach the main branch.

## Review Dimensions

### 1. Correctness
- Does the code do what the PR description claims?
- Are edge cases handled (empty inputs, null values, boundary conditions)?
- Do the tests cover the new behavior and the stated fix?
- Are error paths handled, not just the happy path?

### 2. Security
- Does the change introduce user input that reaches a database query, shell command, or rendered output without sanitization?
- Are authentication and authorization checks present where needed?
- Are secrets or credentials handled safely (not logged, not hardcoded)?
- Does the change respect existing access control patterns?

### 3. Performance
- Does the change introduce N+1 queries, unbounded loops, or unnecessary allocations?
- Are database queries using appropriate indexes?
- Is there potential for resource leaks (unclosed connections, file handles)?
- For hot paths, is the complexity proportional to the input size?

### 4. Maintainability
- Is the code readable without extensive comments?
- Does it follow the project's established patterns and conventions?
- Are new abstractions justified by current use, not hypothetical future needs?
- Is the change scoped appropriately, or does it mix unrelated modifications?

## Output Format

Organize findings by severity (see `prompts/severity-guidelines.md`):

1. **Must Fix** — issues that block merge
2. **Should Fix** — issues worth addressing but not blocking
3. **Nit** — style preferences and minor suggestions
4. **Praise** — call out particularly good patterns worth replicating

Reference specific lines using `file:line` format. Suggest concrete fixes where possible rather than just identifying problems.
