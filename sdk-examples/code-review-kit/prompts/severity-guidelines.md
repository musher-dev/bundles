## Review Severity Guidelines

Classify each review finding using these severity levels to help authors prioritize their response.

### Must Fix (Blocking)

Issues that must be resolved before the PR can merge. These represent correctness or safety problems.

**Examples:**
- Logic error that produces wrong results
- Security vulnerability (SQL injection, XSS, credential exposure)
- Data loss potential (missing transaction, unsafe migration)
- Breaking change to a public API without versioning
- Missing authorization check on a protected resource

### Should Fix (Non-Blocking)

Issues worth addressing that don't block merge but should be tracked for follow-up.

**Examples:**
- Missing error handling for a recoverable failure
- Performance concern that doesn't affect current scale but will cause problems at 10x
- Missing test for an important edge case
- Inconsistency with established project patterns
- Documentation gap for a public-facing change

### Nit (Suggestion)

Style preferences, minor improvements, and optional polish. Authors may choose to address or skip these.

**Examples:**
- Variable naming that could be clearer
- Code that could be simplified with a standard library function
- Minor formatting inconsistency
- Comment that could be more precise
- Import ordering

### Praise

Positive feedback on patterns worth replicating. Recognizing good work is part of a healthy review culture.

**Examples:**
- Clean abstraction that simplifies a complex problem
- Thorough test coverage for edge cases
- Clear error messages that help debugging
- Good documentation of a non-obvious decision
