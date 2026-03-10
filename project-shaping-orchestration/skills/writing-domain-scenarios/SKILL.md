---
name: shaping-writing-domain-scenarios
version: 1.0.0
user-invocable: false
description: Write domain scenario documents that define personas with mental models, domain vocabulary, core workflows, edge cases, exception paths, and open questions. Use when modeling user behavior, mapping workflows, identifying edge cases, building domain vocabulary, or preparing scenarios for downstream UX and technical shaping. Triggered by: domain scenarios, personas, workflows, edge cases, domain vocabulary, user scenarios, exception paths, domain modeling.
allowed-tools:
  - Read
---

# Domain Scenarios — Personas, Workflows, Edge Cases & Terminology

Expert guidance for producing domain scenario artifacts during the shaping process.

## Purpose

Domain scenarios translate the problem frame into concrete human behavior: who does what, in what order, under what conditions, and what can go wrong. They bridge the gap between "we understand the problem" and "we can design a solution" by providing the detailed behavioral context that UX breadboards and architecture sketches need as input.

The domain scenarios artifact exists because abstract problem descriptions do not reveal edge cases, and edge cases are where most implementation complexity hides. By walking through scenarios step by step — including the uncomfortable "what if?" questions — the scenarios surface complexity before the team commits to an architecture or a time budget.

Write domain scenarios after the Context Brief and Problem Frame are complete and before UX breadboarding or architecture sketching begins.

---

## Output Format

```markdown
# Domain Scenarios: <Feature or Problem Area>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Context Brief:** <Reference to the upstream Context Brief>
**Problem Frame:** <Reference to the upstream Problem Frame>

---

## 1. Personas

### <Persona Name> — <Role>

- **Role:** <Job title or function>
- **Mental Model:** <How this person thinks about the domain — what categories they use, what they consider normal vs exceptional, what metaphors guide their expectations>
- **Goals:**
  - <Specific, concrete goal>
  - <Goal>
- **Frustrations:**
  - <Specific, concrete frustration with current state>
  - <Frustration>

### <Persona Name> — <Role>

[Repeat structure for each persona. Include 2-4 personas.]

---

## 2. Domain Vocabulary

| Term | Definition | Synonyms to Avoid | Example Usage |
|------|-----------|-------------------|---------------|
| <Term> | <Precise definition in context of this domain> | <Terms that sound similar but mean different things> | <One sentence showing the term used correctly> |
...

---

## 3. Core Scenarios

### Scenario: <Descriptive Title>

**Persona:** <Which persona performs this scenario>
**Preconditions:**
- <What must be true before this scenario starts>
- <Precondition>

**Steps:**

1. <Concrete action or system response — not "user interacts" but "user clicks Upload">
2. <Step>
3. <Step>
...

**Postconditions:**
- <What is true after successful completion>
- <Postcondition>

**Notes:** <Frequency, importance, or other context about this scenario>

[Repeat for each core scenario. Include 3-6 scenarios covering the primary workflows.]

---

## 4. Edge Cases

| # | Scenario | Trigger Condition | Expected Behavior | Priority |
|---|----------|-------------------|-------------------|----------|
| 1 | <Which core scenario this relates to> | <Specific condition that triggers the edge case> | <What the system should do> | Must Handle / Should Handle / Nice to Handle |
| 2 | ... | ... | ... | ... |
...

---

## 5. Exception Paths

### Exception: <What Goes Wrong>

- **What Goes Wrong:** <Description of the failure>
- **Detection:** <How the failure is detected — by the system, by the user, or both>
- **Recovery:** <What happens to recover — retry, rollback, manual intervention, or graceful degradation>
- **User Communication:** <What the user is told, in specific enough terms to inform UX copy>

[Repeat for each significant exception path. Include 3-5 exception paths.]

---

## 6. Out-of-Scope Scenarios

1. **<Scenario description>** — _Rationale: <Why this is excluded, referencing the Problem Frame appetite or non-goals>_
2. ...

---

## 7. Open Questions

1. **<Question requiring stakeholder input>**
   _Context: <Why this question matters and what depends on the answer>_
   _Suggested respondent: <Who can answer this>_
2. ...
```

---

## Instructions

1. **Read the Context Brief and Problem Frame for foundational understanding.** Absorb the verified facts, glossary, assumptions, trigger scene, success criteria, appetite, and non-goals. The domain scenarios must be consistent with these upstream artifacts — they cannot introduce new facts or contradict the problem frame's boundaries. Note any assumptions particularly relevant to user workflows, as these may need to be tested through scenarios.

2. **Define personas with mental models.** Go beyond job titles. For each persona, articulate how they think about the domain — what categories they use, what they consider normal versus exceptional, what metaphors guide their expectations. A "data analyst" who thinks of imports as "uploading a spreadsheet" has a fundamentally different mental model from one who thinks of imports as "synchronizing two databases." These mental models determine what the persona expects at each step and where surprises cause friction.

3. **Walk through core scenarios step by step, noting pre/postconditions.** Start with the most common, happy-path workflow. Write each step as a concrete action or system response — not "the user interacts with the form" but "the user selects a CSV file from the file picker and clicks Upload." For each step, ask: what input does this require? What output does it produce? What decisions does the persona make? Identify preconditions (what must be true before the scenario starts) and postconditions (what is true after it completes successfully).

4. **Probe edge cases by asking "what if?" for each scenario step.** For every step in every core scenario, ask: what if this input is missing? What if it is malformed? What if the step takes longer than expected? What if it fails partway through? What if the user abandons the workflow and comes back later? What if two users execute this scenario simultaneously? Not every "what if" produces an edge case worth documenting — include only those that are realistic and whose mishandling would cause real harm.

5. **Document exception paths and identify remaining open questions.** For each significant failure mode discovered during edge-case probing, describe the full exception path: what fails, how the failure is detected, how recovery works, and what the user is told. Then review the entire document and list any domain questions that could not be answered from the Context Brief and Problem Frame alone. These open questions represent the boundary of what scenario modeling can determine without additional domain expertise.

---

## Quality Checklist

- [ ] Every persona has a mental model description that goes beyond their job title — it explains how they think about the domain
- [ ] The Domain Vocabulary table covers every term used in the scenarios that has a domain-specific meaning, with synonyms to avoid
- [ ] Every core scenario has explicit preconditions, numbered steps, postconditions, and notes
- [ ] Edge cases reference specific core scenarios and have precise trigger conditions, not vague "something unusual happens"
- [ ] Every exception path includes all four components: what goes wrong, detection, recovery, and user communication
- [ ] Out-of-scope scenarios include a rationale that references the problem frame's appetite or non-goals
- [ ] Open questions include enough context for a stakeholder to answer without reading the full document

---

## Anti-Patterns

### 1. Personas as Job Titles
Listing "Data Analyst" with goals like "analyze data" and frustrations like "tools are slow" provides no insight. A useful persona has a mental model ("thinks of the system as a spreadsheet she uploads to"), specific goals ("needs to reconcile 15,000 shipment records every Monday before the 10 AM standup"), and concrete frustrations ("the import fails silently and she cannot tell which rows were processed").

### 2. Scenario Steps That Skip the Hard Parts
Writing "1. User uploads file, 2. System processes file, 3. User sees results" glosses over the exact points where problems occur. Good scenario steps are granular enough to expose failure modes: what format is the file? How does the user select it? What feedback does the system give during processing?

### 3. Edge Cases Without Priority
Listing 30 edge cases with no priority ranking overwhelms downstream consumers. A "Must Handle" edge case (silent data corruption) demands immediate design attention. A "Nice to Handle" edge case (unusual Unicode characters in a rarely-used field) can be deferred. Without priority, every edge case looks equally urgent.

### 4. Exception Paths That Stop at Detection
"The system detects the error and shows an error message" is not a complete exception path. Recovery is the hard part: does the user retry? Does the system roll back? Is partial progress preserved? Exception paths that stop at detection leave the most important design questions unanswered.

### 5. Domain Vocabulary Defined Too Late
If core scenarios use domain-specific terms that are not in the vocabulary table, readers will interpret them based on their own assumptions. The vocabulary table should be built before or alongside the core scenarios, not as an afterthought.

---

## Examples

### Example 1: Invoice Approval Workflow

A Context Brief documented that finance teams process invoices through a multi-step approval chain. The Problem Frame identified that approvers cannot see the invoice context when they receive an approval request.

**How the artifact looks in practice (abbreviated):**

```markdown
# Domain Scenarios: Invoice Approval Workflow

**Prepared by:** Domain Proxy
**Date:** 2026-02-13
**Status:** Draft
**Context Brief:** Context Brief: Invoice Processing Pain Points, 2026-02-08
**Problem Frame:** Problem Frame: Blind Invoice Approvals, 2026-02-11

---

## 1. Personas

### Maya — Department Approver

- **Role:** Department head who approves invoices under $10,000
- **Mental Model:** Thinks of approval as a judgment call — she needs to see whether the expense was expected, matches a purchase order, and fits within her remaining budget. She mentally compares each invoice against "what I approved buying this quarter."
- **Goals:**
  - Approve or reject invoices within the same business day they arrive
  - Never approve an invoice that exceeds her remaining departmental budget
- **Frustrations:**
  - The approval email contains only vendor name and amount — no purchase order reference
  - She must open three separate systems to check budget remaining, PO details, and invoice history

## 3. Core Scenarios

### Scenario: Standard Invoice Approval Within Budget

**Persona:** Maya — Department Approver
**Preconditions:**
- An invoice has been submitted and routed to Maya's approval queue
- The invoice amount is under $10,000 (Maya's approval threshold)
- A matching purchase order exists in the procurement system

**Steps:**

1. Maya receives a notification that a new invoice requires her approval
2. Maya opens the approval interface and sees the invoice details: vendor name, invoice number, line items, total amount, and submission date
3. Maya reviews the linked purchase order displayed alongside the invoice — she compares line items and quantities
4. Maya checks the budget summary panel showing her department's remaining quarterly budget
5. Maya clicks "Approve" and optionally adds a note
6. The system records the approval with timestamp and routes to the next step

**Postconditions:**
- The invoice status changes from "Pending Approval" to "Approved"
- Maya's departmental budget forecast is updated to reflect the approved amount

**Notes:** Approximately 80% of invoices follow this path. Target time: under 2 minutes.

## 4. Edge Cases

| # | Scenario | Trigger Condition | Expected Behavior | Priority |
|---|----------|-------------------|-------------------|----------|
| 1 | Standard Approval | Invoice amount exactly equals remaining budget, leaving $0 | System displays warning: "This will fully exhaust your Q1 budget" — approval allowed but flagged | Must Handle |
| 2 | Standard Approval | Linked purchase order was canceled since invoice submission | System blocks approval: "Associated PO #4521 was canceled on 2026-02-01" | Must Handle |
```

### Example 2: Content Publishing Pipeline

A Problem Frame identified that content editors cannot preview how their article will look on mobile devices before publishing.

**How the artifact looks in practice (abbreviated):**

```markdown
# Domain Scenarios: Content Publishing Pipeline

**Prepared by:** Domain Proxy
**Date:** 2026-02-15
**Status:** Under Review
**Context Brief:** Context Brief: Publishing Quality Issues, 2026-02-10
**Problem Frame:** Problem Frame: Blind Publishing Without Mobile Preview, 2026-02-12

---

## 2. Domain Vocabulary

| Term | Definition | Synonyms to Avoid | Example Usage |
|------|-----------|-------------------|---------------|
| Draft | An article created but not yet submitted for review — only visible to the author | "Unpublished" (ambiguous — could mean rejected or scheduled) | "She saved the article as a draft to continue editing tomorrow" |
| Slug | The URL-safe identifier generated from the article title | "URL," "permalink" (these refer to the full address, not just the path segment) | "The slug for 'Getting Started' is 'getting-started'" |
| Publishing slot | A scheduled date and time when an article becomes visible | "Go-live date" (implies one-time; slots can be reused) | "The editor assigned the article to the Tuesday 9 AM publishing slot" |

## 5. Exception Paths

### Exception: Image Assets Fail to Load in Preview

- **What Goes Wrong:** The editor opens mobile preview and images display as broken placeholders. Images render correctly in the desktop editor but fail in the preview renderer because they reference internal CDN paths the preview environment cannot access.
- **Detection:** The preview renderer logs a 403 for each failed image. The editor sees broken image icons.
- **Recovery:** The preview renderer falls back to showing alt text in a styled placeholder, with a banner: "1 image could not be loaded in preview — it will render correctly when published."
- **User Communication:** "This preview could not load 1 image. The published version will display correctly. The affected image is in section 3 with alt text 'API key generation dialog.'"

## 7. Open Questions

1. **Should the mobile preview show the exact production rendering engine, or is an approximation acceptable?**
   _Context: Exact match requires running the production build for every preview (15-30 second latency). Approximation using responsive CSS is instant but may miss edge cases._
   _Suggested respondent: Engineering lead for the content delivery platform_
```