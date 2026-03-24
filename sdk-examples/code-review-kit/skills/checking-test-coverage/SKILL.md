---
name: checking-test-coverage
description: >
  Analyze test coverage gaps and suggest targeted tests for uncovered
  code paths. Use when evaluating test adequacy, identifying missing
  edge case tests, or improving confidence before a release.
---

# Checking Test Coverage

Analyze a codebase or changeset to identify gaps in test coverage and recommend targeted tests that maximize confidence.

## Analysis Approach

### 1. Identify What Changed
- Map the files and functions modified in the changeset
- Determine which are business logic vs. configuration vs. infrastructure

### 2. Trace Code Paths
For each modified function:
- Identify the happy path and verify it has a test
- Identify error/exception paths and check for corresponding tests
- Identify boundary conditions (empty inputs, max values, type coercion) and check coverage
- Identify interactions with external systems (databases, APIs, filesystems) and check for integration tests

### 3. Evaluate Existing Tests
- Do tests assert behavior (what the code does) rather than implementation (how it does it)?
- Are test names descriptive enough to serve as documentation?
- Do tests cover the specific scenarios introduced by this change?
- Are there tests that need updating due to changed behavior?

### 4. Recommend New Tests
For each coverage gap, suggest a specific test:
- Name the test clearly: `test_<function>_<scenario>_<expected_outcome>`
- Describe the setup, action, and assertion
- Prioritize tests that catch regressions over tests that verify obvious behavior

## Prioritization

Focus testing effort where it matters most:

1. **Critical path** — authentication, authorization, payment processing, data persistence
2. **Error handling** — failure modes that affect users or data integrity
3. **Edge cases** — boundary values, empty states, concurrent access
4. **Integration points** — external API calls, database queries, message queues

## Output

Produce a summary with:
- Coverage assessment (which paths are tested, which are not)
- Prioritized list of recommended tests with descriptions
- Any existing tests that should be updated or removed
