---
name: reviewing-index-strategy
version: 1.0.0
user-invocable: false
description: >-
  Review and optimize PostgreSQL index strategies including unused index detection,
  write amplification via MVCC, HOT update optimization, covering indexes (INCLUDE),
  partial indexes, expression indexes, composite ordering, fillfactor tuning, EXPLAIN
  ANALYZE interpretation, heap fetch diagnosis, and N+1 query detection. Use when
  auditing index performance, reviewing index-related migrations, investigating slow
  queries, or analyzing query plans. Triggered by: index strategy, index review,
  unused index, covering index, partial index, HOT update, write amplification,
  index performance, composite index, fillfactor, EXPLAIN ANALYZE, query performance,
  heap fetch, N+1.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Index Strategy Review

Comprehensive guide to PostgreSQL indexing decisions, performance analysis, and optimization techniques.

## Purpose

Indexes are the primary mechanism for query performance in PostgreSQL, but they are not free — every index adds write overhead, storage cost, and vacuum work. The goal is not "more indexes" but "the right indexes." This skill provides the analytical framework for evaluating whether an index is helping, hurting, or redundant, along with advanced techniques for extracting maximum performance from minimal index count.

Use this skill when reviewing index strategies in existing schemas, evaluating proposed indexes in migrations, or investigating slow queries that may need index optimization.

---

## Unused Index Detection

Unused indexes consume storage, slow writes, and increase vacuum time without benefiting any query. Detection requires monitoring over a representative time period.

### Query: Find Unused Indexes

```sql
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_size_pretty(pg_total_relation_size(relid)) AS table_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Interpretation Rules

| Condition | Action |
|-----------|--------|
| `idx_scan = 0` and not PK/UNIQUE/FK-supporting | Safe to drop after 30+ day observation |
| `idx_scan = 0` and UNIQUE constraint | Do NOT drop — enforces data integrity regardless of scan count |
| `idx_scan = 0` and supports FK | Do NOT drop — prevents sequential scans on cascading deletes |
| `idx_scan < 10` over 30 days | Candidate for review; may serve rare but important queries |

**Important:** Reset statistics after major application changes: `SELECT pg_stat_reset();` — then observe for a full business cycle before making drop decisions.

---

## Write Amplification via MVCC

PostgreSQL's Multi-Version Concurrency Control (MVCC) creates a new tuple version for every UPDATE. Each index that covers an updated column must also create a new index entry pointing to the new tuple.

### Cost Model

For a table with N indexes, an UPDATE that touches an indexed column:
1. Creates a new heap tuple (dead tuple left behind)
2. Creates N new index entries (one per index covering the changed column)
3. Leaves N dead index entries for vacuum to clean

**Practical impact:** A table with 10 indexes experiences roughly 10x write amplification compared to an unindexed table. On write-heavy tables, this is the primary performance bottleneck.

### Measurement

```sql
-- Compare table size to total index size
SELECT
    relname AS table_name,
    pg_size_pretty(pg_relation_size(oid)) AS table_size,
    pg_size_pretty(pg_indexes_size(oid)) AS total_index_size,
    (SELECT COUNT(*) FROM pg_index WHERE indrelid = c.oid) AS index_count
FROM pg_class c
WHERE relkind = 'r'
  AND relnamespace = 'public'::regnamespace
ORDER BY pg_indexes_size(oid) DESC;
```

**Warning sign:** Total index size exceeding table size on write-heavy tables indicates probable over-indexing.

---

## HOT Update Optimization

Heap-Only Tuple (HOT) updates are PostgreSQL's optimization for avoiding index maintenance on updates that don't change any indexed column.

### How HOT Works

1. UPDATE changes only non-indexed columns
2. PostgreSQL places the new tuple on the same heap page (if space available)
3. No index entries are created or modified
4. The old tuple is marked as redirected, not dead — cheaper for vacuum

### Requirements for HOT

| Requirement | Why |
|-------------|-----|
| No indexed column changes | If an indexed column changes, index entries must be updated |
| New tuple fits on same page | Sufficient free space on the page (controlled by fillfactor) |
| No conflicting concurrent updates | Two HOT chains on the same page can conflict |

### Maximizing HOT Updates

1. **Reduce index count** on frequently-updated tables
2. **Lower fillfactor** to reserve page space for HOT updates:
   ```sql
   ALTER TABLE events SET (fillfactor = 80);
   -- Leaves 20% free space per page for in-place updates
   ```
3. **Keep frequently-updated columns out of indexes** — don't index `updated_at` or `last_used_at` unless queries require it

### Measurement

```sql
SELECT
    relname,
    n_tup_upd AS total_updates,
    n_tup_hot_upd AS hot_updates,
    CASE WHEN n_tup_upd > 0
         THEN round(100.0 * n_tup_hot_upd / n_tup_upd, 1)
         ELSE 0 END AS hot_pct
FROM pg_stat_user_tables
WHERE n_tup_upd > 100
ORDER BY n_tup_upd DESC;
```

**Target:** HOT update percentage > 90% on write-heavy tables. Below 50% indicates over-indexing or insufficient fillfactor.

---

## Covering Indexes (INCLUDE)

A covering index contains all columns needed by a query, enabling an **index-only scan** that never touches the heap.

### Syntax

```sql
CREATE INDEX idx_orders_user_id ON orders (user_id) INCLUDE (amount_cents, created_at);
```

### When to Use

- A query frequently selects specific columns along with the indexed lookup column
- The included columns are not used in WHERE/ORDER BY (they're just returned)
- The table is large and heap access is the bottleneck

### INCLUDE vs Composite Index

| Feature | Composite `(a, b, c)` | Covering `(a) INCLUDE (b, c)` |
|---------|----------------------|-------------------------------|
| WHERE on b, c | Yes — can filter on any prefix | No — b, c are payload only |
| ORDER BY b, c | Yes — sorted by all columns | No — only sorted by key columns |
| Index size | All columns in B-tree nodes | Only key columns in B-tree; included columns in leaf pages |
| Write amplification | Updates to b or c require index maintenance | Updates to b or c do NOT require index maintenance |

**Decision rule:** If b and c are only in the SELECT list (not WHERE or ORDER BY), use INCLUDE to reduce write amplification.

---

## Partial Indexes

A partial index covers only rows matching a WHERE predicate, reducing index size and maintenance cost.

### Syntax

```sql
CREATE INDEX idx_api_keys_active ON api_keys (org_id, created_at)
    WHERE revoked_at IS NULL;
```

### Common Use Cases

| Pattern | Partial Index | Benefit |
|---------|--------------|---------|
| Active records | `WHERE deleted_at IS NULL` | Excludes archived rows from index |
| Pending items | `WHERE status = 'pending'` | Small index for the processing queue |
| Non-null optional FK | `WHERE parent_id IS NOT NULL` | Index only rows with the relationship |
| Recent data | `WHERE created_at > '2025-01-01'` | Index only recent data (requires periodic recreation) |

### Requirements

- Queries must include the partial index's WHERE clause (or a subset) for PostgreSQL to use the index
- The planner must be able to prove that the query's conditions imply the index's predicate

---

## Expression Indexes

An expression index indexes the result of a function or expression, enabling efficient lookups on transformed data.

### Syntax

```sql
CREATE INDEX idx_users_lower_email ON users (lower(email));
```

### Common Use Cases

```sql
-- Case-insensitive email lookup
CREATE INDEX idx_users_lower_email ON users (lower(email));
-- Query must use: WHERE lower(email) = lower($1)

-- JSONB key extraction
CREATE INDEX idx_events_type ON events ((data->>'type'));
-- Query must use: WHERE data->>'type' = 'signup'

-- Date truncation
CREATE INDEX idx_orders_created_date ON orders (date_trunc('day', created_at));
-- Query must use: WHERE date_trunc('day', created_at) = '2025-01-01'
```

**Critical rule:** The query must use the exact same expression as the index definition. `WHERE email = lower($1)` will NOT use an index on `lower(email)`.

---

## Composite Index Ordering

Column order in a composite index determines which queries can use it. PostgreSQL can use a composite index for any left-prefix of its columns.

### Rules

| Index `(a, b, c)` | Query Can Use |
|--------------------|---------------|
| `WHERE a = ?` | Yes — uses first column |
| `WHERE a = ? AND b = ?` | Yes — uses first two columns |
| `WHERE a = ? AND b = ? AND c = ?` | Yes — uses all columns |
| `WHERE b = ?` | No — skips first column |
| `WHERE a = ? AND c = ?` | Partial — uses `a` only, then filters `c` |

### Ordering Strategy

1. **Equality columns first** — columns in `= ?` conditions
2. **Range columns last** — columns in `>`, `<`, `BETWEEN`
3. **High-selectivity columns first** — among equals, put the most selective column first
4. **ORDER BY alignment** — if the query sorts by the indexed columns, the index can provide pre-sorted results

### Example

```sql
-- Query: WHERE org_id = ? AND status = ? AND created_at > ? ORDER BY created_at
-- Optimal index:
CREATE INDEX idx_orders_org_status_created
    ON orders (org_id, status, created_at);
-- org_id (equality), status (equality), created_at (range + sort)
```

---

## Fillfactor Tuning

Fillfactor controls how full PostgreSQL packs each index/table page. Lower values leave free space for HOT updates and reduce page splits.

| Object | Default | Recommended for Write-Heavy |
|--------|---------|----------------------------|
| Table (heap) | 100% | 80-90% |
| B-tree index | 90% | 70-80% |

### When to Adjust

- Tables with frequent updates to non-indexed columns → lower table fillfactor to enable HOT
- Indexes on tables with frequent inserts in non-sequential order → lower index fillfactor to reduce page splits
- Read-only or append-only tables → keep defaults (maximize density)

```sql
-- Set fillfactor on table
ALTER TABLE events SET (fillfactor = 80);

-- Set fillfactor on index
CREATE INDEX idx_events_created_at ON events (created_at) WITH (fillfactor = 80);

-- After changing fillfactor, VACUUM FULL or REINDEX to apply
REINDEX INDEX idx_events_created_at;
```

---

## EXPLAIN ANALYZE Interpretation

`EXPLAIN ANALYZE` executes the query and reports actual execution statistics. It is the definitive tool for understanding how PostgreSQL processes a query.

### Running EXPLAIN ANALYZE

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;
```

Always include `BUFFERS` to see I/O statistics. Use `FORMAT TEXT` for human-readable output.

### Node Types

| Node Type | Meaning | Performance Implication |
|-----------|---------|----------------------|
| Seq Scan | Full table scan | Acceptable for small tables; concerning on large tables |
| Index Scan | B-tree lookup + heap fetch | Standard indexed access; heap fetch may be costly |
| Index Only Scan | B-tree lookup, no heap fetch | Best case — all data from index; requires visibility map |
| Bitmap Index Scan | Builds a bitmap of matching TIDs | Used when too many rows for Index Scan, too few for Seq Scan |
| Bitmap Heap Scan | Fetches heap pages from bitmap | Follows Bitmap Index Scan; lossy on high-match queries |
| Hash Join | Hash one input, probe with other | Fast for equi-joins; memory-intensive |
| Nested Loop | For each outer row, scan inner | Efficient with small outer set + indexed inner; disastrous otherwise |
| Merge Join | Sort both inputs, merge | Efficient when both inputs are pre-sorted |
| Sort | In-memory or on-disk sort | Watch for `Sort Method: external merge` indicating disk spill |

### Key Metrics to Check

| Metric | What to Look For |
|--------|-----------------|
| `actual rows` vs `rows` (estimated) | Large discrepancy indicates stale statistics; run `ANALYZE` |
| `actual time` | Total time in ms for this node (inclusive of children) |
| `Buffers: shared hit` | Pages read from shared buffer cache (fast) |
| `Buffers: shared read` | Pages read from disk (slow) |
| `rows removed by filter` | Rows fetched but discarded — sign of missing or poor index |
| `Sort Method: external merge` | Sort spilled to disk — increase `work_mem` or reduce result set |

### Example EXPLAIN Output Analysis

```
Index Scan using idx_orders_user_id on orders  (cost=0.43..8.45 rows=1 width=100) (actual time=0.05..0.06 rows=1 loops=1)
  Index Cond: (user_id = 42)
  Buffers: shared hit=4
```

**Reading:** Index scan found 1 row (matching estimate). 4 buffer hits, 0 disk reads. Execution time: 0.06ms. This is optimal.

---

## Heap Fetch Diagnosis

An Index Only Scan avoids heap access entirely — but only when the visibility map confirms all matching tuples are visible. When it fails, PostgreSQL falls back to fetching each tuple from the heap, negating the covering index benefit.

### Why Index-Only Scans Fail

| Cause | Detection | Fix |
|-------|-----------|-----|
| Stale visibility map | `Heap Fetches: <large number>` in EXPLAIN output | Run `VACUUM` on the table |
| High update rate | Visibility map invalidated frequently | Increase autovacuum frequency for the table |
| Recently loaded data | Bulk INSERT without subsequent VACUUM | Run `VACUUM` after bulk loads |

### Visibility Map Role

The visibility map tracks which heap pages contain only tuples visible to all transactions. When a page is "all-visible," an Index Only Scan can skip the heap fetch for tuples on that page.

```sql
-- Check visibility map coverage
SELECT
    relname,
    n_tup_mod,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE relname = 'orders';

-- Force vacuum to refresh visibility map
VACUUM orders;
```

### Measurement

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT user_id, amount FROM orders WHERE user_id = 42;

-- Look for:
-- Index Only Scan ... Heap Fetches: 0    ← optimal
-- Index Only Scan ... Heap Fetches: 500  ← visibility map stale, run VACUUM
```

**Target:** Heap Fetches should be 0 or near-0 for Index Only Scans. Any significant heap fetch count indicates the visibility map is stale.

---

## N+1 Query Detection

N+1 queries occur when the application executes one query to fetch a list of N parent records, then N additional queries to fetch related data for each parent. This is the most common ORM-induced performance problem.

### Detection via pg_stat_statements

```sql
-- Find queries executed far more often than expected
SELECT
    query,
    calls,
    mean_exec_time,
    total_exec_time
FROM pg_stat_statements
WHERE query LIKE '%WHERE%_id = $1%'
ORDER BY calls DESC
LIMIT 20;
```

**Signal:** A simple `SELECT ... WHERE parent_id = $1` query with `calls` in the thousands per minute is likely an N+1 pattern.

### Common ORM Patterns

| ORM Pattern | Problem | Fix |
|-------------|---------|-----|
| SQLAlchemy lazy load (default) | Each attribute access fires a query | Use `selectinload()` or `joinedload()` |
| Django `ForeignKey` access | Each `.related_object` fires a query | Use `select_related()` or `prefetch_related()` |
| Prisma implicit relation | Each relation access fires a query | Use `include: { relation: true }` |

### Eager Loading Strategies

| Strategy | SQL Generated | Use When |
|----------|--------------|----------|
| `joinedload` / `select_related` | Single query with JOIN | One-to-one or many-to-one relations |
| `selectinload` / `prefetch_related` | Two queries: parent + `WHERE id IN (...)` | One-to-many or many-to-many relations |
| `subqueryload` | Two queries: parent + subquery | Large parent result sets where IN list is too long |

```python
# SQLAlchemy: N+1 problem
users = session.query(User).all()
for user in users:
    print(user.orders)  # Each access fires SELECT ... WHERE user_id = ?

# SQLAlchemy: Fixed with eager loading
users = session.query(User).options(selectinload(User.orders)).all()
for user in users:
    print(user.orders)  # Already loaded — no additional queries
```

**Rule:** Default ORM relationships to lazy loading for safety, but explicitly add eager loading in every query that iterates over results and accesses relations.

---

## Anti-Patterns

### 1. Index on Every Column

Creating one index per column "for performance." Each index adds write overhead. Composite indexes on actual query patterns are far more effective than single-column indexes on every column.

### 2. Redundant Indexes

An index on `(a)` is redundant if an index on `(a, b)` exists — the composite index serves all queries that the single-column index would. The exception is if `(a)` is significantly smaller and the query never needs `b`.

### 3. Indexing Low-Cardinality Columns

An index on a boolean column or a status column with 3 values is almost never used by the planner — a sequential scan is cheaper when the index would return a large percentage of rows. Use partial indexes instead: `WHERE is_active = true`.

### 4. Missing CONCURRENTLY for Production Indexes

`CREATE INDEX` takes a `SHARE` lock that blocks writes. On production tables, always use `CREATE INDEX CONCURRENTLY` with `-- atlas:txmode none`.

---

## Examples

### Example 1: Index Audit of an Orders Table

**Current indexes:**
```
idx_orders_user_id      (user_id)           — idx_scan: 50,000
idx_orders_status        (status)            — idx_scan: 12
idx_orders_created_at   (created_at)        — idx_scan: 45,000
idx_orders_updated_at   (updated_at)        — idx_scan: 0
idx_orders_user_status  (user_id, status)   — idx_scan: 30,000
```

**Findings:**
- `idx_orders_status`: Near-zero scans, low cardinality — DROP
- `idx_orders_updated_at`: Zero scans — DROP (also blocking HOT updates)
- `idx_orders_user_id`: Redundant with `idx_orders_user_status` — evaluate if standalone user_id lookups justify keeping
- **HOT impact:** Dropping 2 indexes will significantly increase HOT update rate

### Example 2: Designing Indexes for a Query Pattern

**Query:** `SELECT id, amount, created_at FROM orders WHERE org_id = $1 AND status = 'pending' ORDER BY created_at DESC LIMIT 20`

**Optimal index:**
```sql
CREATE INDEX idx_orders_org_pending_created ON orders (org_id, created_at DESC)
    INCLUDE (amount)
    WHERE status = 'pending';
```

**Why:** Partial index excludes non-pending rows. Composite key matches equality (org_id) then sort (created_at DESC). INCLUDE covers the amount column for an index-only scan. No heap access needed.
