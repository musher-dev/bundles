## Coding Guidelines

Language-agnostic standards for writing clean, maintainable production code.

### Naming

- Use names that describe intent, not implementation (`user_accounts` not `data_list`)
- Booleans read as assertions: `is_active`, `has_permission`, `can_retry`
- Functions describe their action: `calculate_total`, `send_notification`, `validate_input`
- Constants are uppercase with underscores: `MAX_RETRIES`, `DEFAULT_TIMEOUT`
- Avoid abbreviations unless universally understood (`url`, `id`, `config`)

### Functions

- Each function does one thing and does it completely
- Keep functions under 30 lines — if longer, extract a named helper
- Use early returns to eliminate nesting
- Accept the narrowest types and return the most specific types
- Document non-obvious parameters and return values

### Error Handling

- Handle errors where you can do something about them — let them propagate otherwise
- Error messages include: what happened, why, and what to do about it
- Use typed/specific errors over generic ones
- Never catch an exception just to log and re-throw without adding context
- Resource cleanup happens in `finally` blocks or equivalent language constructs

### Dependencies

- Justify every new dependency — "it saves 5 lines of code" is rarely sufficient
- Pin versions to avoid surprise breakage
- Prefer well-maintained, widely-used packages with clear licensing
- Wrap third-party APIs behind your own interface to limit blast radius of changes

### Testing

- Test behavior, not implementation details
- Name tests as sentences: `test_expired_token_returns_401`
- Each test has one reason to fail
- Use factories or builders for test data, not copy-pasted fixtures
- Keep tests fast — mock external services, not internal logic

### Documentation

- Code is the primary documentation — write it clearly enough that comments are rarely needed
- Document the "why" in comments, never the "what" (the code already says what)
- Public APIs require docstrings with parameter descriptions and example usage
- Keep README files up to date with setup instructions and project structure
