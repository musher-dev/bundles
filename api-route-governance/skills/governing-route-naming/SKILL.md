---
name: governing-route-naming
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API route URI naming conventions including resource nouns, pluralization, nesting depth limits, kebab-case paths, snake_case query parameters and JSON properties, AIP-136 custom operations with colon-verb syntax, and identifier design with UUIDs and /me alias. Use when designing API routes, reviewing URI patterns, auditing endpoint naming, or enforcing REST naming conventions. Triggered by: route naming, URI design, resource noun, pluralization, nesting depth, kebab-case, snake_case, custom operation, colon verb, AIP-136, identifier design, UUID, /me alias, path design, endpoint naming.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Route Naming Governance

Comprehensive governance for API route URI design, enforcing consistent naming conventions across resource paths, query parameters, and JSON properties.

## Purpose

URI design is the most visible aspect of an API's developer experience. Inconsistent naming -- mixed casing, irregular pluralization, deeply nested paths, unpredictable parameter names -- forces consumers to memorize exceptions instead of learning patterns. This skill encodes canonical URI naming rules that make APIs predictable, discoverable, and self-documenting.

Use this skill when designing new API routes, reviewing existing URI patterns, auditing endpoint naming consistency, or implementing route naming conventions.

---

## Design Principles

### 1. Resource Nouns, Not Verbs

URIs identify resources (nouns), not actions (verbs). The HTTP method supplies the verb.

| Correct | Wrong | Why |
|---------|-------|-----|
| `POST /projects` | `POST /createProject` | HTTP method already means "create" |
| `DELETE /projects/{id}` | `POST /deleteProject` | HTTP method already means "delete" |
| `GET /projects/{id}/members` | `GET /getProjectMembers` | HTTP method already means "retrieve" |

### 2. Plural Resource Names

Collection resources use plural nouns. Singleton sub-resources may use singular when they represent a one-to-one relationship.

| Pattern | Example | When |
|---------|---------|------|
| Plural collection | `/projects`, `/users`, `/invoices` | Always for collections |
| Singular sub-resource | `/projects/{id}/settings` | One-to-one relationship |
| Plural sub-collection | `/projects/{id}/members` | One-to-many relationship |

**Anti-pattern:** Mixing singular and plural for collections (`/project` vs `/users`). Pick plural and enforce it universally.

### 3. Kebab-Case Paths

URI path segments use kebab-case (lowercase with hyphens). Never camelCase, snake_case, or PascalCase in paths.

| Correct | Wrong |
|---------|-------|
| `/api-keys` | `/apiKeys`, `/api_keys`, `/ApiKeys` |
| `/organization-memberships` | `/organizationMemberships` |
| `/billing-invoices` | `/billing_invoices` |

### 4. Snake_Case for Query Parameters and JSON Properties

Query parameters and JSON request/response properties use snake_case. This matches PostgreSQL column naming and avoids JavaScript-centric camelCase that alienates non-JS consumers.

| Layer | Convention | Example |
|-------|-----------|---------|
| URI path segments | kebab-case | `/billing-invoices` |
| Query parameters | snake_case | `?page_size=20&sort_by=created_at` |
| JSON properties | snake_case | `{ "created_at": "...", "org_id": "..." }` |
| HTTP headers | Title-Case-With-Hyphens | `Content-Type`, `X-Request-Id` |

---

## Nesting Depth

### Maximum 3 Segments Deep

URIs must not exceed 3 levels of resource nesting. Deeper nesting creates brittle URLs that are hard to construct and impossible to cache efficiently.

| Depth | Example | Verdict |
|-------|---------|---------|
| 1 | `/projects` | Good |
| 2 | `/projects/{id}/members` | Good |
| 3 | `/organizations/{slug}/projects/{id}/members` | Maximum |
| 4+ | `/organizations/{slug}/projects/{id}/members/{mid}/roles` | Too deep -- flatten |

**Flattening strategy:** Promote deeply nested resources to top-level with filtering:

```
# Instead of depth 4:
GET /organizations/{slug}/projects/{id}/members/{mid}/roles

# Flatten to depth 2 with filter:
GET /members/{mid}/roles?project_id={id}

# Or promote to top-level:
GET /member-roles?member_id={mid}&project_id={id}
```

---

## Custom Operations (AIP-136)

### Colon-Verb Syntax

When standard CRUD methods are insufficient, use the colon-verb pattern from Google's AIP-136:

```
POST /resources/{id}:action
```

The colon separates the resource identifier from the custom action, making it visually distinct from sub-resources.

| Operation | Pattern | Example |
|-----------|---------|---------|
| Archive | `POST /projects/{id}:archive` | Non-destructive soft removal |
| Unarchive | `POST /projects/{id}:unarchive` | Restore archived resource |
| Transfer | `POST /projects/{id}:transfer` | Change ownership |
| Clone | `POST /projects/{id}:clone` | Duplicate resource |
| Export | `POST /reports/{id}:export` | Trigger async export |
| Validate | `POST /schemas:validate` | Validate without persisting |

**Rules for custom operations:**
- Always use `POST` method (custom actions are not idempotent by default)
- Verb must be a present-tense action (`:archive`, not `:archived`)
- Never use custom operations for standard CRUD -- use HTTP methods instead
- Collection-level custom operations omit the ID: `POST /resources:import`

---

## Identifier Design

### UUIDs Over Sequential Integers

Resource identifiers in URIs must be UUIDs (or equivalent opaque identifiers). Never expose sequential integers.

| Correct | Wrong | Why |
|---------|-------|-----|
| `/users/a1b2c3d4-...` | `/users/42` | Sequential IDs leak business data |
| `/projects/7f3e...` | `/projects/1337` | Enumerable IDs enable scraping |

### The `/me` Alias

Provide `/me` as an alias for the authenticated user's own resource. This avoids requiring consumers to know their own user ID:

```
GET /me                    -- Current user profile
GET /me/organizations      -- Current user's organizations
GET /me/api-keys           -- Current user's API keys
```

**Rules:**
- `/me` resolves to the authenticated user's ID server-side
- `/me` must reject unauthenticated requests with `401 Unauthorized`
- Never allow `/me` with org-bound API keys -- use explicit user endpoints instead

### Slug vs UUID in Paths

Some resources support human-readable slugs alongside UUIDs:

```
GET /organizations/acme-corp     -- By slug (human-friendly)
GET /organizations/a1b2c3d4-...  -- By UUID (machine-friendly)
```

**Rules:**
- Slugs must be globally unique within their resource type
- The API must accept both slug and UUID in the same path parameter
- Slugs use kebab-case, are immutable after creation, and have length limits

---

## Review Dimensions

When reviewing or auditing route naming, evaluate these dimensions:

### 1. Resource Naming
- Are all collections plural nouns?
- Are path segments kebab-case?
- Are there any verbs in URI paths (outside custom operations)?

### 2. Casing Consistency
- Are all path segments kebab-case?
- Are all query parameters snake_case?
- Are all JSON properties snake_case?
- Are any mixed conventions present?

### 3. Nesting Depth
- Does any URI exceed 3 levels of resource nesting?
- Could deeply nested resources be flattened with query filters?

### 4. Custom Operations
- Do custom actions use the `:verb` syntax?
- Are any custom operations replaceable by standard CRUD methods?
- Do custom operations use POST?

### 5. Identifier Design
- Are sequential integer IDs exposed in any URI?
- Is the `/me` alias available for user-scoped resources?
- Do slug-capable resources accept both slug and UUID?

### Related Skills

- **`governing-payload-structure`** -- Defines how fields are organized within payloads (semantic field ontology) that complement route-level naming
- **`governing-field-serialization`** -- Extends snake_case naming to JSON field names and defines identifier serialization as opaque strings
- **`governing-mutation-semantics`** -- Defines sub-resource endpoint design for array mutation, complementing route nesting rules

---

## Anti-Patterns

### 1. Verb-Based URIs
Using `POST /createUser` instead of `POST /users`. The HTTP method already conveys the action. Verb-based URIs duplicate intent and create inconsistency.

### 2. Mixed Casing
Combining `camelCase` and `kebab-case` in paths (`/apiKeys` and `/billing-invoices` in the same API). Pick one convention and enforce it everywhere.

### 3. Deeply Nested Hierarchies
URIs like `/orgs/{id}/teams/{id}/projects/{id}/tasks/{id}/comments`. Every nesting level adds a required path parameter that the client must track. Flatten beyond depth 3.

### 4. Sequential Integer IDs in URIs
Exposing `/users/42` lets attackers enumerate all users by incrementing the ID. It also leaks business metrics (user count, growth rate).

### 5. Inconsistent Pluralization
Mixing `/user/{id}/projects` and `/teams` in the same API. If collections are plural, every collection is plural -- no exceptions.

### 6. Sub-Resource for Custom Actions
Using `POST /projects/{id}/archive` (sub-resource) instead of `POST /projects/{id}:archive` (custom operation). Sub-resource syntax implies `archive` is a nested collection, which creates confusion when listing sub-resources.

---

## Examples

### Example 1: API with Mixed Naming Conventions

**Scenario:** Routes use `/apiKeys` for some endpoints and `/billing-invoices` for others. Query parameters use both `pageSize` and `page_size`.

**Findings:**
- **Resource Naming: FAIL** -- Mixed casing in path segments (`apiKeys` is camelCase, `billing-invoices` is kebab-case)
- **Casing Consistency: FAIL** -- Query parameters use both camelCase and snake_case
- **Recommendation:** Standardize all paths to kebab-case (`/api-keys`) and all query parameters to snake_case (`page_size`)

### Example 2: Deeply Nested Project Routes

**Scenario:** `GET /organizations/{org}/projects/{proj}/environments/{env}/deployments/{dep}/logs`

**Findings:**
- **Nesting Depth: FAIL** -- 5 levels of nesting (max is 3)
- **Recommendation:** Flatten to `GET /deployment-logs?deployment_id={dep}` or `GET /deployments/{dep}/logs`

### Example 3: Well-Designed API Surface

**Scenario:** All collections plural kebab-case, query params snake_case, max depth 2, custom operations use `:verb`, UUIDs in all paths, `/me` alias available.

**Findings:**
- All five dimensions: **PASS**
