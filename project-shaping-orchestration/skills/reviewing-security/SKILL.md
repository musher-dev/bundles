---
name: shaping-reviewing-security
version: 1.0.0
user-invocable: false
description: Produce a security review evaluating threat model, authentication, authorization, PII handling, audit logging, data protection, and third-party risk for a shaped architecture sketch. Use when an architecture sketch and API contract need security assessment before implementation. Triggered by: security review, threat model, authentication, authorization, PII handling, audit logging, data protection, security assessment, OWASP.
allowed-tools:
  - Read
---

# Security Review

Expert guidance for producing security review artifacts during the shaping process.

## Purpose

A security review evaluates whether a proposed architecture adequately addresses authentication, authorization, data protection, input validation, audit logging, and third-party integration risks before implementation begins. Catching security gaps during design is orders of magnitude cheaper than remediating vulnerabilities in production.

Use this skill after both an Architecture Sketch and an API Contract Review have been produced. The security review builds on those artifacts to identify threats, assess risk posture, and recommend mitigations.

---

## Output Format

```markdown
# Security Review

**Artifact Reviewed:** <Architecture Sketch and API Contract Review names>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** Security Reviewer

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Authentication | Pass / Partial / Fail | <Brief justification> |
| 2 | Authorization | Pass / Partial / Fail | <Brief justification> |
| 3 | Data Protection | Pass / Partial / Fail | <Brief justification> |
| 4 | Input Validation | Pass / Partial / Fail | <Brief justification> |
| 5 | Audit Logging | Pass / Partial / Fail | <Brief justification> |
| 6 | Third-Party Risk | Pass / Partial / Fail | <Brief justification> |

---

## Threat Model

| # | Threat | Attack Vector | Likelihood | Impact | Mitigation |
|---|--------|--------------|-----------|--------|------------|
| T1 | <Threat description> | <How an attacker exploits this> | Low / Medium / High | Low / Medium / High | <Specific countermeasure> |
...

---

## PII Inventory

| Data Element | Classification | Storage | Access Control | Retention |
|-------------|---------------|---------|---------------|-----------|
| <Field name> | Public / Internal / Confidential / PII | Encrypted / Plaintext / Hashed | <Role/permission required> | <Duration/policy> |
...

---

## Security Controls Checklist

- [ ] HTTPS enforced on all endpoints (no plaintext HTTP)
- [ ] CORS policy restricts allowed origins to known domains
- [ ] Rate limiting applied to authentication endpoints
- [ ] Rate limiting applied to public-facing endpoints
- [ ] CSRF protection on state-changing requests (if cookie-based auth)
- [ ] Security headers configured (CSP, X-Frame-Options, etc.)
- [ ] Secrets stored in environment variables or secret manager, never in code
- [ ] Dependency vulnerability scanning configured
- [ ] Session/token expiration and refresh policies defined
- [ ] Error responses do not leak internal details (stack traces, SQL errors)
- [ ] File upload validation (type, size, content) if applicable
- [ ] Database queries use parameterized statements (no string concatenation)

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

- [ ] Authentication mechanism specified with token lifecycle (issuance, expiration, refresh, revocation)
- [ ] Authorization model defined with per-endpoint access control
- [ ] All PII data elements inventoried with classification, encryption, and retention
- [ ] Input validation strategy documented for all external inputs
- [ ] Audit logging covers all security-relevant actions
- [ ] Third-party integrations have documented trust boundaries
- [ ] No Critical findings remain unresolved

---

## Verdict

**<Approved / Approved with Conditions / Rejected>**

<Summary rationale — 2-3 sentences.>
```

---

## Instructions

1. **Identify the attack surface.** Read the Architecture Sketch and API Contract Review to enumerate every point where external input enters the system: API endpoints, webhook receivers, file uploads, OAuth callbacks, scheduled jobs processing external data, third-party integrations. For each entry point, note what data it accepts, who can access it, and what trust level is assumed. Build the Threat Model.

2. **Evaluate each dimension.**
   - **Authentication:** Check token format and lifecycle, credential storage, brute-force protection, machine-to-machine auth.
   - **Authorization:** Check for defined model (RBAC/ABAC), per-endpoint access control, privilege escalation risks (horizontal and vertical).
   - **Data Protection:** Inventory PII. Check encryption at rest and in transit, hashing of secrets, data minimization, secure deletion.
   - **Input Validation:** Check type/length/format validation, injection risks (SQL, NoSQL, command, SSRF, path traversal). Verify validation at API boundary.
   - **Audit Logging:** Verify logging of auth events, access decisions, data access, admin actions. Verify logs exclude sensitive data.
   - **Third-Party Risk:** Identify external dependencies. Assess trust boundaries, failure modes, data flow direction, least-privilege scoping.

3. **Classify findings by severity.**
   - **Critical:** Missing auth on write endpoints, no authorization model, unencrypted PII storage, injection vulnerabilities, hardcoded credentials.
   - **Major:** Incomplete token lifecycle, partial authorization coverage, missing audit logging for some actions, overly-broad third-party permissions.
   - **Minor:** Missing security headers, unspecified rate limiting for non-critical endpoints, undefined audit log retention.

4. **Compile the Security Controls Checklist.** Mark each item checked or unchecked based on the design. For unchecked items, add corresponding findings.

5. **Render the verdict.** All conditions met, no Critical findings = "Approved." Conditions met with issues = "Approved with Conditions." Critical findings unresolved = "Rejected."

---

## Quality Checklist

- [ ] Every external input entry point is identified and assessed for threats
- [ ] The Threat Model includes at least one threat per major entry point with likelihood, impact, and mitigation
- [ ] Each dimension has a rating with written justification
- [ ] The PII Inventory captures every personal or sensitive data element with protection details
- [ ] The Security Controls Checklist is fully evaluated
- [ ] Findings describe security consequences in concrete terms
- [ ] The verdict is consistent with the Conditions for Approval checklist

---

## Anti-Patterns

### 1. Checklist-Only Review
Walking through the Security Controls Checklist without building a threat model. Checklists catch known categories but miss design-specific threats. The threat model is the core; the checklist is supplementary.

### 2. Assuming the Framework Handles It
Marking auth or validation as "Pass" because the implementation will use a security framework. The review evaluates the design, not the implementation plan.

### 3. Reviewing Authentication Without Authorization
Verifying users must log in without examining what they can do after. A system where every authenticated user can access everything has auth but no authorization.

### 4. Ignoring Data at Rest
Focusing on API-level security without examining storage. HTTPS is necessary but insufficient. PII must be protected in the database, logs, backups, and intermediate storage.

### 5. Treating Third-Party Services as Trusted
Assuming data sent to external services is safe. Third-party integrations expand the attack surface. The review must assess compromise scenarios and least-privilege scoping.

---

## Examples

### Example 1: Multi-Tenant API with Webhooks

**Authorization** is rated **Partial** because RBAC is defined for webhook management (admin-only) but tenant isolation is not addressed in the data layer. `GET /webhooks/{id}` does not specify tenant-scoped authorization — a user could potentially access another organization's webhooks by ID guessing.

**Third-Party Risk** is rated **Partial** because webhook payloads are sent to customer-configured URLs without SSRF protection. An attacker could configure a webhook URL pointing to internal infrastructure (cloud metadata endpoints, internal services).

### Example 2: User Profile Service with PII

**Data Protection** is rated **Fail** because the design does not classify data elements by sensitivity. Email addresses and phone numbers are PII requiring protection, but the design treats them identically to display names.

**Audit Logging** is rated **Partial** because the design logs API requests but not data access events. Admin profile views should be logged for compliance. Request body logging without sanitization would expose PII in plaintext in logs.