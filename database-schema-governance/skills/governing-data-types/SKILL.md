---
name: governing-data-types
version: 1.0.0
user-invocable: false
description: >-
  Audit and enforce PostgreSQL data type selection including text over varchar(n),
  bigint for identifiers, timestamptz for all temporal data, numeric for money,
  enum vs lookup table decisions, JSONB boundaries, and nullability defaults.
  Use when reviewing column type choices, auditing schema type consistency, or
  deciding between competing PostgreSQL types. Triggered by: data type, varchar,
  bigint, timestamptz, enum, JSONB, nullability, column type, numeric, text vs varchar.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Data Type Governance

Canonical rules for PostgreSQL data type selection that optimize for correctness, performance, and maintainability.

## Purpose

Data type choices are permanent — changing a column type on a table with millions of rows requires an exclusive lock and full table rewrite. Getting types right at design time prevents expensive migrations later. This skill encodes the standard type selections for PostgreSQL schemas and the reasoning behind each choice.

Use this skill when designing new tables, reviewing column type selections in migrations, or auditing existing schemas for type consistency.

---

## Text Types

| Rule | Use | Avoid | Rationale |
|------|-----|-------|-----------|
| Unconstrained text | `text` | `varchar(n)` | PostgreSQL stores both identically; `varchar(n)` adds a check constraint that requires a migration to change |
| Length validation | Application-layer or `CHECK` constraint | `varchar(255)` | A CHECK constraint can be altered without rewriting the table; `varchar(n)` cannot |

**When `varchar(n)` is acceptable:** External system integration where the length limit is a protocol-level requirement (e.g., SMS body at 160 chars), not a business rule.

**Detection:** Scan for `varchar` columns — each should have a documented reason for the length limit, or be changed to `text`.

---

## Integer Types

| Use Case | Type | Rationale |
|----------|------|-----------|
| Primary keys | `bigint GENERATED ALWAYS AS IDENTITY` | 32-bit `integer` exhausts at ~2.1B rows; `bigint` is 8 bytes and future-proof |
| Foreign keys | `bigint` | Must match referenced PK type |
| Counters, quantities | `integer` | Safe when bounded by business logic |
| Small enumerations | `smallint` | 2-byte storage for values < 32K |

**Anti-pattern:** Using `serial` or `bigserial`. These are legacy syntax for sequences. Use `GENERATED ALWAYS AS IDENTITY` which is SQL-standard, prevents manual override, and is clearer in intent.

---

## Temporal Types

| Rule | Use | Avoid | Rationale |
|------|-----|-------|-----------|
| All timestamps | `timestamptz` | `timestamp` | `timestamp` (without timezone) silently drops timezone info; `timestamptz` stores UTC and converts on display |
| Date-only values | `date` | `timestamptz` | When time component is meaningless (birthdays, effective dates) |
| Durations | `interval` | Integer columns storing seconds/minutes | `interval` supports arithmetic and human-readable output |

**Critical rule:** There is no valid use case for `timestamp without time zone` in a multi-user application. Always use `timestamptz`.

**Detection:** Scan for columns of type `timestamp` (without `tz`) — every one is a bug unless the application is provably single-timezone.

---

## Monetary Types

| Rule | Use | Avoid | Rationale |
|------|-----|-------|-----------|
| Currency amounts | `numeric(precision, scale)` or `integer` (cents) | `money`, `float`, `double precision` | `money` is locale-dependent; `float`/`double` have rounding errors |
| Storing cents | `bigint` | `numeric` | When the application consistently works in minor units |

**Decision rule:** If the application does arithmetic in the database (SUM, AVG), use `numeric`. If the database only stores and retrieves values, `bigint` cents is simpler.

---

## Enum vs Lookup Table

| Factor | PostgreSQL ENUM | Lookup Table |
|--------|----------------|--------------|
| Adding values | `ALTER TYPE ... ADD VALUE` (append only, no removal) | INSERT a row |
| Removing values | Impossible without type recreation | DELETE or soft-delete a row |
| Reordering | Impossible | Update a `sort_order` column |
| Metadata | None | Additional columns (label, description, is_active) |
| FK enforcement | Type system | Foreign key constraint |
| Migration complexity | Requires `ALTER TYPE` in a separate transaction | Standard DML |

**Decision rule:** Use a **lookup table** by default. Use a PostgreSQL `ENUM` only when:
1. The set is truly fixed and small (< 10 values)
2. Values will never be removed or reordered
3. No metadata is needed beyond the value itself

**Examples of safe ENUMs:** `principal_type` ('membership', 'service_account'), `environment` ('production', 'staging', 'development').

**Examples requiring lookup tables:** Status workflows, permission levels, product categories — anything where business users might add or remove values.

---

## JSONB Boundaries

| Appropriate Use | Inappropriate Use |
|----------------|-------------------|
| Metadata bags with unpredictable keys | Structured data with known, queryable fields |
| Audit log context/details | Core business entities (users, orders) |
| External API response caching | Replacing normalized relations |
| User preferences/settings | Data that appears in WHERE clauses frequently |
| Schema-less event payloads | Data requiring referential integrity |

**Decision rule:** If you need to query a JSON key in a WHERE clause more than occasionally, extract it to a column. JSONB in hot-path queries has measurably higher CPU cost than indexed columns.

**Performance note:** GIN indexes on JSONB support `@>` (containment) queries efficiently but do not help with arbitrary key-path lookups. Plan indexes around actual query patterns.

---

## Nullability

| Rule | Convention | Rationale |
|------|-----------|-----------|
| Default stance | `NOT NULL` | NULLs propagate through expressions and comparisons, creating subtle bugs |
| Explicit DEFAULT | Always pair `NOT NULL` with `DEFAULT` for new columns | Adding a `NOT NULL` column without `DEFAULT` to an existing table requires a full table scan |
| When NULL is correct | Temporal optionality | `revoked_at NULL` means "not yet revoked" — the absence of a value carries semantic meaning |
| Boolean columns | `NOT NULL DEFAULT false` | A three-valued boolean (`true`/`false`/`null`) is almost never intentional |

**Detection:** Scan for columns without explicit `NOT NULL` constraints. Each nullable column should have a documented reason for allowing NULL.

**Migration safety:** Adding a `NOT NULL` column to a large table requires `DEFAULT` to avoid an `ACCESS EXCLUSIVE` lock during backfill. Without `DEFAULT`, PostgreSQL rewrites every row.

---

## Identity and Key Strategy

### Surrogate vs Natural Key Trade-offs

| Factor | Surrogate Key (UUID/bigint) | Natural Key (email, code) |
|--------|---------------------------|--------------------------|
| Stability | Immutable by design | May change (email, name) requiring cascading updates |
| Size | Fixed (16 bytes UUID, 8 bytes bigint) | Variable; can be large for composite naturals |
| Joins | Single-column joins | Multi-column joins for composites |
| Meaning | Opaque; requires lookup for human context | Self-documenting; readable in queries |
| External exposure | Safe; reveals nothing about internals | May leak PII or business data |

**Decision rule:** Use surrogate keys by default. Use natural keys only for lookup/reference tables where the value is the identity (e.g., `country_code`, `currency_code`) and provably immutable.

**Junction tables exception:** Pure junction tables may use composite natural keys as PK (`PRIMARY KEY (user_id, role_id)`) when the junction has no attributes and no child references.

### UUID Variants

| Variant | Use Case | Rationale |
|---------|----------|-----------|
| UUIDv7 | Primary keys exposed to clients | Time-sortable, no information leakage about row count |
| UUIDv4 | Legacy compatibility | Random, no ordering — causes index fragmentation on B-tree |
| TSID | Alternative to UUIDv7 | Smaller (64-bit), time-sortable, but less ecosystem support |

### UUIDv4 vs UUIDv7 B-tree Fragmentation

UUIDv4 values are random, causing inserts to land on arbitrary B-tree leaf pages. This produces:

| Problem | Mechanism | Impact |
|---------|-----------|--------|
| Page splits | Random inserts fill pages unevenly, forcing splits | Write amplification; fragmented tree structure |
| Poor cache utilization | Sequential inserts touch different pages | Buffer pool thrashing; increased I/O |
| Index bloat | Split pages are ~50% full on average | 2x storage vs sequential inserts |

UUIDv7 embeds a timestamp prefix, making values monotonically increasing within a time window. This produces sequential B-tree inserts — similar to `bigint GENERATED ALWAYS AS IDENTITY` — avoiding page splits and enabling efficient buffer usage.

**Migration path:** Do not migrate existing UUIDv4 PKs to UUIDv7 unless measured fragmentation is causing performance issues. For new tables, default to UUIDv7.

### External-Facing ID Strategies

Internal database IDs should never appear in URLs or API responses. Strategies for external identifiers:

| Strategy | Implementation | Properties |
|----------|---------------|------------|
| `public_id` column | `public_id uuid NOT NULL DEFAULT gen_random_uuid() UNIQUE` | Decoupled from PK; can rotate without schema change |
| Hashids/Sqids | Application-layer encoding of integer PK | Short, URL-safe; reversible; no extra column |
| TSID | 64-bit time-sorted ID as PK | Compact; time-sortable; no extra column needed |
| Prefixed IDs | `usr_abc123`, `org_xyz789` | Type-safe; self-documenting; requires application encoding |

**Recommendation:** For new systems, use UUIDv7 as PK and expose it directly (it reveals only creation time, not row count). For systems with integer PKs, add a `public_id` UUID column for API exposure.

### Key Generation Location

| Location | Mechanism | Trade-offs |
|----------|-----------|------------|
| Database | `GENERATED ALWAYS AS IDENTITY` or `DEFAULT gen_random_uuid()` | Guaranteed unique; no round-trip needed for ID; cannot pre-generate client-side |
| Application | UUID library in app code | Can assign ID before INSERT (optimistic patterns); must handle collisions |
| Hybrid | Application generates UUIDv7, database has DEFAULT as fallback | Best of both; application can pre-assign, database catches omissions |

**Recommendation:** Use `GENERATED ALWAYS AS IDENTITY` for integer PKs (prevents manual override). For UUID PKs, generate in the application with a database DEFAULT as fallback.

---

## Anti-Patterns

### 1. varchar(255) Everywhere

Applying `varchar(255)` as a default without considering actual constraints. This is a MySQL convention carried into PostgreSQL where it provides no storage benefit over `text`.

### 2. timestamp Without Timezone

Using `timestamp` instead of `timestamptz`. The stored value is ambiguous — it depends on the `timezone` setting of the session that wrote it.

### 3. float for Money

Using `double precision` or `real` for monetary amounts. IEEE 754 floating point cannot represent 0.10 exactly, leading to rounding errors in financial calculations.

### 4. Implicit Nullability

Omitting `NOT NULL` and allowing columns to be nullable by default. This creates three-valued logic (`true`/`false`/`null`) for booleans and makes aggregations (`SUM`, `COUNT`) behave unexpectedly.

---

## Examples

### Example 1: Type Audit of a Products Table

**Schema under review:**
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price FLOAT NOT NULL,
    description VARCHAR(1000),
    created TIMESTAMP DEFAULT NOW()
);
```

**Findings:**
- **`id SERIAL`:** FAIL — use `bigint GENERATED ALWAYS AS IDENTITY`
- **`name VARCHAR(255)`:** WARN — use `text` unless 255 is a protocol requirement
- **`price FLOAT`:** FAIL — use `numeric(12,2)` or `bigint` (cents) for monetary values
- **`description VARCHAR(1000)`:** FAIL — use `text` with optional CHECK constraint
- **`created TIMESTAMP`:** FAIL — use `timestamptz`; rename to `created_at`; add `NOT NULL`

### Example 2: JSONB vs Column Decision

**Scenario:** A `users` table has a `preferences jsonb` column. Queries frequently filter by `preferences->>'theme'` and `preferences->>'locale'`.

**Findings:**
- **JSONB usage:** WARN — `theme` and `locale` are known, frequently-queried keys. Extract to `theme text NOT NULL DEFAULT 'light'` and `locale text NOT NULL DEFAULT 'en'`.
- **Remaining JSONB:** If other preference keys are unpredictable and rarely queried, keep them in JSONB.

### Example 3: Well-Typed Schema

**Schema under review:**
```sql
CREATE TABLE invoices (
    id bigint GENERATED ALWAYS AS IDENTITY,
    org_id bigint NOT NULL REFERENCES organizations(id),
    amount_cents bigint NOT NULL,
    currency text NOT NULL DEFAULT 'usd',
    is_paid boolean NOT NULL DEFAULT false,
    paid_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now(),
    CONSTRAINT pk_invoices PRIMARY KEY (id)
);
```

**Findings:** All type selections pass. Integer PK with `GENERATED ALWAYS`, `bigint` for money in cents, `text` for currency code, `boolean` with `NOT NULL DEFAULT`, `timestamptz` for all temporal columns, `paid_at` is correctly nullable (absence means not paid).
