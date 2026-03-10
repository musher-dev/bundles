---
name: scoring-schema-maturity
version: 1.0.0
user-invocable: false
description: >-
  Score PostgreSQL schemas against a 5-level maturity model (Ad-Hoc through Optimizing)
  using diagnostic questions, a prioritized audit checklist by severity, and recommended
  next steps by timeframe. Use when assessing overall database quality, prioritizing
  technical debt remediation, or establishing a schema improvement roadmap. Triggered by:
  schema maturity, maturity model, maturity score, database quality, audit checklist,
  technical debt, maturity level, schema assessment.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Schema Maturity Scoring

5-level maturity model with diagnostic questions, audit checklists, and improvement roadmaps for PostgreSQL schemas.

## Purpose

Schema maturity scoring provides an objective framework for assessing where a database schema falls on the spectrum from "quick prototype" to "production-hardened." Without a maturity model, teams debate individual schema decisions in isolation — should we add comments? should we fix naming? — without a shared framework for prioritization. The maturity model sequences improvements so teams fix the highest-impact issues first.

Use this skill when conducting initial schema assessments, prioritizing technical debt remediation, reporting schema quality to stakeholders, or establishing a schema improvement roadmap for a team.

---

## Maturity Levels

### Level 1: Ad-Hoc

**Characteristics:**
- No naming conventions — mix of camelCase, snake_case, singular/plural
- `serial` or `integer` primary keys
- `varchar(255)` as default text type
- `timestamp` without timezone
- No explicit constraint names (PostgreSQL auto-generates)
- No schema documentation (zero COMMENT ON statements)
- No index strategy — either no indexes or indexes on every column

**Diagnostic signals:** `information_schema` queries reveal mixed naming patterns, implicit constraints, and no pg_description entries.

### Level 2: Consistent

**Characteristics:**
- Naming conventions established and mostly followed
- `bigint` or UUID primary keys
- `text` preferred over `varchar(n)`
- `timestamptz` used consistently
- Foreign keys defined on all relationships
- Explicit `NOT NULL` on required columns
- Basic indexes on FK columns and common query patterns

**Diagnostic signals:** Naming audit passes >80% of checks. All relationships have FK constraints. Nullable columns are intentional.

### Level 3: Governed

**Characteristics:**
- All Level 2 plus:
- Explicit constraint names following conventions (`pk_`, `fk_`, `unique_`, `check_`)
- Partial indexes for soft-delete patterns
- COMMENT ON for all tables and key columns
- Atlas (or equivalent) managing migrations with lint enabled
- Autovacuum tuned for high-write tables
- No schema anti-patterns (soft delete booleans, unindexed FKs)

**Diagnostic signals:** Atlas lint passes with zero warnings. Documentation audit shows >90% coverage. Anti-pattern scan returns zero findings.

### Level 4: Measured

**Characteristics:**
- All Level 3 plus:
- Index usage monitoring via `pg_stat_user_indexes`
- Cache hit ratio tracking (>99%)
- Dead tuple monitoring with alerts
- XID age monitoring with thresholds
- Query performance baselines established
- Covering indexes and expression indexes for hot-path queries
- Schema documentation enforced via CI/CD linting

**Diagnostic signals:** Operational health dashboard exists. Index audit shows zero unused indexes. HOT update ratio >90% on write-heavy tables.

### Level 5: Optimizing

**Characteristics:**
- All Level 4 plus:
- Fillfactor tuned per table based on measured write patterns
- Keyset pagination for all user-facing list endpoints
- RLS enforced for tenant isolation
- Schema changes tested under realistic load before deployment
- Automated schema drift detection
- Regular (quarterly) schema audits with documented findings
- Knowledge sharing — team members can explain schema decisions

**Diagnostic signals:** Load test results for schema changes exist. Quarterly audit history is documented. Team onboarding includes schema walkthrough.

---

## Diagnostic Questions

Use these questions to quickly assess the current maturity level. Each "yes" answer earns the point for its level.

### Horizon (Where Are You Going?)

| Question | Level |
|----------|-------|
| Do you have naming conventions documented? | 2 |
| Are naming conventions enforced by tooling (lint/CI)? | 3 |
| Do you track schema quality metrics over time? | 4 |
| Do you have quarterly schema review cadence? | 5 |

### Concurrency (How Do You Handle Contention?)

| Question | Level |
|----------|-------|
| Do all relationships have FK constraints? | 2 |
| Do you use advisory locks or serializable isolation for TOCTOU-vulnerable operations? | 3 |
| Do you monitor lock contention and query wait times? | 4 |
| Do you load-test schema changes before deployment? | 5 |

### Vacuum (How Do You Manage Dead Tuples?)

| Question | Level |
|----------|-------|
| Is autovacuum enabled (not disabled)? | 2 |
| Have you tuned autovacuum for high-write tables? | 3 |
| Do you monitor dead tuple ratios and XID age? | 4 |
| Do you have automated alerts for vacuum-related thresholds? | 5 |

### Lock-Queue (How Do You Deploy Schema Changes?)

| Question | Level |
|----------|-------|
| Do you use a migration tool (not manual SQL)? | 2 |
| Do you lint migrations for destructive and data-dependent changes? | 3 |
| Do you use CONCURRENTLY for production index creation? | 4 |
| Do you have zero-downtime migration runbooks? | 5 |

### Lifecycle (How Do You Manage Schema Over Time?)

| Question | Level |
|----------|-------|
| Are all columns explicitly typed (no implicit defaults)? | 2 |
| Do all tables have COMMENT ON documentation? | 3 |
| Do you track unused indexes and remove them? | 4 |
| Do you have automated schema drift detection? | 5 |

### Scoring

Count "yes" answers per level. The schema's maturity level is the highest level where ALL questions are answered "yes," with all lower levels also complete.

| Yes Count by Level | Assessment |
|--------------------|------------|
| Level 2: 5/5 | Ready to advance to Governed |
| Level 3: 5/5 | Ready to advance to Measured |
| Level 4: 5/5 | Ready to advance to Optimizing |
| Level 5: 5/5 | Fully mature |

**Partial scores:** If Level 3 has 3/5 "yes" answers, the schema is "Level 2 with partial Level 3 adoption." Focus on completing Level 3 before attempting Level 4.

---

## Prioritized Audit Checklist

### High Severity (Fix Within 0-30 Days)

These issues cause data corruption, security breaches, or service outages if left unaddressed.

| # | Check | Detection |
|---|-------|-----------|
| 1 | XID age below 800M | `SELECT age(datfrozenxid) FROM pg_database` |
| 2 | No unindexed foreign keys | FK constraints without matching indexes |
| 3 | All PKs are `bigint` or UUID (not `serial`/`integer`) | `information_schema.columns` WHERE column_default LIKE 'nextval%' |
| 4 | `timestamptz` used everywhere (no bare `timestamp`) | `information_schema.columns` WHERE data_type = 'timestamp without time zone' |
| 5 | No plaintext secrets in schema (credential tables use hashes) | Columns named `key`, `secret`, `token`, `password` without `_hash` suffix |
| 6 | RLS enabled on tenant-scoped tables (if multi-tenant) | `SELECT relname FROM pg_class WHERE relrowsecurity = false` |
| 7 | No `float`/`double precision` for monetary values | `information_schema.columns` for money-related column names |

### Medium Severity (Fix Within 30-90 Days)

These issues cause performance degradation, maintenance burden, or developer confusion.

| # | Check | Detection |
|---|-------|-----------|
| 8 | Consistent naming conventions across all tables | Naming audit script |
| 9 | All constraints explicitly named | `pg_constraint` WHERE conname matches auto-generated patterns |
| 10 | No soft-delete boolean columns (`is_deleted`) | Column name scan |
| 11 | `NOT NULL` on all columns without documented NULL semantics | Nullable column audit |
| 12 | Autovacuum tuned for tables with >10K writes/hour | Per-table `n_tup_ins + n_tup_upd + n_tup_del` vs autovacuum settings |
| 13 | Unused indexes identified and removed | `pg_stat_user_indexes` WHERE `idx_scan = 0` |
| 14 | JSONB columns not used for structured, frequently-queried data | JSONB columns on high-scan tables |

### Low Severity (Fix Within 90+ Days)

These issues affect long-term maintainability and team velocity.

| # | Check | Detection |
|---|-------|-----------|
| 15 | COMMENT ON for all tables | `pg_description` audit |
| 16 | COMMENT ON for all boolean, nullable, and JSONB columns | Column-level documentation audit |
| 17 | Covering indexes for hot-path queries | EXPLAIN ANALYZE on top-10 queries |
| 18 | Fillfactor tuned on write-heavy tables | `pg_class.reloptions` check |
| 19 | Keyset pagination replacing OFFSET on user-facing endpoints | Application code audit |
| 20 | Schema documentation enforced via CI/CD linting | Pipeline configuration audit |
| 21 | Quarterly schema audit cadence established | Process documentation |

---

## Recommended Next Steps by Timeframe

### 0-30 Days: Stabilize

Focus on data safety and correctness.

1. Run XID age check — escalate if > 500M
2. Add missing FK indexes
3. Audit for `timestamp` columns — plan migration to `timestamptz`
4. Audit for `serial`/`integer` PKs — plan migration to `bigint`
5. Verify autovacuum is enabled and not blocked
6. Check for credential columns storing plaintext

### 30-90 Days: Standardize

Establish conventions and tooling.

1. Document naming conventions and get team agreement
2. Configure Atlas (or chosen migration tool) with lint policies
3. Add explicit constraint names to new migrations
4. Replace soft-delete booleans with timestamp patterns
5. Add `NOT NULL` to columns that should never be null
6. Tune autovacuum for high-write tables
7. Remove unused indexes

### 90+ Days: Optimize

Continuous improvement and measurement.

1. Add COMMENT ON to all tables and key columns
2. Configure CI/CD documentation linting
3. Implement covering indexes for hot-path queries
4. Tune fillfactor on write-heavy tables
5. Replace OFFSET pagination with keyset pagination
6. Establish quarterly audit cadence
7. Set up operational health monitoring dashboard
8. Document schema decisions for team onboarding

---

## Anti-Patterns

### 1. Skipping Levels

Attempting Level 4 (Measured) without completing Level 2 (Consistent). Monitoring index usage is meaningless if the indexes don't follow naming conventions and half of them are redundant.

### 2. Scoring Without Action

Running the maturity assessment and filing the results without creating improvement tasks. The assessment is only valuable if it drives prioritized action.

### 3. Perfectionism

Refusing to ship until every check passes. The maturity model is a roadmap, not a gate. High-severity items block shipping; low-severity items are tracked debt.

---

## Examples

### Example 1: Startup Schema Assessment

**Assessment:**
- Naming: Mixed camelCase and snake_case → Level 1
- PKs: `serial` on most tables → Level 1
- Types: `varchar(255)` and `timestamp` widespread → Level 1
- FKs: Some relationships missing FK constraints → Level 1
- Documentation: Zero COMMENT ON statements → Level 1
- Tooling: Manual SQL migrations, no lint → Level 1

**Score:** Level 1 (Ad-Hoc)

**30-day plan:** Fix PK types, add missing FKs with indexes, switch to `timestamptz`, adopt Atlas with destructive lint enabled.

### Example 2: Growing SaaS Schema Assessment

**Assessment:**
- Naming: snake_case everywhere, plural tables → Level 2+
- PKs: `bigint` everywhere → Level 2+
- Types: `text`, `timestamptz`, `numeric` used correctly → Level 2+
- Constraints: Mix of auto-generated and explicit names → Partial Level 3
- Documentation: COMMENT ON on ~40% of tables → Partial Level 3
- Tooling: Atlas with destructive lint, no naming lint → Partial Level 3
- Monitoring: No index usage tracking, no XID monitoring → Level 2

**Score:** Level 2 (Consistent) with partial Level 3

**30-day plan:** Enable naming lint in Atlas, add explicit constraint names to new migrations.
**90-day plan:** Achieve 100% COMMENT ON coverage, tune autovacuum, set up pg_stat monitoring.

### Example 3: Mature Platform Schema Assessment

**Assessment:**
- All Level 2-3 checks pass
- Index monitoring active, unused indexes removed quarterly → Level 4
- Cache hit ratio >99.5%, dead tuple ratio <2% → Level 4
- XID monitoring with PagerDuty alerts → Level 4
- Missing: No load testing for schema changes, no automated drift detection → Not Level 5

**Score:** Level 4 (Measured)

**90-day plan:** Add schema change load testing to CI/CD, implement drift detection, establish quarterly review cadence.
