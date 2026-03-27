# Security Policy Framework

A section-by-section authoring guide for `SECURITY.md` files. Each section includes its purpose, what to include, a quality test, and an example format.

## Section-by-Section Guide

### 1. Supported Versions

**Purpose:** Tell reporters which versions will actually receive patches so they do not waste time reporting issues in unsupported releases.

**What to include:**
- A table of version ranges with support status
- The policy for how long a version remains supported after a new major release
- End-of-life dates for older versions, if applicable

**Quality test:** Could a reporter look at this table and immediately know whether their version will receive a fix?

**Example format:**

```markdown
## Supported Versions

| Version | Supported          |
|---------|--------------------|
| 3.x     | :white_check_mark: |
| 2.x     | :white_check_mark: Until 2026-09-01 |
| 1.x     | :x:                |
| < 1.0   | :x:                |
```

---

### 2. Reporting a Vulnerability

**Purpose:** Give the reporter a single, unambiguous channel for private disclosure. Remove all friction from the reporting process.

**What to include:**
- The preferred reporting method (GitHub Security Advisories, email, or platform)
- What information to include in the report (affected version, reproduction steps, potential impact)
- An explicit statement that reporters should NOT open public issues for security vulnerabilities
- Optional: a PGP key for encrypted email communication

**Quality test:** Could a first-time reporter submit a complete, useful report without asking any clarifying questions?

**Example format:**

```markdown
## Reporting a Vulnerability

**Please do not open a public issue for security vulnerabilities.**

Use [GitHub Security Advisories](https://github.com/OWNER/REPO/security/advisories/new)
to report vulnerabilities privately.

Include in your report:
- Affected version(s)
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

If you prefer email, contact security@example.com.
```

---

### 3. Response Timeline

**Purpose:** Set expectations so the reporter knows when they will hear back and what happens next.

**What to include:**
- Acknowledgment SLA (time to first response confirming receipt)
- Triage SLA (time to confirm whether the report is valid and assess severity)
- Fix SLA by severity level (time from confirmation to patch release)
- Communication cadence during the fix window (e.g., weekly updates)

**Quality test:** Does every SLA use a specific number? Are the timelines realistic for the project's maintainer capacity?

**Example format:**

```markdown
## Response Timeline

| Step | Timeframe |
|------|-----------|
| Acknowledgment | Within 48 hours |
| Severity assessment | Within 1 week |
| Fix for Critical/High | Within 30 days |
| Fix for Medium/Low | Within 90 days or next release |

We will provide status updates at least every 2 weeks while a fix is in progress.
```

---

### 4. Disclosure Policy

**Purpose:** Define the coordinated disclosure process so both parties agree on when and how the vulnerability becomes public.

**What to include:**
- The disclosure model (coordinated disclosure is the standard)
- The default embargo period (typically 90 days from report)
- How credit will be given to the reporter
- What happens if the deadline passes without a fix

**Quality test:** Would both the maintainer and the reporter agree this process is fair?

**Example format:**

```markdown
## Disclosure Policy

We follow coordinated disclosure:

1. Reporter submits the vulnerability privately.
2. We acknowledge receipt and begin work on a fix.
3. We coordinate a public disclosure date with the reporter.
4. We release the fix and publish a security advisory with credit to the reporter.

The default embargo period is **90 days** from the initial report. If we cannot
ship a fix within that window, we will negotiate an extension or disclose with
available mitigations.
```

---

### 5. Scope

**Purpose:** Define what is in scope for security reporting so researchers focus their efforts productively.

**What to include:**
- Repositories, services, or domains that are in scope
- Vulnerability types that are in scope (e.g., RCE, injection, authentication bypass)
- Explicit out-of-scope items (e.g., social engineering, denial of service against production, issues in third-party dependencies)

**Quality test:** Could a researcher read this section and know, before starting any testing, whether their finding qualifies?

**Example format:**

```markdown
## Scope

### In Scope
- This repository and its published packages
- Authentication and authorization logic
- Data exposure or injection vulnerabilities
- Cryptographic implementation issues

### Out of Scope
- Social engineering attacks against maintainers or users
- Denial of service attacks against production infrastructure
- Vulnerabilities in third-party dependencies (report these upstream)
- Issues that require physical access to a user's device
```

---

### 6. Safe Harbor

**Purpose:** Assure good-faith reporters that they will not face legal action, account suspension, or retaliation for responsible disclosure.

**What to include:**
- An explicit safe harbor statement covering legal action and account status
- The conditions that qualify as "good faith" (e.g., no data destruction, no accessing other users' data beyond proof of concept)
- A commitment not to pursue legal action against reporters who follow the policy

**Quality test:** Would a security researcher feel confident reporting to this project without consulting a lawyer first?

**Example format:**

```markdown
## Safe Harbor

We consider security research conducted in accordance with this policy to be:

- Authorized under applicable anti-hacking laws
- Exempt from DMCA restrictions on circumvention
- Exempt from restrictions in our Terms of Service that would interfere
  with security research

We will not pursue legal action against or suspend the account of anyone
who reports a vulnerability in good faith following this policy.

"Good faith" means:
- You do not access or modify other users' data beyond the minimum
  necessary to demonstrate the vulnerability
- You do not degrade or disrupt our services
- You do not publicly disclose the vulnerability before we have had
  a reasonable opportunity to address it
```

---

## Coordinated Disclosure Process

The standard coordinated disclosure process protects all parties and produces the best outcomes:

1. **Private report** -- The researcher submits the vulnerability through the designated private channel. No public issues, tweets, or blog posts at this stage.

2. **Acknowledgment** -- The maintainer confirms receipt within the stated SLA. This is a receipt confirmation, not a validity assessment.

3. **Triage and assessment** -- The maintainer reproduces the issue, assesses severity, and determines the affected versions.

4. **Private fix development** -- The fix is developed in a private branch or security advisory draft. The reporter may be invited to review the fix.

5. **Coordinated public disclosure** -- The maintainer and reporter agree on a disclosure date. On that date:
   - The fix is released across all supported versions
   - A security advisory is published with a description, severity, and affected versions
   - The reporter receives credit (unless they prefer anonymity)
   - A CVE is requested if applicable

6. **Post-disclosure** -- The advisory is updated with any additional context. The reporter may publish their own write-up.

---

## RFC 9116 Reference

For web-facing projects, consider adding a `security.txt` file following [RFC 9116](https://www.rfc-editor.org/rfc/rfc9116). This is a machine-readable file placed at `/.well-known/security.txt` that provides:

- Contact information for security reports
- A link to the security policy
- Preferred language for communication
- An expiration date to ensure the file stays current

```text
# /.well-known/security.txt
Contact: https://github.com/OWNER/REPO/security/advisories/new
Policy: https://github.com/OWNER/REPO/blob/main/SECURITY.md
Preferred-Languages: en
Expires: 2027-03-27T00:00:00.000Z
```

This complements `SECURITY.md` by giving automated tools and security researchers a standardized discovery mechanism. The `Expires` field forces periodic review of the file's accuracy.

---

## Safe Harbor Template

The following is a reusable safe harbor clause that projects can adapt to their legal context. Replace bracketed placeholders with project-specific values.

```markdown
## Safe Harbor

[Project Name] pledges to the following safe harbor for security researchers
who:

1. Make a good-faith effort to avoid privacy violations, data destruction,
   and service disruption during their research.
2. Report vulnerabilities through the designated private channel before any
   public disclosure.
3. Allow a reasonable time for the issue to be resolved before disclosing
   publicly (we request [90 days] as a default).

In return, we commit to:

- **No legal action**: We will not initiate legal action against researchers
  who follow this policy, including claims under the CFAA, DMCA, or
  equivalent international laws.
- **No account penalties**: We will not suspend or terminate the account of
  a researcher acting in good faith.
- **Good-faith response**: We will acknowledge your report within
  [48 hours], provide a severity assessment within [1 week], and keep you
  informed of remediation progress.
- **Credit**: We will publicly credit you in the security advisory (unless
  you prefer anonymity).

If at any time you are unsure whether your research complies with this
policy, contact us at [security contact] before proceeding. We would rather
hear about a potential issue than have researchers stay silent out of
uncertainty.
```

Adapt the bracketed values to match the project's actual SLAs and legal jurisdiction. For open-source projects maintained by volunteers, extend the timelines and note that maintainers contribute on a best-effort basis.
