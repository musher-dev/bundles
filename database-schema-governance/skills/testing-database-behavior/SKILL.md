---
name: testing-database-behavior
version: 1.0.0
user-invocable: false
description: >-
  Design and review dynamic behavioral tests for PostgreSQL using pgTAP for RLS
  validation, trigger and constraint side-effect testing, RegreSQL for query plan
  baselines, EXPLAIN cost tracking, and seed data strategy. Covers SET ROLE
  tenant isolation proofs, performance regression detection, and when NOT to
  automate tests to avoid false positives. Use when writing pgTAP tests,
  designing RLS test suites, or setting up performance regression baselines.
  Triggered by: pgTAP, RLS testing, behavioral test, performance regression,
  query plan baseline, RegreSQL, database testing.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Database Behavioral Testing

Framework for dynamic testing of PostgreSQL schemas covering RLS policy validation, constraint enforcement, trigger side-effects, and query performance regression.

## Purpose

Static analysis (linting, naming checks, type audits) catches structural defects but cannot verify runtime behavior. Does the RLS policy actually prevent Tenant A from reading Tenant B's data? Does the trigger correctly update `updated_at`? Does the query plan stay within acceptable cost bounds after a schema change? Behavioral tests answer these questions by executing real SQL against a real PostgreSQL instance.

Use this skill when designing pgTAP test suites, validating RLS policies with tenant isolation proofs, testing trigger and constraint side-effects, setting up query plan regression baselines, or deciding which behaviors warrant automated tests and which do not.

---

## pgTAP Overview

pgTAP is a PostgreSQL extension that provides TAP-compatible test functions for writing unit tests directly in SQL.

### Installation

```sql
CREATE EXTENSION IF NOT EXISTS pgtap;
```

### Test Structure

```sql
BEGIN;

SELECT plan(3);  -- Declare the number of tests

SELECT has_table('users', 'Table users should exist');
SELECT has_column('users', 'email', 'Users should have email column');
SELECT col_is_pk('users', 'id', 'id should be the primary key');

SELECT * FROM finish();
ROLLBACK;  -- Tests run in a transaction and are rolled back
```

### Running Tests

```bash
# Run all tests in a directory
pg_prove -d mydb tests/*.sql

# Run with verbose output
pg_prove -d mydb -v tests/*.sql
```

### Test Categories

| Category | What It Verifies | pgTAP Functions |
|----------|-----------------|-----------------|
| Schema structure | Tables, columns, types, constraints exist | `has_table`, `has_column`, `col_type_is` |
| RLS policies | Tenant isolation, role-based access | `SET ROLE` + custom queries |
| Constraints | CHECK, FK, UNIQUE enforcement | INSERT/UPDATE that should fail |
| Triggers | Side-effects on DML operations | INSERT + verify side-effect occurred |
| Functions | Stored procedure behavior | `results_eq`, `throws_ok` |

---

## RLS Testing with Tenant Isolation Proofs

RLS policies are the most critical behavioral tests. A misconfigured policy can expose one tenant's data to another — a security breach. Static analysis cannot verify RLS correctness; only runtime tests can.

### SET ROLE Pattern

pgTAP tests use `SET ROLE` to simulate different tenants and verify isolation.

```sql
BEGIN;
SELECT plan(4);

-- Setup: Create test data for two tenants
INSERT INTO tenants (id, name) VALUES
    ('aaaa-1111', 'Tenant A'),
    ('bbbb-2222', 'Tenant B');

INSERT INTO orders (id, tenant_id, amount) VALUES
    ('order-a1', 'aaaa-1111', 100),
    ('order-a2', 'aaaa-1111', 200),
    ('order-b1', 'bbbb-2222', 300);

-- Test 1: Tenant A sees only their own orders
SET LOCAL app.tenant_id = 'aaaa-1111';
SET ROLE tenant_user;

SELECT results_eq(
    'SELECT count(*)::integer FROM orders',
    ARRAY[2],
    'Tenant A should see exactly 2 orders'
);

-- Test 2: Tenant A cannot see Tenant B orders
SELECT is_empty(
    $$SELECT * FROM orders WHERE tenant_id = 'bbbb-2222'$$,
    'Tenant A should not see Tenant B orders'
);

-- Test 3: Tenant B sees only their own orders
RESET ROLE;
SET LOCAL app.tenant_id = 'bbbb-2222';
SET ROLE tenant_user;

SELECT results_eq(
    'SELECT count(*)::integer FROM orders',
    ARRAY[1],
    'Tenant B should see exactly 1 order'
);

-- Test 4: Tenant B cannot modify Tenant A orders
SELECT throws_ok(
    $$UPDATE orders SET amount = 999 WHERE id = 'order-a1'$$,
    'new row violates row-level security policy',
    'Tenant B should not be able to modify Tenant A orders'
);

RESET ROLE;
SELECT * FROM finish();
ROLLBACK;
```

### Isolation Proof Checklist

Every RLS test suite should verify:

| Test | What It Proves |
|------|---------------|
| Tenant A reads own data | SELECT policy works for owner |
| Tenant A cannot read other tenant | SELECT policy excludes non-owner |
| Tenant A inserts own data | INSERT policy works for owner |
| Tenant A cannot insert as other tenant | INSERT policy prevents spoofing |
| Tenant A updates own data | UPDATE policy works for owner |
| Tenant A cannot update other tenant | UPDATE policy excludes non-owner |
| Tenant A deletes own data | DELETE policy works for owner |
| Tenant A cannot delete other tenant | DELETE policy excludes non-owner |
| Superuser bypasses RLS | `FORCE ROW LEVEL SECURITY` not set unless intended |
| No tenant set returns empty | Missing `app.tenant_id` returns no rows, not all rows |

### The Empty-Tenant-ID Test

The most commonly missed RLS test: what happens when `app.tenant_id` is not set?

```sql
-- Test: No tenant context should return zero rows, not all rows
RESET ROLE;
SET ROLE tenant_user;
-- Deliberately do NOT set app.tenant_id

SELECT is_empty(
    'SELECT * FROM orders',
    'No tenant context should return zero rows'
);
```

If this test fails (returns all rows), the RLS policy has a NULL-handling bug — `tenant_id = current_setting('app.tenant_id')` evaluates to `tenant_id = ''` which matches nothing, but if the policy uses `IS NOT DISTINCT FROM` or a permissive default, it may match everything.

---

## Constraint and Trigger Testing

### Testing CHECK Constraints

```sql
BEGIN;
SELECT plan(2);

-- Test: Valid status is accepted
SELECT lives_ok(
    $$INSERT INTO orders (id, tenant_id, status, amount)
      VALUES ('test-1', 'aaaa-1111', 'pending', 100)$$,
    'Valid status should be accepted'
);

-- Test: Invalid status is rejected
SELECT throws_ok(
    $$INSERT INTO orders (id, tenant_id, status, amount)
      VALUES ('test-2', 'aaaa-1111', 'invalid_status', 100)$$,
    '23514',  -- check_violation error code
    NULL,
    'Invalid status should be rejected by CHECK constraint'
);

SELECT * FROM finish();
ROLLBACK;
```

### Testing Trigger Side-Effects

```sql
BEGIN;
SELECT plan(2);

-- Test: INSERT sets created_at via trigger
INSERT INTO orders (id, tenant_id, status, amount)
VALUES ('test-1', 'aaaa-1111', 'pending', 100);

SELECT isnt(
    (SELECT created_at FROM orders WHERE id = 'test-1'),
    NULL,
    'created_at should be set by trigger on INSERT'
);

-- Test: UPDATE sets updated_at via trigger
UPDATE orders SET amount = 200 WHERE id = 'test-1';

SELECT isnt(
    (SELECT updated_at FROM orders WHERE id = 'test-1'),
    NULL,
    'updated_at should be set by trigger on UPDATE'
);

SELECT * FROM finish();
ROLLBACK;
```

### Testing Foreign Key Enforcement

```sql
BEGIN;
SELECT plan(1);

-- Test: FK prevents orphaned references
SELECT throws_ok(
    $$INSERT INTO orders (id, tenant_id, user_id, amount)
      VALUES ('test-1', 'aaaa-1111', 'nonexistent-user', 100)$$,
    '23503',  -- foreign_key_violation error code
    NULL,
    'FK should prevent inserting with non-existent user_id'
);

SELECT * FROM finish();
ROLLBACK;
```

---

## Performance Regression Testing

### EXPLAIN Cost Tracking

Track query plan costs across schema changes to detect performance regressions.

```sql
-- Capture the estimated cost of a critical query
EXPLAIN (FORMAT JSON)
SELECT o.id, o.amount, u.email
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.tenant_id = 'aaaa-1111'
  AND o.status = 'pending'
ORDER BY o.created_at DESC
LIMIT 20;
```

### Cost Comparison Strategy

| Metric | Source | Threshold |
|--------|--------|-----------|
| Total cost | `EXPLAIN` output `"Total Cost"` | Alert if >15% increase |
| Planning time | `EXPLAIN ANALYZE` output | Alert if >2x increase |
| Execution time | `EXPLAIN ANALYZE` output | Alert if >2x increase |
| Rows scanned | `EXPLAIN ANALYZE` `"Actual Rows"` | Alert if >10x increase |

### RegreSQL for Query Plan Baselines

RegreSQL captures query plans as baseline files and compares against them on each test run.

**Workflow:**
1. Define critical queries in `.sql` files
2. Run `regresql` to capture baseline plans
3. On schema change, re-run and compare against baselines
4. Any plan change beyond the threshold triggers an alert

### When to Track Performance

| Scenario | Track? | Rationale |
|----------|--------|-----------|
| Adding an index | Yes | Verify the index is used by expected queries |
| Dropping an index | Yes | Verify no critical query plan degrades |
| Adding a column | Usually no | Column addition rarely changes plans |
| Changing column type | Yes | May affect index usage and cast costs |
| Adding a FK constraint | Sometimes | FK scan cost on INSERT/UPDATE may increase |

---

## Seed Data Strategy

Behavioral tests require representative data to produce meaningful results. The seed data strategy determines test reliability.

### Requirements

| Requirement | Rationale |
|-------------|-----------|
| Multiple tenants (minimum 2) | RLS isolation requires cross-tenant verification |
| Sufficient volume per tenant | Performance tests need enough rows for index vs seq scan to matter |
| Edge case data | NULL values, empty strings, boundary dates, max-length strings |
| Referential completeness | All FK references must resolve (parent rows exist) |
| Deterministic | Same seed data every run for reproducible tests |

### Seed Data Structure

```sql
-- seeds/test_data.sql
-- Deterministic test data for behavioral tests

-- Tenants
INSERT INTO tenants (id, name) VALUES
    ('tenant-aaa', 'Acme Corp'),
    ('tenant-bbb', 'Beta Inc');

-- Users (per tenant)
INSERT INTO users (id, tenant_id, email) VALUES
    ('user-a1', 'tenant-aaa', 'alice@acme.com'),
    ('user-a2', 'tenant-aaa', 'bob@acme.com'),
    ('user-b1', 'tenant-bbb', 'carol@beta.com');

-- Orders (per tenant, various states)
INSERT INTO orders (id, tenant_id, user_id, status, amount) VALUES
    ('order-a1', 'tenant-aaa', 'user-a1', 'pending', 100),
    ('order-a2', 'tenant-aaa', 'user-a1', 'completed', 200),
    ('order-a3', 'tenant-aaa', 'user-a2', 'cancelled', 50),
    ('order-b1', 'tenant-bbb', 'user-b1', 'pending', 300);
```

### Volume Seed for Performance Tests

For performance regression tests, a small deterministic seed is insufficient. Use a generator:

```sql
-- Generate 10,000 orders per tenant for performance baseline
INSERT INTO orders (id, tenant_id, user_id, status, amount)
SELECT
    gen_random_uuid(),
    CASE WHEN i % 2 = 0 THEN 'tenant-aaa' ELSE 'tenant-bbb' END,
    CASE WHEN i % 2 = 0 THEN 'user-a1' ELSE 'user-b1' END,
    (ARRAY['pending', 'completed', 'cancelled'])[1 + (i % 3)],
    (i % 1000) + 1
FROM generate_series(1, 20000) AS s(i);

ANALYZE;  -- Update statistics after bulk insert
```

---

## When NOT to Automate

Not every database behavior warrants an automated test. False positives erode trust in the test suite.

### Do Not Automate

| Behavior | Why Not |
|----------|---------|
| PostgreSQL built-in constraint enforcement | Testing that NOT NULL rejects NULL is testing PostgreSQL, not your schema |
| Exact query plan shape | Plans change with PostgreSQL version, statistics, and data distribution |
| Exact timing thresholds | Execution time varies by hardware and load |
| Extension internal behavior | Test your usage of extensions, not the extension itself |
| Auto-generated column defaults | `DEFAULT now()` works — testing it adds noise |

### Do Automate

| Behavior | Why |
|----------|-----|
| RLS tenant isolation | Security-critical; misconfiguration is a data breach |
| Custom trigger logic | Side-effects are invisible without explicit verification |
| Complex CHECK constraints | Business rules encoded as constraints need regression protection |
| Cross-table consistency | Constraints that span multiple tables via triggers or functions |
| Critical query performance | High-traffic queries where a plan regression causes incidents |

### The False Positive Rule

**If a test fails more than twice without a real bug, remove it.** False positives train developers to ignore test failures, which defeats the purpose of the entire test suite. Better to have 20 reliable tests than 100 flaky ones.

---

## Anti-Patterns

### 1. Testing PostgreSQL Instead of Your Schema

Writing tests that verify PostgreSQL's constraint engine works correctly (e.g., testing that UNIQUE actually prevents duplicates on a simple column). These tests pass on every PostgreSQL version and add maintenance cost without catching real bugs.

### 2. Hardcoded Expected Plans

Asserting exact query plan output (specific node types, exact costs). Plans change with PostgreSQL minor versions, `ANALYZE` statistics, and data distribution. Test cost thresholds, not plan shapes.

### 3. Missing Rollback in Tests

Running pgTAP tests without wrapping them in `BEGIN/ROLLBACK`. Test data leaks into the database, causing subsequent test runs to fail or produce incorrect results.

### 4. Single-Tenant RLS Tests

Testing RLS with only one tenant. The test passes because the policy returns the only tenant's data — it would also pass with no policy at all. Always test with at least two tenants and verify cross-tenant isolation.

### 5. No ANALYZE After Seed Data

Loading seed data for performance tests without running `ANALYZE`. PostgreSQL's planner uses stale statistics and produces plans that don't reflect the actual data distribution, making performance baselines meaningless.

---

## Examples

### Example 1: Complete RLS Test Suite

A multi-tenant application needs to verify `orders` table isolation.

**Test file:** `tests/rls_orders.sql`
- 10 tests covering SELECT, INSERT, UPDATE, DELETE for two tenants
- Empty tenant context test
- Superuser bypass verification
- Seed data with 2 tenants, 3 users, 4 orders

**Run:** `pg_prove -d testdb tests/rls_orders.sql`

### Example 2: Trigger Side-Effect Verification

An `updated_at` trigger should fire on every UPDATE.

**Test file:** `tests/trigger_updated_at.sql`
- INSERT a row, verify `updated_at` is NULL
- UPDATE the row, verify `updated_at` is not NULL
- UPDATE again, verify `updated_at` changed

### Example 3: Performance Baseline Setup

A team wants to track query plan costs for their 5 most critical queries.

1. Create `tests/perf/` directory with one `.sql` file per query
2. Each file contains `EXPLAIN (FORMAT JSON)` for the query
3. CI captures the `Total Cost` from each plan
4. Compare against stored baselines (previous CI run)
5. Alert (advisory) if any query cost increases by >15%
6. Store new baselines when the increase is intentional (schema change)
