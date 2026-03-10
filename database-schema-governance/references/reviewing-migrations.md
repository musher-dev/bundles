# Reviewing Migrations

Every generated migration must be reviewed before committing. This document covers
how to read generated SQL, understand Atlas lint codes, and handle overrides when
a lint warning is intentional.

## Reading Generated SQL

Migration files live in a `migrations/` directory and are named with timestamps:

```
migrations/
├── atlas.sum
├── 20250101120000_initial.sql
├── 20250115120000_add_accounts.sql
└── 20250220120000_add_invoices.sql
```

A typical migration file:

```sql
CREATE TABLE "invoices" (
    "id" uuid NOT NULL DEFAULT gen_random_uuid(),
    "created_at" timestamptz NOT NULL DEFAULT now(),
    "updated_at" timestamptz,
    "deleted_at" timestamptz,
    "workspace_id" uuid NOT NULL,
    "amount_cents" integer NOT NULL,
    CONSTRAINT "pk_invoices" PRIMARY KEY ("id"),
    CONSTRAINT "fk_invoices_workspace_id_workspaces" FOREIGN KEY ("workspace_id")
        REFERENCES "workspaces" ("id")
);
CREATE INDEX "ix_invoices_created_at" ON "invoices" ("created_at");
CREATE INDEX "ix_invoices_deleted_at" ON "invoices" ("deleted_at");
CREATE INDEX "ix_invoices_workspace_id" ON "invoices" ("workspace_id");
```

When reviewing, check:

1. **Table and column names** match the SQLModel definition
2. **Constraint names** follow the naming convention (`pk_`, `fk_`, `ix_`, `uq_`, `ck_`)
3. **No unexpected DROP statements** — Atlas may generate drops if you renamed something
4. **Correct types** — `timestamptz` for datetimes, `uuid` for IDs, `varchar(n)` for bounded strings
5. **Indexes on foreign keys** — every FK column should have a corresponding index

## Atlas Directives

Atlas supports special SQL comments that control migration behavior.

### Transaction Mode

By default, Atlas wraps each migration in a transaction. Some operations (like
`CREATE INDEX CONCURRENTLY`) cannot run inside a transaction. Use the `txmode` directive:

```sql
-- atlas:txmode none

CREATE INDEX CONCURRENTLY "ix_users_email" ON "users" ("email");
```

This is **required** whenever a migration contains `CONCURRENTLY`. Without it,
PostgreSQL will error.

### Lint Suppression

Suppress a specific lint code for a migration file:

```sql
-- atlas:nolint DS102

DROP TABLE "deprecated_feature";
```

You can suppress multiple codes:

```sql
-- atlas:nolint DS102 DS103

DROP TABLE "deprecated_feature";
ALTER TABLE "users" DROP COLUMN "legacy_field";
```

Lint suppression directives must appear at the top of the migration file, before any
SQL statements.

## Lint Codes Reference

### Destructive Changes (DS)

Operations that permanently destroy data.

| Code | Rule | What It Catches |
|------|------|-----------------|
| DS101 | Drop schema | `DROP SCHEMA` statements |
| DS102 | Drop table | `DROP TABLE` statements |
| DS103 | Drop column | `ALTER TABLE ... DROP COLUMN` statements |

These are errors by default. Dropping a table or column destroys data permanently.

### Data-Dependent Changes (MF)

Operations that may fail or lose data depending on existing rows.

| Code | Rule | What It Catches |
|------|------|-----------------|
| MF101 | Add unique index on existing data | Unique constraint on a column that may have duplicates |
| MF102 | Add non-nullable column without default | `ALTER TABLE ADD COLUMN ... NOT NULL` without `DEFAULT` |
| MF103 | Modify column type | Changing column type that may lose data |
| MF104 | Add check constraint to existing data | Check constraint that existing rows may violate |

MF102 is the most common. If you add a required field to an existing table, you need
either a default value or a multi-stage migration (see [Data Migrations](data-migrations.md)).

### Backward Incompatible (BC)

Operations that break existing application code.

| Code | Rule | What It Catches |
|------|------|-----------------|
| BC101 | Rename table | `ALTER TABLE ... RENAME TO` |
| BC102 | Rename column | `ALTER TABLE ... RENAME COLUMN` |

Renames break any code that references the old name. Use the expand-contract pattern
instead: add the new name, migrate data, drop the old name.

### Concurrent Index (PG)

PostgreSQL-specific rules for safe index creation.

| Code | Rule | What It Catches |
|------|------|-----------------|
| PG101 | Non-concurrent index on existing table | `CREATE INDEX` without `CONCURRENTLY` |
| PG102 | Missing `txmode none` directive | Concurrent index inside a transaction block |
| PG103 | Index not verified after creation | Concurrent index may be left `INVALID` |

`CREATE INDEX CONCURRENTLY` avoids locking the table during index creation. The diff
policy (`concurrent_index.create = true`) makes Atlas generate concurrent indexes
automatically, but you should verify the `txmode none` directive is present.

## Overriding Lint Warnings

Sometimes a lint warning is expected. For example, dropping a table that was
intentionally deprecated.

### File-Level Override

Add a `nolint` directive at the top of the migration file:

```sql
-- atlas:nolint DS102
-- Dropping deprecated_feature table — approved in PR #42

DROP TABLE "deprecated_feature";
```

Always include a comment explaining why the override is acceptable.

### Config-Level Override

For project-wide overrides (rare), adjust the lint block in `atlas.hcl`:

```hcl
lint {
  destructive {
    error = false  # Downgrade to warning (not recommended)
  }
}
```

Prefer file-level overrides over config-level changes. Config-level changes affect
all migrations, file-level overrides are scoped to one migration.

## Editing Generated Migrations

Atlas generates migrations, but sometimes you need to edit them:

- Adding a `-- atlas:txmode none` directive for concurrent operations
- Adding data backfill SQL alongside schema changes
- Splitting a large migration into smaller steps

**After editing any migration file, you must regenerate the checksum:**

```bash
atlas migrate hash
```

If you skip this step, `atlas migrate validate` will fail because `atlas.sum`
no longer matches the file contents.

**When editing is acceptable:**

- Adding directives (`txmode`, `nolint`)
- Adding comments for reviewer context
- Reordering independent statements within a migration

**When editing is not acceptable:**

- Changing the schema semantics (add/remove columns, change types) — regenerate instead
- Editing migrations that have already been applied to any shared environment
