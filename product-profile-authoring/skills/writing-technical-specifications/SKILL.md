---
name: writing-technical-specifications
version: 1.0.0
user-invocable: false
description: Author technical specification documents covering architecture overview, integrations, data flows, system limits, compliance posture, and non-functional requirements. Use when writing technical specs, documenting system architecture, cataloging integrations, defining compliance posture, or specifying system limits. Triggered by: technical specification, technical.md, architecture documentation, system limits, compliance posture, data flows, integrations, non-functional requirements.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Technical Specification Authoring

## Purpose

Author `technical.md` files that document a product's architecture, integrations, data flows, system limits, and compliance posture. This document serves the engineering and security audiences — it answers "how is it built?" and "is it safe?"

## Instructions

Follow these steps when authoring a technical specification:

1. **Read existing product context**
   - Read `product.yaml` for canonical metadata (integrations, security_compliance)
   - Read `positioning.md` for differentiated capabilities (often architecture-driven)
   - Read any existing `technical.md` to understand current state
   - Examine the actual codebase if available

2. **Document Architecture Overview**
   - Describe the high-level system architecture in 2-3 paragraphs
   - List all major components with their responsibilities and technology choices
   - Reference architecture diagrams in `assets/` if available
   - Keep descriptions factual — document what exists, not what's aspirational

3. **Document Integrations**
   - List all external systems the product connects to
   - For each integration: direction (inbound/outbound/bidirectional), protocol, and purpose
   - Include both production integrations and development/CI integrations

4. **Document Data Flows**
   - Describe the primary data paths through the system
   - Use the format: `[Source] → [Processing] → [Destination]`
   - Highlight any data transformation, enrichment, or validation steps
   - Note where PII or sensitive data flows

5. **Document System Limits**
   - Rate limits, storage limits, concurrency limits
   - Include both hard limits (enforced) and soft limits (recommended)
   - Note any limits that are configurable vs. fixed

6. **Complete the Compliance Matrix**
   - Use the framework from `./references/compliance-matrix.md`
   - For each framework: status, evidence location, last assessment date
   - Status options: Compliant, In Progress, N/A
   - Be honest about N/A — not every product needs every framework

7. **Document Non-Functional Requirements**
   - Availability targets (SLA)
   - Latency targets (p50, p99)
   - Recovery targets (RTO, RPO)
   - Include current measurements alongside targets

8. **Write the file**
   - Write to `products/<slug>/technical.md`
   - Update `product.yaml` `integrations` and `security_compliance` fields to match

## Guidelines

- **DO**: Document what exists today, not what's planned — use the roadmap for future state
- **DO**: Include specific numbers for limits and targets — "fast" is not a specification
- **DO**: Note where PII flows through the system — this is critical for compliance
- **DO**: Cross-reference `product.yaml` fields when writing integrations and compliance sections
- **DON'T**: Include implementation details like code snippets — this is architecture-level documentation
- **DON'T**: Mark a framework as "Compliant" without citing evidence (audit report, certification, assessment)
- **DON'T**: Skip the "N/A" designation — explicitly stating non-applicability shows the assessment was done

## Edge Cases

- **Early-stage product**: Many sections may be sparse. Document what's decided, note what's TBD
- **Serverless/managed architecture**: Components are services, not servers. Document the service topology
- **Multi-tenant SaaS**: Explicitly document tenant isolation boundaries in architecture and data flows
- **Open-source product**: Include self-hosted deployment architecture alongside cloud architecture

## Supporting Files

- [Compliance Matrix Reference](./references/compliance-matrix.md)
