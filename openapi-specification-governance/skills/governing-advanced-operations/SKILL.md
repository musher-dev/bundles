---
name: governing-advanced-operations
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of advanced OpenAPI operations including QUERY method via additionalOperations, webhooks vs callbacks distinction, streaming media types (text/event-stream, application/jsonl), itemSchema for streaming responses, link objects for HATEOAS-style navigation, and Overlay specification for spec patching. Use when designing webhook specifications, implementing streaming endpoints, using the QUERY method, adding link objects, or applying spec overlays. Triggered by: QUERY method, additionalOperations, webhooks, callbacks, streaming, text/event-stream, application/jsonl, itemSchema, link objects, Overlay, spec patching, SSE, server-sent events, webhook specification, advanced operations.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Advanced Operations Governance

Comprehensive governance for advanced OpenAPI operation types -- QUERY method, webhooks, callbacks, streaming, link objects, and the Overlay specification.

## Purpose

Modern APIs go beyond CRUD. They stream real-time data, accept complex queries with request bodies, push events via webhooks, and express navigable relationships between resources. OpenAPI 3.1.0 and 3.2.0 provide first-class constructs for these patterns, but misuse produces specs that confuse SDK generators and mislead consumers. This skill governs the correct application of advanced operation types.

Use this skill when designing webhook specifications, implementing streaming endpoints, adding link objects, or using the QUERY method in OpenAPI documents.

---

## QUERY Method (3.2.0+)

### The Problem QUERY Solves

Complex search operations need a request body for structured query parameters. HTTP GET semantics prohibit request bodies. The workaround -- using POST for search -- breaks safe/idempotent expectations and confuses caches.

### `additionalOperations` for QUERY

OpenAPI 3.2.0 introduces `additionalOperations` to support non-standard HTTP methods:

```yaml
paths:
  /projects:
    get:
      operationId: listProjects
      summary: "List projects with simple filters"
      parameters:
        - name: status
          in: query
          schema:
            type: string
    additionalOperations:
      query:
        operationId: searchProjects
        summary: "Search projects with complex query"
        requestBody:
          required: true
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProjectSearchQuery"
        responses:
          "200":
            content:
              application/json:
                schema:
                  $ref: "#/components/schemas/PaginatedProjectList"
```

### QUERY Method Rules

| Rule | Rationale |
|------|-----------|
| QUERY is safe and idempotent | Like GET, it reads data -- never mutates |
| Use for complex queries that exceed URL length limits | Simple filters belong in GET query parameters |
| Request body defines the query structure | Not URL-encoded parameters |
| Response is identical to the equivalent GET collection | Same envelope, pagination, schema |
| Provide a GET alternative for simple queries | QUERY supplements GET, does not replace it |

### Pre-3.2.0 Workaround

Until 3.2.0 tooling is widely adopted, use POST with clear documentation:

```yaml
paths:
  /projects:search:
    post:
      operationId: searchProjects
      summary: "Search projects (POST workaround for QUERY method)"
      description: |
        This endpoint accepts a structured search query in the request body.
        It is semantically safe and idempotent -- it reads data, never mutates.
        This POST endpoint will migrate to the QUERY method when tooling supports it.
```

---

## Webhooks vs Callbacks

### Webhooks (3.1.0+)

Webhooks are server-initiated events pushed to consumer-registered URLs. They are top-level objects in the spec, not tied to a specific API path:

```yaml
webhooks:
  projectCreated:
    post:
      operationId: onProjectCreated
      summary: "Fired when a new project is created"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ProjectEvent"
      responses:
        "200":
          description: "Webhook acknowledged"
        "410":
          description: "Endpoint no longer active -- unsubscribe"

  projectDeleted:
    post:
      operationId: onProjectDeleted
      summary: "Fired when a project is deleted"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ProjectEvent"
      responses:
        "200":
          description: "Webhook acknowledged"
```

### Callbacks

Callbacks are tied to a specific API operation and describe the API calling back to a consumer-provided URL:

```yaml
paths:
  /exports:
    post:
      operationId: createExport
      summary: "Start an async export job"
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                callback_url:
                  type: string
                  format: uri
      callbacks:
        exportComplete:
          "{$request.body#/callback_url}":
            post:
              summary: "Export job completed"
              requestBody:
                content:
                  application/json:
                    schema:
                      $ref: "#/components/schemas/ExportResult"
              responses:
                "200":
                  description: "Callback acknowledged"
```

### Webhooks vs Callbacks Decision

| Factor | Webhooks | Callbacks |
|--------|----------|-----------|
| Scope | Global, not tied to an operation | Tied to a specific operation |
| Registration | Out-of-band (settings, UI, API) | Inline (in the request body) |
| Lifecycle | Persistent subscription | Per-request |
| Use case | Event notifications (entity changes) | Async job completion |
| Spec location | Top-level `webhooks` object | Under `paths.{path}.{method}.callbacks` |

### Webhook Design Rules

| Rule | Rationale |
|------|-----------|
| Use `post` method for webhook deliveries | Webhooks push data to consumers |
| Define both 200 and error response expectations | Document what your server expects back |
| Include event type and idempotency key in payload | Consumers need to deduplicate and route |
| Document retry policy in the description | Consumers need to know delivery guarantees |
| Use consistent event payload schema | One envelope across all webhook types |

### Webhook Event Payload Pattern

```yaml
components:
  schemas:
    WebhookEvent:
      type: object
      required:
        - event_id
        - event_type
        - occurred_at
        - data
      properties:
        event_id:
          type: string
          format: uuid
          description: "Unique event identifier for idempotency"
        event_type:
          type: string
          description: "Event type identifier"
          example: "project.created"
        occurred_at:
          type: string
          format: date-time
          description: "When the event occurred (UTC, RFC 3339)"
        data:
          type: object
          description: "Event-specific payload"
```

---

## Streaming Media Types

### Server-Sent Events (SSE)

For real-time streaming of events to the client:

```yaml
paths:
  /projects/{projectId}/events:
    get:
      operationId: streamProjectEvents
      summary: "Stream real-time project events"
      responses:
        "200":
          description: "Event stream"
          content:
            text/event-stream:
              schema:
                type: string
                description: "SSE stream of ProjectEvent objects"
              x-itemSchema:
                $ref: "#/components/schemas/ProjectEvent"
```

### JSON Lines (JSONL)

For streaming large result sets line by line:

```yaml
paths:
  /exports/{exportId}/download:
    get:
      operationId: downloadExport
      summary: "Download export as streaming JSONL"
      responses:
        "200":
          description: "JSONL stream"
          content:
            application/jsonl:
              schema:
                type: string
                description: "Newline-delimited JSON stream"
              x-itemSchema:
                $ref: "#/components/schemas/ExportRow"
```

### `itemSchema` (3.2.0+)

OpenAPI 3.2.0 introduces `itemSchema` to describe the structure of individual items in a stream:

```yaml
content:
  text/event-stream:
    schema:
      type: string
    itemSchema:
      $ref: "#/components/schemas/ProjectEvent"
```

Pre-3.2.0, use `x-itemSchema` as the extension equivalent.

### Streaming Rules

| Rule | Rationale |
|------|-----------|
| Use `text/event-stream` for SSE | Standard media type for Server-Sent Events |
| Use `application/jsonl` for JSON Lines | Standard media type for newline-delimited JSON |
| Always provide `itemSchema` or `x-itemSchema` | SDK generators need to know the element type |
| Document connection lifecycle | Reconnection, heartbeats, terminal events |
| Streaming responses are always 200 | Errors after stream start use in-band error events |

---

## Link Objects

### HATEOAS-Style Navigation

Link objects express relationships between operations, enabling clients to navigate the API by following links:

```yaml
paths:
  /projects/{projectId}:
    get:
      operationId: getProject
      responses:
        "200":
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProjectResponse"
          links:
            listMembers:
              operationId: listProjectMembers
              parameters:
                projectId: "$response.body#/id"
              description: "List members of this project"
            getOrganization:
              operationId: getOrganization
              parameters:
                organizationId: "$response.body#/organization_id"
              description: "Get the owning organization"
```

### Link Object Fields

| Field | Purpose | Example |
|-------|---------|---------|
| `operationId` | Target operation | `listProjectMembers` |
| `parameters` | Map response data to target params | `projectId: "$response.body#/id"` |
| `description` | Human-readable description | `"List members of this project"` |
| `server` | Override server for the linked operation | `{url: "https://..."}` |

### Link Rules

| Rule | Rationale |
|------|-----------|
| Link `operationId` must reference an existing operation | Broken links are worse than no links |
| Use runtime expressions for parameter mapping | `$response.body#/field` links data to navigation |
| Links are documentation, not enforcement | Clients may navigate freely |
| Add links for natural navigation flows | Get resource → list sub-resources |

---

## Overlay Specification (3.2.0+)

### Purpose

Overlays allow declarative patching of OpenAPI specs without modifying the base document:

```yaml
# overlay.yaml
overlay: "1.0.0"
info:
  title: "Internal API Overlay"
actions:
  - target: "$.paths['/internal/metrics']"
    remove: true
  - target: "$.paths.*.*.x-internal"
    update:
      x-audience: "team-internal"
  - target: "$.info"
    update:
      x-environment: "staging"
```

### Overlay Use Cases

| Use Case | Overlay Action |
|----------|---------------|
| Remove internal endpoints for public spec | `remove` on internal paths |
| Add environment-specific server URLs | `update` on `servers` array |
| Add vendor-specific extensions for a doc tool | `update` with `x-` fields |
| Redact sensitive schemas for partner spec | `remove` on restricted schemas |

### Overlay Rules

| Rule | Rationale |
|------|-----------|
| Base spec must be valid without overlay | Overlay is additive/subtractive, not corrective |
| One overlay per audience/environment | `overlay-public.yaml`, `overlay-internal.yaml` |
| Never use overlays to fix spec errors | Fix the base spec instead |
| Overlays must be versioned alongside the base spec | They are part of the spec lifecycle |

---

## Review Dimensions

When reviewing or auditing advanced operations, evaluate these dimensions:

### 1. QUERY Method Usage
- Are complex search operations using QUERY (or documented POST workaround)?
- Is the QUERY operation marked as safe and idempotent?
- Is a GET alternative available for simple queries?

### 2. Webhook Completeness
- Are all event types documented as webhook objects?
- Do webhook payloads include event_id, event_type, and occurred_at?
- Is the retry policy documented?

### 3. Streaming Specification
- Do streaming endpoints use correct media types?
- Is `itemSchema` (or `x-itemSchema`) provided for stream element typing?
- Is the connection lifecycle documented?

### 4. Link Coverage
- Do resource endpoints link to related operations?
- Do all link operationIds reference existing operations?
- Are parameter mappings using runtime expressions?

### 5. Callback Correctness
- Are callbacks tied to the correct initiating operation?
- Do callback URLs use runtime expressions from the request?
- Are callback response expectations documented?

---

## Anti-Patterns

### 1. POST for Search Without Documentation
Using POST for search queries without noting the semantic intent is safe/idempotent. Consumers assume POST mutates.

### 2. Webhooks in Paths
Defining webhook deliveries as regular path items instead of top-level `webhooks` objects. Webhooks are not consumer-callable endpoints.

### 3. Streaming Without Item Schema
Streaming endpoints that define `schema: {type: string}` without `itemSchema`. Consumers cannot generate typed stream handlers.

### 4. Circular Links
Links that create navigation loops (A links to B, B links to A) without a clear entry point. Links should form a directed graph.

### 5. Overlays as Spec Fixes
Using overlays to patch schema errors instead of fixing the base spec. Overlays should only customize, not correct.

---

## Guardrails

1. **QUERY is safe and idempotent** -- document this explicitly until tooling understands the method
2. **Webhooks go in the top-level `webhooks` object** -- not in `paths`
3. **Every stream needs `itemSchema`** -- consumers need typed elements
4. **Links must reference existing operations** -- validate in CI
5. **Callbacks are per-request, webhooks are subscriptions** -- do not conflate them
6. **Overlays customize, they do not fix** -- the base spec must stand alone
