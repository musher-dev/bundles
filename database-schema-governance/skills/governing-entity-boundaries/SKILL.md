---
name: governing-entity-boundaries
version: 1.0.0
user-invocable: false
description: >-
  Govern entity boundary decisions including normalization diagnostics (1NF through
  3NF), denormalization justification, table width heuristics, junction table design,
  and entity decomposition criteria. Use when designing table structures, reviewing
  schema decomposition, or evaluating normalization trade-offs. Triggered by: entity
  boundaries, normalization, denormalization, junction table, table decomposition,
  many-to-many, table width, 3NF, entity design.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Entity Boundary Governance

Decision framework for table decomposition, normalization levels, and junction table design in PostgreSQL schemas.

## Purpose

Entity boundaries — where one table ends and another begins — are the most consequential structural decisions in schema design. Too few tables produce "god tables" with dozens of nullable columns. Too many tables produce join explosions that degrade query performance and developer comprehension. This skill provides the heuristics and diagnostic checks for making boundary decisions consistently.

Use this skill when designing new schemas, reviewing table structures for decomposition opportunities, or evaluating whether denormalization is justified.

---

## Normalization Diagnostics

Normalization eliminates redundancy by ensuring each fact is stored exactly once. The standard target is 3NF (Third Normal Form).

### 1NF: Atomic Values

Every column contains a single, indivisible value. No repeating groups, no arrays storing structured data that should be rows.

| Violation | Example | Fix |
|-----------|---------|-----|
| Comma-separated values | `tags text = 'urgent,billing,vip'` | Junction table `entity_tags` or `text[]` with GIN index |
| Embedded structure | `address text = '123 Main St, Springfield, IL'` | Separate columns: `street`, `city`, `state` |
| Repeating columns | `phone1`, `phone2`, `phone3` | Child table `phone_numbers` |

**Detection:** Scan for columns containing comma-separated data patterns, numbered column suffixes (`_1`, `_2`, `_3`), or `text` columns storing structured data.

### 2NF: Full Functional Dependency

Every non-key column depends on the entire primary key, not just part of it. Relevant only for composite primary keys.

| Violation | Example | Fix |
|-----------|---------|-----|
| Partial dependency | `order_items(order_id, product_id, product_name)` — `product_name` depends only on `product_id` | Move `product_name` to `products` table |

**Detection:** In tables with composite PKs, check whether any non-key column depends on only one component of the PK.

### 3NF: No Transitive Dependencies

Every non-key column depends directly on the primary key, not through another non-key column.

| Violation | Example | Fix |
|-----------|---------|-----|
| Transitive dependency | `orders(id, customer_id, customer_name, customer_city)` — `customer_city` depends on `customer_id`, not `order.id` | Access `customer_name` and `customer_city` via FK to `customers` |
| Derived values | `orders(id, subtotal, tax, total)` — `total` = `subtotal + tax` | Compute `total` at query time or use a generated column |

**Detection:** For each non-key column, ask: "Does this column describe the entity identified by the PK, or does it describe something referenced by another column?"

---

## Denormalization Decision Matrix

Denormalization is justified only when measured query performance requires it and the maintenance cost is accepted.

| Justification | Technique | Maintenance Cost |
|---------------|-----------|-----------------|
| Read-heavy aggregation | Materialized view | `REFRESH MATERIALIZED VIEW CONCURRENTLY` on schedule |
| Dashboard counters | Summary table with trigger-based updates | Trigger complexity; eventual consistency risk |
| Frequently joined columns | Redundant column with FK source-of-truth | Application must update both; risk of drift |
| Search/display optimization | Denormalized read model | Separate write and read paths; CQRS pattern |

### Decision Criteria

Before denormalizing, confirm ALL of these:

1. **Measured bottleneck** — the join is proven slow via `EXPLAIN ANALYZE`, not assumed slow
2. **Read-to-write ratio > 100:1** — the table is read far more than written
3. **Documented** — a comment or ADR explains why the denormalization exists
4. **Maintained** — a trigger, materialized view, or application logic keeps the denormalized data consistent
5. **Bounded** — the denormalization is limited to specific columns, not wholesale table duplication

---

## Table Width Heuristics

### Too Wide: Decompose

A table is a decomposition candidate when:

| Signal | Threshold | Action |
|--------|-----------|--------|
| Column count | > 30 columns | Evaluate for entity extraction |
| Nullable column percentage | > 50% nullable | Suggests optional sub-entities |
| Column name prefixes | `billing_`, `shipping_`, `contact_` | Extract to separate tables |
| Mixed update frequency | Some columns updated hourly, others yearly | Separate hot and cold columns |
| Feature-gated columns | Columns only relevant for specific plans/features | Separate to feature-specific tables |

### Too Fragmented: Consolidate

A schema is over-normalized when:

| Signal | Threshold | Action |
|--------|-----------|--------|
| Join depth | > 3 joins for common queries | Consider consolidation |
| 1:1 relationships | Tables always joined together | Merge unless separation has a specific reason |
| Single-column child tables | Child table has only FK + one data column | Merge into parent |
| Identical lifecycles | Two tables always created/deleted together | Merge into one |

---

## Junction Table Design

Junction tables implement many-to-many relationships. Design choices affect query performance, constraint enforcement, and extensibility.

### Composite PK vs Surrogate + Unique

| Approach | Schema | Trade-offs |
|----------|--------|------------|
| Composite PK | `PRIMARY KEY (user_id, role_id)` | No surrogate ID; natural key is the identity; simpler for pure M2M |
| Surrogate + Unique | `id bigint PK, UNIQUE (user_id, role_id)` | Enables FK references to the junction row; needed if junction has child tables |

**Decision rule:** Use composite PK for pure join tables with no attributes and no child references. Use surrogate PK when the junction row is itself an entity that other tables reference.

### Junction Tables with Attributes

When the relationship carries data, the junction table becomes a first-class entity.

```sql
-- Pure junction (composite PK)
CREATE TABLE project_members (
    project_id bigint NOT NULL REFERENCES projects(id),
    user_id bigint NOT NULL REFERENCES users(id),
    PRIMARY KEY (project_id, user_id)
);

-- Junction with attributes (surrogate PK)
CREATE TABLE enrollments (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    course_id bigint NOT NULL REFERENCES courses(id),
    student_id bigint NOT NULL REFERENCES students(id),
    enrolled_at timestamptz NOT NULL DEFAULT now(),
    grade text,
    completed_at timestamptz,
    UNIQUE (course_id, student_id)
);
```

**Rule:** If a junction table has more than 2 columns beyond the FKs, it is an entity — give it a name reflecting its domain concept (e.g., `enrollments`, `memberships`, `assignments`), not a generic name (e.g., `course_student`).

### Index Strategy for Junctions

```sql
-- Composite PK (project_id, user_id) creates an index on (project_id, user_id)
-- This supports: WHERE project_id = ? AND user_id = ?
-- AND: WHERE project_id = ?
-- But NOT: WHERE user_id = ?

-- Add reverse lookup index:
CREATE INDEX idx_project_members_user_id ON project_members (user_id);
```

**Rule:** Always add a reverse-lookup index on the second column of a composite PK junction table. Without it, queries starting from the second entity require a sequential scan.

---

## Entity Boundary Heuristics

### Single Table Criteria

Keep as a single table when:
- All columns describe one real-world concept
- All columns have the same lifecycle (created/updated/deleted together)
- Column count is < 20
- Nullable percentage is < 30%
- No column name prefixes suggesting sub-entities

### Split Criteria

Split into multiple tables when:
- Columns divide into distinct real-world concepts
- Subsets of columns have independent lifecycles
- A column group is optional (nullable) for most rows
- A column group is accessed by different query patterns than the rest
- A polymorphic type column determines which other columns are relevant

### Polymorphic Entities

When a table has a "type" column and different columns are relevant per type:

| Approach | When to Use |
|----------|------------|
| Single Table Inheritance | Few types (< 5), mostly shared columns, simple queries |
| Class Table Inheritance | Many type-specific columns, type-specific constraints needed |
| Shared FK to base table | Each type has its own table, all share a FK to a common parent |

**Decision rule:** If more than 40% of columns are only relevant for a specific type, use Class Table Inheritance.

---

## Anti-Patterns

### 1. God Table

A single table with 50+ columns covering multiple concepts (user profile, billing info, preferences, audit data). Causes wide row reads, excessive nullable columns, and impossible-to-understand schemas.

### 2. Premature Decomposition

Splitting a 10-column table into three 4-column tables "for normalization." The join cost exceeds any benefit, and every query requires multiple joins.

### 3. Generic Junction Names

Naming junction tables `table1_table2` (e.g., `user_role`) instead of domain-meaningful names (e.g., `role_assignments`). When the junction acquires attributes, the name becomes misleading.

### 4. Missing Reverse Index

Creating a junction table with composite PK `(a_id, b_id)` without an index on `b_id`. Queries starting from entity B require a full table scan.

### 5. EAV (Entity-Attribute-Value)

Storing entity attributes as rows (`entity_id, attribute_name, attribute_value`) instead of columns. Prevents type safety, constraint enforcement, and efficient querying. Use JSONB if schema flexibility is genuinely needed.

---

## Examples

### Example 1: Decomposing a Wide Users Table

**Schema under review:**
```sql
CREATE TABLE users (
    id bigint PRIMARY KEY,
    email text NOT NULL,
    name text NOT NULL,
    billing_street text,
    billing_city text,
    billing_state text,
    billing_zip text,
    billing_country text,
    shipping_street text,
    shipping_city text,
    shipping_state text,
    shipping_zip text,
    shipping_country text,
    stripe_customer_id text,
    plan_name text,
    plan_started_at timestamptz,
    created_at timestamptz NOT NULL DEFAULT now()
);
```

**Findings:**
- `billing_` and `shipping_` prefixes indicate address sub-entities — extract to `addresses` table with a `type` column
- `stripe_customer_id`, `plan_name`, `plan_started_at` describe a subscription — extract to `subscriptions` table
- Core `users` table retains: `id`, `email`, `name`, `created_at`

### Example 2: Junction Table Design Decision

**Scenario:** Users can belong to organizations with a role and join date.

**Analysis:** The relationship carries attributes (`role`, `joined_at`), making it a first-class entity.

**Recommendation:**
```sql
CREATE TABLE memberships (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id),
    organization_id bigint NOT NULL REFERENCES organizations(id),
    role text NOT NULL DEFAULT 'member',
    joined_at timestamptz NOT NULL DEFAULT now(),
    UNIQUE (user_id, organization_id)
);

CREATE INDEX idx_memberships_organization_id ON memberships (organization_id);
```
