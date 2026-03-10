---
name: auditing-operational-health
version: 1.0.0
user-invocable: false
description: >-
  Audit PostgreSQL operational health including XID wraparound monitoring, table and
  index bloat detection, autovacuum tuning, dead tuple tracking, cache hit ratios,
  and sequential scan ratios. Use when investigating database performance degradation,
  reviewing vacuum configuration, or monitoring production health metrics. Triggered by:
  operational health, XID wraparound, autovacuum, table bloat, dead tuples, vacuum,
  cache hit ratio, transaction ID, sequential scan, pg_stat.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Operational Health Auditing

PostgreSQL operational health monitoring covering XID wraparound, bloat, autovacuum, and performance metrics.

## Purpose

Operational health issues in PostgreSQL are silent killers — they degrade gradually until a threshold is crossed and the system fails catastrophically. XID wraparound can force emergency single-user-mode recovery. Unchecked bloat can exhaust disk space. Misconfigured autovacuum can leave dead tuples consuming memory and I/O. This skill provides the queries, thresholds, and tuning guidance for proactive monitoring.

Use this skill when setting up database monitoring, investigating unexplained performance degradation, tuning autovacuum for high-write workloads, or conducting periodic operational health audits.

---

## XID Wraparound Monitoring

PostgreSQL uses 32-bit transaction IDs (XIDs). Every transaction consumes one XID. When the XID space (approximately 2 billion usable IDs) is exhausted without vacuum freezing old tuples, PostgreSQL enters emergency shutdown to prevent data corruption.

### How It Works

- Each row stores the XID of the transaction that created it
- VACUUM freezes old rows by marking them as visible to all future transactions
- `datfrozenxid` tracks the oldest unfrozen XID per database
- `age(datfrozenxid)` measures how close the database is to wraparound

### Monitoring Query

```sql
SELECT
    datname,
    age(datfrozenxid) AS xid_age,
    pg_size_pretty(pg_database_size(oid)) AS db_size,
    CASE
        WHEN age(datfrozenxid) > 1200000000 THEN 'CRITICAL'
        WHEN age(datfrozenxid) > 800000000 THEN 'WARNING'
        ELSE 'OK'
    END AS status
FROM pg_database
WHERE datallowconn
ORDER BY age(datfrozenxid) DESC;
```

### Per-Table XID Age

```sql
SELECT
    schemaname,
    relname,
    age(relfrozenxid) AS xid_age,
    last_autovacuum,
    last_vacuum,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_stat_user_tables
ORDER BY age(relfrozenxid) DESC
LIMIT 20;
```

### Thresholds

| XID Age | Status | Action |
|---------|--------|--------|
| < 500M | OK | Normal operation |
| 500M – 800M | Watch | Verify autovacuum is running; investigate blocked vacuums |
| 800M – 1.2B | Warning | Manual VACUUM FREEZE on oldest tables; investigate locks |
| > 1.2B | Critical | Emergency vacuum required; risk of protective shutdown |
| > 2B | Shutdown | PostgreSQL refuses new writes to prevent data loss |

### Common Causes of High XID Age

- Long-running transactions preventing vacuum from freezing tuples
- Autovacuum not keeping up with write volume (needs tuning)
- Prepared transactions left open (`pg_prepared_xacts`)
- Replication slots preventing vacuum from advancing

---

## Table and Index Bloat

Bloat occurs when dead tuples (from UPDATEs and DELETEs) accumulate faster than vacuum can reclaim them. The table and its indexes grow larger than the live data warrants.

### Table Bloat Detection

```sql
SELECT
    schemaname,
    relname AS table_name,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
    pg_size_pretty(pg_relation_size(relid)) AS table_size,
    pg_size_pretty(pg_indexes_size(relid)) AS index_size,
    n_live_tup,
    n_dead_tup,
    CASE WHEN n_live_tup > 0
         THEN round(100.0 * n_dead_tup / (n_live_tup + n_dead_tup), 1)
         ELSE 0 END AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

### Index Bloat Detection

```sql
-- Estimate index bloat by comparing actual size to estimated size
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC
LIMIT 20;
```

### Bloat Thresholds

| Metric | Healthy | Investigate | Critical |
|--------|---------|-------------|----------|
| Dead tuple percentage | < 5% | 5-20% | > 20% |
| Table size vs live data | < 1.5x | 1.5-3x | > 3x |
| Index size vs table size | < 2x | 2-5x | > 5x |

### Remediation

| Approach | Downtime | Use When |
|----------|----------|----------|
| Wait for autovacuum | None | Dead tuple % is moderate, autovacuum is running |
| Manual `VACUUM` | None | Autovacuum is behind; need immediate cleanup |
| `VACUUM FULL` | Yes (ACCESS EXCLUSIVE lock) | Extreme bloat; need to reclaim disk space |
| `pg_repack` | Minimal | Need to reclaim space without downtime |
| `REINDEX CONCURRENTLY` | None | Index-specific bloat |

---

## Autovacuum Tuning

Autovacuum is PostgreSQL's background process for reclaiming dead tuples and preventing XID wraparound. Default settings are conservative and often insufficient for high-write workloads.

### Key Parameters

| Parameter | Default | Purpose | Tune When |
|-----------|---------|---------|-----------|
| `autovacuum_vacuum_threshold` | 50 | Min dead tuples before vacuum triggers | Small tables vacuumed too rarely |
| `autovacuum_vacuum_scale_factor` | 0.2 (20%) | Fraction of table size added to threshold | Large tables vacuumed too rarely |
| `autovacuum_analyze_threshold` | 50 | Min changed rows before analyze triggers | Query plans going stale |
| `autovacuum_analyze_scale_factor` | 0.1 (10%) | Fraction of table size added to threshold | Large tables analyzed too rarely |
| `autovacuum_vacuum_cost_delay` | 2ms | Delay between vacuum I/O operations | Vacuum too slow or too aggressive |
| `autovacuum_vacuum_cost_limit` | 200 | I/O cost limit per vacuum cycle | Vacuum not keeping up with writes |
| `autovacuum_max_workers` | 3 | Concurrent vacuum workers | Many tables need simultaneous vacuuming |
| `autovacuum_freeze_max_age` | 200M | XID age that triggers anti-wraparound vacuum | Never lower this without understanding implications |

### Per-Table Overrides

High-write tables often need more aggressive vacuum settings:

```sql
ALTER TABLE events SET (
    autovacuum_vacuum_scale_factor = 0.01,    -- Trigger at 1% dead tuples instead of 20%
    autovacuum_vacuum_threshold = 1000,       -- Minimum 1000 dead tuples
    autovacuum_analyze_scale_factor = 0.005   -- Analyze at 0.5% changes
);
```

### Monitoring Autovacuum Activity

```sql
-- Currently running vacuum operations
SELECT
    pid,
    datname,
    relid::regclass AS table_name,
    phase,
    heap_blks_total,
    heap_blks_scanned,
    CASE WHEN heap_blks_total > 0
         THEN round(100.0 * heap_blks_scanned / heap_blks_total, 1)
         ELSE 0 END AS progress_pct
FROM pg_stat_progress_vacuum;
```

```sql
-- Last vacuum times per table
SELECT
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    vacuum_count,
    autovacuum_count,
    n_dead_tup
FROM pg_stat_user_tables
ORDER BY last_autovacuum NULLS FIRST
LIMIT 20;
```

---

## Dead Tuple Tracking

Dead tuples are row versions left behind by UPDATEs and DELETEs. They consume space and I/O until vacuum reclaims them.

### Monitoring Query

```sql
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    last_autovacuum,
    CASE WHEN n_live_tup > 0
         THEN round(100.0 * n_dead_tup / n_live_tup, 1)
         ELSE 0 END AS dead_to_live_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### What Blocks Vacuum From Reclaiming Dead Tuples

| Blocker | Detection | Fix |
|---------|-----------|-----|
| Long-running transactions | `SELECT * FROM pg_stat_activity WHERE state != 'idle' AND xact_start < now() - interval '1 hour'` | Terminate or fix the long transaction |
| Abandoned prepared transactions | `SELECT * FROM pg_prepared_xacts` | `ROLLBACK PREPARED '<name>'` |
| Replication slot lag | `SELECT * FROM pg_replication_slots WHERE active = false` | Drop inactive slots |
| Hot standby feedback | `hot_standby_feedback = on` on replica | Consider disabling or tuning `max_standby_streaming_delay` |

---

## Cache Hit Ratios

PostgreSQL caches frequently accessed data in shared buffers. A low cache hit ratio means the database is reading from disk instead of memory.

### Database-Level Cache Hit Ratio

```sql
SELECT
    datname,
    blks_hit,
    blks_read,
    CASE WHEN (blks_hit + blks_read) > 0
         THEN round(100.0 * blks_hit / (blks_hit + blks_read), 2)
         ELSE 0 END AS cache_hit_pct
FROM pg_stat_database
WHERE datname = current_database();
```

### Table-Level Cache Hit Ratio

```sql
SELECT
    schemaname,
    relname,
    heap_blks_hit,
    heap_blks_read,
    CASE WHEN (heap_blks_hit + heap_blks_read) > 0
         THEN round(100.0 * heap_blks_hit / (heap_blks_hit + heap_blks_read), 2)
         ELSE 0 END AS cache_hit_pct
FROM pg_statio_user_tables
WHERE (heap_blks_hit + heap_blks_read) > 100
ORDER BY cache_hit_pct ASC
LIMIT 20;
```

### Thresholds

| Cache Hit % | Status | Action |
|-------------|--------|--------|
| > 99% | Excellent | Working set fits in memory |
| 95-99% | Good | Acceptable for most workloads |
| 90-95% | Watch | Consider increasing `shared_buffers` or optimizing queries |
| < 90% | Poor | Working set exceeds memory; increase `shared_buffers` or reduce working set |

---

## Sequential Scan Ratios

High sequential scan ratios on large tables indicate missing or unused indexes.

### Monitoring Query

```sql
SELECT
    schemaname,
    relname,
    seq_scan,
    idx_scan,
    CASE WHEN (seq_scan + idx_scan) > 0
         THEN round(100.0 * seq_scan / (seq_scan + idx_scan), 1)
         ELSE 0 END AS seq_scan_pct,
    seq_tup_read,
    n_live_tup,
    pg_size_pretty(pg_relation_size(relid)) AS table_size
FROM pg_stat_user_tables
WHERE (seq_scan + idx_scan) > 100
ORDER BY seq_scan_pct DESC;
```

### Interpretation

| Table Size | High Seq Scan % | Likely Cause |
|------------|----------------|--------------|
| < 1MB | Normal | Small tables are faster to seq scan than index scan |
| 1MB - 100MB | Investigate | May need an index or queries may not match existing indexes |
| > 100MB | Problem | Missing index, index not matching query pattern, or stale statistics |

**Note:** `seq_tup_read / seq_scan` gives the average rows read per sequential scan. If this is close to the total row count, the scan is reading the entire table each time.

---

## Anti-Patterns

### 1. Ignoring Autovacuum Warnings

Treating autovacuum as "it just works" without monitoring. Default settings are insufficient for tables with more than a few thousand writes per hour.

### 2. VACUUM FULL as Routine Maintenance

Using `VACUUM FULL` regularly instead of tuning autovacuum. `VACUUM FULL` takes an `ACCESS EXCLUSIVE` lock, blocking all reads and writes for the duration of the rewrite.

### 3. Disabling Autovacuum

Setting `autovacuum = off` to "reduce overhead." This guarantees XID wraparound and eventual database shutdown.

---

## Examples

### Example 1: Investigating Slow Query Performance

**Symptoms:** Application queries on the `events` table have degraded from 5ms to 200ms over the past month.

**Audit findings:**
- `n_dead_tup`: 2.4M (45% of live tuples)
- `last_autovacuum`: 3 days ago
- `cache_hit_pct`: 87%
- `seq_scan_pct`: 62% on a 4GB table

**Diagnosis:** Dead tuple accumulation is causing bloat, reducing cache efficiency, and forcing sequential scans. Autovacuum is not keeping up with write volume.

**Remediation:**
1. Run `VACUUM ANALYZE events` immediately
2. Tune per-table autovacuum: `ALTER TABLE events SET (autovacuum_vacuum_scale_factor = 0.02)`
3. Add missing indexes based on query patterns
4. Monitor dead tuple ratio weekly

### Example 2: XID Wraparound Alert

**Monitoring alert:** Database XID age is 900M.

**Audit findings:**
- Oldest table `audit_logs`: `age(relfrozenxid) = 890M`
- `pg_prepared_xacts`: one abandoned prepared transaction from 5 days ago
- Autovacuum running but blocked by the prepared transaction

**Remediation:**
1. `ROLLBACK PREPARED 'txn_abc123'`
2. Run `VACUUM FREEZE audit_logs` manually
3. Add monitoring for abandoned prepared transactions
4. Verify XID age drops below 500M within 24 hours

### Example 3: Healthy Operational State

**Audit findings:**
- XID age: 120M (OK)
- Dead tuple ratio: < 2% on all tables
- Cache hit ratio: 99.7%
- Sequential scan ratio: < 5% on tables > 1MB
- Autovacuum: running regularly, no tables overdue

**Assessment:** Operational health is excellent. No action needed. Next audit in 30 days.
