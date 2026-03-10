---
name: shaping-reviewing-api-contracts
version: 1.0.0
user-invocable: false
description: Produce an API contract review evaluating endpoint coverage, naming consistency, error responses, pagination, versioning, and authorization for a shaped architecture sketch. Use when an architecture sketch proposes API endpoints that need validation before implementation begins. Triggered by: api contract review, endpoint coverage, error handling, pagination, naming consistency, REST conventions, API design review.
allowed-tools:
  - Read
---

# API Contract Review

Expert guidance for producing API contract review artifacts during the shaping process.

## Purpose

An API contract review evaluates whether the API surface implied by an Architecture Sketch is complete, consistent, and well-designed. It catches gaps in endpoint coverage, inconsistent naming, missing error responses, and absent pagination before the implementation team builds the wrong thing.

Use this skill after an Architecture Sketch has been produced and the proposed components include API endpoints. The review compares the endpoints against the UX Breadboard flows and Domain Scenarios to verify that every user action has a corresponding API operation, and that the API design follows consistent conventions.

---

## Output Format

```markdown
# API Contract Review

**Artifact Reviewed:** <Name of the Architecture Sketch>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** API Contract Designer

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Endpoint Coverage | Pass / Partial / Fail | <Brief justification> |
| 2 | Naming Consistency | Pass / Partial / Fail | <Brief justification> |
| 3 | Error Responses | Pass / Partial / Fail | <Brief justification> |
| 4 | Pagination & Filtering | Pass / Partial / Fail | <Brief justification> |
| 5 | Versioning Strategy | Pass / Partial / Fail | <Brief justification> |
| 6 | Auth & Permissions | Pass / Partial / Fail | <Brief justification> |

---

## Endpoint Inventory

| Method | Path | Purpose | Request Body | Response | Auth Required |
|--------|------|---------|-------------|----------|---------------|
| GET | /resources | List all resources | — | Paginated list | Yes |
| POST | /resources | Create a resource | Resource payload | Created resource | Yes |
| GET | /resources/{id} | Get single resource | — | Resource detail | Yes |
...

---

## Missing Endpoints

Endpoints implied by Domain Scenarios or UX Breadboard flows but not present
in the Architecture Sketch.

| # | Implied By | Expected Endpoint | Gap Description |
|---|-----------|-------------------|-----------------|
| 1 | <Scenario or breadboard flow> | <Method + path> | <What user action has no API backing> |
...

---

## Naming Audit

Inconsistencies found in resource naming, URL patterns, or conventions.

| # | Resource/Endpoint | Issue | Suggestion |
|---|-------------------|-------|------------|
| 1 | <Path or resource name> | <What is inconsistent> | <Recommended fix> |
...

---

## Findings

### Critical

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Major

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Minor

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

---

## Recommendations

1. <Priority-ordered recommendation with rationale>
2. ...

---

## Conditions for Approval

- [ ] Every UX Breadboard flow has corresponding API endpoints that support all user actions
- [ ] Resource names use consistent pluralization, casing, and URL patterns
- [ ] Error responses are enumerated for each endpoint with appropriate status codes
- [ ] List endpoints include pagination, and filtering/sorting where needed
- [ ] Authorization requirements are specified per endpoint
- [ ] No Critical findings remain unresolved

---

## Verdict

**<Approved / Approved with Conditions / Rejected>**

<Summary rationale — 2-3 sentences.>
```

---

## Instructions

1. **Build the endpoint inventory.** Read the Architecture Sketch and extract every API endpoint mentioned or implied. For each, record the HTTP method, path, purpose, request body shape, response shape, and authentication requirement. If the sketch does not specify all of these, note what is missing.

2. **Cross-reference against UX Breadboard and Domain Scenarios.** For every affordance in the UX Breadboard that triggers a server action, verify that a corresponding endpoint exists. For every domain scenario step that reads or writes data, verify API coverage. Record any gaps in the Missing Endpoints table.

3. **Audit naming conventions.** Check that all resource names use the same pluralization convention (all plural or all singular). Verify URL patterns are consistent (nested resources vs flat with query params). Check that path parameter names match the resource they reference. Record inconsistencies in the Naming Audit table.

4. **Evaluate error handling, pagination, and auth.** For each endpoint, verify: (a) error responses are defined for common failure cases (validation errors, not found, conflict, unauthorized); (b) list endpoints include pagination parameters; (c) authorization requirements are explicit. Rate each dimension and record findings.

5. **Render the verdict.** Check all Conditions for Approval. If all pass, the verdict is "Approved." If minor issues remain, "Approved with Conditions." If critical gaps exist (missing endpoints for core flows, no error handling), "Rejected."

---

## Quality Checklist

- [ ] The endpoint inventory includes every API endpoint mentioned or implied in the Architecture Sketch
- [ ] Missing endpoints are traced back to specific UX Breadboard flows or Domain Scenarios
- [ ] The naming audit checks pluralization, casing, URL structure, and parameter naming
- [ ] Error responses are evaluated per endpoint, not just globally
- [ ] Pagination is evaluated for every list endpoint
- [ ] Authorization requirements are specified per endpoint, not assumed
- [ ] The verdict is consistent with the Conditions for Approval checklist

---

## Anti-Patterns

### 1. Reviewing Only Explicit Endpoints
Checking only the endpoints the Architecture Sketch names while ignoring endpoints implied by UX flows. If the breadboard shows a "delete" affordance but the sketch does not mention a DELETE endpoint, that is a coverage gap.

### 2. Assuming Consistent Error Handling
Marking error responses as "Pass" because "the framework handles errors." The review evaluates whether error cases are identified and documented, not whether a framework exists. Each endpoint has different failure modes.

### 3. Ignoring Pagination Until Implementation
Treating pagination as an implementation detail. If the UX shows a list that could have thousands of items and the API returns all items in one response, that is a design failure discovered too late.

### 4. Naming Review Without Convention Reference
Noting naming issues without referencing a consistent convention. The review should either reference an existing API style guide or propose one. "This name is wrong" is not useful without "this name should be X because the convention is Y."

### 5. Auth Requirements as an Afterthought
Listing "Auth: Yes" for all endpoints without distinguishing between different permission levels. A list endpoint and a delete endpoint may both require auth but at different permission levels.

---

## Examples

### Example 1: Webhook Management API Review

An Architecture Sketch proposes endpoints for CRUD operations on webhook configurations. The UX Breadboard shows flows for creating, testing, and viewing delivery logs.

**Endpoint Coverage** is rated **Partial** because the sketch includes endpoints for creating and listing webhooks but does not include an endpoint for testing a webhook (sending a sample payload). The UX Breadboard shows a "Send test" affordance in the WEBHOOK_DETAIL place. Missing endpoint: `POST /webhooks/{id}/test`.

**Naming Consistency** is rated **Fail** because the sketch uses `/webhooks` for the resource but `/webhook-logs/{id}` for delivery logs (singular vs plural inconsistency), and the delivery log path uses the webhook ID as a path parameter instead of nesting: `/webhooks/{id}/deliveries` would be more consistent.

### Example 2: User Profile API Review

An Architecture Sketch proposes endpoints for viewing and editing user profiles. The Domain Scenarios include uploading a profile photo and changing email (which requires verification).

**Error Responses** is rated **Partial** because the email change endpoint does not define what happens when the new email is already taken (409 Conflict expected) or when the verification token expires. The photo upload endpoint does not specify size or format validation errors.

**Auth & Permissions** is rated **Partial** because all endpoints are marked "Auth: Yes" but the sketch does not distinguish between a user editing their own profile (self-service) and an admin editing another user's profile (admin permission). These require different authorization checks.