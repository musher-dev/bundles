## Code Style

Apply these conventions when generating or modifying code.

### General

- Match the existing style of the file being edited — do not impose a different style
- Prefer explicit over implicit (named arguments, clear variable names, explicit return types)
- Keep functions short and focused on a single responsibility
- Avoid deep nesting — use early returns and guard clauses

### Naming

- Use descriptive names that convey intent, not implementation
- Boolean variables and functions should read as assertions: `is_valid`, `has_permission`, `can_retry`
- Avoid abbreviations unless they are universally understood in the domain (`url`, `id`, `config`)
- Collection variables should be plural: `users`, `items`, `results`

### Error Handling

- Handle errors at the appropriate level — don't catch and re-throw without adding context
- Use specific error types over generic ones when the language supports it
- Include actionable information in error messages: what failed, why, and what to do about it
- Never swallow errors silently

### Dependencies

- Prefer standard library solutions over third-party packages for simple operations
- When suggesting a dependency, verify it is actively maintained and widely used
- Pin dependency versions in examples to avoid breaking changes
