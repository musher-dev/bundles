---
name: writing-commit-messages
description: >
  Generate Conventional Commits-compliant commit messages from staged changes
  or a description of work done.
---

# Writing Commit Messages

Generate structured commit messages that follow the Conventional Commits specification from staged changes or a description of the work.

## Approach

### 1. Analyze the Changes

Examine the staged diff or provided description:

- Identify which files were added, modified, or deleted
- Determine the primary intent — is this a feature, fix, refactor, docs update, or chore?
- Note the scope — which module, component, or area of the codebase is affected

### 2. Select the Commit Type

Choose the appropriate Conventional Commits type:

| Type | When to use |
|------|-------------|
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `docs` | Documentation-only changes |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating tests |
| `chore` | Build process, dependencies, tooling |
| `perf` | Performance improvements |
| `ci` | CI/CD configuration changes |

### 3. Determine the Scope

Identify a short scope token in parentheses:

- Use the module, package, or component name (e.g., `auth`, `api`, `cli`)
- Omit the scope if the change spans multiple unrelated areas
- Keep it to one word when possible

### 4. Write the Message

Structure the commit message:

- **Subject line**: `{type}({scope}): {description}` — imperative mood, lowercase, no period, max 72 characters
- **Body** (optional): explain *why* the change was made, not *what* changed (the diff shows that)
- **Footer** (optional): reference issues (`Closes #123`), note breaking changes (`BREAKING CHANGE: ...`)

## Output Format

```
{type}({scope}): {short description}

{Optional body explaining motivation and context}

{Optional footer with references or breaking change notes}
```

## Examples

Single-line:
```
feat(api): add pagination to the list users endpoint
```

With body and footer:
```
fix(auth): prevent token refresh race condition

Two concurrent requests could both attempt to refresh an expired token,
causing one to fail with a 401. Added a mutex to serialize refresh calls.

Closes #42
```

## Guidelines

- Use imperative mood in the subject ("add", not "added" or "adds")
- Keep the subject line under 72 characters
- Do not end the subject line with a period
- Separate subject from body with a blank line
- The body should explain *why*, not *what*
- If the change is trivial (typo fix, formatting), a subject line alone is sufficient
- Flag breaking changes in both the type (`feat!`) and the footer
