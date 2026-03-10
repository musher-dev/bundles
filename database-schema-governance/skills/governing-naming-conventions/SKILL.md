---
name: governing-naming-conventions
version: 1.0.0
user-invocable: false
description: >-
  Audit and enforce PostgreSQL naming conventions for tables, columns, primary keys,
  foreign keys, booleans, timestamps, indexes, and constraints. Use when reviewing
  schema identifiers, enforcing naming standards, or auditing existing schemas for
  naming consistency. Triggered by: naming convention, identifier naming, table name,
  column name, constraint name, index name, snake_case, pk naming, fk naming.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Naming Convention Governance

Comprehensive rules for PostgreSQL identifier naming across tables, columns, keys, indexes, and constraints.

## Purpose

Consistent naming conventions eliminate ambiguity, reduce cognitive load during code review, and enable automated linting. A schema where every identifier follows predictable patterns is one where developers can guess column names without checking documentation. This skill codifies the canonical naming rules and provides detection methods for violations.

Use this skill when reviewing new schema designs, auditing existing databases for naming drift, or configuring Atlas naming lint rules.

---

## Table Naming

| Rule | Convention | Example | Rationale |
|------|-----------|---------|-----------|
| Plural | Always plural | `users`, `organizations` | A table holds many rows; plural reflects the collection |
| snake_case | Lowercase with underscores | `api_keys`, `audit_logs` | PostgreSQL folds unquoted identifiers to lowercase |
| No prefixes | No `tbl_` or schema-name prefixes | `users` not `tbl_users` | Prefixes add noise without information |
| Join tables | Alphabetical combination | `organization_users` | Predictable ordering eliminates guesswork |

**Detection:** Query `information_schema.tables` for names containing uppercase, singular nouns, or prefix patterns like `tbl_`.

---

## Column Naming

| Rule | Convention | Example | Rationale |
|------|-----------|---------|-----------|
| Singular | Describes one value | `email` not `emails` | Each cell holds one value |
| snake_case | Lowercase with underscores | `first_name`, `org_id` | Consistency with table names |
| No table prefix | Never repeat the table name | `users.name` not `users.user_name` | The table context is always known in queries |
| Descriptive | Self-documenting | `expires_at` not `exp` | Abbreviations create ambiguity |

**Exception:** The primary key column `id` is the one case where the table name prefix is intentionally omitted for brevity. Foreign keys use the referenced table name as prefix (see below).

---

## Primary Key Naming

| Rule | Convention | Example |
|------|-----------|---------|
| Column name | `id` | `users.id` |
| Constraint name | `pk_<table>` | `pk_users` |
| Type | `bigint` (generated) or `uuid` | `id bigint GENERATED ALWAYS AS IDENTITY` |

**Anti-pattern:** Naming the PK `user_id` on the `users` table. This creates confusion with the FK pattern used on other tables.

---

## Foreign Key Naming

| Rule | Convention | Example |
|------|-----------|---------|
| Column name | `<referenced_table_singular>_id` | `api_keys.org_id` → `organizations.id` |
| Constraint name | `fk_<table>_<column>_<referenced_table>` | `fk_api_keys_org_id_organizations` |

**Self-referential FK:** Use a descriptive prefix: `parent_id`, `manager_id`, `reply_to_id`.

**Detection:** Scan for FK columns that don't match the `<table>_id` pattern, or FK constraints that don't follow the `fk_` prefix convention.

---

## Boolean Column Naming

| Prefix | Usage | Example |
|--------|-------|---------|
| `is_` | State or status | `is_active`, `is_verified` |
| `has_` | Possession or capability | `has_mfa`, `has_billing` |
| `does_` | Action capability (rare) | `does_notify` |

**Anti-pattern:** Boolean columns named `active`, `verified`, or `enabled` without a prefix. These read ambiguously in WHERE clauses: `WHERE active` vs `WHERE is_active`.

**Detection:** Query for `boolean` columns whose names don't start with `is_`, `has_`, or `does_`.

---

## Timestamp Column Naming

| Column | Convention | Notes |
|--------|-----------|-------|
| Creation time | `created_at` | Immutable, set once at INSERT |
| Update time | `updated_at` | Set on every UPDATE |
| Soft-delete time | `deleted_at` or `revoked_at` | Domain-specific verb preferred |
| Domain events | `<verb>_at` | `published_at`, `verified_at`, `expires_at` |

**Rule:** All timestamp columns end with `_at`. All timestamps use `timestamptz` (see data type governance).

**Anti-pattern:** Columns named `created`, `modified`, `timestamp`, or `date` without the `_at` suffix.

---

## Index Naming

| Type | Convention | Example |
|------|-----------|---------|
| Standard index | `idx_<table>_<columns>` | `idx_users_email` |
| Unique index | `unique_<table>_<columns>` | `unique_users_email` |
| Partial index | `idx_<table>_<columns>_<condition>` | `idx_api_keys_expires_at_active` |
| Expression index | `idx_<table>_<expression>` | `idx_users_lower_email` |

**Detection:** Query `pg_indexes` for index names that don't follow the `idx_` or `unique_` prefix convention.

---

## Constraint Naming

| Type | Convention | Example |
|------|-----------|---------|
| Primary key | `pk_<table>` | `pk_users` |
| Foreign key | `fk_<table>_<column>_<ref_table>` | `fk_api_keys_org_id_organizations` |
| Unique | `unique_<table>_<columns>` | `unique_users_email` |
| Check | `check_<table>_<description>` | `check_users_email_format` |
| Exclusion | `excl_<table>_<description>` | `excl_schedules_no_overlap` |

**Why explicit names matter:** PostgreSQL auto-generates constraint names like `users_email_key` or `api_keys_org_id_fkey`. These auto-generated names are unpredictable across environments and make migration scripts brittle. Always name constraints explicitly.

---

## SQLAlchemy/SQLModel Naming Convention

Map these rules to the SQLAlchemy `naming_convention` dict:

```python
NAMING_CONVENTION: dict[str, str] = {
    "ix": "idx_%(column_0_label)s",
    "uq": "unique_%(table_name)s_%(column_0_name)s",
    "ck": "check_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}
```

This ensures all ORM-generated constraints follow the same naming rules as hand-written SQL.

---

## Anti-Patterns

### 1. Auto-Generated Constraint Names

Relying on PostgreSQL's auto-generated names (`users_pkey`, `api_keys_org_id_fkey`). These names vary across environments and break migration scripts that reference constraints by name.

### 2. Table Name as Column Prefix

`users.user_name`, `users.user_email`. The table context is always known — the prefix is redundant and makes queries verbose.

### 3. Inconsistent Pluralization

Mixing `user` and `organizations` in the same schema. Pick plural (recommended) and enforce it everywhere.

### 4. Abbreviated Column Names

`usr_nm`, `org`, `amt`. Abbreviations save keystrokes but cost readability. Disk space for longer column names is irrelevant.

---

## Examples

### Example 1: Naming Audit of a Users Table

**Schema under review:**
```sql
CREATE TABLE user (
    user_id SERIAL PRIMARY KEY,
    userName VARCHAR(255),
    active BOOLEAN,
    created TIMESTAMP
);
```

**Findings:**
- **Table name:** FAIL — singular `user`, should be `users`
- **PK column:** FAIL — `user_id` should be `id`; constraint should be named `pk_users`
- **Column `userName`:** FAIL — camelCase, should be `name` (no table prefix, snake_case)
- **Column `active`:** FAIL — boolean without prefix, should be `is_active`
- **Column `created`:** FAIL — missing `_at` suffix, should be `created_at`; type should be `timestamptz`

### Example 2: Foreign Key Naming Review

**Schema under review:**
```sql
ALTER TABLE api_keys ADD CONSTRAINT api_keys_organization_fkey
    FOREIGN KEY (organization) REFERENCES organizations(id);
```

**Findings:**
- **FK column:** FAIL — `organization` should be `org_id` (singular referenced table + `_id`)
- **Constraint name:** FAIL — should be `fk_api_keys_org_id_organizations`

### Example 3: Well-Named Schema

**Schema under review:**
```sql
CREATE TABLE organizations (
    id bigint GENERATED ALWAYS AS IDENTITY,
    name text NOT NULL,
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT now(),
    updated_at timestamptz NOT NULL,
    CONSTRAINT pk_organizations PRIMARY KEY (id)
);
```

**Findings:** All naming conventions pass. Table is plural, PK is `id` with explicit constraint name, boolean has `is_` prefix, timestamps use `_at` suffix with `timestamptz`.
