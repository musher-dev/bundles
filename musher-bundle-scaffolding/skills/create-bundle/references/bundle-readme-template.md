# Bundle README Template

Every bundle directory must contain a `README.md` file at its root. This document defines the template.

## Template

```markdown
# {Display Name}

{One-sentence description of what the bundle does.}

## Metadata

- **Domain:** {domain term from controlled vocabulary}
- **Risk Level:** {low | medium | high}
- **Technologies:** {comma-separated list, or "None (vendor-agnostic)"}
- **Aliases:** {comma-separated shorthand names}

## Components

### Skills
- {skill list, or "None"}

### Agents
- {agent list, or "None"}

### Tools
- {tool list, or "None"}
```

## Field Descriptions

### Required Fields

| Field | Description |
|---|---|
| Display Name | Human-readable name in title case. Example: `Database Schema Governance` |
| Description | One-sentence summary of what the bundle does |
| Domain | Domain term from the controlled vocabulary |
| Risk Level | One of: `low`, `medium`, `high`. Reflects the blast radius of the bundle's operations |

### Optional Fields

| Field | Default | Description |
|---|---|---|
| Technologies | `None (vendor-agnostic)` | Technology dependencies (e.g., `aws, terraform`). "None (vendor-agnostic)" if none |
| Aliases | *(empty)* | Alternative names or abbreviations for discoverability |
| Skills | `None` | Skill identifiers using `gerund-concept` convention (e.g., `reviewing-migrations`) |
| Agents | `None` | Agent identifiers using `role_noun` convention (e.g., `schema_reviewer`) |
| Tools | `None` | Tool identifiers using `verb_noun` convention (e.g., `validate_schema`) |

## Component Naming Conventions

| Component Type | Convention | Separator | Example |
|---|---|---|---|
| Tools | `verb_noun` | underscore | `validate_schema`, `generate_migration` |
| Skills | `gerund-concept` | kebab-case | `reviewing-migrations`, `enforcing-standards` |
| Agents | `role_noun` | underscore | `schema_reviewer`, `compliance_auditor` |

## Additional Sections

Bundles with extra component types (e.g., `rules`, `references`) should add corresponding sections under Components:

```markdown
### Rules
- {rule list}

### References
- {reference list}
```

## Example

```markdown
# Database Schema Governance

Enforces schema standards, reviews migrations, and audits drift across database environments.

## Metadata

- **Domain:** database-schema
- **Risk Level:** high
- **Technologies:** None (vendor-agnostic)
- **Aliases:** schema-gov, db-governance

## Components

### Skills
- None

### Agents
- None

### Tools
- None
```
