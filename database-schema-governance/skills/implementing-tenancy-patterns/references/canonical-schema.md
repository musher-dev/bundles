# Canonical Multi-Tenant Schema

Complete table definitions implementing the User-Owned Organization tenancy model.

---

## users

Global identity table. Not tenant-scoped.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  display_name VARCHAR(255) NOT NULL,
  avatar_url TEXT,
  email_verified_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_email ON users(email);
```

**Design notes:**
- `email` is the only globally unique field — it is login identity
- No password hash here; authentication is delegated to an auth provider or a separate `user_credentials` table
- No `tenant_id` — users are global entities

---

## organizations

The tenant boundary. Every resource in the system belongs to exactly one Organization.

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL UNIQUE,
  type VARCHAR(20) NOT NULL CHECK (type IN ('personal', 'corporate')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_organizations_slug ON organizations(slug);
```

**Design notes:**
- `type` distinguishes personal orgs (auto-created per user, single member) from corporate orgs (multi-member)
- `slug` is globally unique for URL routing (e.g., `/orgs/acme-corp`)
- No authentication fields — identity lives on `users`

---

## memberships

Bridge between users and organizations. Defines who belongs where and with what role.

```sql
CREATE TABLE memberships (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  role_id UUID NOT NULL REFERENCES roles(id),
  status VARCHAR(20) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'invited', 'suspended')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_memberships_user_org
  ON memberships(user_id, organization_id);
CREATE INDEX idx_memberships_user ON memberships(user_id);
CREATE INDEX idx_memberships_org ON memberships(organization_id);
```

**Design notes:**
- One membership per user per organization (enforced by unique index)
- `role_id` references the `roles` table, not a raw string
- `status` supports invitation flows and account suspension
- Cascade delete: removing a user or org removes their memberships

---

## roles

Role definitions with tenant-scoping support for customization.

```sql
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_default BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_roles_org_name
  ON roles(organization_id, name);
CREATE INDEX idx_roles_org ON roles(organization_id);
```

**Design notes:**
- `organization_id IS NULL` → global template role (e.g., "Owner", "Member", "Viewer")
- `organization_id IS NOT NULL` → tenant-customized role
- `is_default` marks the role assigned to new members when no role is specified
- The unique index on `(organization_id, name)` allows different orgs to have roles with the same name

---

## permissions

Atomic capability strings. Application logic checks these, never role names.

```sql
CREATE TABLE permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key VARCHAR(255) NOT NULL UNIQUE,
  description TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_permissions_key ON permissions(key);
```

**Capability string convention:** `resource.action`

| Key | Description |
|-----|-------------|
| `projects.create` | Create new projects |
| `projects.read` | View project details |
| `projects.update` | Modify project settings |
| `projects.delete` | Delete projects |
| `members.invite` | Invite new members |
| `members.remove` | Remove existing members |
| `roles.manage` | Create and modify custom roles |
| `billing.manage` | Manage billing and subscription |

---

## role_permissions

Join table linking roles to their granted permissions.

```sql
CREATE TABLE role_permissions (
  role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  permission_id UUID NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);

CREATE INDEX idx_role_permissions_role ON role_permissions(role_id);
```

**Design notes:**
- Composite primary key prevents duplicate grants
- Cascade delete: removing a role or permission cleans up the join
- Query pattern: "Does user X have permission Y in org Z?" joins `memberships → roles → role_permissions → permissions`

---

## projects

Tenant-scoped resource. Replaces the "workspace" concept.

```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,
  archived_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_projects_tenant_slug ON projects(tenant_id, slug);
CREATE UNIQUE INDEX idx_projects_tenant_name ON projects(tenant_id, name);
CREATE INDEX idx_projects_tenant_created ON projects(tenant_id, created_at DESC);
```

**Design notes:**
- `tenant_id` is the ownership FK — projects belong to organizations, not users
- `slug` and `name` are unique within a tenant, not globally
- `archived_at` supports soft-archive without soft-delete complexity

---

## audit_logs

Tenant-scoped compliance log. Created from day one, not retrofitted.

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

CREATE INDEX idx_audit_logs_tenant_created
  ON audit_logs(tenant_id, created_at DESC);
CREATE INDEX idx_audit_logs_tenant_entity
  ON audit_logs(tenant_id, entity_type, entity_id);
CREATE INDEX idx_audit_logs_actor
  ON audit_logs(actor_user_id, created_at DESC);
```

**Design notes:**
- No `ON DELETE CASCADE` from organizations — audit logs must survive org deletion for compliance
- No `updated_at` — audit logs are append-only, never modified
- `metadata` captures context-specific details as JSON (e.g., changed fields, previous values)
- `entity_type` + `entity_id` enable filtering logs for any resource without polymorphic FKs

---

## Adding New Resource Tables

When adding a new tenant-scoped resource table, follow this template:

```sql
CREATE TABLE <resource_name> (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  -- resource-specific columns here
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Tenant-scoped uniqueness (if applicable)
CREATE UNIQUE INDEX idx_<resource>_tenant_<unique_field>
  ON <resource_name>(tenant_id, <unique_field>);

-- Activity feed index
CREATE INDEX idx_<resource>_tenant_created
  ON <resource_name>(tenant_id, created_at DESC);

-- Additional tenant-scoped lookup indexes as needed
CREATE INDEX idx_<resource>_tenant_<filter_field>
  ON <resource_name>(tenant_id, <filter_field>);
```

---

## Permission Check Query

The canonical query for checking whether a user has a specific permission in an organization:

```sql
SELECT EXISTS (
  SELECT 1
  FROM memberships m
    JOIN roles r ON m.role_id = r.id
    JOIN role_permissions rp ON r.id = rp.role_id
    JOIN permissions p ON rp.permission_id = p.id
  WHERE m.user_id = :user_id
    AND m.organization_id = :organization_id
    AND m.status = 'active'
    AND p.key = :permission_key
) AS has_permission;
```
