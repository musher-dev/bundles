# Evaluation Rubric

Every proposed bundle name is scored against four criteria. Each criterion is scored 0-3. A name must score **10 or higher out of 12** to pass.

## Criteria

### 1. Distinctiveness & Collision Risk

How uniquely does the name identify this bundle within the repository?

| Score | Description |
|---|---|
| 0 | Indistinguishable from an existing or likely future bundle name |
| 1 | Ambiguous; could plausibly describe multiple unrelated bundles |
| 2 | Clear within context but shares a segment with another bundle |
| 3 | Unambiguous; no realistic collision with other bundle names |

### 2. Breadth without Vagueness

Does the name cover the right scope — neither too narrow nor uselessly broad?

| Score | Description |
|---|---|
| 0 | Uses a prohibited vague term or is so generic it conveys nothing |
| 1 | Overly narrow (excludes obvious related functionality) or overly broad |
| 2 | Reasonable scope with minor over/under-inclusion |
| 3 | Precisely scoped to the bundle's responsibility boundary |

### 3. Semantic Discoverability

Could a developer guess the bundle's purpose from its name alone?

| Score | Description |
|---|---|
| 0 | Name is opaque or misleading without reading documentation |
| 1 | Partially guessable but requires domain-specific knowledge |
| 2 | Most developers would infer the general purpose correctly |
| 3 | Name immediately and clearly communicates what the bundle does |

### 4. Structural Consistency

Does the name follow the grammar and conventions defined in `naming-grammar.md`?

| Score | Description |
|---|---|
| 0 | Violates grammar (wrong order, camelCase, prohibited term, etc.) |
| 1 | Follows grammar but uses a non-standard or unjustified new term |
| 2 | Follows grammar; new term is justified and documented |
| 3 | Uses only established vocabulary in the correct `[tech]-[domain]-[purpose]` order |

## Passing Examples

| Name | D&CR | BwV | SD | SC | Total |
|---|---|---|---|---|---|
| `database-schema-governance` | 3 | 3 | 3 | 3 | **12** |
| `aws-identity-access-auditing` | 3 | 3 | 2 | 3 | **11** |
| `release-pipeline-orchestration` | 3 | 3 | 3 | 3 | **12** |

## Failing Examples

| Name | D&CR | BwV | SD | SC | Total | Issue & Revision Guidance |
|---|---|---|---|---|---|---|
| `utils` | 0 | 0 | 0 | 0 | **0** | Prohibited term; no grammar structure. Start over with a specific description. |
| `aws-tools` | 1 | 0 | 1 | 0 | **2** | Prohibited term "tools"; too broad. Narrow to a specific domain and purpose. |
| `schema-helper` | 1 | 1 | 1 | 0 | **3** | Prohibited term "helper"; missing purpose segment. Identify the core action. |
| `iam-policy-checker` | 2 | 2 | 2 | 1 | **7** | Non-standard terms; not kebab-case grammar. Map to `identity-access` domain + `auditing` purpose. |

## Revision Process

When a name scores below 10:

1. Identify which criteria scored lowest
2. Re-examine the domain and purpose vocabulary for better-fitting terms
3. Reassemble and re-score
4. If still below 10 after two attempts, present options to the user with trade-off analysis
