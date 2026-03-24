## Code Review Checklist

Use this checklist when reviewing pull requests to ensure consistent, thorough reviews.

### Before Reading Code
- [ ] Read the PR description and linked issues to understand intent
- [ ] Check the PR size — if >400 lines, consider requesting it be split
- [ ] Verify the branch is up to date with the target branch

### Correctness
- [ ] The code does what the PR description claims
- [ ] Edge cases are handled (nulls, empty collections, boundary values)
- [ ] Error messages are actionable and include relevant context
- [ ] Database migrations are reversible and safe for zero-downtime deploys

### Security
- [ ] User input is validated and sanitized before use
- [ ] Authentication and authorization checks are in place
- [ ] No secrets or credentials are hardcoded or logged
- [ ] SQL queries use parameterized statements, not string interpolation

### Testing
- [ ] New behavior has corresponding tests
- [ ] Bug fixes include a regression test
- [ ] Tests assert behavior, not implementation details
- [ ] Test names describe the scenario and expected outcome

### Performance
- [ ] No N+1 query patterns introduced
- [ ] Large datasets are paginated, not loaded into memory
- [ ] Expensive operations are not inside loops
- [ ] Database queries use appropriate indexes

### Maintainability
- [ ] Code follows existing project conventions
- [ ] New abstractions are justified by current needs
- [ ] No dead code, commented-out blocks, or TODO items without issue links
- [ ] Public APIs include clear documentation
