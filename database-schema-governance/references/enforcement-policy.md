# Enforcement Policy

Complete classification matrix for database quality gate rules. Each rule is classified as hard-block (fails the pipeline) or advisory (produces a warning). This reference is consumed by the `enforcing-quality-gates` skill and by CI pipeline configurations.

## Classification Principles

1. **Hard-block** rules prevent defects that cause data loss, downtime, or security breaches
2. **Advisory** rules flag style, documentation, and optimization improvements
3. Rules start as advisory and are promoted to hard-block only after the team has established a clean baseline
4. Formatting rules are never hard-block (see rationale below)

## Rule Matrix

### Hard-Block Rules

These rules fail the CI pipeline. A PR cannot merge until they are resolved or overridden via break-glass.

| Rule | Tool | Rationale | Failure Mode If Skipped |
|------|------|-----------|------------------------|
| Missing `CONCURRENTLY` on index operations | Atlas lint / Squawk | `CREATE INDEX` acquires `SHARE` lock, blocking all writes for duration of index build | Production write outage proportional to table size |
| `DROP TABLE` without expand-contract | Atlas lint (DS102) | Destroys data permanently; running application code referencing the table crashes | Immediate data loss and application errors |
| `DROP COLUMN` without verification | Atlas lint (DS103) | Destroys column data; application code reading the column crashes | Data loss and application errors |
| `RENAME TABLE` or `RENAME COLUMN` | Atlas lint (BC101, BC102) | Breaks all application code referencing the old name | Application errors on deploy |
| Direct `SET NOT NULL` on large table | Squawk | Requires full table scan under `ACCESS EXCLUSIVE` lock | Production read/write outage for duration of scan |
| Missing `NOT VALID` on constraint addition | Atlas lint (CD101) / Squawk | Validates all existing rows under `ACCESS EXCLUSIVE` lock | Production lock-out proportional to table size |
| Missing `lock_timeout` | Squawk | DDL waits indefinitely for lock, blocking all subsequent queries via lock queue cascading | Cascading connection pool exhaustion |
| Missing `statement_timeout` | Squawk | Long-running migration holds locks and consumes resources indefinitely | Resource exhaustion, replication lag |
| Migration integrity failure | Atlas `migrate validate` | Tampered or diverged migration files produce unpredictable schema state | Silent schema corruption |
| Naming convention violation | Custom script | Inconsistent naming compounds across the schema, creating confusion and bugs | Permanent naming debt that grows with every migration |
| Missing RLS on tenant-scoped table | Custom analyzer | Tenant data accessible without policy enforcement | Cross-tenant data exposure (security breach) |

### Advisory Rules

These rules produce warnings in PR comments but do not fail the pipeline.

| Rule | Tool | Rationale | Why Not Hard-Block |
|------|------|-----------|-------------------|
| Missing `COMMENT ON` for tables | Custom script | Undocumented tables slow onboarding and increase tribal knowledge dependency | No runtime impact; can be added retroactively |
| Missing `COMMENT ON` for non-obvious columns | Custom script | Column purpose unclear without documentation | No runtime impact |
| SQL formatting violations | SQLFluff | Inconsistent formatting creates noisy diffs and review friction | Style-only; blocking would cause bypass behavior |
| Missing FK index | Custom script / Atlas analyzer | Unindexed FK degrades JOIN performance but doesn't cause outage | Performance impact is gradual, not catastrophic |
| Query plan cost increase >15% | RegreSQL | Performance regression on critical queries | Plans vary by data distribution; needs human judgment |
| Unused index detected | Custom script | Write amplification from maintaining unused index | Requires workload analysis before removal |
| `JSONB` column without GIN index | Custom script | Queries on JSONB without index do full table scan | May be intentional for low-query columns |

## Promotion Path

Rules move from advisory to hard-block through a defined process:

| Step | Action | Duration |
|------|--------|----------|
| 1. Introduce as advisory | Add the rule, produce warnings only | 2-4 weeks |
| 2. Measure baseline | Count existing violations and new violations per PR | 2 weeks |
| 3. Clean baseline | Resolve all existing violations in a dedicated PR | 1 sprint |
| 4. Promote to hard-block | Change rule classification in CI configuration | Permanent |

**Prerequisite for promotion:** Zero existing violations. If the codebase has pre-existing violations, promoting to hard-block forces unrelated PRs to fix them — creating friction and bypass behavior.

## Override Mechanism

### Per-Migration Override

For rules enforced by Atlas lint, use file-level directives:

```sql
-- atlas:nolint DS102
-- Approved: dropping deprecated_feature table per ADR-042

DROP TABLE "deprecated_feature";
```

### Per-PR Override

For rules enforced by custom scripts or Squawk, use a PR label or comment-based mechanism:

```
/override missing-rls
Reason: This table stores global reference data, not tenant-scoped data.
Approved by: @dba-team
```

### Override Audit

All overrides must be:
- **Justified** — a comment explaining why the rule does not apply
- **Approved** — by a designated reviewer (DBA, tech lead)
- **Tracked** — searchable in PR history for periodic audit

## Formatting Policy Rationale

Formatting rules are deliberately excluded from hard-block classification for the following reasons:

| Factor | Impact of Hard-Block |
|--------|---------------------|
| Emergency hotfixes | Blocked by trailing whitespace during an incident |
| New contributors | First PR rejected for style, not substance |
| Generated SQL | Atlas-generated migrations may not match formatter rules |
| Tool disagreement | SQLFluff and pg_formatter may produce different output for edge cases |
| Diminishing returns | Formatting adds consistency but prevents zero bugs |

Formatting is enforced through auto-fix in pre-commit hooks and advisory warnings in CI. If a developer consistently ignores formatting, escalate through code review discussion — not pipeline enforcement.
