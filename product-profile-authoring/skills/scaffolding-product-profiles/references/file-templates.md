# Markdown File Templates

Templates for the four markdown files in a product profile directory. Each template provides the framework-specific structure with placeholder sections.

---

## overview.md — PR/FAQ Format (Working Backwards)

```markdown
# [Product Name] — Overview

## Press Release

### Heading
[Product name and category in < 10 words]

### Subheading
[Who is the target customer and what benefit do they get? One sentence.]

### Summary Paragraph
[2-3 sentences: what is the product, what does it do, why does it matter?]

### Problem Paragraph
[Describe the current pain point. What does the world look like without this product?]

### Solution Paragraph
[How does the product solve the problem? Focus on the mechanism, not features.]

### Stakeholder Quote
> "[Internal quote from a leader explaining why this product was built and why it matters now.]"
>
> — [Name], [Title]

### Customer Experience
[Walk through a typical first-use experience. What does the customer do? What happens?]

### Customer Quote
> "[Quote from a real or representative customer describing the outcome they achieved.]"
>
> — [Name], [Title], [Company]

### Call to Action
[One clear next step: try it, sign up, read docs, talk to sales.]

## External FAQ

1. **[Question a customer would ask]**
   [Answer]

2. **[Question about pricing or availability]**
   [Answer]

3. **[Question about how it compares to alternatives]**
   [Answer]

## Internal FAQ

1. **[Question about feasibility or timeline]**
   [Answer]

2. **[Question about resource requirements]**
   [Answer]

3. **[Question about risks or dependencies]**
   [Answer]
```

---

## positioning.md — Dunford 5-Component Framework

```markdown
# [Product Name] — Positioning

## 1. Competitive Alternatives

What would customers use if this product didn't exist?

| Alternative | Type | Limitation |
|---|---|---|
| [Alternative 1] | [Direct / Indirect / Status quo] | [Key limitation] |
| [Alternative 2] | [Direct / Indirect / Status quo] | [Key limitation] |
| [Alternative 3] | [Direct / Indirect / Status quo] | [Key limitation] |

## 2. Differentiated Capabilities

What can this product do that alternatives cannot?

| Capability | Why Alternatives Can't Match | Proof |
|---|---|---|
| [Capability 1] | [Reason] | [Evidence] |
| [Capability 2] | [Reason] | [Evidence] |
| [Capability 3] | [Reason] | [Evidence] |

## 3. Customer Value

What value do the differentiated capabilities deliver?

| Capability | Value Delivered | Measurable Outcome |
|---|---|---|
| [Capability 1] | [Value] | [Metric or result] |
| [Capability 2] | [Value] | [Metric or result] |
| [Capability 3] | [Value] | [Metric or result] |

## 4. Target Customer Segments

Who cares most about the value this product delivers?

### Primary Segment
- **Role:** [Job title / function]
- **Company:** [Size, stage, industry]
- **Trigger:** [What event makes them search for a solution?]
- **Must-have:** [Non-negotiable requirement]

### Secondary Segment (if applicable)
- **Role:** [Job title / function]
- **Company:** [Size, stage, industry]
- **Trigger:** [What event makes them search for a solution?]
- **Must-have:** [Non-negotiable requirement]

## 5. Market Category

What category does this product belong to, and how should it be framed?

- **Category:** [Existing, searchable market category]
- **Modifier:** [What makes this product's approach distinct within the category]
- **Positioning statement:** [Product] is a [category] that helps [audience] achieve [outcome] by [mechanism].
```

---

## roadmap.md — Now/Next/Later Format

```markdown
# [Product Name] — Roadmap

*Last updated: [date]*

## Now — Active & Committed

Items actively in progress with committed resources.

| Initiative | Outcome | Owner | Target |
|---|---|---|---|
| [Initiative 1] | [Expected outcome] | [Team/person] | [Date or sprint] |
| [Initiative 2] | [Expected outcome] | [Team/person] | [Date or sprint] |

## Next — Prioritized & Flexible

Prioritized items that will start when current work completes. Order may shift.

| Initiative | Outcome | Dependencies |
|---|---|---|
| [Initiative 1] | [Expected outcome] | [Blocker or prerequisite] |
| [Initiative 2] | [Expected outcome] | [Blocker or prerequisite] |

## Later — Aspirational & Uncommitted

Directional ideas under consideration. No commitment to timeline or delivery.

| Initiative | Outcome | Open Questions |
|---|---|---|
| [Initiative 1] | [Expected outcome] | [What we need to learn first] |
| [Initiative 2] | [Expected outcome] | [What we need to learn first] |

## Dependencies

- [External dependency and its impact on roadmap items]

## Assumptions

- [Key assumption that roadmap items depend on]

## Open Questions

- [Unresolved question that could change priorities]
```

---

## technical.md — Technical Specification Format

```markdown
# [Product Name] — Technical Specification

## Architecture Overview

[High-level description of the system architecture. Include a diagram reference if available.]

### Components

| Component | Responsibility | Technology |
|---|---|---|
| [Component 1] | [What it does] | [Stack/language/framework] |
| [Component 2] | [What it does] | [Stack/language/framework] |

## Integrations

| System | Direction | Protocol | Purpose |
|---|---|---|---|
| [System 1] | [Inbound / Outbound / Bidirectional] | [REST / gRPC / Webhook / SDK] | [What data flows] |
| [System 2] | [Inbound / Outbound / Bidirectional] | [REST / gRPC / Webhook / SDK] | [What data flows] |

## Data Flows

[Describe primary data flows through the system. Include sequence or flow descriptions.]

1. **[Flow name]**: [Source] → [Processing] → [Destination]
2. **[Flow name]**: [Source] → [Processing] → [Destination]

## System Limits

| Dimension | Limit | Notes |
|---|---|---|
| [Rate limits] | [Value] | [Context] |
| [Storage limits] | [Value] | [Context] |
| [Concurrency] | [Value] | [Context] |

## Compliance Posture

| Framework | Status | Evidence | Last Assessed |
|---|---|---|---|
| GDPR | [Compliant / In Progress / N/A] | [Link or description] | [Date] |
| SOC 2 Type II | [Compliant / In Progress / N/A] | [Link or description] | [Date] |
| ISO 27001 | [Compliant / In Progress / N/A] | [Link or description] | [Date] |
| PCI DSS | [Compliant / In Progress / N/A] | [Link or description] | [Date] |
| HIPAA | [Compliant / In Progress / N/A] | [Link or description] | [Date] |
| EU AI Act | [Compliant / In Progress / N/A] | [Link or description] | [Date] |

## Non-Functional Requirements

| Attribute | Target | Current | Measurement |
|---|---|---|---|
| Availability | [SLA target] | [Current metric] | [How measured] |
| Latency (p99) | [Target] | [Current] | [How measured] |
| Recovery (RTO/RPO) | [Target] | [Current] | [How measured] |
```
