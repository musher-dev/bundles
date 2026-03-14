---
name: governing-operation-identifiers
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI operation and component identifiers including operationId naming conventions (camelCase verbNoun), uniqueness enforcement, component key naming regex, schema naming (UpperCamelCase), parameter naming (snake_case for query, camelCase for path), and SDK generation impact of identifier choices. Use when reviewing operationId naming, auditing component key conventions, enforcing schema naming, or optimizing specs for SDK generation. Triggered by: operationId, operation ID, component key, schema naming, parameter naming, SDK generation, client generation, camelCase, UpperCamelCase, snake_case, identifier naming, spec naming, component naming, operation naming.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Operation Identifier Governance

Comprehensive governance for identifiers within OpenAPI specifications -- operationIds, component keys, schema names, and parameter names -- ensuring consistency, uniqueness, and optimal SDK generation.

## Purpose

Every identifier in an OpenAPI spec becomes a symbol in generated code. A poorly named `operationId` becomes a confusing function name in every SDK. An inconsistent schema key becomes an unpredictable class name across languages. This skill enforces naming conventions that produce clean, predictable, idiomatic code in generated clients across all target languages.

**Boundary:** This skill governs identifiers within the OpenAPI specification document itself. For framework-specific operation ID configuration (e.g., FastAPI's `operation_id` parameter), see `api-route-governance/governing-openapi-contracts`.

Use this skill when naming operations, schemas, parameters, or component keys in OpenAPI specifications.

---

## operationId Naming

### Convention: camelCase verbNoun

Every operation must have an explicit `operationId` following `camelCase` with a verb prefix:

```yaml
paths:
  /projects:
    get:
      operationId: listProjects
    post:
      operationId: createProject
  /projects/{projectId}:
    get:
      operationId: getProject
    put:
      operationId: updateProject
    delete:
      operationId: deleteProject
  /projects/{projectId}/members:
    get:
      operationId: listProjectMembers
    post:
      operationId: addProjectMember
  /projects/{projectId}:archive:
    post:
      operationId: archiveProject
```

### Verb Vocabulary

Use a controlled set of verb prefixes:

| Verb | Semantics | HTTP Method |
|------|-----------|-------------|
| `list` | Retrieve a collection | GET |
| `get` | Retrieve a single resource | GET |
| `create` | Create a new resource | POST |
| `update` | Full replacement of a resource | PUT |
| `patch` | Partial update of a resource | PATCH |
| `delete` | Remove a resource | DELETE |
| `search` | Complex query with body | POST (or QUERY) |

Custom operation verbs should be domain-specific and self-documenting:

| Verb | Example | Semantics |
|------|---------|-----------|
| `archive` | `archiveProject` | Move to archived state |
| `activate` | `activateSubscription` | Enable a resource |
| `validate` | `validateWebhook` | Check without persisting |
| `export` | `exportReport` | Generate downloadable output |
| `revoke` | `revokeApiKey` | Invalidate a credential |

### Uniqueness Requirement

`operationId` must be unique across the entire API specification:

- No two operations may share the same `operationId`, even across different paths
- SDK generators use `operationId` as function names -- duplicates cause compilation errors
- Validate uniqueness as a CI check (Spectral rule: `operation-operationId-unique`)

### Why camelCase (Not snake_case)

The OpenAPI specification community convention and most SDK generators expect camelCase:

| Language | camelCase `listProjects` | snake_case `list_projects` |
|----------|------------------------|---------------------------|
| JavaScript/TypeScript | `client.listProjects()` (native) | `client.list_projects()` (non-idiomatic) |
| Python | `client.list_projects()` (auto-converted) | `client.list_projects()` (native) |
| Java | `client.listProjects()` (native) | `client.listProjects()` (requires conversion) |
| Go | `client.ListProjects()` (auto-converted) | `client.ListProjects()` (requires conversion) |

Most generators convert camelCase to language-idiomatic forms. Starting with camelCase produces the best results across all targets.

---

## Component Key Naming

### Schema Keys: UpperCamelCase

Schema component keys use UpperCamelCase (PascalCase) and become class/type names in generated code:

```yaml
components:
  schemas:
    Project:
      type: object
      properties: ...
    ProjectCreate:
      type: object
      properties: ...
    ProjectResponse:
      type: object
      properties: ...
    PaginatedProjectList:
      type: object
      properties: ...
    ProblemDetail:
      type: object
      properties: ...
```

**Key regex:** `^[A-Z][a-zA-Z0-9]+$`

### Schema Naming Conventions

| Pattern | Usage | Example |
|---------|-------|---------|
| `{Resource}` | Core domain entity | `Project`, `Organization` |
| `{Resource}Create` | Request body for creation | `ProjectCreate` |
| `{Resource}Update` | Request body for full update | `ProjectUpdate` |
| `{Resource}Patch` | Request body for partial update | `ProjectPatch` |
| `{Resource}Response` | Response body | `ProjectResponse` |
| `Paginated{Resource}List` | Collection response | `PaginatedProjectList` |
| `{Resource}Summary` | Abbreviated representation | `ProjectSummary` |

### Parameter Keys

Parameter component keys use camelCase:

```yaml
components:
  parameters:
    projectId:
      name: project_id
      in: path
      schema:
        type: string
        format: uuid
    pageSize:
      name: page_size
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
```

**Key regex:** `^[a-z][a-zA-Z0-9]+$`

Note: The component key (`projectId`) follows camelCase for the reference name, while the `name` field (`project_id`) follows the wire format convention for the parameter location.

### Response Keys

Response component keys use UpperCamelCase descriptive names:

```yaml
components:
  responses:
    NotFound:
      description: "Resource not found"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
    ValidationError:
      description: "Request validation failed"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
```

**Key regex:** `^[A-Z][a-zA-Z0-9]+$`

### Security Scheme Keys

Security scheme component keys use camelCase:

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
```

---

## Parameter Naming (Wire Format)

### Query Parameters: snake_case

```yaml
parameters:
  - name: page_size
    in: query
  - name: sort_by
    in: query
  - name: created_after
    in: query
  - name: include_archived
    in: query
```

### Path Parameters: snake_case or camelCase

Path parameters should match the URL convention. For kebab-case URLs, use snake_case parameter names:

```yaml
paths:
  /projects/{project_id}/members/{member_id}:
    get:
      parameters:
        - name: project_id
          in: path
        - name: member_id
          in: path
```

### Header Parameters: Kebab-Title-Case

```yaml
parameters:
  - name: X-Request-Id
    in: header
  - name: X-Correlation-Id
    in: header
```

### Cookie Parameters: snake_case

```yaml
parameters:
  - name: session_token
    in: cookie
```

---

## SDK Generation Impact

### How Identifiers Map to Generated Code

| Spec Identifier | TypeScript | Python | Java | Go |
|----------------|------------|--------|------|-----|
| `operationId: listProjects` | `listProjects()` | `list_projects()` | `listProjects()` | `ListProjects()` |
| Schema: `ProjectResponse` | `ProjectResponse` | `ProjectResponse` | `ProjectResponse` | `ProjectResponse` |
| Schema: `ProblemDetail` | `ProblemDetail` | `ProblemDetail` | `ProblemDetail` | `ProblemDetail` |
| Param: `page_size` | `pageSize` | `page_size` | `pageSize` | `PageSize` |

### SDK Generation Rules

1. **operationId becomes a function name** -- unclear IDs create confusing APIs
2. **Schema keys become class/type names** -- inconsistent casing breaks naming conventions across languages
3. **Parameter names become function arguments** -- overly long or abbreviated names reduce usability
4. **Enum values become constants** -- must be valid identifiers in all target languages

### Common SDK Generation Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Duplicate method names | Non-unique operationIds | Ensure global uniqueness |
| Unnamed request body class | Missing schema key for inline schema | Extract to named component |
| Confusing method name | Generic operationId like `getData` | Use specific verb+noun: `getProjectMetrics` |
| Invalid identifier in target language | Schema key starting with number or containing special characters | Use `^[A-Z][a-zA-Z0-9]+$` regex |

---

## Review Dimensions

When reviewing or auditing operation identifiers, evaluate these dimensions:

### 1. operationId Completeness
- Does every operation have an explicit `operationId`?
- Are all operationIds unique across the spec?
- Do operationIds follow camelCase verbNoun convention?

### 2. Verb Consistency
- Is the verb vocabulary consistent (list/get/create/update/delete)?
- Do custom verbs accurately describe the operation semantics?
- Are verbs from the controlled vocabulary used where applicable?

### 3. Schema Key Conventions
- Do all schema keys follow UpperCamelCase?
- Do schema names follow the `{Resource}{Suffix}` pattern?
- Are schemas extracted from inline definitions into named components?

### 4. Parameter Naming
- Are query parameters snake_case?
- Are path parameters consistent with URL conventions?
- Are header parameters Kebab-Title-Case?

### 5. SDK Readability
- Will generated function names be self-documenting?
- Will generated class names be idiomatic in target languages?
- Are there any identifiers that would produce invalid symbols?

---

## Anti-Patterns

### 1. Missing operationId
Auto-generated IDs (e.g., `get_projects__project_id__get`) produce unreadable SDK methods and break when paths change.

### 2. Inconsistent Verb Usage
Mixing `fetch`, `retrieve`, `find`, and `get` for the same semantic operation. Pick one verb per semantic meaning.

### 3. snake_case Schema Keys
`project_response` instead of `ProjectResponse`. Most generators expect UpperCamelCase for type names.

### 4. Inline Schemas Without Names
Request/response bodies defined inline without a schema name produce `InlineObject1`, `InlineObject2` in generated code.

### 5. Overly Generic operationIds
`getData`, `processRequest`, `handleAction` -- these become meaningless function names across every SDK.

### 6. Path Parameters Leaking Implementation
`{id}` instead of `{projectId}` -- the parameter name should indicate which resource it identifies.

---

## Guardrails

1. **Every operation must have an explicit operationId** -- never rely on auto-generation
2. **operationIds must be globally unique** -- enforce via CI linting
3. **Schema keys must match `^[A-Z][a-zA-Z0-9]+$`** -- ensures valid type names in all languages
4. **Use the controlled verb vocabulary** -- consistency across the spec reduces cognitive load
5. **Test with at least one SDK generator** -- the generated code reveals naming issues that manual review misses
6. **Inline schemas must be extracted** -- every request/response body deserves a named schema
