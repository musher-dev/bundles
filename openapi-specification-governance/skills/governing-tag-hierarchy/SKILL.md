---
name: governing-tag-hierarchy
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI tag taxonomy including 3.2.0 hierarchical tags with parent, summary, and kind fields, tag classification types (navigation, badge, lifecycle, audience), migration strategies from flat tags and x-tagGroups vendor extensions, and tag governance for documentation portals and SDK grouping. Use when designing tag taxonomy, reviewing tag structure, migrating from flat tags, implementing hierarchical tags, or organizing API endpoints by domain. Triggered by: tag hierarchy, hierarchical tags, tag taxonomy, x-tagGroups, tag classification, tag kind, tag parent, flat tags, tag migration, endpoint grouping, tag governance, API tags, documentation tags, navigation tags.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Tag Hierarchy Governance

Comprehensive governance for OpenAPI tag taxonomy -- from flat tag lists through vendor extensions to the 3.2.0 hierarchical tag specification.

## Purpose

Tags are the primary navigation mechanism in API documentation portals. A flat, ungoverned tag list produces a wall of unrelated groups that consumers cannot scan. Hierarchical tags with classification metadata enable documentation portals to render navigation trees, badge indicators, and audience-filtered views. This skill enforces tag structure that scales from 5 endpoints to 500.

**Boundary:** This skill governs tag structure within the OpenAPI specification document. For framework-specific tag configuration (e.g., FastAPI's `openapi_tags` parameter), see `api-route-governance/governing-openapi-contracts`.

Use this skill when designing tag taxonomy, migrating from flat tags, or implementing hierarchical tags for large API surfaces.

---

## Tag Fundamentals

### Basic Tag Structure (3.0.x / 3.1.0)

Tags in current OpenAPI versions are flat string labels with optional metadata:

```yaml
tags:
  - name: Projects
    description: "Create, read, update, and delete projects"
  - name: Organizations
    description: "Manage organizations and membership"
  - name: Authentication
    description: "Token management and authentication flows"

paths:
  /projects:
    get:
      tags:
        - Projects
```

### Tag Rules (All Versions)

| Rule | Rationale |
|------|-----------|
| Every operation must have at least one tag | Untagged operations appear in "default" group |
| One primary tag per operation | Multiple tags duplicate the operation in docs |
| Tags use PascalCase plural nouns | Consistency with resource naming |
| Every tag must have a top-level definition with description | Undeclared tags appear without context |
| Tag order in the `tags` array determines display order | Most important resources first |

---

## Tag Classification

### Classification Types

Tags serve different purposes beyond navigation. Classify each tag by its function:

| Kind | Purpose | Example | Display Treatment |
|------|---------|---------|-------------------|
| `navigation` | Primary grouping for endpoint browsing | `Projects`, `Organizations` | Sidebar section |
| `badge` | Operational status indicator | `Beta`, `Deprecated`, `Experimental` | Visual badge on endpoint |
| `lifecycle` | Release stage of the endpoint group | `GA`, `Preview`, `Sunset` | Status indicator |
| `audience` | Who should use these endpoints | `Public`, `Partner`, `Internal` | Visibility filter |

### Applying Classification (Current Versions)

Without native support, encode classification in extensions:

```yaml
tags:
  - name: Projects
    description: "CRUD operations for projects"
    x-kind: navigation
  - name: Beta
    description: "Endpoints in beta -- subject to breaking changes"
    x-kind: badge
  - name: Partner
    description: "Available to contracted partners only"
    x-kind: audience
```

### Multiple Tags With Classification

When an operation needs both a navigation tag and a badge tag:

```yaml
paths:
  /projects/{id}/analytics:
    get:
      tags:
        - Projects     # navigation: where to find it
        - Beta         # badge: stability indicator
```

**Rule:** At most one navigation tag and one badge/lifecycle/audience tag per operation. Never assign two navigation tags.

---

## 3.2.0 Hierarchical Tags

### New Tag Object Fields

OpenAPI 3.2.0 introduces structured tag objects with hierarchy support:

```yaml
tags:
  - name: Platform
    summary: "Core platform APIs"
    kind: navigation
  - name: Projects
    summary: "Project management"
    parent: Platform
    kind: navigation
  - name: Organizations
    summary: "Organization and membership"
    parent: Platform
    kind: navigation
  - name: Integrations
    summary: "Third-party integration APIs"
    kind: navigation
  - name: Webhooks
    summary: "Webhook configuration and delivery"
    parent: Integrations
    kind: navigation
  - name: Beta
    summary: "Subject to breaking changes without notice"
    kind: badge
```

### 3.2.0 Tag Fields

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Tag identifier (unchanged from 3.0) |
| `summary` | string | Short description for navigation display |
| `description` | string | Detailed description (Markdown) |
| `parent` | string | Name of the parent tag (creates hierarchy) |
| `kind` | string | Classification: `navigation`, `badge`, etc. |
| `externalDocs` | object | Link to external documentation |

### Hierarchy Rules

| Rule | Rationale |
|------|-----------|
| Maximum 3 levels deep | Deeper trees create navigation fatigue |
| Parent tags may or may not have operations directly | A parent can be purely organizational |
| Child tags inherit the parent's audience scope | Internal parent means internal children |
| Hierarchy must form a tree (no cycles, no multiple parents) | Display rendering requires tree structure |

### Hierarchy Visualization

```
Platform (navigation)
├── Projects (navigation)
├── Organizations (navigation)
└── Members (navigation)
Integrations (navigation)
├── Webhooks (navigation)
└── OAuth Apps (navigation)
Beta (badge)
Deprecated (badge)
Partner (audience)
```

---

## Migration from Flat Tags

### Step 1: Inventory Current Tags

Collect all tags used across the specification:

```yaml
# Before: flat, ungoverned tags
tags:
  - name: projects
  - name: orgs
  - name: members
  - name: auth
  - name: webhooks
  - name: admin
  - name: beta
```

### Step 2: Classify Each Tag

Assign a classification to each tag:

| Current Tag | Classification | Action |
|-------------|---------------|--------|
| `projects` | navigation | Rename to `Projects` (PascalCase) |
| `orgs` | navigation | Rename to `Organizations` (full word) |
| `members` | navigation | Keep as `Members` |
| `auth` | navigation | Rename to `Authentication` |
| `webhooks` | navigation | Keep as `Webhooks` |
| `admin` | audience | Rename to `Internal` |
| `beta` | badge | Rename to `Beta` |

### Step 3: Design Hierarchy

Group navigation tags under parent categories:

```yaml
tags:
  - name: Platform
    description: "Core platform resources"
    x-kind: navigation
  - name: Projects
    description: "Project management"
    x-kind: navigation
    x-parent: Platform
  - name: Organizations
    description: "Organization and membership management"
    x-kind: navigation
    x-parent: Platform
```

### Step 4: Apply to Operations

Update every operation's tag assignment to use the new taxonomy.

---

## Migration from `x-tagGroups`

### Current `x-tagGroups` Pattern

Redocly and other tools support `x-tagGroups` for visual grouping:

```yaml
x-tagGroups:
  - name: Platform
    tags:
      - Projects
      - Organizations
      - Members
  - name: Integrations
    tags:
      - Webhooks
      - OAuth Apps
```

### Migration to 3.2.0 Hierarchical Tags

Map each `x-tagGroups` group to a parent tag:

```yaml
# Before: x-tagGroups (vendor extension)
x-tagGroups:
  - name: Platform
    tags: [Projects, Organizations, Members]

# After: 3.2.0 hierarchical tags
tags:
  - name: Platform
    kind: navigation
  - name: Projects
    parent: Platform
    kind: navigation
  - name: Organizations
    parent: Platform
    kind: navigation
  - name: Members
    parent: Platform
    kind: navigation
```

**Migration rules:**
- Each `x-tagGroups` entry becomes a parent tag with `kind: navigation`
- Child tags gain `parent` field pointing to the group name
- Remove `x-tagGroups` after migration (retain as fallback only during transition)
- Validate that documentation portals render both formats during transition

---

## Review Dimensions

When reviewing or auditing tag hierarchy, evaluate these dimensions:

### 1. Tag Coverage
- Does every operation have at least one tag?
- Are there untagged operations falling into the "default" group?
- Is there exactly one navigation tag per operation?

### 2. Tag Definitions
- Is every used tag defined in the top-level `tags` array?
- Does every tag have a description?
- Are tag names PascalCase plural nouns?

### 3. Classification Consistency
- Are tags classified by kind (navigation, badge, lifecycle, audience)?
- Is classification applied consistently (via native fields or extensions)?
- Are badge tags used appropriately (not as navigation)?

### 4. Hierarchy Quality
- Does the hierarchy have at most 3 levels?
- Does the hierarchy form a valid tree (no cycles)?
- Are parent tags meaningful organizational categories?

### 5. Migration Status
- Has `x-tagGroups` been migrated to hierarchical tags (or planned)?
- Are extension-based workarounds documented for pre-3.2.0 specs?
- Is there a transition plan for tooling that doesn't yet support 3.2.0?

---

## Anti-Patterns

### 1. Untagged Operations
Operations without tags appear in a "default" or "Other" group, breaking navigation.

### 2. Tag Proliferation
Creating a unique tag for every endpoint instead of grouping by resource domain. 50 tags is 50 sidebar items.

### 3. Inconsistent Tag Casing
Mixing `projects`, `Projects`, and `PROJECTS` across the spec. Tags are case-sensitive.

### 4. Multiple Navigation Tags
Assigning `tags: [Projects, Analytics]` to one operation -- it appears in both navigation groups, confusing consumers.

### 5. Undeclared Tags
Using tags in operations without defining them in the top-level `tags` array. Documentation portals render them without description or ordering.

### 6. Verb Tags
Tags like `Create`, `Update`, `Delete` that classify by action instead of resource domain. Tags should group by what (resources), not how (actions).

---

## Guardrails

1. **Every operation must have exactly one navigation tag** -- zero means lost; two means duplicated
2. **Every tag must be declared with a description** -- undeclared tags create confusion
3. **Tags are PascalCase plural nouns** -- consistency with resource naming conventions
4. **Hierarchy depth must not exceed 3 levels** -- deeper trees create navigation fatigue
5. **Badge and lifecycle tags supplement, never replace, navigation tags** -- an operation is always navigable
6. **Plan for 3.2.0 migration** -- use extensions today that map cleanly to native fields
