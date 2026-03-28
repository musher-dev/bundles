# Database Schema Governance

Enforce PostgreSQL schema standards from naming conventions through migration safety and operational health. This bundle catches anti-patterns before they reach production, scores your schema against a maturity model, and helps you design tables, indexes, partitions, and multi-tenant structures that follow proven conventions.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/database-schema-governance
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our database schema — check naming conventions, data types, indexes, and anti-patterns
```

## What's Inside

The bundle ships three agents and eighteen skills covering schema design, migration safety, operational health, and CI enforcement.

The **schema_auditor** agent runs comprehensive read-only audits covering naming conventions, data type choices, anti-patterns (soft deletes, unindexed foreign keys, over-indexing, JSONB misuse, offset pagination), index strategy, documentation completeness, and overall maturity. It produces severity-prioritized findings with a maturity score on a 5-level scale (Ad-Hoc → Optimizing).

The **database-designer** agent handles hands-on schema work — designing tables with proper normalization, constraints, and naming; implementing multi-tenant patterns; and configuring credential schemas with lifecycle management.

The **pipeline_architect** agent focuses on CI/CD infrastructure — quality gate pipelines, SQL formatting toolchains (SQLFluff, pg_formatter), schema drift detection workflows, pre-commit hooks, and pgTAP behavioral test suites.

Governance skills span the full lifecycle: entity boundary decisions (1NF-3NF, denormalization justification), data type enforcement (text over varchar, bigint for IDs, timestamptz everywhere), logic placement decisions (database constraints vs application layer), index optimization (covering indexes, HOT updates, EXPLAIN ANALYZE), migration safety (lock risk, CONCURRENTLY enforcement, expand-contract), data lifecycle (partitioning, archival, retention), and operational health monitoring (XID wraparound, autovacuum tuning, bloat detection).

## Usage Examples

**Score schema maturity**
```
Score our PostgreSQL schema against the maturity model — where are we on the Ad-Hoc to Optimizing scale?
```
The schema_auditor evaluates your schema across all dimensions and produces a maturity score with a prioritized improvement roadmap.

**Review a migration for safety**
```
Review this migration for production safety — check lock risk, timeout mandates, backward compatibility, and WAL impact
```
Assesses whether the migration can run safely with zero downtime using expand-contract choreography.

**Design a multi-tenant schema**
```
Design the database schema for our multi-tenant API using the User-Owned Organization model with RBAC
```
The database-designer creates tenant-scoped tables with proper isolation, membership, and role structures.

**Detect anti-patterns**
```
Scan our schema for anti-patterns — soft deletes, unindexed foreign keys, over-indexing, JSONB in hot paths, offset pagination
```
Identifies the six most common PostgreSQL schema problems with detection methods and remediation steps.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `governing-naming-conventions` | Table, column, PK, FK, boolean, timestamp, index, and constraint naming |
| `governing-data-types` | text over varchar(n), bigint for IDs, timestamptz, numeric for money, enum vs lookup |
| `governing-entity-boundaries` | Normalization diagnostics (1NF-3NF), denormalization, junction tables, table width |
| `governing-logic-placement` | Database layer vs application layer decision matrix — triggers, RLS, constraints |
| `governing-data-lifecycle` | Partitioning, archival, retention policies, materialized views, WAL impact |
| `governing-credential-schemas` | Multi-tenant API credential storage — polymorphic principals, hash storage, audit trails |
| `governing-schema-documentation` | COMMENT ON, Atlas comment attributes, custom linting rules |
| `implementing-tenancy-patterns` | User-Owned Organization model, tenant isolation, RBAC schema, membership tables |
| `detecting-schema-antipatterns` | Six anti-patterns — soft deletes, unindexed FKs, over-indexing, JSONB misuse, offset pagination |
| `detecting-schema-drift` | Production drift detection, catalog-vs-VCS comparison, emergency reconciliation |
| `reviewing-index-strategy` | Unused indexes, covering indexes, HOT updates, write amplification, EXPLAIN ANALYZE |
| `reviewing-migration-safety` | Lock risk, timeout mandates, CONCURRENTLY enforcement, expand-contract, WAL impact |
| `scoring-schema-maturity` | 5-level maturity model with diagnostic questions and prioritized audit checklist |
| `auditing-operational-health` | XID wraparound, autovacuum tuning, table/index bloat, cache hit ratios, dead tuples |
| `explaining-atlas` | Ariga Atlas concepts, CLI commands, atlas.hcl configuration, Python integration |
| `formatting-sql` | SQLFluff, pg_formatter, atlas fmt — tool comparison, diff-quality, pre-commit hooks |
| `enforcing-quality-gates` | 5-phase CI/CD pipeline (local → pre-commit → PR → staging → production), tool orchestration |
| `testing-database-behavior` | pgTAP for RLS validation, trigger testing, RegreSQL for query plan baselines |

### Agents

| Agent | Role |
|-------|------|
| `schema_auditor` | Read-only schema audits with maturity scoring and severity-prioritized findings |
| `database-designer` | Hands-on schema design — normalization, constraints, tenancy, credentials |
| `pipeline_architect` | CI/CD quality gates, SQL formatting, drift detection, pgTAP test suites |

### Rules

| Rule | Purpose |
|------|---------|
| `database-patterns` | UUID PKs, timestamp columns, soft deletes, naming conventions, pagination indexes |

### References

| Reference | Purpose |
|-----------|---------|
| `reviewing-migrations` | Migration review guidance |
| `data-migrations` | Data migration patterns |
| `migration-safety` | Production migration safety checklist |
| `enforcement-policy` | Schema governance enforcement policy |
| `tooling-ecosystem` | Tool comparison and ecosystem overview |

</details>

---

**Domain:** database-schema · **Technologies:** PostgreSQL (harness-agnostic) · **Aliases:** db-naming-standards, schema-conventions, database-standards
