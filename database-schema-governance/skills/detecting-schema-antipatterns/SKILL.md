---
name: detecting-schema-antipatterns
version: 1.0.0
user-invocable: false
description: >-
  Detect and remediate six common PostgreSQL schema anti-patterns: soft deletes,
  unindexed foreign keys, over-indexing, JSONB misuse in hot paths, offset pagination,
  and implicit nullability. Each anti-pattern includes mechanism, harm, detection method,
  and remediation. Use when auditing schemas for structural problems or reviewing
  migrations for known pitfalls. Triggered by: anti-pattern, soft delete, is_deleted,
  unindexed foreign key, over-indexing, write amplification, offset pagination, schema smell.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Schema Anti-Pattern Detection

Six high-impact PostgreSQL schema anti-patterns with detection methods and remediations.

## Purpose

Schema anti-patterns are structural choices that appear reasonable at small scale but degrade performance, correctness, or maintainability as the system grows. Unlike bugs, they don't cause immediate failures — they create slow-building technical debt that becomes expensive to fix once data volumes grow. This skill provides the mechanism, harm, detection, and remediation for each pattern.

Use this skill when auditing existing schemas for structural problems, reviewing new migrations for known pitfalls, or investigating unexplained performance degradation.

---

## Anti-Pattern 1: Soft Deletes via Boolean Flag

### Mechanism

Adding an `is_deleted boolean DEFAULT false` or `deleted boolean` column to tables instead of physically removing rows. Every query must include `WHERE is_deleted = false` to exclude logically deleted records.

### Harm

- **Query correctness risk:** Every query that forgets the filter returns stale data. This is not a hypothetical — it is the most common source of "ghost data" bugs in production.
- **Index bloat:** Deleted rows remain in indexes, degrading scan performance proportionally to the deleted-to-active ratio.
- **Cascading complexity:** Joins between tables with soft deletes require filters on both sides, creating combinatorial WHERE clause growth.
- **UNIQUE constraint breakage:** A unique index on `(email)` prevents re-registration after soft-delete. Workarounds (partial unique index, composite unique with `is_deleted`) add fragility.

### Detection

```sql
-- Find boolean columns likely used for soft deletes
SELECT table_name, column_name
FROM information_schema.columns
WHERE data_type = 'boolean'
  AND column_name IN ('is_deleted', 'deleted', 'is_archived', 'archived', 'is_removed');
```

Also grep codebase for `WHERE.*is_deleted\s*=\s*false` or `WHERE.*deleted\s*=\s*false` to find filter patterns.

### Remediation

Replace with a **timestamp pattern**: `deleted_at timestamptz` (NULL means active). Benefits:
- Partial index `WHERE deleted_at IS NULL` covers active-row queries efficiently
- The timestamp provides forensic value (when was it deleted?)
- Pair with a view: `CREATE VIEW active_users AS SELECT * FROM users WHERE deleted_at IS NULL`
- For true archival needs, move deleted rows to a separate `_archive` table on a schedule

---

## Anti-Pattern 2: Unindexed Foreign Keys

### Mechanism

Creating a foreign key constraint without a corresponding index on the referencing column. PostgreSQL does NOT automatically create indexes on FK columns (unlike MySQL).

### Harm

- **Slow cascading deletes:** `ON DELETE CASCADE` requires scanning the child table for matching rows. Without an index, this is a sequential scan.
- **Slow joins:** Every join on the FK column becomes a sequential scan on the child table.
- **Lock escalation:** Unindexed FKs can cause `ShareLock` on the parent table during child inserts, creating unexpected contention.
- **Exponential degradation:** Performance degrades linearly with child table size — invisible at 1K rows, catastrophic at 1M.

### Detection

```sql
-- Find FK columns without indexes
SELECT
    tc.table_name,
    kcu.column_name,
    tc.constraint_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
    ON tc.constraint_name = kcu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND NOT EXISTS (
    SELECT 1
    FROM pg_indexes pi
    WHERE pi.tablename = tc.table_name
      AND pi.indexdef LIKE '%' || kcu.column_name || '%'
  );
```

### Remediation

Add an index on every FK column. This should be part of the standard FK creation pattern:

```sql
ALTER TABLE api_keys ADD CONSTRAINT fk_api_keys_org_id_organizations
    FOREIGN KEY (org_id) REFERENCES organizations(id);
CREATE INDEX idx_api_keys_org_id ON api_keys (org_id);
```

**Exception:** If the FK column is already the leading column of a composite index, a separate single-column index is redundant.

---

## Anti-Pattern 3: Over-Indexing

### Mechanism

Adding indexes speculatively "just in case" or creating an index for every column. Common in teams that treat indexing as free optimization.

### Harm

- **Write amplification:** Every INSERT, UPDATE, or DELETE must update every index on the table. PostgreSQL's MVCC creates a new index entry for every row version, multiplying write I/O.
- **HOT update prevention:** Heap-Only Tuple (HOT) updates skip index maintenance when no indexed column changes. Over-indexing reduces HOT update eligibility, forcing full index maintenance on every update.
- **Vacuum overhead:** More indexes means more work for autovacuum to reclaim dead tuples across all index structures.
- **Storage waste:** Unused indexes consume disk, memory (buffer cache), and backup bandwidth.

### Detection

```sql
-- Find indexes with zero scans since last stats reset
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE 'pk_%'
  AND indexrelname NOT LIKE 'unique_%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Note:** PK and UNIQUE indexes may show zero scans but still enforce constraints — do not drop them based on scan count alone.

### Remediation

1. Monitor `pg_stat_user_indexes.idx_scan` over a full business cycle (minimum 30 days)
2. Drop indexes with zero scans that are not PK, UNIQUE, or FK-supporting
3. Before dropping, verify with `EXPLAIN ANALYZE` on known query patterns
4. Aim for a rule of thumb: no more indexes than columns on tables with heavy write traffic

---

## Anti-Pattern 4: JSONB Misuse in Hot Paths

### Mechanism

Using JSONB columns for structured, frequently-queried data instead of normalized columns. Often starts as a "flexible schema" decision and persists after the schema stabilizes.

### Harm

- **CPU overhead:** JSONB extraction (`->`, `->>`) is measurably slower than column access. At hot-path query volumes (>1000 QPS), this adds up.
- **Index limitations:** GIN indexes support containment (`@>`) but not arbitrary key-path lookups. Expression indexes on specific keys work but are fragile and don't compose.
- **Null handling:** Missing keys in JSONB return NULL silently, with no schema enforcement. `NOT NULL` constraints cannot be applied to JSONB keys.
- **Type coercion:** All JSONB values extracted via `->>` are text. Numeric comparisons require explicit casting, which is error-prone and index-unfriendly.

### Detection

```sql
-- Find JSONB columns on tables with high sequential scan counts
SELECT
    s.relname AS table_name,
    c.column_name,
    s.seq_scan,
    s.seq_tup_read
FROM pg_stat_user_tables s
JOIN information_schema.columns c
    ON c.table_name = s.relname
WHERE c.data_type = 'jsonb'
  AND s.seq_scan > 1000
ORDER BY s.seq_tup_read DESC;
```

Also grep application code for JSONB operator patterns (`->`, `->>`, `@>`, `#>`) in hot-path queries.

### Remediation

1. Identify JSONB keys that appear in WHERE, ORDER BY, or JOIN clauses
2. Extract those keys to dedicated columns with proper types and constraints
3. Retain JSONB only for truly unstructured, rarely-queried metadata
4. Add a CHECK constraint or application validation for remaining JSONB structure

---

## Anti-Pattern 5: Offset Pagination

### Mechanism

Using `LIMIT n OFFSET m` for paginated queries. The database must scan and discard `m` rows before returning `n` results.

### Harm

- **Linear degradation:** Page 1 scans 20 rows. Page 1000 scans 20,000 rows and discards 19,980. Cost grows linearly with page depth.
- **Inconsistent results:** Concurrent inserts or deletes between page fetches cause rows to be skipped or duplicated.
- **Resource waste:** Deep pages consume CPU, I/O, and memory proportional to the offset, not the page size.

### Detection

Grep application code and query logs for `OFFSET` usage:

```sql
-- In pg_stat_statements (if enabled)
SELECT query, calls, mean_exec_time
FROM pg_stat_statements
WHERE query ILIKE '%offset%'
ORDER BY mean_exec_time DESC;
```

### Remediation

Replace with **keyset pagination** (cursor-based):

```sql
-- Instead of: SELECT * FROM events ORDER BY created_at DESC LIMIT 20 OFFSET 1000
-- Use:
SELECT * FROM events
WHERE created_at < $last_seen_created_at
ORDER BY created_at DESC
LIMIT 20;
```

**Requirements for keyset pagination:**
- A stable, indexed sort column (typically `created_at` or `id`)
- The client passes the last seen value instead of a page number
- Ties must be broken with a secondary sort column (e.g., `id`)

**When OFFSET is acceptable:** Admin dashboards with low page depth (< 100 pages), reporting queries, or COUNT-based UIs where total page count is displayed.

---

## Anti-Pattern 6: Implicit Nullability

### Mechanism

Omitting `NOT NULL` constraints and allowing columns to be nullable by default. PostgreSQL defaults to nullable when no constraint is specified.

### Harm

- **Three-valued logic:** NULL propagates through comparisons (`NULL = NULL` is NULL, not TRUE), causing subtle query bugs.
- **Aggregation surprises:** `COUNT(column)` excludes NULLs; `COUNT(*)` includes them. `SUM` of an all-NULL column returns NULL, not 0.
- **Application bugs:** ORMs map NULL to language-specific null types (None, nil, null) which may not be handled consistently.
- **Index waste:** NULL values are included in B-tree indexes but excluded from default unique constraints. Partial indexes must explicitly handle NULL.

### Detection

```sql
-- Find nullable columns that might need NOT NULL
SELECT table_name, column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
  AND is_nullable = 'YES'
  AND column_name NOT IN ('deleted_at', 'revoked_at', 'expires_at', 'last_used_at')
ORDER BY table_name, ordinal_position;
```

Review each nullable column: does NULL carry semantic meaning, or was `NOT NULL` simply omitted?

### Remediation

1. Audit every nullable column — document why NULL is valid or add `NOT NULL`
2. For new columns: always specify `NOT NULL DEFAULT <value>` unless NULL has explicit semantic meaning
3. For existing columns: add `NOT NULL` constraint with `ALTER TABLE ... SET NOT NULL` (requires a full table scan to verify no existing NULLs)
4. Use `ALTER TABLE ... ADD CONSTRAINT ... NOT VALID` followed by `VALIDATE CONSTRAINT` for zero-downtime migration on large tables

---

## Audit Checklist

When scanning a schema for anti-patterns, check each table against:

| # | Anti-Pattern | Quick Check | Severity |
|---|-------------|-------------|----------|
| 1 | Soft delete boolean | Columns named `is_deleted`, `deleted` | High |
| 2 | Unindexed FK | FK constraints without matching index | High |
| 3 | Over-indexing | Indexes with `idx_scan = 0` for 30+ days | Medium |
| 4 | JSONB in hot path | JSONB columns on high-scan tables | Medium |
| 5 | Offset pagination | `OFFSET` in application queries | Medium |
| 6 | Implicit nullability | Nullable columns without documented reason | Low |

---

## Examples

### Example 1: Schema with Multiple Anti-Patterns

**Schema under review:**
```sql
CREATE TABLE orders (
    id serial PRIMARY KEY,
    user_id integer REFERENCES users(id),
    data jsonb,
    is_deleted boolean DEFAULT false,
    total float
);
CREATE INDEX idx_orders_data ON orders USING GIN (data);
CREATE INDEX idx_orders_total ON orders (total);
CREATE INDEX idx_orders_is_deleted ON orders (is_deleted);
```

**Findings:**
- **Soft delete:** `is_deleted boolean` — replace with `deleted_at timestamptz`
- **Unindexed FK:** `user_id` has FK but no index — add `idx_orders_user_id`
- **Over-indexing:** `idx_orders_is_deleted` on a low-cardinality boolean is wasteful; `idx_orders_total` is likely unused
- **JSONB misuse:** `data jsonb` with GIN index — investigate what keys are queried; extract to columns if structured
- **Implicit nullability:** `user_id`, `data`, `total` are all nullable without documented reason

### Example 2: Clean Schema After Remediation

```sql
CREATE TABLE orders (
    id bigint GENERATED ALWAYS AS IDENTITY,
    user_id bigint NOT NULL REFERENCES users(id),
    amount_cents bigint NOT NULL,
    metadata jsonb,
    deleted_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT pk_orders PRIMARY KEY (id)
);
CREATE INDEX idx_orders_user_id ON orders (user_id);
CREATE INDEX idx_orders_created_at ON orders (created_at) WHERE deleted_at IS NULL;
```

All six anti-patterns addressed: timestamp soft delete, indexed FK, minimal indexes, JSONB only for unstructured metadata, no offset pagination in schema, explicit nullability.
