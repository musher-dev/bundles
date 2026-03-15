---
name: governing-component-reuse
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI component reuse patterns including $ref usage, inline schema anti-patterns, component organization strategies, schema composition with allOf oneOf and discriminator, RFC 9457 ProblemDetails as reusable component, and SDK generation impact of composition choices. Use when reviewing $ref patterns, auditing inline schemas, organizing components, designing schema composition, or implementing discriminated unions. Triggered by: $ref, component reuse, inline schema, allOf, oneOf, discriminator, schema composition, ProblemDetails, RFC 9457, components section, schema organization, DRY schemas, ref pattern, reusable schema.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Component Reuse Governance

Comprehensive governance for OpenAPI component reuse -- `$ref` patterns, schema composition, discriminated unions, and shared component organization.

## Purpose

OpenAPI specs without component reuse become maintenance nightmares. The same error format duplicated across 50 endpoints diverges silently. An inline schema edited in one path drifts from its twin in another. This skill enforces the DRY principle at the spec level: extract once into `components`, reference everywhere with `$ref`, compose with `allOf`/`oneOf`/`discriminator`.

**Boundary:** This skill governs reuse patterns within the OpenAPI specification document. For framework-specific schema implementation (Pydantic models, response_model), see `api-route-governance/governing-schema-implementation`.

Use this skill when organizing components, designing schema composition, or auditing specs for inline duplication.

---

## `$ref` Patterns

### Basic Component Reference

Extract reusable schemas into `components/schemas` and reference them:

```yaml
paths:
  /projects:
    get:
      responses:
        "200":
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/PaginatedProjectList"
        "401":
          $ref: "#/components/responses/Unauthorized"
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ProjectCreate"
      responses:
        "201":
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProjectResponse"
        "422":
          $ref: "#/components/responses/ValidationError"
```

### `$ref` Rules

| Rule | Rationale |
|------|-----------|
| Every request body schema must be a `$ref` | Inline request bodies produce unnamed classes in SDKs |
| Every response schema must be a `$ref` | Ensures consistent types across endpoints |
| Error responses should use response-level `$ref` | Shares both schema and description |
| Parameters used in multiple paths must be a `$ref` | Prevents parameter definition drift |
| Security schemes are always in `components` | Required by the spec |

### `$ref` with Sibling Properties (3.1.0+)

OpenAPI 3.1.0 allows properties alongside `$ref`, eliminating the need for `allOf` wrappers:

```yaml
# 3.1.0 -- $ref with siblings (clean)
properties:
  owner:
    $ref: "#/components/schemas/UserSummary"
    description: "The user who created this project"

# 3.0.x workaround -- allOf wrapper (avoid if on 3.1.0)
properties:
  owner:
    allOf:
      - $ref: "#/components/schemas/UserSummary"
    description: "The user who created this project"
```

---

## Inline Schema Anti-Patterns

### What Constitutes an Inline Schema

An inline schema is any schema defined directly in a path item rather than referenced from `components`:

```yaml
# Anti-pattern: inline schema
paths:
  /projects:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                description:
                  type: string
```

### When Inline Schemas Are Acceptable

| Scenario | Inline OK? | Reason |
|----------|-----------|--------|
| Simple wrapper with one property | Yes | Extraction adds noise without value |
| Enum-only schema used in one place | Yes | Single-use enum is self-documenting inline |
| Any schema used in 2+ places | No | Must extract to prevent drift |
| Any request/response body | No | SDK generators need named types |
| Any schema with 3+ properties | No | Complexity warrants extraction and naming |

### Extraction Process

1. Identify the inline schema
2. Name it following UpperCamelCase convention (see `governing-operation-identifiers`)
3. Move to `components/schemas`
4. Replace with `$ref`
5. Search for duplicate definitions across the spec and consolidate

---

## Component Organization

### Component Sections

OpenAPI defines these component categories:

```yaml
components:
  schemas:        # Data models
  responses:      # Reusable response definitions
  parameters:     # Reusable parameter definitions
  examples:       # Reusable examples
  requestBodies:  # Reusable request bodies
  headers:        # Reusable header definitions
  securitySchemes: # Authentication definitions
  links:          # Reusable link definitions
  callbacks:      # Reusable callback definitions
  pathItems:      # Reusable path items (3.1.0+)
```

### Schema Organization Strategy

Group schemas logically within `components/schemas`:

```yaml
components:
  schemas:
    # --- Domain: Projects ---
    Project:
      type: object
    ProjectCreate:
      type: object
    ProjectUpdate:
      type: object
    ProjectPatch:
      type: object
    ProjectResponse:
      type: object
    ProjectSummary:
      type: object
    PaginatedProjectList:
      type: object

    # --- Domain: Organizations ---
    Organization:
      type: object
    OrganizationResponse:
      type: object

    # --- Shared: Error Responses ---
    ProblemDetail:
      type: object
    ValidationError:
      type: object

    # --- Shared: Pagination ---
    PaginationMeta:
      type: object
    PaginationLinks:
      type: object
```

### Multi-File Specs

For multi-file spec directory layout, file-level `$ref` strategies, bundling, and root file conventions, see `governing-spec-structure`.

---

## Schema Composition

### `allOf` -- Inheritance / Extension

Use `allOf` to compose schemas through extension:

```yaml
components:
  schemas:
    ResourceBase:
      type: object
      properties:
        id:
          type: string
          format: uuid
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time

    ProjectResponse:
      allOf:
        - $ref: "#/components/schemas/ResourceBase"
        - type: object
          properties:
            name:
              type: string
            slug:
              type: string
          required:
            - name
            - slug
```

**`allOf` rules:**
- Use for "is-a" relationships (ProjectResponse is a ResourceBase with extra fields)
- All `allOf` schemas are merged -- property names must not conflict
- SDK generators produce inheritance hierarchies or flattened types
- Limit `allOf` depth to 2 levels -- deeper nesting confuses generators

### `oneOf` -- Discriminated Unions

Use `oneOf` with `discriminator` for polymorphic types:

```yaml
components:
  schemas:
    Notification:
      oneOf:
        - $ref: "#/components/schemas/EmailNotification"
        - $ref: "#/components/schemas/SlackNotification"
        - $ref: "#/components/schemas/WebhookNotification"
      discriminator:
        propertyName: channel
        mapping:
          email: "#/components/schemas/EmailNotification"
          slack: "#/components/schemas/SlackNotification"
          webhook: "#/components/schemas/WebhookNotification"

    EmailNotification:
      type: object
      required: [channel, recipient]
      properties:
        channel:
          type: string
          enum: [email]
        recipient:
          type: string
          format: email

    SlackNotification:
      type: object
      required: [channel, slack_channel]
      properties:
        channel:
          type: string
          enum: [slack]
        slack_channel:
          type: string
```

**`oneOf` + `discriminator` rules:**
- Always pair `oneOf` with `discriminator` -- without it, validators must try every schema
- The discriminator property must exist in every variant schema
- Each variant's discriminator value must be a string constant (`enum` with single value)
- SDK generators produce tagged unions or sealed classes

### `oneOf` Without Discriminator (Avoid)

```yaml
# Anti-pattern: oneOf without discriminator
components:
  schemas:
    PaymentMethod:
      oneOf:
        - $ref: "#/components/schemas/CreditCard"
        - $ref: "#/components/schemas/BankTransfer"
      # No discriminator -- validators must try each schema
```

This forces runtime trial-and-error deserialization and produces poor SDK types.

### `anyOf` -- Use Sparingly

`anyOf` means "matches one or more of these schemas." It has limited SDK support:

- Most generators treat `anyOf` as `oneOf` (pick one)
- Use only when the value genuinely can match multiple schemas simultaneously
- Prefer `oneOf` with discriminator for union types

---

## RFC 9457 ProblemDetails as Reusable Component

### Standard Error Schema

Define RFC 9457 Problem Details as a reusable component:

```yaml
components:
  schemas:
    ProblemDetail:
      type: object
      required:
        - type
        - title
        - status
      properties:
        type:
          type: string
          format: uri
          description: "URI reference identifying the problem type"
          example: "https://api.acme.com/problems/not-found"
        title:
          type: string
          description: "Short human-readable summary"
          example: "Resource Not Found"
        status:
          type: integer
          description: "HTTP status code"
          example: 404
        detail:
          type: string
          description: "Human-readable explanation specific to this occurrence"
          example: "Project with ID '123' does not exist"
        instance:
          type: string
          format: uri
          description: "URI identifying this specific occurrence"
        errors:
          type: array
          description: "Field-level validation errors (for 422 responses)"
          items:
            $ref: "#/components/schemas/FieldError"

    FieldError:
      type: object
      required:
        - field
        - message
      properties:
        field:
          type: string
          description: "JSON Pointer to the field"
          example: "/name"
        message:
          type: string
          description: "Human-readable error message"
          example: "must not be blank"
        code:
          type: string
          description: "Machine-readable error code"
          example: "required"
```

### Reusable Error Responses

```yaml
components:
  responses:
    BadRequest:
      description: "Malformed request syntax"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
    Unauthorized:
      description: "Missing or invalid credentials"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
    Forbidden:
      description: "Insufficient permissions"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
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
    TooManyRequests:
      description: "Rate limit exceeded"
      headers:
        Retry-After:
          schema:
            type: integer
          description: "Seconds until rate limit resets"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetail"
```

---

## Review Dimensions

When reviewing or auditing component reuse, evaluate these dimensions:

### 1. Inline Schema Prevalence
- How many request/response bodies are defined inline vs referenced?
- Are there duplicate schema definitions across different paths?
- Do inline schemas exceed the acceptable complexity threshold?

### 2. `$ref` Hygiene
- Do all `$ref` values resolve to existing components?
- Are circular references avoided (or intentionally used for recursive types)?
- Is the `$ref` path style consistent (`#/components/schemas/` not mixed formats)?

### 3. Composition Quality
- Is `allOf` used for inheritance (not as a `$ref` workaround)?
- Do all `oneOf` unions have a `discriminator`?
- Is `allOf` depth limited to 2 levels?
- Is `anyOf` avoided in favor of `oneOf` where possible?

### 4. Component Organization
- Are schemas grouped logically by domain?
- Are shared schemas (errors, pagination) clearly separated?
- For multi-file specs, is the directory structure consistent?

### 5. Error Response Reuse
- Is `ProblemDetail` defined as a reusable schema?
- Are standard error responses (401, 403, 404, 422) defined as reusable responses?
- Do all endpoints reference the shared error responses?

---

## Anti-Patterns

### 1. Copy-Paste Schemas
The same schema structure duplicated across endpoints instead of extracted into `components`. When one copy is updated, the others drift.

### 2. `allOf` for `$ref` Workaround
Using `allOf` with a single `$ref` just to add a description -- unnecessary in 3.1.0 where `$ref` supports siblings.

### 3. Deeply Nested `allOf`
Three or more levels of `allOf` inheritance. SDK generators produce confusing class hierarchies or fail entirely.

### 4. `oneOf` Without Discriminator
Forces validators into trial-and-error matching and produces weak SDK types (typically `object` or `any`).

### 5. Circular `$ref` Without Purpose
Unintentional circular references that cause infinite recursion in generators. Circular refs should only exist for genuinely recursive data structures (trees, linked lists).

### 6. Per-Endpoint Error Schemas
Each endpoint defines its own error format instead of using `ProblemDetail`. Consumers must handle N different error shapes.

---

## Guardrails

1. **Every request/response body must use `$ref`** -- inline bodies produce unnamed SDK types
2. **Extract any schema used in 2+ places** -- DRY at the spec level prevents drift
3. **Always pair `oneOf` with `discriminator`** -- validators and generators depend on it
4. **Limit `allOf` depth to 2 levels** -- deeper nesting breaks generators
5. **Define `ProblemDetail` once, reference everywhere** -- RFC 9457 compliance through reuse
6. **Validate `$ref` resolution in CI** -- broken references produce silent generation failures
