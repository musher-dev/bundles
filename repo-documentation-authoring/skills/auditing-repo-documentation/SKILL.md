---
name: auditing-repo-documentation
version: 1.0.0
user-invocable: true
description: Score a repository's documentation health across five dimensions (Presence, Completeness, Currency, Discoverability, Quality) and produce a severity-prioritized remediation report. Covers README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, SUPPORT, CHANGELOG, LICENSE, issue templates, PR template, CODEOWNERS, GOVERNANCE, and FUNDING.yml. Use when auditing repo health, assessing documentation gaps, or prioritizing documentation improvements. Triggered by: repo audit, documentation health, community health check, repo health score, documentation gaps, missing docs.
allowed-tools: Read, Glob, Grep
---

# Audit Repository Documentation

## Purpose

Score a repository's documentation health against GitHub's recommended community health files and produce a prioritized remediation report. This is a read-only assessment — it identifies gaps and scores quality but does not create or modify files. The output is the input to the `repo_docs_author` agent's work.

## Instructions

Execute these steps in order. Each step builds on prior findings.

### Step 1: Discovery

Locate all community health files in the repository:

- Glob for root-level files: `README*`, `CONTRIBUTING*`, `CODE_OF_CONDUCT*`, `SECURITY*`, `SUPPORT*`, `CHANGELOG*`, `LICENSE*`, `GOVERNANCE*`
- Glob for `.github/` files: `ISSUE_TEMPLATE/`, `PULL_REQUEST_TEMPLATE*`, `CODEOWNERS`, `FUNDING.yml`
- Check for org-level defaults in `../.github/` (if applicable)
- Catalog every file found with its path and size

### Step 2: Detect Project Type

Determine the project type to calibrate scoring:

- Inspect package manifests, source layout, and README content
- Classify as: Library, Application, Framework, CLI Tool, or Org `.github`
- Apply project-type weighting from `./references/audit-rubric.md`

### Step 3: Score Presence

For each recommended file, score whether it exists:

| Score | Criteria |
|---|---|
| 0 | File does not exist anywhere in the repository |
| 1 | File exists but is empty or contains only the scaffold placeholder |
| 2 | File exists with substantive content |
| 3 | File exists with comprehensive content covering all recommended sections |

### Step 4: Score Completeness

For each existing file, score whether it covers all recommended sections:

- Compare file content against the section frameworks defined in the writing skills
- Check for required sections (e.g., README must have description and installation; SECURITY must have supported versions and reporting channel)
- Score 0-3 based on section coverage percentage

### Step 5: Score Currency

For each existing file, score whether it is up-to-date:

| Signal | What to Check |
|---|---|
| Stale versions | Does SECURITY's supported versions table match actual releases? |
| Dead links | Do URLs in SUPPORT and CONTRIBUTING resolve? (check format, not HTTP) |
| Outdated dates | Does CHANGELOG reference recent releases? |
| Stale references | Do CODEOWNERS teams/handles match current org? |

Score 0-3 based on freshness signals found.

### Step 6: Score Discoverability

For each file, score whether GitHub can detect and surface it:

| Check | Why It Matters |
|---|---|
| Correct filename | GitHub recognizes exact filenames (e.g., `CONTRIBUTING.md`, not `contributing.md`) |
| Correct location | Root or `.github/` as appropriate |
| Correct format | Issue templates must be `.yml` for form rendering |
| Case sensitivity | GitHub is case-sensitive for community health files |

Score 0-3 based on discoverability compliance.

### Step 7: Score Quality

For each existing file with content, score against best-practice frameworks:

- README: Does it pass the 30-second test (what, who, how)?
- CONTRIBUTING: Does it pass the first-PR test (setup in under 60 minutes)?
- SECURITY: Does it have concrete SLAs and a safe harbor clause?
- CODE_OF_CONDUCT: Does it have an enforcement process with named contacts?
- CHANGELOG: Does it follow Keep a Changelog format?

Score 0-3 based on framework adherence.

### Step 8: Synthesize Scorecard

Compile all scores into the health scorecard format from `./references/audit-rubric.md`:

1. Calculate per-file scores (average across 5 dimensions)
2. Calculate per-dimension scores (average across all files)
3. Calculate overall health score (weighted average accounting for project type)
4. Classify each finding by severity
5. Generate prioritized remediation roadmap

## Severity Classification

| Severity | Definition | Examples |
|---|---|---|
| Critical | Missing file that blocks repository function or legal compliance | No README, no LICENSE |
| High | Missing file that impacts security or contributor experience | No SECURITY, stale CONTRIBUTING |
| Medium | Missing or incomplete file that reduces discoverability | No issue templates, incomplete sections |
| Low | Missing optional file or style issue | No FUNDING.yml, minor formatting |

## Output Format

Structure every audit report as follows:

```
# Repository Documentation Health Report

**Date**: YYYY-MM-DD
**Repository**: {repo-name}
**Project Type**: {detected-type}
**Files Audited**: {count}

---

## Health Score: {score}/100

| Dimension | Score | Grade |
|---|---|---|
| Presence | {n}/3.0 | {A/B/C/D/F} |
| Completeness | {n}/3.0 | {A/B/C/D/F} |
| Currency | {n}/3.0 | {A/B/C/D/F} |
| Discoverability | {n}/3.0 | {A/B/C/D/F} |
| Quality | {n}/3.0 | {A/B/C/D/F} |

---

## File-Level Scores

| File | Presence | Complete | Current | Discover | Quality | Avg |
|---|---|---|---|---|---|---|
| README.md | {0-3} | {0-3} | {0-3} | {0-3} | {0-3} | {n} |
| ... | ... | ... | ... | ... | ... | ... |

---

## Findings

| # | Severity | File | Finding | Remediation |
|---|---|---|---|---|
| 1 | Critical | ... | ... | ... |
| ... | ... | ... | ... | ... |

---

## Remediation Roadmap

### Immediate (Critical)
- {findings}

### Short-Term (High)
- {findings}

### Medium-Term (Medium)
- {findings}

### Backlog (Low)
- {findings}
```

## Grade Scale

| Grade | Score Range | Meaning |
|---|---|---|
| A | 2.5 - 3.0 | Excellent — meets or exceeds best practices |
| B | 2.0 - 2.4 | Good — minor improvements possible |
| C | 1.5 - 1.9 | Adequate — notable gaps exist |
| D | 1.0 - 1.4 | Poor — significant remediation needed |
| F | 0.0 - 0.9 | Failing — critical files missing |

## Guidelines

- **DO**: Audit against the project type — don't penalize an application for missing CHANGELOG
- **DO**: Report evidence for every score (quote the specific gap or issue)
- **DO**: Prioritize findings by severity in the remediation roadmap
- **DO**: Note files that exist at the org level (`.github` repo defaults)
- **DON'T**: Suggest content — that's the `repo_docs_author` agent's job
- **DON'T**: Modify any files — this is a read-only assessment
- **DON'T**: Penalize for missing optional files (mark them as recommendations instead)
- **DON'T**: Score files that are intentionally absent (e.g., GOVERNANCE for a solo project)

## Edge Cases

- **Org-level `.github` repo**: Audit as the default provider — score coverage of org-wide defaults
- **Monorepo**: Audit root-level community health files; note that packages may need individual READMEs
- **Private/internal repository**: Adjust expectations — CODE_OF_CONDUCT and FUNDING.yml are typically not needed
- **Newly created repository**: All files will be missing — present as a fresh setup opportunity, not a failure

## Supporting Files

- [Audit Rubric](./references/audit-rubric.md)
