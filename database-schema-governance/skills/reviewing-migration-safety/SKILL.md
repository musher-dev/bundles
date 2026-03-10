---
name: reviewing-migration-safety
version: 1.0.0
user-invocable: false
description: >-
  Review PostgreSQL migration safety including lock risk assessment, timeout
  mandates, CONCURRENTLY enforcement, expand-contract choreography, WAL impact
  analysis, backward compatibility checks, and Atlas lint code mapping. Use when
  reviewing migrations for production safety, evaluating lock impact, or planning
  zero-downtime schema changes. Triggered by: migration safety, lock risk, lock
  timeout, statement timeout, CONCURRENTLY, expand-contract, WAL impact, backward
  compatibility, Atlas lint, destructive change, zero-downtime migration.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Migration Safety Review

Structured detection and remediation framework for PostgreSQL migration risks in production environments.

## Purpose

Migrations are the highest-risk operations in database management. A single missing `CONCURRENTLY` keyword or an unguarded `ALTER TABLE` can lock a production table for minutes, cascading into connection pool exhaustion and application downtime. This skill provides the analytical framework for evaluating migration safety before execution — covering lock acquisition, timeout configuration, backward compatibility, and WAL impact.

Use this skill when reviewing migration files before merge, evaluating the production impact of schema changes, or designing multi-phase migration strategies for zero-downtime deployments.

---

## Lock Risk Assessment

Every DDL statement acquires a lock. The critical concern is `ACCESS EXCLUSIVE` — it blocks all other operations including `SELECT`.

### Lock Modes by Operation

| Operation | Lock Acquired | Blocks Reads | Blocks Writes |
|-----------|--------------|--------------|---------------|
| `CREATE INDEX` | `SHARE` | No | Yes |
| `CREATE INDEX CONCURRENTLY` | `SHARE UPDATE EXCLUSIVE` | No | No |
| `ALTER TABLE ADD COLUMN` (nullable, no default) | `ACCESS EXCLUSIVE` | Yes | Yes |
| `ALTER TABLE ADD COLUMN ... DEFAULT` (PG 11+) | `ACCESS EXCLUSIVE` (brief) | Yes (brief) | Yes (brief) |
| `ALTER TABLE DROP COLUMN` | `ACCESS EXCLUSIVE` | Yes | Yes |
| `ALTER TABLE ADD CONSTRAINT ... NOT VALID` | `SHARE UPDATE EXCLUSIVE` | No | No |
| `ALTER TABLE VALIDATE CONSTRAINT` | `SHARE UPDATE EXCLUSIVE` | No | No |
| `ALTER TABLE SET NOT NULL` | `ACCESS EXCLUSIVE` | Yes | Yes |
| `DROP TABLE` | `ACCESS EXCLUSIVE` | Yes | Yes |
| `DROP INDEX` | `ACCESS EXCLUSIVE` | Yes | Yes |
| `DROP INDEX CONCURRENTLY` | `SHARE UPDATE EXCLUSIVE` | No | No |

### Lock Queue Cascading

PostgreSQL locks are queued. If Transaction A holds a `SHARE` lock and Transaction B requests `ACCESS EXCLUSIVE`, Transaction B waits. But now Transaction C requesting `SHARE` also waits behind B — even though C is compatible with A. This cascading effect means a single `ACCESS EXCLUSIVE` request can block all subsequent queries on the table.

**Detection:** Any migration containing `ALTER TABLE`, `DROP TABLE`, `DROP INDEX` (without `CONCURRENTLY`), or `CREATE INDEX` (without `CONCURRENTLY`) on tables with active traffic is a lock risk.

---

## Timeout Mandates

Every migration session must set explicit timeouts to prevent unbounded lock waits.

### Required Settings

```sql
-- Set BEFORE any DDL operation
SET LOCAL lock_timeout = '5s';       -- Fail if lock not acquired within 5 seconds
SET LOCAL statement_timeout = '30s'; -- Fail if statement runs longer than 30 seconds
```

### Timeout Selection

| Table Size | lock_timeout | statement_timeout |
|------------|-------------|-------------------|
| < 100K rows | 5s | 30s |
| 100K - 1M rows | 3s | 60s |
| 1M - 10M rows | 2s | 120s |
| > 10M rows | 1s | Use batched approach |

**Rule:** If a migration cannot complete within `statement_timeout`, it must be redesigned as a batched operation or use a non-blocking approach.

---

## CONCURRENTLY Enforcement

Index operations on production tables must use `CONCURRENTLY` to avoid blocking writes.

### Rules

```sql
-- REQUIRED for production tables
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders (user_id);
DROP INDEX CONCURRENTLY idx_orders_old_index;

-- CONCURRENTLY cannot run inside a transaction block
-- In Atlas, annotate with:
-- atlas:txmode none
```

### CONCURRENTLY Constraints

| Constraint | Implication |
|------------|-------------|
| Cannot run in a transaction | Atlas requires `-- atlas:txmode none` annotation |
| Can fail partway through | Leaves an `INVALID` index; must DROP and retry |
| Requires two table scans | Slower than regular `CREATE INDEX` |
| Blocks other `CONCURRENTLY` operations | Only one per table at a time |

**Detection:** Scan migration files for `CREATE INDEX` or `DROP INDEX` without `CONCURRENTLY`. Flag as HIGH severity on any table with production traffic.

---

## Expand-Contract Choreography

The expand-contract pattern enables zero-downtime schema changes by separating the migration into phases that are individually backward-compatible.

### Phase Sequence

| Phase | Migration Action | Application Action |
|-------|-----------------|-------------------|
| **1. Expand** | Add new column/table (nullable) | Write to both old and new; read from old |
| **2. Backfill** | Batch-update existing rows | Continue dual-write |
| **3. Switch** | (none) | Read from new; write to both |
| **4. Contract** | Drop old column/table, add NOT NULL | Write only to new |

### Common Scenarios

**Renaming a column** (`address` to `billing_address`):
1. Add `billing_address` (nullable)
2. Deploy dual-write code
3. Backfill `billing_address` from `address` in batches
4. Switch reads to `billing_address`
5. Drop `address` column

**Changing a column type** (`varchar` to `text`):
1. Add new `text` column
2. Dual-write + backfill
3. Switch reads
4. Drop old column, rename new

**Adding NOT NULL constraint**:
1. Add CHECK constraint as `NOT VALID`: `ALTER TABLE t ADD CONSTRAINT ck_t_col_nn CHECK (col IS NOT NULL) NOT VALID;`
2. Backfill any NULL values
3. Validate: `ALTER TABLE t VALIDATE CONSTRAINT ck_t_col_nn;`
4. Optionally add formal `SET NOT NULL` (instantaneous after validated CHECK exists in PG 12+)

---

## WAL Impact Analysis

Write-Ahead Log (WAL) volume determines replication lag and recovery time. Migrations that generate excessive WAL can cause replica lag spikes.

### Operation WAL Costs

| Operation | WAL Impact | Alternative |
|-----------|-----------|-------------|
| `DELETE FROM large_table` | Heavy — logs every deleted row | `DROP TABLE` or `TRUNCATE` (instant, minimal WAL) |
| Mass `UPDATE` (backfill) | Heavy — logs every row change | Batch in chunks of 1,000-10,000 with `pg_sleep(0.1)` between batches |
| `CREATE INDEX` | Moderate | `CREATE INDEX CONCURRENTLY` generates more WAL but avoids locks |
| Partition `DETACH` | Minimal | Preferred over `DELETE` for removing old data |
| `TRUNCATE` | Minimal | Preferred over `DELETE` for full table clearing |

### Batching Strategy

```sql
-- Instead of: DELETE FROM events WHERE created_at < '2024-01-01';
-- Use batched approach:
DO $$
DECLARE
    batch_size CONSTANT int := 10000;
    rows_deleted int;
BEGIN
    LOOP
        DELETE FROM events
        WHERE id IN (
            SELECT id FROM events
            WHERE created_at < '2024-01-01'
            LIMIT batch_size
        );
        GET DIAGNOSTICS rows_deleted = ROW_COUNT;
        EXIT WHEN rows_deleted = 0;
        PERFORM pg_sleep(0.1);  -- Allow replication to catch up
        COMMIT;
    END LOOP;
END $$;
```

---

## Atlas Lint Code Mapping

Atlas lint detects destructive and backward-incompatible changes. Map lint codes to migration review decisions.

### Key Lint Codes

| Code | Meaning | Severity | Action |
|------|---------|----------|--------|
| `DS101` | Dropping table | Critical | Require expand-contract; verify no references |
| `DS102` | Dropping column | Critical | Verify column is unused in all application versions |
| `BC101` | Renaming table | High | Use expand-contract with view aliasing |
| `BC102` | Renaming column | High | Use expand-contract pattern |
| `MF101` | Missing `NOT NULL` on new column | Medium | Add NOT NULL with DEFAULT or use NOT VALID CHECK |
| `CD101` | Adding constraint without `NOT VALID` | High | Use `NOT VALID` + separate `VALIDATE` step |

---

## Backward Compatibility Checks

Every migration must be backward-compatible with the currently-deployed application version. This means the application code running during the migration must continue to function.

### Compatibility Matrix

| Change | Backward Compatible? | Requirement |
|--------|---------------------|-------------|
| Add nullable column | Yes | None |
| Add column with DEFAULT | Yes (PG 11+) | Brief ACCESS EXCLUSIVE lock |
| Drop column | No | Application must stop reading column first |
| Rename column | No | Use expand-contract |
| Change column type | No | Use expand-contract |
| Add NOT NULL | No | Use NOT VALID CHECK + VALIDATE |
| Drop table | No | Application must stop referencing table first |
| Add index CONCURRENTLY | Yes | Use `-- atlas:txmode none` |
| Add FK constraint NOT VALID | Yes | Validates only new rows |

---

## Anti-Patterns

### 1. Missing Lock Timeout

Running DDL migrations without `SET LOCAL lock_timeout`. A long-running query holds a conflicting lock, and the migration waits indefinitely — blocking all subsequent queries via lock queue cascading.

### 2. Non-Concurrent Index Creation

Using `CREATE INDEX` instead of `CREATE INDEX CONCURRENTLY` on production tables. Acquires a `SHARE` lock that blocks all writes for the duration of the index build.

### 3. Big-Bang Column Drop

Dropping a column in a single migration without verifying that no running application version reads it. Causes immediate errors in any request that references the column.

### 4. Monolithic Backfill

Running `UPDATE table SET new_col = old_col` on a multi-million row table in a single transaction. Generates massive WAL, holds row locks for the entire duration, and blocks autovacuum.

### 5. NOT NULL Without NOT VALID

Adding `ALTER TABLE t ALTER COLUMN c SET NOT NULL` directly on a large table. Requires a full table scan under ACCESS EXCLUSIVE lock. Use a CHECK constraint with NOT VALID instead.

### 6. Missing Transaction Mode Annotation

Using `CREATE INDEX CONCURRENTLY` inside an Atlas migration without `-- atlas:txmode none`. CONCURRENTLY cannot run inside a transaction block — the migration fails.

---

## Examples

### Example 1: Safe Index Addition

```sql
-- atlas:txmode none

SET LOCAL lock_timeout = '5s';

CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders (user_id);
```

**Why safe:** CONCURRENTLY avoids SHARE lock. `lock_timeout` prevents indefinite wait. `txmode none` ensures no transaction wrapping.

### Example 2: Safe Column Rename (Expand Phase)

```sql
SET LOCAL lock_timeout = '3s';
SET LOCAL statement_timeout = '30s';

-- Phase 1: Expand — add new column
ALTER TABLE users ADD COLUMN display_name text;

-- Phase 2: Backfill (in application or separate migration)
-- UPDATE users SET display_name = name WHERE display_name IS NULL;
-- (run in batches)
```

**Why safe:** New nullable column is backward-compatible. Old code continues reading `name`. New code dual-writes both columns.

### Example 3: Unsafe Migration (Flagged)

```sql
-- UNSAFE: Multiple issues
ALTER TABLE orders DROP COLUMN legacy_status;
CREATE INDEX idx_orders_created ON orders (created_at);
ALTER TABLE orders ALTER COLUMN amount SET NOT NULL;
```

**Issues:**
- Column drop without verifying application compatibility
- `CREATE INDEX` without `CONCURRENTLY`
- `SET NOT NULL` without NOT VALID CHECK pattern
- No `lock_timeout` or `statement_timeout`
