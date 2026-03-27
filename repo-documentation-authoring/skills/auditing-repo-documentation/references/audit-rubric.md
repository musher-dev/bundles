# Audit Rubric

Detailed scoring rubric for repository documentation health audits. Covers all 12 community health file types across 5 audit dimensions.

---

## Scoring Scale

Every file is scored 0-3 on each dimension:

| Score | Meaning |
|---|---|
| 0 | Absent or completely non-functional |
| 1 | Present but minimal — placeholder content, scaffold stub, or severely incomplete |
| 2 | Functional — covers core requirements with minor gaps |
| 3 | Exemplary — comprehensive, current, and follows best practices |

---

## Per-File Scoring Criteria

### README.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No README | Exists but empty or single line | Has multiple sections | Comprehensive with all standard sections |
| Completeness | — | Title and description only | Adds installation and usage | Includes quick start, badges, contributing link, license |
| Currency | — | References deprecated versions/tools | Mostly current | All links valid, examples tested, versions current |
| Discoverability | — | Wrong filename (e.g., readme.txt) | Correct name, wrong case | `README.md` at repo root |
| Quality | — | Wall of text, no structure | Structured but dry | Passes 30-second test, copy-pasteable examples |

### CONTRIBUTING.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has content | Comprehensive guide |
| Completeness | — | "PRs welcome" only | Setup + PR process | Full workflow: setup, standards, testing, PR, recognition |
| Currency | — | References obsolete tools/processes | Mostly current | All commands verified, CI references match |
| Discoverability | — | Wrong filename | Correct name | `CONTRIBUTING.md` at root, linked from README |
| Quality | — | Vague instructions | Specific but untested | Every command copy-pasteable, verification steps included |

### CODE_OF_CONDUCT.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has content | Based on recognized standard |
| Completeness | — | Standards only, no enforcement | Standards + enforcement contact | Full Contributor Covenant: standards, scope, enforcement ladder |
| Currency | — | Outdated Covenant version | Current version | Current version with active enforcement contact |
| Discoverability | — | Wrong filename | Correct name | `CODE_OF_CONDUCT.md` at root, GitHub detects it |
| Quality | — | No enforcement process | Has enforcement but vague | Named contact, concrete ladder, clear scope |

### SECURITY.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has content | Comprehensive policy |
| Completeness | — | Contact email only | Adds versions table | Full policy: versions, reporting, timeline, scope, safe harbor |
| Currency | — | Supported versions outdated | Versions mostly current | Versions match latest releases, contact monitored |
| Discoverability | — | Wrong filename | Correct name | `SECURITY.md` at root, GitHub links from Security tab |
| Quality | — | No SLAs or process | Has SLAs | Concrete SLAs, safe harbor, multiple reporting channels |

### SUPPORT.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has content | Comprehensive support guide |
| Completeness | — | "Open an issue" only | Channel matrix | Full guide: channels, before-you-ask, response expectations |
| Currency | — | Dead links to channels | Links mostly valid | All channel links valid, response times realistic |
| Discoverability | — | Wrong filename | Correct name | `SUPPORT.md` at root |
| Quality | — | Vague routing | Specific channels per need | Clear routing, honest response expectations, "before you ask" checklist |

### CHANGELOG.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has entries | Follows Keep a Changelog format |
| Completeness | — | Unsorted list of changes | Categorized entries | All 6 categories used as needed, [Unreleased] section present |
| Currency | — | Last entry months behind latest release | Recent but incomplete | Matches latest release, [Unreleased] has pending changes |
| Discoverability | — | Wrong filename or location | Correct name at root | `CHANGELOG.md` at root, linked from README |
| Quality | — | Git log dump | Human-readable entries | User-facing language, version comparison links, SemVer aligned |

### LICENSE

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Placeholder text | Valid license text | Recognized license with correct attribution |
| Completeness | — | Partial license text | Full license text | Full text with year and copyright holder |
| Currency | — | Outdated year | Current year | Year current, matches project ownership |
| Discoverability | — | Wrong filename | Correct name | `LICENSE` at root, GitHub detects and displays it |
| Quality | — | Unrecognized license | OSI-approved license | SPDX-identifiable, matches `license` field in package manifest |

### Issue Templates (.github/ISSUE_TEMPLATE/)

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No templates | Legacy markdown templates | YAML form templates | Bug + Feature + config.yml |
| Completeness | — | Single template | Two templates | Bug report + feature request + config.yml with contact links |
| Currency | — | Outdated labels or fields | Mostly current | Labels match repo, fields relevant |
| Discoverability | — | Wrong location | In `.github/ISSUE_TEMPLATE/` | Correct location, renders as issue forms on GitHub |
| Quality | — | Minimal fields | Structured fields | Required fields marked, dropdowns for common values, helpful descriptions |

### Pull Request Template (.github/PULL_REQUEST_TEMPLATE.md)

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No template | Exists but empty | Has content | Comprehensive checklist |
| Completeness | — | Description prompt only | Description + checklist | Description, change type, testing, full checklist, related issues |
| Currency | — | References outdated process | Mostly current | Matches current CI and review process |
| Discoverability | — | Wrong location or name | Correct path | `.github/PULL_REQUEST_TEMPLATE.md`, auto-populates PRs |
| Quality | — | Too long (discourages PRs) | Reasonable length | Completable in 2 minutes, uses checkboxes, HTML comments for guidance |

### CODEOWNERS (.github/CODEOWNERS)

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has rules | Comprehensive ownership map |
| Completeness | — | Default owner only | Key directories covered | All active directories mapped, file-type patterns included |
| Currency | — | References departed team members | Mostly current | All handles/teams valid in current org |
| Discoverability | — | Wrong location | `.github/CODEOWNERS` | Correct path, GitHub assigns reviewers automatically |
| Quality | — | Single catch-all rule | Directory-level rules | Team-based ownership, appropriate granularity, comments explaining patterns |

### GOVERNANCE.md

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has content | Comprehensive governance doc |
| Completeness | — | Lists maintainers only | Adds roles and decision model | Full: roles, decision-making, maintainer path, conflict resolution |
| Currency | — | Maintainer list outdated | Mostly current | Maintainer list matches actual committers, model reflects reality |
| Discoverability | — | Wrong filename | Correct name | `GOVERNANCE.md` at root |
| Quality | — | Vague decision process | Named model | Concrete criteria for advancement, specific decision-making rules |

### FUNDING.yml (.github/FUNDING.yml)

| Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Presence | No file | Exists but empty | Has platforms | Active funding configured |
| Completeness | — | Invalid YAML | One platform listed | All active platforms listed |
| Currency | — | Links to closed accounts | Mostly valid | All platform links active and accepting contributions |
| Discoverability | — | Wrong location | `.github/FUNDING.yml` | Correct path, Sponsor button appears |
| Quality | — | Invalid syntax | Valid YAML | Only active platforms, renders Sponsor button correctly |

---

## Project-Type Weighting

Not all files are equally important for every project type. Apply these weights when calculating the overall health score:

| File | Library | Application | Framework | CLI Tool | Org `.github` |
|---|---|---|---|---|---|
| README | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 |
| CONTRIBUTING | 1.0 | 0.7 | 1.0 | 1.0 | 1.0 |
| CODE_OF_CONDUCT | 1.0 | 0.5 | 1.0 | 1.0 | 1.0 |
| SECURITY | 1.0 | 1.0 | 1.0 | 1.0 | 1.0 |
| SUPPORT | 0.7 | 0.5 | 0.7 | 0.7 | 1.0 |
| CHANGELOG | 1.0 | 0.7 | 1.0 | 1.0 | 0.0 |
| LICENSE | 1.0 | 1.0 | 1.0 | 1.0 | 0.7 |
| Issue Templates | 1.0 | 0.5 | 1.0 | 0.7 | 1.0 |
| PR Template | 1.0 | 0.7 | 1.0 | 0.7 | 1.0 |
| CODEOWNERS | 0.7 | 0.7 | 0.7 | 0.7 | 1.0 |
| GOVERNANCE | 0.5 | 0.3 | 0.7 | 0.5 | 1.0 |
| FUNDING.yml | 0.3 | 0.3 | 0.3 | 0.3 | 0.3 |

**Weight interpretation**: 1.0 = full weight, 0.7 = recommended, 0.5 = optional, 0.3 = nice-to-have, 0.0 = not applicable.

---

## Health Score Calculation

1. **Per-file score**: Average of the 5 dimension scores (0-3 scale)
2. **Weighted file score**: Per-file score multiplied by project-type weight
3. **Overall score**: Sum of weighted file scores divided by sum of weights, normalized to 0-100 scale

```
overall = (sum(file_score[i] * weight[i]) / sum(weight[i])) * (100/3)
```

---

## Scorecard Output Template

```
## Health Scorecard

### Overall: {score}/100 ({grade})

### By Dimension
| Dimension | Avg Score | Grade |
|---|---|---|
| Presence | {n}/3.0 | {grade} |
| Completeness | {n}/3.0 | {grade} |
| Currency | {n}/3.0 | {grade} |
| Discoverability | {n}/3.0 | {grade} |
| Quality | {n}/3.0 | {grade} |

### By File
| File | P | C | Cu | D | Q | Avg | Weight | Weighted |
|---|---|---|---|---|---|---|---|---|
| README.md | {0-3} | {0-3} | {0-3} | {0-3} | {0-3} | {n} | {w} | {n*w} |
| ... | | | | | | | | |
```
