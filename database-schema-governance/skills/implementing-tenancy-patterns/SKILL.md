---
name: implementing-tenancy-patterns
version: 1.0.0
description: "Implement, audit, and optimize multi-tenant database schemas following the User-Owned Organization model. Use when creating tenant-scoped tables, reviewing migrations for isolation compliance, adding RBAC schema, or detecting tenancy anti-patterns. Triggered by: tenancy, multi-tenant, tenant_id, organization model, tenant isolation, RBAC schema, polymorphic ownership, membership table, tenant scoping, shared schema."
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Implementing Tenancy Patterns

Expert guidance for implementing, auditing, and optimizing multi-tenant database schemas using the User-Owned Organization model.

## Purpose

This skill encodes the User-Owned Organization multi-tenancy architecture as actionable governance for database schemas. It covers the full lifecycle: auditing existing schemas for isolation compliance, implementing new tenant-scoped tables, adjusting schemas that violate tenancy rules, and optimizing tenant isolation patterns.

---

## Canonical Tenancy Model

The User-Owned Organization pattern establishes four structural primitives:

- **Organization** — the sole tenant boundary. Every Organization is typed as either `personal` (single-user) or `corporate` (multi-user). All resources belong to an Organization, never directly to a User.
- **User** — a global identity that authenticates independently of any tenant. Users own no resources directly; they access resources through Organization membership.
- **Membership** — the bridge between User and Organization. Each Membership carries a role reference and a status (active, invited, suspended). A User may belong to multiple Organizations.
- **Every resource table** gets a `tenant_id UUID NOT NULL FK -> organizations` column. No exceptions.

---

## Terminology Mandate

### Banned Terms (as structural DB entities)

| Banned Term | Required Replacement | Rationale |
|-------------|---------------------|-----------|
| `Account` | `User` (for identity) or `Organization` (for tenant) | "Account" conflates identity and tenancy |
| `Workspace` | `Project` | "Workspace" implies UI, not data boundary |
| `Team` | `Organization` + `Membership` | "Team" is a subset concern, not a tenant boundary |

### Required Terms

Use `User`, `Organization`, `Project`, and `Membership` as the canonical structural entities. These terms must appear in table names, column names, and FK references exactly as specified.

---

## Implementation Rules

When creating or modifying tables, enforce these rules:

### 1. Tenant Scoping

Every resource table must include:

```sql
tenant_id UUID NOT NULL REFERENCES organizations(id)
```

No resource table may exist without `tenant_id`. The only tables exempt from this rule are `users` (global identity) and `organizations` (the tenant table itself).

### 2. Tenant-Scoped Uniqueness

Uniqueness constraints must be scoped to the tenant:

```sql
-- Correct: tenant-scoped uniqueness
UNIQUE(tenant_id, name)
UNIQUE(tenant_id, slug)
UNIQUE(tenant_id, email)  -- for org-scoped invitations

-- Wrong: global uniqueness on tenant-owned resources
UNIQUE(name)
UNIQUE(slug)
```

The only global uniqueness constraints allowed are on the `users` table (e.g., `UNIQUE(email)` for login identity).

### 3. Roles Table Design

The `roles` table uses a nullable `organization_id` to distinguish global templates from tenant-customized roles:

```sql
organization_id UUID REFERENCES organizations(id)
-- NULL = global template role (e.g., "Owner", "Member")
-- UUID = tenant-customized role
```

### 4. Permission Checking

Permissions are checked by capability string, never by role name:

```sql
-- Correct: check capability
SELECT 1 FROM role_permissions rp
  JOIN permissions p ON rp.permission_id = p.id
  WHERE rp.role_id = :role_id AND p.key = 'projects.create';

-- Wrong: check role name
SELECT 1 FROM memberships WHERE role = 'admin';
```

### 5. Audit Logging

Include an `audit_logs` table with `tenant_id` + `actor_user_id` from day one:

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES organizations(id),
  actor_user_id UUID NOT NULL REFERENCES users(id),
  action VARCHAR(255) NOT NULL,
  entity_type VARCHAR(255) NOT NULL,
  entity_id UUID NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## Anti-Pattern Detection

Flag and fix the following when found in schemas or migrations:

### 1. Polymorphic Ownership

```sql
-- Anti-pattern: polymorphic owner
owner_id UUID NOT NULL,
owner_type VARCHAR(50) NOT NULL  -- 'user' | 'organization'
```

**Fix:** Replace with explicit `tenant_id UUID NOT NULL REFERENCES organizations(id)`. Resources always belong to Organizations.

### 2. Resources FK'd to Users

```sql
-- Anti-pattern: resource owned by user
created_by UUID NOT NULL REFERENCES users(id)  -- as ownership, not attribution
```

**Fix:** Add `tenant_id` as the ownership FK. `created_by` may remain as an attribution column, but it is not the ownership boundary.

### 3. Identity Credentials on Tenant Table

```sql
-- Anti-pattern: auth fields on organizations
ALTER TABLE organizations ADD COLUMN password_hash VARCHAR(255);
ALTER TABLE organizations ADD COLUMN api_key VARCHAR(255);
```

**Fix:** Authentication belongs on the `users` table. API keys belong in a dedicated `api_keys` table with `tenant_id`.

### 4. Missing tenant_id

Any resource table without `tenant_id` is a tenancy violation. Flag it immediately with a recommendation to add the column.

### 5. Global Uniqueness on Tenant Resources

```sql
-- Anti-pattern
CREATE UNIQUE INDEX idx_projects_slug ON projects(slug);
```

**Fix:** `CREATE UNIQUE INDEX idx_projects_tenant_slug ON projects(tenant_id, slug);`

### 6. Role Name Checking

```sql
-- Anti-pattern: branching on role name
WHERE membership.role_name = 'admin'
```

**Fix:** Check permissions via capability strings through the `role_permissions` + `permissions` join.

---

## Index Strategy

### Leading tenant_id

All indexes on resource tables must lead with `tenant_id`:

```sql
-- Activity feeds
CREATE INDEX idx_resource_tenant_created
  ON resources(tenant_id, created_at DESC);

-- Tenant-scoped lookups
CREATE INDEX idx_resource_tenant_status
  ON resources(tenant_id, status);

-- Tenant-scoped search
CREATE INDEX idx_resource_tenant_name
  ON resources(tenant_id, name);
```

### Membership Lookups

```sql
-- "What orgs does this user belong to?"
CREATE INDEX idx_memberships_user ON memberships(user_id);

-- "Who belongs to this org?"
CREATE INDEX idx_memberships_org ON memberships(organization_id);

-- "What role does this user have in this org?"
CREATE UNIQUE INDEX idx_memberships_user_org
  ON memberships(user_id, organization_id);
```

### Permission Lookups

```sql
-- "What permissions does this role grant?"
CREATE INDEX idx_role_permissions_role
  ON role_permissions(role_id);
```

---

## Migration Safety

### New Resource Tables

Always include `tenant_id` in the initial `CREATE TABLE`. Never create a resource table with the intent to "add tenant_id later."

### Adding tenant_id to Existing Tables

Use a three-step expand-contract migration:

```sql
-- Step 1: Add nullable column
ALTER TABLE existing_resources ADD COLUMN tenant_id UUID REFERENCES organizations(id);

-- Step 2: Backfill (in batches)
UPDATE existing_resources SET tenant_id = (
  SELECT organization_id FROM /* derive from existing relationships */
) WHERE tenant_id IS NULL;

-- Step 3: Add NOT NULL constraint (separate migration, after backfill confirmed)
ALTER TABLE existing_resources ALTER COLUMN tenant_id SET NOT NULL;
```

### Breaking Changes

Use the Expand-Contract pattern:

1. **Expand:** Add new column/table alongside old
2. **Migrate:** Backfill data, update application to write to both
3. **Contract:** Remove old column/table after verification

Never rename or drop columns in a single migration step on populated tables.

---

## Reference

See `./references/canonical-schema.md` for the complete canonical table definitions, constraints, indexes, and FK relationships that implement this tenancy model.
