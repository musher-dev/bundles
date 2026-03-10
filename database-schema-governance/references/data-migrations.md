# Data Migrations

Atlas manages database **structure** (DDL) — tables, columns, indexes, constraints. It
does not manage **data** (DML) — INSERT, UPDATE, DELETE. This document covers the
boundary between schema and data changes, and the patterns for handling changes that
touch both.

## Schema vs Data

| Change Type | Tool | Examples |
|-------------|------|---------|
| Structural (DDL) | Atlas | Add table, add column, add index, add constraint |
| Data (DML) | Application script | Backfill column values, seed data, fix inconsistencies |
| Hybrid | Multi-stage | Add NOT NULL column to populated table, rename with data preservation |

Atlas generates `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX`, `DROP` — pure schema
operations. It does not generate `INSERT`, `UPDATE`, or `DELETE`.

For pure data changes (backfilling a column, seeding reference data), write a standalone
script. For changes that require both schema and data work, use the expand-contract
pattern.

## The Expand-Contract Pattern

When a schema change depends on data being in a certain state (e.g., adding a NOT NULL
column to a table that already has rows), you cannot do it in one step. The
expand-contract pattern breaks it into stages.

### Example: Adding a Required Column to an Existing Table

You want to add a `region` column to the `accounts` table, NOT NULL, with no default.

**Stage 1: Expand — Add the column as nullable (Atlas)**

Edit the model:

```sql
-- Add the column as nullable
ALTER TABLE accounts ADD COLUMN region VARCHAR(10);
```

Generate and apply:

```bash
atlas migrate diff --env local
atlas migrate lint --env local --latest 1
atlas migrate apply --env local
```

**Stage 2: Backfill — Populate existing rows (application script)**

Write a standalone script that backfills existing rows:

```sql
UPDATE accounts SET region = 'us-east-1' WHERE region IS NULL;
```

For large tables, batch the updates (see "Use Batching for Large Tables" below).

**Stage 3: Contract — Make the column NOT NULL (Atlas)**

Update the model to make the column required, then generate a migration:

```sql
-- Atlas generates this after the model is updated
ALTER TABLE accounts ALTER COLUMN region SET NOT NULL;
```

Generate and apply:

```bash
atlas migrate diff --env local
atlas migrate lint --env local --latest 1
atlas migrate apply --env local
```

### Why Three Stages?

A single migration with `ADD COLUMN region varchar(10) NOT NULL` fails on a populated
table because existing rows have NULL for the new column. Atlas lint catches this as
`MF102` (non-nullable column without default).

The expand-contract pattern:

1. **Expand** — the schema accommodates both old and new data
2. **Backfill** — existing data is brought up to the new standard
3. **Contract** — the schema enforces the new standard

## When to Use Multi-Stage

| Scenario | Why Single-Step Fails | Multi-Stage Approach |
|----------|----------------------|---------------------|
| Add NOT NULL column to populated table | Existing rows have no value | Nullable -> backfill -> NOT NULL |
| Add unique constraint to populated column | Existing duplicates violate constraint | Deduplicate -> add constraint |
| Rename a column with data preservation | BC102 lint, application breaks | Add new column -> copy data -> drop old column |
| Split a column into two | Data must be parsed/transformed | Add new columns -> transform data -> drop old column |
| Merge two columns into one | Data must be combined | Add new column -> combine data -> drop old columns |

## Running Backfill Scripts

### Keep Scripts Separate

Backfill scripts are standalone Python files. They are **not** embedded in migration
SQL files. This keeps schema changes (deterministic, generated) separate from data
changes (application-specific, often manual).

### Use Batching for Large Tables

For tables with many rows, batch the updates to avoid long-running transactions:

```python
"""Backfill region for existing accounts (batched)."""

import asyncio
from sqlalchemy import text
from your_app.database.engine import get_async_session

BATCH_SIZE = 1000

async def backfill():
    async with get_async_session() as session:
        while True:
            result = await session.execute(
                text("""
                    UPDATE accounts
                    SET region = 'us-east-1'
                    WHERE id IN (
                        SELECT id FROM accounts
                        WHERE region IS NULL
                        LIMIT :batch_size
                    )
                """),
                {"batch_size": BATCH_SIZE},
            )
            await session.commit()

            if result.rowcount == 0:
                break

asyncio.run(backfill())
```

### Idempotency

Backfill scripts should be safe to run multiple times. Use `WHERE region IS NULL`
(or equivalent) to skip rows that have already been backfilled. If the script is
interrupted and restarted, it should resume where it left off.

### Ordering

The typical sequence for a PR that involves both schema and data changes:

1. **PR 1:** Expand migration (add nullable column) + backfill script
2. Run backfill script in each environment after deploying PR 1
3. **PR 2:** Contract migration (add NOT NULL constraint)

Keeping these in separate PRs ensures the backfill has completed before the constraint
is enforced. For development environments where the table is empty or small, both PRs
can be combined.
