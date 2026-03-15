---
name: governing-spec-structure
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of multi-file OpenAPI specification structure including directory layout conventions for paths, components/schemas, components/parameters, and components/responses, file-level $ref strategies, bundling with Redocly CLI into single openapi.json, split vs monolithic anti-patterns, and root file conventions. Use when designing multi-file spec layouts, auditing spec directory structure, reviewing $ref file strategies, or configuring spec bundling. Triggered by: multi-file spec, spec structure, spec layout, $ref files, Redocly bundle, split spec, spec directory, paths directory, schemas directory, spec organization, file structure, spec files.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Spec Structure Governance

Comprehensive governance for multi-file OpenAPI specification organization -- directory layout, file-level `$ref` strategies, bundling, and root file conventions.

## Purpose

A single monolithic `openapi.yaml` file works for small APIs but becomes unmanageable as specifications grow beyond 20-30 operations. Files exceeding 2,000 lines produce merge conflicts on every PR, make code review impractical, and obscure the logical structure of the API. This skill enforces a multi-file directory layout that maps the API's logical structure to the filesystem, uses file-level `$ref` to connect the pieces, and bundles the result into a single file for tooling consumption.

**Boundary:** This skill governs how the spec is split across files within the spec directory. For where the spec directory lives in the repository and how it relates to generated artifacts, see `contract-enforcement-governance/governing-contract-structure`. For component reuse patterns within files (`$ref` to components), see `governing-component-reuse`.

Use this skill when designing multi-file spec layouts, auditing spec directory structure, or configuring spec bundling.

---

## Directory Layout

### Recommended Structure

```
openapi/
  openapi.yaml                 # Root file (entry point)
  paths/                       # Path items, one file per resource
    projects.yaml
    organizations.yaml
    users.yaml
  components/
    schemas/                   # Domain schemas, one file per resource domain
      project.yaml             # Project, ProjectCreate, ProjectUpdate, ProjectResponse
      organization.yaml
      common.yaml              # ProblemDetail, FieldError, PaginationMeta
    parameters/                # Shared parameters
      pagination.yaml          # page, page_size query params
      path-ids.yaml            # project_id, org_id path params
    responses/                 # Shared error responses
      errors.yaml              # BadRequest, Unauthorized, Forbidden, NotFound, ValidationError
    examples/                  # Shared examples (optional)
      project-examples.yaml
```

### Layout Rules

| Rule | Rationale |
|------|-----------|
| One file per resource domain under `paths/` | Each resource evolves independently; reduces merge conflicts |
| One file per resource domain under `components/schemas/` | Groups related schemas (Create, Update, Response) together |
| Shared schemas in a dedicated `common.yaml` | ProblemDetail, pagination, and other cross-cutting schemas are easy to find |
| Parameters and responses in their own directories | Separates concerns; parameters and responses are referenced differently than schemas |
| Root file contains only metadata and `$ref` pointers | Provides a table of contents; no inline definitions |

### File Naming Convention

| Convention | Example | Rationale |
|-----------|---------|-----------|
| Kebab-case filenames | `project-members.yaml` | Consistent with URL conventions |
| Resource-domain grouping | `project.yaml` (contains Project, ProjectCreate, ProjectResponse) | One mental model per file |
| Singular nouns for schema files | `project.yaml` not `projects.yaml` | Schema files describe a domain, not a collection |
| Plural nouns for path files | `projects.yaml` not `project.yaml` | Path files describe collection endpoints (`/projects`) |

---

## Root File Conventions

### Root File Structure

The root `openapi.yaml` serves as the entry point and table of contents:

```yaml
openapi: "3.1.0"
info:
  title: "Acme Platform API"
  summary: "Manage projects, organizations, and users"
  description: |
    Full API description with authentication and rate limiting details.
  version: "1.2.0"
  contact:
    name: "Platform Team"
    email: "platform@acme.com"
  license:
    name: "Proprietary"
  x-api-id: "acme-platform"
  x-audience: "company-internal"

servers:
  - url: "https://api.acme.com/v1"
    description: "Production"
  - url: "https://api.staging.acme.com/v1"
    description: "Staging"

externalDocs:
  description: "Acme Platform Documentation"
  url: "https://docs.acme.com"

tags:
  - name: Projects
    description: "Manage projects and project settings"
  - name: Organizations
    description: "Manage organizations and memberships"
  - name: Users
    description: "User profiles and preferences"

paths:
  /projects:
    $ref: "./paths/projects.yaml#/projects"
  /projects/{project_id}:
    $ref: "./paths/projects.yaml#/projects~1{project_id}"
  /organizations:
    $ref: "./paths/organizations.yaml#/organizations"
  /users/me:
    $ref: "./paths/users.yaml#/users~1me"

components:
  schemas:
    Project:
      $ref: "./components/schemas/project.yaml#/Project"
    ProjectCreate:
      $ref: "./components/schemas/project.yaml#/ProjectCreate"
    ProjectResponse:
      $ref: "./components/schemas/project.yaml#/ProjectResponse"
    ProblemDetail:
      $ref: "./components/schemas/common.yaml#/ProblemDetail"
    FieldError:
      $ref: "./components/schemas/common.yaml#/FieldError"

  parameters:
    PageParam:
      $ref: "./components/parameters/pagination.yaml#/PageParam"
    PageSizeParam:
      $ref: "./components/parameters/pagination.yaml#/PageSizeParam"

  responses:
    BadRequest:
      $ref: "./components/responses/errors.yaml#/BadRequest"
    Unauthorized:
      $ref: "./components/responses/errors.yaml#/Unauthorized"
    NotFound:
      $ref: "./components/responses/errors.yaml#/NotFound"
    ValidationError:
      $ref: "./components/responses/errors.yaml#/ValidationError"
```

### Root File Rules

| Rule | Rationale |
|------|-----------|
| Root file contains all metadata (info, servers, tags, externalDocs) | Single place for API-level configuration |
| Root file has no inline path items, schemas, or responses | Everything is referenced via `$ref` |
| Paths section enumerates every endpoint with `$ref` | Serves as a table of contents for the API surface |
| Components section enumerates every reusable component with `$ref` | Tooling can discover all components from the root |
| Tags are defined in the root file, not in path files | Tag taxonomy is an API-level concern |

---

## File-Level `$ref` Strategies

### Strategy 1: Root-Level Pointers (Recommended)

The root file points to definitions within domain files:

```yaml
# openapi.yaml
paths:
  /projects:
    $ref: "./paths/projects.yaml#/projects"

# paths/projects.yaml
projects:
  get:
    operationId: listProjects
    tags: [Projects]
    # ...
  post:
    operationId: createProject
    tags: [Projects]
    # ...
```

### Strategy 2: Direct File `$ref`

Each file is a complete path item or schema:

```yaml
# openapi.yaml
paths:
  /projects:
    $ref: "./paths/projects.yaml"

# paths/projects.yaml (file IS the path item)
get:
  operationId: listProjects
  # ...
post:
  operationId: createProject
  # ...
```

### `$ref` Strategy Rules

| Rule | Rationale |
|------|-----------|
| Choose one strategy and use it consistently | Mixed strategies confuse contributors |
| Cross-file `$ref` always uses relative paths (`./`) | Absolute paths break when the spec is moved |
| Schema files may reference other schema files | `ProjectResponse` can `$ref` to `common.yaml#/ProblemDetail` |
| Path files reference schemas via root-relative paths | `$ref: "#/components/schemas/ProjectCreate"` ensures bundled output resolves correctly |
| Avoid circular cross-file references | File A referencing file B which references file A breaks bundlers |

---

## Bundling

### Why Bundle

Multi-file specs must be bundled into a single file for consumption by:
- SDK generators (openapi-ts, openapi-generator)
- Breaking change detectors (oasdiff)
- Conformance testers (Schemathesis)
- Documentation renderers (Redoc, Swagger UI)
- Linters that operate on the resolved spec

### Redocly CLI Bundling

```bash
# Bundle to JSON (recommended for tooling)
redocly bundle openapi/openapi.yaml -o generated/openapi.bundled.json

# Bundle to YAML
redocly bundle openapi/openapi.yaml -o generated/openapi.bundled.yaml
```

### Redocly Configuration

```yaml
# redocly.yaml
apis:
  main:
    root: openapi/openapi.yaml

rules:
  no-unresolved-refs: error
  no-unused-components: warn
  spec-components-invalid-map-name: error

output:
  ext: json
```

### Bundling Rules

| Rule | Rationale |
|------|-----------|
| Bundle output is JSON (not YAML) | JSON is unambiguous, faster to parse, and universally supported |
| Bundle output lives in a `generated/` directory | Clearly separates authored from generated artifacts |
| Bundle must resolve all `$ref` pointers | Unresolved refs break downstream tools |
| Bundle output must be deterministic | Same input produces same output for caching |
| `redocly bundle` is the recommended tool | Handles all `$ref` strategies, validates during bundling |

---

## Split vs Monolithic Decision

### When to Split

| Signal | Action |
|--------|--------|
| Spec exceeds 500 lines | Consider splitting |
| More than 10 operations | Split paths by resource |
| Multiple contributors | Split to reduce merge conflicts |
| Multiple resource domains | One file per domain |
| Schemas used across resources | Extract to shared schema files |

### When Monolithic Is Acceptable

| Signal | Action |
|--------|--------|
| Fewer than 10 operations | Single file is manageable |
| Single contributor | No merge conflict risk |
| Prototype or spike | Speed over structure |
| Internal tool with limited lifespan | Overhead of splitting is not justified |

### Anti-Patterns

| Anti-Pattern | Problem |
|--------------|---------|
| One file per operation | Over-splitting; dozens of tiny files add navigation overhead |
| Splitting schemas but not paths | Inconsistent structure; paths become the bottleneck |
| No root file (flat directory of fragments) | No entry point; tooling cannot discover the spec |
| Monolithic spec with 50+ operations | Merge conflicts, difficult review, slow editing |
| Generated bundle checked in alongside source files | Confusion about which is authoritative |

---

## Review Dimensions

When reviewing or auditing spec structure, evaluate these dimensions:

### 1. Directory Organization
- Does the layout follow the recommended structure?
- Are path files grouped by resource domain?
- Are schema files grouped by resource domain?
- Are shared components in dedicated files?

### 2. Root File Quality
- Does the root file contain all metadata?
- Are all paths and components enumerated with `$ref`?
- Is there any inline content in the root file?
- Are tags defined at the root level?

### 3. `$ref` Consistency
- Is one `$ref` strategy used consistently?
- Are all cross-file references relative?
- Are there circular cross-file references?
- Do path files use root-relative component references?

### 4. Bundling
- Is bundling configured and working?
- Is the bundle output in a generated directory?
- Is the bundle deterministic?
- Do downstream tools consume the bundle, not source files?

---

## Guardrails

1. **Root file must contain only metadata and `$ref` pointers** -- no inline definitions
2. **One `$ref` strategy, used consistently** -- mixed strategies confuse contributors
3. **All cross-file references must be relative** -- absolute paths break portability
4. **Bundle output must be in a `generated/` directory** -- separates authored from generated
5. **Downstream tools consume the bundle, not source files** -- ensures all consumers see the resolved spec
6. **Split when the spec exceeds 500 lines or 10 operations** -- keep files reviewable
