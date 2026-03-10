---
name: shaping-writing-architecture-sketches
version: 1.0.0
user-invocable: false
description: Produce architecture sketch documents that define the solution boundary, component inventory, data shape, integration points, data flow, and hard parts for a shaped project. Use when the technical approach needs to be sketched before implementation begins. Triggered by: architecture sketch, system design, components, boundaries, data shape, integration points, technical shaping, hard parts, solution boundary.
allowed-tools:
  - Read
---

# Architecture Sketches — Components, Boundaries, Data Shape & Hard Parts

Expert guidance for producing architecture sketch artifacts during the shaping process.

## Purpose

An architecture sketch defines the technical shape of a solution at the right level of abstraction for shaping — detailed enough to identify hard parts and integration risks, abstract enough to leave implementation decisions to the building team. It is not a system design document or a technical specification; it is a sketch that answers "what pieces are involved and where is the complexity?"

Use this skill after the UX Breadboard and Domain Scenarios are complete. The architecture sketch translates user-facing flows into technical components, identifies what already exists versus what must be built, and calls out the technically challenging aspects that could derail the project.

---

## Output Format

```markdown
# Architecture Sketch: <Feature or Project Name>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Problem Frame:** <Reference to upstream Problem Frame>
**UX Breadboard:** <Reference to upstream UX Breadboard>

---

## 1. Solution Boundary

<One paragraph defining what is inside and outside the technical scope. State
explicitly what this solution WILL and WILL NOT touch. Reference the Problem
Frame appetite and non-goals to anchor the boundary.>

---

## 2. Component Inventory

| Component | Type | Responsibility | Owner |
|-----------|------|---------------|-------|
| <Component name> | Existing / New / Modified | <What this component does in the context of the solution> | <Team or domain that owns it> |
| <Component name> | <Type> | <Responsibility> | <Owner> |
...

---

## 3. Data Shape

### <Entity Name>

| Field | Type | Notes |
|-------|------|-------|
| <field_name> | <type> | <constraints, relationships, or behavioral notes> |
...

**Relationships:** <Describe how this entity relates to others>

### <Entity Name>

[Repeat for each significant entity. This is NOT a database schema — it is
just enough shape to reason about data flow and storage.]

---

## 4. Integration Points

| System | Direction | Protocol | Data Exchanged | Failure Mode |
|--------|-----------|----------|---------------|--------------|
| <External or internal system> | Inbound / Outbound / Bidirectional | <REST, webhook, queue, etc.> | <What data flows through this point> | <What happens when this integration fails> |
...

---

## 5. Data Flow

Primary scenario data flow, showing how data moves through components.

```
User -> [Component A] -> [Component B] -> [Store]
                      -> [External API]
```

<Prose description of the flow: what triggers it, what each component does
with the data, where the data lands, and what the user sees at the end.>

---

## 6. Hard Parts

Technically challenging aspects of this solution. Each hard part explains WHY
it is hard and sketches possible approaches.

1. **<Hard Part Name>**
   _Why it is hard:_ <Explanation of the technical challenge>
   _Possible approaches:_ <1-2 approaches with trade-offs>

2. **<Hard Part Name>**
   _Why it is hard:_ <Explanation>
   _Possible approaches:_ <Approaches>

...

---

## 7. What We're NOT Designing

Architectural decisions explicitly deferred. These are NOT non-goals (which
are product scope decisions) — these are technical decisions left to the
implementation team.

1. <Deferred decision> — <Why it is deferred and who should decide during build>
2. <Deferred decision> — <Reason>
...
```

---

## Instructions

1. **Read the Problem Frame and UX Breadboard for scope and user flows.** The Problem Frame defines the boundaries (appetite, non-goals, constraints). The UX Breadboard defines the places, affordances, and transitions that the architecture must support. Every place in the breadboard that loads data or performs an action implies at least one component interaction.

2. **Identify components needed — classify as existing, new, or modified.** Walk through each UX flow and identify what technical components are involved at each step. For each component, determine whether it already exists in the system, needs to be built from scratch, or needs modification. Be honest about the "Modified" category — modifying an existing component is often harder than building a new one because of existing consumers and backward compatibility.

3. **Sketch data shapes — just enough to reason about, not full schemas.** For each significant entity, list its key fields, types, and relationships. The goal is not a database design — it is enough shape to answer: "can the data support the UX flows?" and "where does complexity live in the data model?" If two entities have a relationship, describe it. If a field has important constraints, note them.

4. **Map integration points and their failure modes.** Every external system, API, queue, or service that the solution touches is an integration point. For each, document the direction of data flow, the protocol, what data is exchanged, and what happens when the integration fails. Failure modes are critical — they drive error states in the UX Breadboard and risk entries in the Risk Register.

5. **Call out hard parts honestly.** Hard parts are aspects of the solution where the team could spend unbounded time, where the approach is uncertain, or where failure is non-obvious. For each hard part, explain why it is hard (not just that it is hard) and sketch 1-2 possible approaches with their trade-offs. Hard parts are the most important section of the sketch — they drive spike work, risk assessment, and appetite decisions.

---

## Quality Checklist

- [ ] The Solution Boundary paragraph explicitly states what is in scope and out of scope technically, referencing the Problem Frame
- [ ] Every component in the Component Inventory is classified as Existing, New, or Modified with a clear responsibility
- [ ] Data shapes include enough fields and relationships to support all UX Breadboard flows without being a full database schema
- [ ] Every integration point includes a failure mode — what happens when the external system is unavailable or returns errors
- [ ] The data flow section traces the primary scenario from user action through components to final state
- [ ] Hard parts explain WHY something is hard, not just THAT it is hard, with possible approaches
- [ ] What We're NOT Designing lists deferred technical decisions with rationale for deferral

---

## Anti-Patterns

### 1. Premature Detail
Writing a full database schema, API specification, or class hierarchy. The architecture sketch is a sketch — it defines shape and boundaries, not implementation. If you are specifying column types, HTTP status codes, or method signatures, you have gone too deep. Leave those decisions to the building team.

### 2. Missing Failure Modes
Listing integration points without describing what happens when they fail. Every external dependency is a potential failure point. "Calls the payment API" is incomplete without "if the payment API is down, the order is saved in pending state and retried via background job." Failure modes drive error states and risk assessment.

### 3. Optimistic Component Classification
Marking components as "Existing" without verifying they actually support the needed functionality. An existing user service might exist but not support the new query pattern the solution needs. The sketch should note what the existing component provides and what additional capability is needed.

### 4. Hard Parts Omitted for Confidence
Skipping the Hard Parts section because "we know how to do this" or because it feels like admitting weakness. Every non-trivial project has hard parts. If the sketch has no hard parts, either the solution is trivial (in which case it does not need shaping) or the author has not thought deeply enough about implementation complexity.

### 5. Architecture Without Traceability
Drawing components and data flows that do not map back to UX Breadboard flows. Every component should exist because a user flow requires it. Every data shape should exist because a place in the breadboard displays or modifies that data. Untraceable components are likely over-engineering.

---

## Examples

### Example 1: Webhook Delivery System

**Context:** The UX Breadboard shows a flow where users configure webhook endpoints, the system sends events to those endpoints, and users view delivery logs. The Problem Frame scopes this to HTTP webhooks with retry logic.

**How the artifact looks in practice (abbreviated):**

```markdown
# Architecture Sketch: Webhook Delivery System

## 1. Solution Boundary

This solution covers webhook endpoint configuration (CRUD), event delivery with
retry, and delivery log viewing. It does NOT cover webhook signature verification
on the receiver side, custom payload transformations, or real-time delivery
streaming. The delivery mechanism is fire-and-retry with exponential backoff,
not guaranteed exactly-once delivery.

## 2. Component Inventory

| Component | Type | Responsibility | Owner |
|-----------|------|---------------|-------|
| Webhook endpoint API | New | CRUD for endpoint configs (URL, secret, events) | Platform team |
| Event dispatcher | Modified | Route domain events to webhook delivery queue | Platform team |
| Delivery worker | New | HTTP POST to configured URLs with retry logic | Platform team |
| Delivery log store | New | Record every delivery attempt with status and response | Platform team |
| Delivery log API | New | Read-only access to delivery history per endpoint | Platform team |

## 6. Hard Parts

1. **Retry timing under load**
   _Why it is hard:_ If 100 endpoints all fail simultaneously (e.g., a popular
   receiver goes down), the retry queue floods. Exponential backoff per endpoint
   is straightforward, but the aggregate retry volume could overwhelm the worker
   pool.
   _Possible approaches:_ (a) Circuit breaker per destination host — stop retrying
   all endpoints targeting a downed host after N consecutive failures. (b) Global
   rate limiter on the retry queue with priority based on attempt count.
```

### Example 2: Bulk CSV Import

**Context:** The UX Breadboard shows upload, validation preview, error correction, and import confirmation flows. The Problem Frame scopes this to CSV only, with a maximum file size.

**How the artifact looks in practice (abbreviated):**

```markdown
# Architecture Sketch: Bulk CSV Import

## 3. Data Shape

### ImportJob
| Field | Type | Notes |
|-------|------|-------|
| id | UUID | Primary identifier |
| status | enum | pending, validating, preview, importing, complete, failed |
| file_reference | string | Path to uploaded file in object storage |
| row_count | integer | Total rows in the CSV |
| valid_count | integer | Rows that passed validation |
| error_count | integer | Rows with validation errors |
| created_by | UUID | FK to user who initiated the import |

### ImportRow
| Field | Type | Notes |
|-------|------|-------|
| job_id | UUID | FK to ImportJob |
| row_number | integer | Position in the original CSV |
| data | JSON | Parsed row data as key-value pairs |
| errors | JSON | Array of validation errors for this row, empty if valid |
| status | enum | valid, invalid, imported, skipped |

**Relationships:** ImportJob has many ImportRows. An ImportJob belongs to a User.

## 6. Hard Parts

1. **Large file validation latency**
   _Why it is hard:_ A 10,000-row CSV takes time to parse and validate. The user
   cannot stare at a spinner for 30 seconds. But we need all rows validated before
   showing the preview.
   _Possible approaches:_ (a) Background validation with progress polling — user
   uploads, sees a progress bar, preview loads when ready. (b) Streaming validation
   that shows results as they come in — more complex but better UX for very large files.
```