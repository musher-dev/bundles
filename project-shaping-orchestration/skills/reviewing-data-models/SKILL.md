---
name: shaping-reviewing-data-models
version: 1.0.0
user-invocable: false
description: Produce a data model review evaluating normalization, referential integrity, naming conventions, temporal patterns, migration safety, and index strategy for a shaped architecture sketch. Use when an architecture sketch proposes data shapes that need validation before implementation begins. Triggered by: data model review, normalization, constraints, migration safety, schema design, entity relationships, data integrity, database design review.
allowed-tools:
  - Read
---

# Data Model Review

Expert guidance for producing data model review artifacts during the shaping process.

## Purpose

A data model review evaluates whether the data shapes proposed in an Architecture Sketch form a sound, maintainable, and safely deployable schema. It catches normalization issues, missing constraints, unsafe migration patterns, and naming inconsistencies before any data is written to production.

Use this skill after an Architecture Sketch has been produced and the data shape section needs formal review. Catching data model issues during shaping is orders of magnitude cheaper than fixing them after data is in production.

---

## Output Format

```markdown
# Data Model Review

**Artifact Reviewed:** <Name of the Architecture Sketch>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** Data Modeler

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Normalization | Pass / Partial / Fail | <Brief justification> |
| 2 | Referential Integrity | Pass / Partial / Fail | <Brief justification> |
| 3 | Naming Conventions | Pass / Partial / Fail | <Brief justification> |
| 4 | Temporal Patterns | Pass / Partial / Fail | <Brief justification> |
| 5 | Migration Safety | Pass / Partial / Fail | <Brief justification> |
| 6 | Index Strategy | Pass / Partial / Fail | <Brief justification> |

---

## Entity Relationship Summary

<Text description of key entities and their relationships. Describe cardinality
(one-to-one, one-to-many, many-to-many), ownership semantics (parent-child,
association), and lifecycle dependencies (cascade delete, orphan protection).>

---

## Constraint Inventory

| Entity | Field | Constraint Type | Rationale |
|--------|-------|----------------|-----------|
| ... | ... | NOT NULL / UNIQUE / FK / CHECK / DEFAULT | ... |
...

---

## Migration Risk Assessment

| Change | Risk Level | Mitigation | Requires Downtime? |
|--------|-----------|------------|-------------------|
| ... | Low / Medium / High | ... | Yes / No |
...

---

## Findings

### Critical

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Major

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Minor

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

---

## Recommendations

1. <Priority-ordered recommendation with rationale>
2. ...

---

## Conditions for Approval

- [ ] All entities are normalized to at least 3NF or denormalization is explicitly justified
- [ ] Foreign key constraints are defined for every cross-entity reference
- [ ] Table and column names follow a single consistent naming convention
- [ ] Temporal columns (created_at, updated_at) are present on all mutable entities
- [ ] Every proposed schema change has a migration risk assessment
- [ ] Query access patterns are identified and index strategy is documented
- [ ] No Critical findings remain unresolved

---

## Verdict

**<Approved / Approved with Conditions / Rejected>**

<Summary rationale — 2-3 sentences.>
```

---

## Instructions

1. **Map the entity landscape.** Read the Architecture Sketch and extract every entity, table, or data shape proposed. For each entity, record its fields, data types, and stated relationships. Write the Entity Relationship Summary describing how entities connect, including cardinality and lifecycle dependencies. If data types or relationships are unspecified, note them as "Unspecified."

2. **Evaluate each dimension.** Walk through the six review dimensions:
   - **Normalization:** Check for redundant fields, repeated groups, transitive dependencies. Verify 3NF. If denormalization exists, check whether it is justified by documented read performance requirements.
   - **Referential Integrity:** Verify every cross-entity reference has a defined foreign key. Check cascade/restrict behavior on delete. Identify orphan risks.
   - **Naming Conventions:** Check table name pluralization and casing consistency. Verify column naming patterns (snake_case, boolean prefixes, FK naming).
   - **Temporal Patterns:** Verify created_at/updated_at on mutable entities. Check soft-delete patterns where required. Verify timezone handling.
   - **Migration Safety:** Assess each schema change for production deployment risk. Identify destructive operations. Determine zero-downtime compatibility.
   - **Index Strategy:** Identify expected query patterns. Check for indexes on FKs and frequently filtered fields. Evaluate composite index needs.

3. **Classify findings by severity.**
   - **Critical:** Missing FKs allowing orphaned records, destructive migrations with no rollback, 1NF violations.
   - **Major:** Missing temporal columns, no FK indexes, naming violations across entities, NOT NULL additions without defaults.
   - **Minor:** Boolean naming inconsistencies, missing check constraints, minor pluralization issues.

4. **Formulate recommendations.** Priority-ordered, each specifying the exact change needed and referencing the finding it addresses.

5. **Render the verdict.** All conditions met with no Critical findings = "Approved." Conditions met with Minor/Major = "Approved with Conditions." Critical findings unresolved = "Rejected."

---

## Quality Checklist

- [ ] Every entity in the Architecture Sketch is analyzed with fields, types, and relationships documented
- [ ] The Entity Relationship Summary describes cardinality and lifecycle dependencies in plain language
- [ ] Each dimension has a rating with a written justification, not just a Pass/Partial/Fail label
- [ ] The Constraint Inventory includes every FK, unique, and NOT NULL constraint with rationale
- [ ] The Migration Risk Assessment covers every schema change with risk level, mitigation, and downtime impact
- [ ] Findings include impact statements describing data integrity consequences
- [ ] The verdict is consistent with the Conditions for Approval checklist

---

## Anti-Patterns

### 1. Reviewing in Isolation from Access Patterns
Evaluating the model without considering how it will be queried. A perfectly normalized schema that requires five joins for every page load is a design failure. Always cross-reference against known read and write patterns.

### 2. Treating All Denormalization as Wrong
Flagging every denormalized field without evaluating the performance justification. Some read-heavy patterns legitimately benefit from controlled denormalization. The review should assess whether the justification is documented.

### 3. Ignoring Migration Sequencing
Reviewing the final schema state without considering the steps to get there. A column rename requires an expand-contract migration with data backfill. The review must evaluate the migration path, not just the destination.

### 4. Skipping Constraint Rationale
Listing constraints without explaining why each exists. A NOT NULL on a description field is a design choice that affects UX. The Constraint Inventory must capture reasoning.

### 5. Assuming the ORM Handles It
Treating referential integrity as an application concern. FK constraints, unique constraints, and check constraints must be enforced at the database level regardless of application-layer validation.

---

## Examples

### Example 1: Multi-Tenant Workspace Model

An Architecture Sketch proposes `organization`, `workspace`, and `member` entities.

**Referential Integrity** is rated **Partial** because the `member` entity references both `workspace_id` and `user_id` without specifying cascade behavior. If a workspace is deleted, should members be cascade-deleted or should deletion be blocked?

**Migration Safety** is rated **Fail** because the sketch proposes adding a NOT NULL `tier` column to the existing `organization` table without a default value. For a populated table, this requires a three-step migration: add nullable column, backfill, then alter to NOT NULL.

### Example 2: Event Log Schema

An Architecture Sketch proposes an `event_log` entity with `event_type`, `actor_id`, `target_type`, `target_id`, `payload` (JSON), and `occurred_at`.

**Index Strategy** is rated **Partial** because the sketch mentions queries by `actor_id` and `target_type + target_id` but proposes no indexes. Recommended: single-column index on `actor_id` and composite index on `(target_type, target_id, occurred_at)`.

**Temporal Patterns** is rated **Partial** because the entity has `occurred_at` but not `created_at`. These can differ if events are processed asynchronously.