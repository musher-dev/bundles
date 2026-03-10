# Naming Reference

Naming examples and conventions for skills. Referenced from the main SKILL.md during name derivation (Phase 1).

---

## Required Constraints

All skill names MUST satisfy these rules:

- [ ] Lowercase letters, numbers, and hyphens only
- [ ] Total length <= 64 characters
- [ ] Does not start or end with a hyphen

## Recommended Convention: Gerund-Noun

The Gerund-Noun pattern (`[gerund]-[noun]`) is the recommended naming style. It produces self-documenting names that communicate the skill action and target at a glance.

**Note:** Description quality matters more than directory name syntax. A skill with a clear, trigger-rich description will be discovered correctly regardless of whether its name follows gerund-noun perfectly. Use this convention as helpful guidance, not a strict gate.

---

## Examples by Category

### Creation Skills

| Intent | Derived Name |
|--------|--------------|
| Create React components | `scaffolding-components` |
| Generate API clients | `generating-clients` |
| Write migration scripts | `writing-migrations` |
| Bootstrap new services | `scaffolding-services` |

### Modification Skills

| Intent | Derived Name |
|--------|--------------|
| Refactor legacy code | `refactoring-legacy` |
| Migrate to new framework | `migrating-frameworks` |
| Optimize database queries | `optimizing-queries` |
| Fix accessibility issues | `fixing-accessibility` |

### Analysis Skills

| Intent | Derived Name |
|--------|--------------|
| Audit security vulnerabilities | `auditing-security` |
| Review pull requests | `reviewing-prs` |
| Analyze performance bottlenecks | `analyzing-performance` |
| Explain complex algorithms | `explaining-algorithms` |

### Execution Skills

| Intent | Derived Name |
|--------|--------------|
| Deploy to production | `deploying-production` |
| Run integration tests | `running-integration` |
| Test API endpoints | `testing-endpoints` |

### Data Skills

| Intent | Derived Name |
|--------|--------------|
| Query analytics data | `querying-analytics` |
| Format log output | `formatting-logs` |
| Validate API responses | `validating-responses` |

---

## Scoped Skill Examples

For monorepos with distinct areas, add a scope prefix:

| Scope | Example Names |
|-------|---------------|
| `frontend-` | `frontend-scaffolding-components`, `frontend-auditing-accessibility` |
| `api-` | `api-generating-routes`, `api-validating-schemas` |
| `infra-` | `infra-deploying-services`, `infra-auditing-security` |
| `docs-` | `docs-generating-api`, `docs-reviewing-structure` |
| `db-` | `db-writing-migrations`, `db-optimizing-queries` |

---

## Non-Gerund Names

Some skills have domain-specific names that do not follow the gerund-noun pattern. This is acceptable when the name is well-known and the description is clear:

| Name | Why It Works |
|------|-------------|
| `generate-skill` | Established tool name |
| `generate-agent` | Established tool name |
| `compliance-openapi` | Domain-specific compound |
| `ship` | Single action verb, universally understood |

The key question: **Will someone reading this name understand what the skill does?** If yes, the name works.
