# Compliance Matrix

Maps regulatory and industry frameworks to their documentation requirements for the technical specification's Compliance Posture section.

## Framework → Documentation Requirements

### GDPR (General Data Protection Regulation)

| Requirement | What to Document |
|---|---|
| Data inventory | What personal data is collected, where it's stored, retention periods |
| Legal basis | Consent, legitimate interest, or contractual necessity for each data type |
| Data subject rights | How deletion, export, and access requests are handled |
| Cross-border transfers | Whether data leaves the EU/EEA, transfer mechanisms (SCCs, adequacy decisions) |
| Data Protection Impact Assessment | Whether a DPIA has been conducted for high-risk processing |
| Breach notification | Process for detecting and reporting breaches within 72 hours |

**Applies when**: Product processes personal data of EU/EEA residents.

### SOC 2 Type II

| Requirement | What to Document |
|---|---|
| Trust Service Criteria | Which criteria are in scope (Security, Availability, Processing Integrity, Confidentiality, Privacy) |
| Audit period | Start and end dates of the most recent audit |
| Auditor | Name of the auditing firm |
| Report availability | How customers can request the report (NDA-gated, trust center, sales) |
| Remediation items | Any noted exceptions and their remediation status |

**Applies when**: Product handles customer data and customers require third-party assurance.

### ISO 27001

| Requirement | What to Document |
|---|---|
| Certification scope | Which systems and processes are covered |
| Certification body | Accredited body that issued the certificate |
| Certificate validity | Issue date and expiration date |
| Statement of Applicability | Which Annex A controls are implemented |
| Surveillance audits | Dates of most recent surveillance audit |

**Applies when**: Enterprise customers require ISO certification, especially in EU markets.

### PCI DSS

| Requirement | What to Document |
|---|---|
| PCI level | Merchant or service provider level (1-4) |
| SAQ type | Which Self-Assessment Questionnaire applies |
| Cardholder data environment | Where card data is stored, processed, or transmitted |
| Tokenization | Whether card data is tokenized and by which provider |
| QSA assessment | Whether an external Qualified Security Assessor is used |

**Applies when**: Product stores, processes, or transmits payment card data.

### HIPAA

| Requirement | What to Document |
|---|---|
| PHI handling | What Protected Health Information is stored or processed |
| BAA status | Whether Business Associate Agreements are in place with subprocessors |
| Encryption | Encryption at rest and in transit for PHI |
| Access controls | Role-based access to PHI, audit logging |
| Breach notification | HIPAA-specific breach notification procedures |

**Applies when**: Product handles Protected Health Information for covered entities.

### EU AI Act

| Requirement | What to Document |
|---|---|
| Risk classification | Unacceptable, High, Limited, or Minimal risk tier |
| AI system inventory | Which AI/ML models are used and their purpose |
| Transparency obligations | How users are informed they're interacting with AI |
| Human oversight | Mechanisms for human review of AI outputs |
| Data governance | Training data quality, bias testing, documentation |
| Technical documentation | Model cards, performance metrics, known limitations |

**Applies when**: Product uses AI/ML models and serves EU users.

## Status Definitions

| Status | Meaning | Evidence Required |
|---|---|---|
| **Compliant** | Meets all framework requirements with current evidence | Audit report, certification, or documented assessment |
| **In Progress** | Actively working toward compliance | Timeline, gap analysis, remediation plan |
| **N/A** | Framework does not apply to this product | Brief justification (e.g., "No payment card data processed") |

## Compliance Posture Table Template

```markdown
| Framework | Status | Evidence | Last Assessed |
|---|---|---|---|
| GDPR | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
| SOC 2 Type II | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
| ISO 27001 | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
| PCI DSS | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
| HIPAA | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
| EU AI Act | [Compliant / In Progress / N/A] | [Link or description] | [YYYY-MM-DD] |
```

## Usage Notes

- Always assess all 6 frameworks — marking N/A is better than omitting
- "Compliant" without evidence is a claim, not a fact — always cite the evidence
- Review compliance posture quarterly or when architecture changes significantly
- Cross-reference with `product.yaml` `security_compliance` field to keep both in sync
