---
name: governing-schema-documentation
version: 1.0.0
user-invocable: false
description: >-
  Enforce PostgreSQL schema documentation standards using COMMENT ON, Atlas comment
  attributes, and custom linting rules. Covers what to document (constraint rationale,
  boolean semantics, historical anomalies), documentation enforcement via CI/CD, and
  schema.rule.hcl linting. Use when auditing schema documentation completeness or
  establishing documentation standards. Triggered by: schema documentation, COMMENT ON,
  Atlas comment, schema linting, documentation standard, column comment, table comment,
  schema rule.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Schema Documentation Governance

Standards and enforcement for PostgreSQL schema documentation using COMMENT ON and Atlas linting.

## Purpose

Schema documentation is the difference between a database that a team can maintain and one that only the original author understands. Column names convey what data is stored; comments convey why the column exists, what constraints mean, and what edge cases to watch for. Without documentation, every schema question requires archeological investigation through application code, git history, and tribal knowledge.

Use this skill when establishing documentation standards for a new schema, auditing existing schemas for documentation gaps, or configuring Atlas linting rules to enforce documentation.

---

## COMMENT ON Standards

PostgreSQL's `COMMENT ON` stores documentation directly in the database catalog, accessible via `\d+` in psql, pg_description queries, and schema inspection tools.

### Syntax

```sql
COMMENT ON TABLE users IS 'Registered platform users. One row per human identity.';
COMMENT ON COLUMN users.is_verified IS 'True after email verification link is clicked. Required for API access.';
COMMENT ON COLUMN users.deleted_at IS 'Soft-delete timestamp. NULL means active. Filtered by partial index idx_users_active.';
COMMENT ON INDEX idx_users_email IS 'Supports login-by-email lookup. Expression index on lower(email) for case-insensitive matching.';
COMMENT ON CONSTRAINT check_users_email_format ON users IS 'Validates basic email format. Application-layer validation is more thorough; this prevents obviously invalid data from migrations or manual SQL.';
```

### What to Document

| Object | Document | Example Comment |
|--------|----------|-----------------|
| Tables | Purpose, cardinality expectation, lifecycle | `'API keys for programmatic access. Soft-deleted via revoked_at, never hard-deleted.'` |
| Columns with constraints | Constraint rationale | `'Must be positive. Zero-value invoices use a separate credit_notes table.'` |
| Boolean columns | What true/false means in business terms | `'True when the org has completed onboarding. Controls dashboard feature visibility.'` |
| Nullable columns | What NULL signifies | `'NULL means the key has not been used yet. Updated async on each API call.'` |
| Timestamp columns | Mutability and trigger behavior | `'Set by trigger on every UPDATE. Never set by application code directly.'` |
| JSONB columns | Expected structure and boundaries | `'Unstructured request metadata. Keys vary by action type. Not queried in WHERE clauses.'` |
| Indexes | What queries the index supports | `'Supports tenant-scoped chronological queries on the audit log dashboard.'` |
| Constraints | Why the constraint exists | `'Prevents overlapping date ranges for the same resource. Uses tstzrange exclusion.'` |
| Historical anomalies | Why something looks wrong | `'Column is varchar(100) instead of text for legacy compatibility with external system X. Safe to migrate after Q3 deprecation.'` |

### What NOT to Document

- **Self-evident columns:** `COMMENT ON COLUMN users.email IS 'The user email'` adds no value
- **Implementation details that change:** `'Uses B-tree index'` — the catalog already shows this
- **Application behavior:** `'Triggers a welcome email'` — this belongs in application docs

---

## Atlas Comment Attribute

In Atlas HCL schema files, the `comment` attribute maps directly to PostgreSQL `COMMENT ON`:

```hcl
table "users" {
  schema = schema.public
  comment = "Registered platform users. One row per human identity."

  column "id" {
    type = bigint
    identity {}
  }

  column "is_verified" {
    type    = boolean
    null    = false
    default = false
    comment = "True after email verification link is clicked. Required for API access."
  }

  column "deleted_at" {
    type    = timestamptz
    null    = true
    comment = "Soft-delete timestamp. NULL means active. Filtered by partial index idx_users_active."
  }
}
```

When using SQLModel/SQLAlchemy as the schema source, comments are set via the `comment` parameter:

```python
class User(SQLModelBase, table=True):
    __tablename__ = "users"
    __table_args__ = {"comment": "Registered platform users. One row per human identity."}

    is_verified: bool = Field(
        default=False,
        sa_column_kwargs={"comment": "True after email verification link is clicked. Required for API access."}
    )
```

---

## Custom Linting Rules

Atlas supports custom linting rules via `schema.rule.hcl` files that enforce documentation standards as part of CI/CD.

### Require Table Comments

```hcl
# schema.rule.hcl
rule "table-comment-required" {
  description = "Every table must have a COMMENT ON"

  match {
    table {}
  }

  assert {
    condition = table.comment != ""
    message   = "Table '${table.name}' is missing a COMMENT ON statement"
  }
}
```

### Require Column Comments on Specific Types

```hcl
rule "boolean-comment-required" {
  description = "Boolean columns must document what true/false means"

  match {
    column {
      type = "boolean"
    }
  }

  assert {
    condition = column.comment != ""
    message   = "Boolean column '${table.name}.${column.name}' must have a comment explaining true/false semantics"
  }
}

rule "jsonb-comment-required" {
  description = "JSONB columns must document expected structure"

  match {
    column {
      type = "jsonb"
    }
  }

  assert {
    condition = column.comment != ""
    message   = "JSONB column '${table.name}.${column.name}' must have a comment documenting expected structure"
  }
}

rule "nullable-comment-required" {
  description = "Nullable columns must document what NULL signifies"

  match {
    column {
      null = true
    }
  }

  assert {
    condition = column.comment != ""
    message   = "Nullable column '${table.name}.${column.name}' must have a comment explaining what NULL means"
  }
}
```

### Integrating Rules in atlas.hcl

```hcl
lint {
  rule "schema.rule.hcl" {
    error = true
  }
}
```

---

## Documentation Enforcement via CI/CD

### Pipeline Integration

Add a documentation check to the migration CI pipeline:

```yaml
# In CI/CD pipeline
steps:
  - name: Lint migrations
    run: atlas migrate lint --env local --latest 1

  - name: Check documentation
    run: |
      # Count tables without comments
      UNDOCUMENTED=$(atlas schema inspect --env local --format '{{ range .Tables }}{{ if eq .Comment "" }}{{ .Name }}{{ printf "\n" }}{{ end }}{{ end }}' | wc -l)
      if [ "$UNDOCUMENTED" -gt 0 ]; then
        echo "ERROR: $UNDOCUMENTED tables without COMMENT ON"
        exit 1
      fi
```

### Audit Query for Existing Schemas

```sql
-- Find tables without comments
SELECT c.relname AS table_name
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
LEFT JOIN pg_description d ON d.objoid = c.oid AND d.objsubid = 0
WHERE n.nspname = 'public'
  AND c.relkind = 'r'
  AND d.description IS NULL
ORDER BY c.relname;

-- Find columns without comments
SELECT
    c.relname AS table_name,
    a.attname AS column_name,
    format_type(a.atttypid, a.atttypmod) AS data_type
FROM pg_attribute a
JOIN pg_class c ON c.oid = a.attrelid
JOIN pg_namespace n ON n.oid = c.relnamespace
LEFT JOIN pg_description d ON d.objoid = a.attrelid AND d.objsubid = a.attnum
WHERE n.nspname = 'public'
  AND c.relkind = 'r'
  AND a.attnum > 0
  AND NOT a.attisdropped
  AND d.description IS NULL
ORDER BY c.relname, a.attnum;
```

---

## Documentation Quality Guidelines

### Comment Length

- **Tables:** 1-2 sentences. Purpose + key lifecycle notes.
- **Columns:** 1 sentence. Focus on semantics, not the obvious.
- **Constraints:** 1 sentence. Focus on why, not what.
- **Maximum:** Keep comments under 200 characters. Longer documentation belongs in external docs linked from the comment.

### Comment Style

| Do | Don't |
|----|-------|
| `'NULL means the key has not been used yet.'` | `'This column can be null.'` |
| `'Prevents duplicate scope grants per key.'` | `'Unique constraint on key_id and scope_name.'` |
| `'Legacy: varchar(100) for compat with System X. Migrate to text after Q3.'` | `'Do not change this column.'` |
| `'Set by before-update trigger. Never set by application.'` | `'Updated automatically.'` |

### When to Update Comments

- When a column's semantics change (even if the type doesn't)
- When a constraint is added or modified
- When a historical anomaly is resolved (remove the legacy note)
- During every schema review — stale comments are worse than no comments

---

## Anti-Patterns

### 1. Tautological Comments

`COMMENT ON COLUMN users.email IS 'The user email address'` — restates the column name without adding information. Either add real context or omit the comment.

### 2. Comments as Code Documentation

`COMMENT ON TABLE users IS 'Used by UserService.getById()'` — application-level references change constantly and will go stale. Document schema semantics, not application usage.

### 3. Missing NULL Semantics

A nullable column without a comment explaining what NULL means. Every nullable column must document the business meaning of absence.

### 4. Undocumented JSONB Structure

A JSONB column with no comment about expected keys, value types, or query patterns. Without structure documentation, every developer must read application code to understand the data.

---

## Examples

### Example 1: Auditing Documentation Completeness

**Schema under review:** `api_keys` table with 14 columns, 0 comments.

**Audit findings:**
- **Table comment:** MISSING — add purpose and lifecycle notes
- **`principal_type`:** MISSING — must document enum values and their meanings
- **`key_hash`:** MISSING — must document hashing algorithm and that plaintext is never stored
- **`revoked_at`:** MISSING — must document NULL semantics (NULL = active)
- **`last_used_at`:** MISSING — must document async update behavior
- **`checksum`:** MISSING — must document its role in gateway pre-validation

**Priority:** Boolean columns, nullable columns, and JSONB columns first. Then all remaining columns.

### Example 2: Well-Documented Table

```sql
COMMENT ON TABLE api_keys IS 'API credentials for programmatic access. Soft-deleted via revoked_at. Keys belong to exactly one org via org_id.';
COMMENT ON COLUMN api_keys.principal_type IS 'Polymorphic discriminator: membership (human) or service_account (machine).';
COMMENT ON COLUMN api_keys.key_hash IS 'SHA-256 hash of the full API key. Plaintext is shown once at creation and never stored.';
COMMENT ON COLUMN api_keys.revoked_at IS 'NULL means active. Non-null means permanently revoked. Never hard-delete rows.';
COMMENT ON COLUMN api_keys.last_used_at IS 'Updated asynchronously on each API call. NULL means never used. Used for stale key detection.';
COMMENT ON COLUMN api_keys.checksum IS 'CRC32/Base62 checksum for gateway-level pre-validation before database lookup.';
```

**Assessment:** All high-priority columns documented. Comments explain semantics, not obvious facts.

### Example 3: Atlas Lint Catching Missing Comments

**CI output:**
```
schema.rule.hcl: boolean-comment-required: Boolean column 'users.is_verified' must have a comment explaining true/false semantics
schema.rule.hcl: nullable-comment-required: Nullable column 'api_keys.last_used_at' must have a comment explaining what NULL means
schema.rule.hcl: jsonb-comment-required: JSONB column 'audit_logs.metadata' must have a comment documenting expected structure
```

**Action:** Add COMMENT ON for each flagged column before the migration can merge.
