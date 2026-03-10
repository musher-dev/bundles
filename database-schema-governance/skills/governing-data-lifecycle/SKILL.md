---
name: governing-data-lifecycle
version: 1.0.0
user-invocable: false
description: >-
  Govern data lifecycle architecture including table partitioning strategies,
  partition management, archival patterns, data retention policies, WAL impact
  comparison, materialized view refresh, and soft delete lifecycle progression.
  Use when designing partitioning schemes, planning data archival, implementing
  retention policies, or managing data growth. Triggered by: data lifecycle,
  partitioning, partition, archival, data retention, purge, materialized view,
  soft delete lifecycle, data growth, table partitioning.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Data Lifecycle Governance

Decision framework for managing data from creation through archival and permanent deletion in PostgreSQL.

## Purpose

Every table grows. Without lifecycle management, tables accumulate rows indefinitely — degrading query performance, increasing backup times, and consuming storage. This skill provides the patterns for partitioning tables at creation, archiving data when it ages out of operational relevance, and purging data when retention periods expire. The goal is to make data growth a managed concern, not a crisis.

Use this skill when designing new high-growth tables, planning archival strategies for existing data, implementing retention policies, or evaluating partitioning as a performance optimization.

---

## Table Partitioning Strategies

Partitioning divides a logical table into physical segments. PostgreSQL supports declarative partitioning since version 10.

### Partition Types

| Strategy | Partition Key | Use Case | Example |
|----------|--------------|----------|---------|
| Range | Date/timestamp column | Time-series data, logs, events | Monthly partitions on `created_at` |
| List | Status or type column | Multi-tenant isolation, category separation | Per-tenant partitions on `tenant_id` |
| Hash | High-cardinality column | Even distribution when no natural range/list key | Hash on `user_id` for parallel query |

### Range Partitioning (Most Common)

```sql
CREATE TABLE events (
    id bigint GENERATED ALWAYS AS IDENTITY,
    tenant_id bigint NOT NULL,
    event_type text NOT NULL,
    payload jsonb,
    created_at timestamptz NOT NULL DEFAULT now()
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE events_2025_01 PARTITION OF events
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE events_2025_02 PARTITION OF events
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');
```

### List Partitioning

```sql
CREATE TABLE documents (
    id bigint GENERATED ALWAYS AS IDENTITY,
    tenant_id bigint NOT NULL,
    content text,
    created_at timestamptz NOT NULL DEFAULT now()
) PARTITION BY LIST (tenant_id);

CREATE TABLE documents_tenant_1 PARTITION OF documents
    FOR VALUES IN (1);

CREATE TABLE documents_tenant_2 PARTITION OF documents
    FOR VALUES IN (2);
```

### Hash Partitioning

```sql
CREATE TABLE sessions (
    id bigint GENERATED ALWAYS AS IDENTITY,
    user_id bigint NOT NULL,
    data jsonb,
    created_at timestamptz NOT NULL DEFAULT now()
) PARTITION BY HASH (user_id);

CREATE TABLE sessions_p0 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE sessions_p1 PARTITION OF sessions
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
```

### Partition Key Selection

| Criterion | Guidance |
|-----------|----------|
| Query patterns | Choose a key that appears in most WHERE clauses — enables partition pruning |
| Growth rate | Range partitioning on time for append-heavy tables |
| Data distribution | Hash partitioning when data is skewed and no natural range exists |
| Retention alignment | Range on time enables dropping old partitions instead of DELETE |

---

## Partition Management

### Adding Partitions

Create future partitions before they are needed. Automate with a scheduled job or pre-create several periods ahead.

```sql
-- Create next month's partition
CREATE TABLE events_2025_03 PARTITION OF events
    FOR VALUES FROM ('2025-03-01') TO ('2025-04-01');
```

**Rule:** Always create the next partition before the current one fills. A missing partition causes INSERT failures.

### Dropping Partitions

Dropping a partition is a metadata-only operation — instant, regardless of row count, and generates minimal WAL.

```sql
-- Remove data older than retention period
DROP TABLE events_2024_01;
```

### Detaching Partitions

Detaching preserves the data as a standalone table for archival or analysis without affecting the parent table.

```sql
-- Detach for archival (keeps data accessible)
ALTER TABLE events DETACH PARTITION events_2024_06;

-- The detached table still exists as events_2024_06
-- Can be moved to cheaper storage, exported, or dropped later
```

---

## Archival Patterns

### Archive Table Pattern

Move aged data to a structurally identical archive table.

```sql
-- Create archive with identical structure
CREATE TABLE orders_archive (LIKE orders INCLUDING ALL);

-- Move old data in batches
WITH moved AS (
    DELETE FROM orders
    WHERE created_at < now() - interval '2 years'
    RETURNING *
)
INSERT INTO orders_archive SELECT * FROM moved;
```

### Partition Detach for Cold Storage

For partitioned tables, detaching old partitions is the preferred archival mechanism.

| Approach | WAL Impact | Lock Impact | Speed |
|----------|-----------|-------------|-------|
| `DELETE FROM` + `INSERT INTO archive` | Heavy | Row locks during delete | Slow (row-by-row) |
| `ALTER TABLE DETACH PARTITION` | Minimal | Brief ACCESS EXCLUSIVE | Instant |
| `DROP TABLE` (partition) | Minimal | Brief ACCESS EXCLUSIVE | Instant |

**Recommendation:** Design tables for partitioned archival from the start. Retrofitting partitioning onto existing tables requires a full data migration.

### Tiered Storage

| Tier | Age | Storage | Access Pattern |
|------|-----|---------|---------------|
| Hot | 0-90 days | Primary database | Real-time queries |
| Warm | 90 days - 1 year | Read replica or detached partition | Occasional lookups |
| Cold | 1-5 years | Object storage (S3/GCS) export | Compliance/audit only |
| Deleted | > retention period | Permanently removed | None |

---

## Data Retention Policies

### Policy Definition

| Element | Description | Example |
|---------|-------------|---------|
| Retention period | How long data is kept in the primary database | 2 years for orders |
| Archive period | How long archived data is kept | 5 years for financial records |
| Legal hold | Override that prevents deletion | Active litigation preserves all related data |
| Purge schedule | Automated deletion frequency | Weekly batch purge of expired data |

### Automated Purge

```sql
-- Purge function for partitioned tables
CREATE OR REPLACE FUNCTION purge_old_partitions(
    parent_table text,
    retention_interval interval
) RETURNS void AS $$
DECLARE
    partition_name text;
BEGIN
    FOR partition_name IN
        SELECT inhrelid::regclass::text
        FROM pg_inherits
        WHERE inhparent = parent_table::regclass
        AND inhrelid::regclass::text ~ '\d{4}_\d{2}$'
    LOOP
        -- Extract date from partition name and compare
        -- DROP partition if older than retention_interval
        EXECUTE format('DROP TABLE IF EXISTS %I', partition_name);
        RAISE NOTICE 'Dropped partition: %', partition_name;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Legal Hold Implementation

```sql
-- Legal hold table
CREATE TABLE legal_holds (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    target_table text NOT NULL,
    hold_reason text NOT NULL,
    started_at timestamptz NOT NULL DEFAULT now(),
    ended_at timestamptz,
    created_by text NOT NULL
);

-- Check for legal holds before purging
-- If any active hold exists for the target table, skip purge
```

---

## WAL Impact Comparison

Understanding WAL costs is critical for choosing between deletion strategies.

| Operation | WAL Generated | Lock Duration | Use When |
|-----------|--------------|---------------|----------|
| `DELETE FROM ... WHERE ...` | High (per-row) | Row-level locks | Selective deletion from non-partitioned tables |
| `TRUNCATE TABLE` | Minimal | ACCESS EXCLUSIVE (brief) | Clearing entire tables |
| `DROP TABLE` (partition) | Minimal | ACCESS EXCLUSIVE (brief) | Removing time-range partitions |
| `ALTER TABLE DETACH PARTITION` | Minimal | ACCESS EXCLUSIVE (brief) | Archiving partitions |

**Key insight:** `DELETE` of 10 million rows generates ~10 million WAL records. `DROP TABLE` on a partition with 10 million rows generates a single WAL record. Design for partitioned deletion.

---

## Materialized View Refresh

Materialized views cache query results for read performance. Managing their staleness is a lifecycle concern.

### Refresh Strategies

| Strategy | Command | Blocking | Use When |
|----------|---------|----------|----------|
| Full refresh | `REFRESH MATERIALIZED VIEW mv` | Yes — exclusive lock | Small views, can tolerate brief downtime |
| Concurrent refresh | `REFRESH MATERIALIZED VIEW CONCURRENTLY mv` | No — requires UNIQUE index | Large views, continuous availability required |

### Concurrent Refresh Requirements

```sql
-- Materialized view must have a UNIQUE index for CONCURRENTLY
CREATE MATERIALIZED VIEW monthly_revenue AS
    SELECT date_trunc('month', created_at) AS month,
           org_id,
           SUM(amount_cents) AS total_cents
    FROM orders
    WHERE status = 'completed'
    GROUP BY 1, 2;

CREATE UNIQUE INDEX idx_monthly_revenue_month_org
    ON monthly_revenue (month, org_id);

-- Now concurrent refresh works:
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_revenue;
```

### Staleness Trade-offs

| Refresh Frequency | Staleness | Cost |
|-------------------|-----------|------|
| Every minute | < 1 min | High CPU; only for critical dashboards |
| Every 15 minutes | < 15 min | Moderate; good for operational views |
| Hourly | < 1 hour | Low; suitable for reporting |
| Daily | < 24 hours | Minimal; suitable for analytics |

---

## Soft Delete Lifecycle

Soft deletes (`deleted_at` timestamp) are the first stage of a lifecycle progression, not a permanent state.

### Lifecycle Stages

| Stage | State | Duration | Action |
|-------|-------|----------|--------|
| 1. Active | `deleted_at IS NULL` | Indefinite | Normal operations |
| 2. Soft-deleted | `deleted_at IS NOT NULL` | 30-90 days | Recoverable; excluded from queries via partial index |
| 3. Archived | Moved to archive table | 1-5 years | Not in primary tables; available for compliance |
| 4. Purged | Permanently deleted | After retention | Irrecoverable |

### Implementation

```sql
-- Stage 1 → 2: Soft delete
UPDATE orders SET deleted_at = now() WHERE id = $1;

-- Stage 2 → 3: Archive (batch job)
WITH archived AS (
    DELETE FROM orders
    WHERE deleted_at < now() - interval '90 days'
    RETURNING *
)
INSERT INTO orders_archive SELECT * FROM archived;

-- Stage 3 → 4: Purge (batch job)
DELETE FROM orders_archive
WHERE deleted_at < now() - interval '5 years';
```

### Partial Index for Active Records

```sql
-- Ensure queries on active records are fast
CREATE INDEX idx_orders_active ON orders (user_id, created_at)
    WHERE deleted_at IS NULL;
```

---

## Anti-Patterns

### 1. Unbounded Tables

Tables without any partitioning, archival, or retention strategy that grow indefinitely. Eventually causes slow queries, long backup times, and storage exhaustion.

### 2. DELETE for Bulk Removal

Using `DELETE FROM` to remove millions of rows instead of dropping a partition. Generates massive WAL, holds row locks, and blocks autovacuum.

### 3. Permanent Soft Deletes

Setting `deleted_at` but never progressing rows through the lifecycle to archival and purge. The table grows indefinitely, and the partial index on active records covers a shrinking fraction of total rows.

### 4. Missing Partition Pre-creation

Failing to create future partitions before they are needed. When the current partition fills and no next partition exists, INSERTs fail with an error.

### 5. Synchronous Materialized View Refresh

Refreshing materialized views in the request path (blocking user requests). Refresh should be async — scheduled or triggered by background jobs.

### 6. Retrofitting Partitioning

Attempting to add partitioning to an existing large table in-place. PostgreSQL does not support converting a regular table to a partitioned table — requires creating a new partitioned table and migrating data.

---

## Examples

### Example 1: Designing a Partitioned Events Table

**Requirement:** Event log table expected to grow by 10M rows/month. Queries filter by `tenant_id` and `created_at`. Retention is 1 year.

**Design:**
```sql
CREATE TABLE events (
    id bigint GENERATED ALWAYS AS IDENTITY,
    tenant_id bigint NOT NULL,
    event_type text NOT NULL,
    payload jsonb,
    created_at timestamptz NOT NULL DEFAULT now(),
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (created_at);

-- Pre-create 3 months ahead
CREATE TABLE events_2025_01 PARTITION OF events
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
-- ... etc

-- Monthly job: create next partition, drop 13-month-old partition
```

**Why:** Range partitioning on `created_at` aligns with retention policy (DROP old partitions instead of DELETE). PK includes partition key as required by PostgreSQL.

### Example 2: Soft Delete Lifecycle for Orders

**Current state:** `orders` table has `deleted_at` column. Soft-deleted rows are never cleaned up. Table has 50M rows, 30M soft-deleted.

**Recommendation:**
1. Create `orders_archive` table with identical structure
2. Batch-move rows where `deleted_at < now() - interval '90 days'` to archive
3. Schedule weekly purge of archive rows older than 5 years
4. Add partial index `WHERE deleted_at IS NULL` for active order queries
