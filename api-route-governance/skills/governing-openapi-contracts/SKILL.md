---
name: governing-openapi-contracts
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI specification compliance and FastAPI configuration including 3.1.0 baseline requirements, metadata completeness, root_path configuration, operation ID conventions, docstring truncation with form feed, response model declarations, and tag taxonomy. Use when reviewing OpenAPI specs, auditing FastAPI configuration, enforcing API documentation standards, or implementing OpenAPI compliance. Triggered by: OpenAPI, openapi, FastAPI, swagger, API spec, API specification, operation ID, root_path, API documentation, tag taxonomy, response model, API metadata, openapi.json, spec compliance.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# OpenAPI Contract Governance

Comprehensive governance for OpenAPI specification compliance and FastAPI-specific configuration, ensuring API documentation is accurate, complete, and useful.

## Purpose

The OpenAPI specification is the machine-readable contract between an API and its consumers. An incomplete or inaccurate OpenAPI spec generates incorrect client libraries, misleading documentation, and broken integrations. In FastAPI applications, the OpenAPI spec is auto-generated from code -- making code-level configuration the source of truth. This skill enforces standards that keep the generated spec accurate, complete, and useful.

Use this skill when configuring FastAPI's OpenAPI generation, reviewing OpenAPI specs, auditing API documentation completeness, or implementing spec compliance.

---

## OpenAPI Version Baseline

### 3.1.0 Baseline

All new APIs must target OpenAPI 3.1.0 as the minimum specification version:

```python
app = FastAPI(
    openapi_version="3.1.0",
    # ...
)
```

**Key 3.1.0 features over 3.0.x:**
- Full JSON Schema 2020-12 alignment (nullable via `type: ["string", "null"]` instead of `nullable: true`)
- Webhooks as first-class objects
- `pathItems` for reusable path definitions
- `$ref` alongside sibling properties (no need for `allOf` workarounds)

### 3.2.0 Roadmap Awareness

OpenAPI 3.2.0 (upcoming) will introduce:
- **QUERY method** -- Standardized replacement for POST-based search endpoints
- **Structured tags** -- Tags with metadata objects instead of bare strings
- **Overlay specification** -- Declarative spec patching without modifying the base spec

**Action:** Do not use 3.2.0 features yet. Design APIs so they can adopt these features when FastAPI supports them.

---

## FastAPI Metadata Configuration

### Required Metadata Fields

```python
app = FastAPI(
    title="Acme Platform API",
    summary="REST API for the Acme multi-tenant platform",
    description="Provides CRUD operations for projects, organizations, and members.",
    version="1.0.0",
    openapi_version="3.1.0",
    terms_of_service="https://acme.com/terms",
    contact={
        "name": "API Support",
        "url": "https://acme.com/support",
        "email": "api-support@acme.com",
    },
    license_info={
        "name": "Proprietary",
        "url": "https://acme.com/license",
    },
    servers=[
        {"url": "https://api.acme.com/api/v1", "description": "Production"},
        {"url": "https://staging-api.acme.com/api/v1", "description": "Staging"},
    ],
)
```

### Metadata Completeness Checklist

| Field | Required | Purpose |
|-------|----------|---------|
| `title` | Yes | API name in generated docs |
| `summary` | Yes | One-line description |
| `description` | Yes | Detailed overview (supports Markdown) |
| `version` | Yes | API version (semver) |
| `openapi_version` | Yes | Spec version (3.1.0) |
| `terms_of_service` | Recommended | Legal terms URL |
| `contact` | Recommended | Support contact info |
| `license_info` | Recommended | License details |
| `servers` | Recommended | Base URL list for environments |

### `root_path` Configuration

When the API runs behind a reverse proxy or API gateway that strips a path prefix, configure `root_path`:

```python
# If the gateway routes /api/v1/* to the FastAPI app at /*
app = FastAPI(root_path="/api/v1")
```

**Rules:**
- Set `root_path` when a proxy strips path prefixes
- The OpenAPI spec's `servers` URLs will include the `root_path`
- Test that the Swagger UI at `/docs` correctly constructs full URLs
- Never hardcode the proxy prefix in route paths -- use `root_path`

---

## Operation ID Conventions

### Naming Pattern

Operation IDs are the unique identifiers for each endpoint in the OpenAPI spec. They become function names in generated client libraries.

**Pattern:** `{verb}_{resource}` or `{verb}_{resource}_{sub_resource}`

| Method | Path | Operation ID |
|--------|------|-------------|
| GET | `/projects` | `list_projects` |
| POST | `/projects` | `create_project` |
| GET | `/projects/{id}` | `get_project` |
| PUT | `/projects/{id}` | `update_project` |
| DELETE | `/projects/{id}` | `delete_project` |
| GET | `/projects/{id}/members` | `list_project_members` |
| POST | `/projects/{id}:archive` | `archive_project` |

**Rules:**
- Use snake_case (these become function names in Python/Ruby clients)
- Use verb prefixes: `list_`, `get_`, `create_`, `update_`, `delete_`
- Custom operations drop the method prefix: `archive_project`, not `post_archive_project`
- Operation IDs must be unique across the entire API
- FastAPI sets `operation_id` on route decorators:

```python
@router.get("/projects", operation_id="list_projects")
async def list_projects():
    ...
```

### Auto-Generated Operation IDs

FastAPI auto-generates operation IDs from function names. If you rely on this, ensure function names follow the convention:

```python
# Good: function name = operation ID
async def list_projects():  # operation_id: "list_projects"
    ...

# Bad: generic function name
async def get():  # operation_id: "get" (ambiguous)
    ...
```

---

## Docstring Truncation

### Form Feed Character (`\f`)

FastAPI uses the first paragraph of a route's docstring as the `summary` and subsequent paragraphs as the `description` in the OpenAPI spec. Use the form feed character (`\f`) to truncate what appears in the spec:

```python
@router.get("/projects/{project_id}")
async def get_project(project_id: UUID):
    """Retrieve a single project by ID.

    Returns the full project object including settings and member count.
    \f
    Internal implementation notes that should NOT appear in the OpenAPI spec:
    - Queries the projects table with a LEFT JOIN on members
    - Caches result for 60 seconds
    - Falls back to read replica on primary failure
    """
```

**Rules:**
- Everything before `\f` appears in the OpenAPI spec (visible to consumers)
- Everything after `\f` is internal documentation (invisible to consumers)
- Use `\f` to prevent leaking implementation details into the public spec
- The first line before any blank line becomes the `summary`; subsequent lines before `\f` become the `description`

---

## Response Model Declarations

### Explicit Response Models

Every endpoint must declare its response model for accurate OpenAPI spec generation:

```python
@router.get("/projects/{project_id}", response_model=ProjectResponse)
async def get_project(project_id: UUID) -> ProjectResponse:
    ...

@router.post("/projects", response_model=ProjectResponse, status_code=201)
async def create_project(body: ProjectCreate) -> ProjectResponse:
    ...
```

### Multiple Response Codes

Declare all possible response codes with their models:

```python
@router.post(
    "/projects",
    response_model=ProjectResponse,
    status_code=201,
    responses={
        409: {"model": ProblemDetail, "description": "Project slug already exists"},
        422: {"model": ProblemDetail, "description": "Validation error"},
    },
)
async def create_project(body: ProjectCreate) -> ProjectResponse:
    ...
```

### Response Model Rules

| Rule | Implementation |
|------|---------------|
| Every endpoint has `response_model` | Ensures schema in spec |
| Success code matches behavior | 201 for creation, 204 for deletion |
| Error responses are declared | 4xx/5xx in `responses` dict |
| Use `response_model_exclude_none=True` | Omit null fields from response |

---

## Tag Taxonomy

### Organizing Endpoints by Tags

Tags group related endpoints in the OpenAPI spec and generated documentation:

```python
@router.get("/projects", tags=["Projects"])
async def list_projects():
    ...

@router.get("/organizations/{org}/members", tags=["Members"])
async def list_members():
    ...
```

### Tag Rules

| Rule | Example |
|------|---------|
| One primary tag per endpoint | `tags=["Projects"]` |
| Tags use PascalCase plural nouns | `"Projects"`, `"Members"`, `"API Keys"` |
| Define tag metadata in FastAPI | `tags_metadata` list |
| Order tags by importance | Most-used resources first |

### Tag Metadata

```python
tags_metadata = [
    {
        "name": "Projects",
        "description": "Create, read, update, and delete projects.",
    },
    {
        "name": "Members",
        "description": "Manage organization membership and roles.",
    },
    {
        "name": "Authentication",
        "description": "Token management and authentication flows.",
    },
]

app = FastAPI(openapi_tags=tags_metadata)
```

---

## Review Dimensions

When reviewing or auditing OpenAPI compliance, evaluate these dimensions:

### 1. Spec Version and Metadata
- Is the OpenAPI version set to 3.1.0?
- Are all required metadata fields populated (title, summary, description, version)?
- Is `root_path` configured correctly for the deployment environment?

### 2. Operation IDs
- Do all endpoints have explicit, unique operation IDs?
- Do operation IDs follow the `verb_resource` convention?
- Will generated client libraries have readable function names?

### 3. Response Declarations
- Does every endpoint declare a `response_model`?
- Are error responses (4xx, 5xx) declared in the `responses` dict?
- Do success status codes match the endpoint behavior (201 for creation)?

### 4. Documentation Quality
- Do endpoints have meaningful summaries and descriptions?
- Is `\f` used to exclude internal notes from the spec?
- Are tags applied consistently with metadata defined?

---

## Anti-Patterns

### 1. Missing Response Models
Endpoints without `response_model` produce `{}` schemas in the OpenAPI spec. Consumers cannot generate typed clients.

### 2. Default OpenAPI Version
Not setting `openapi_version`, which defaults to 3.0.x in some FastAPI versions. Explicitly set 3.1.0.

### 3. Leaked Implementation Details
Docstrings that describe database queries, cache strategies, or internal service calls without `\f` truncation. These details appear in the public spec.

### 4. Generic Operation IDs
Auto-generated operation IDs like `get_project_project_id_get` from FastAPI's default naming. Set explicit `operation_id` values.

### 5. Missing Error Responses
Only declaring the success response model. Without declared error responses, generated clients lack error type definitions.

### 6. Untagged Endpoints
Endpoints without tags appear in an "default" group in Swagger UI, making the documentation hard to navigate.

---

## Examples

### Example 1: Minimal FastAPI Configuration

**Scenario:** FastAPI app has `title` only. No `openapi_version`, no `summary`, no `contact`, no `servers`.

**Findings:**
- **Spec Version and Metadata: FAIL** -- Missing required metadata fields, no explicit OpenAPI version
- **Recommendation:** Add all required metadata, set `openapi_version="3.1.0"`, configure servers list

### Example 2: Well-Configured API

**Scenario:** 3.1.0 version set, all metadata populated, every endpoint has operation_id and response_model, tags with metadata, `\f` used in docstrings, error responses declared.

**Findings:**
- All four dimensions: **PASS**
