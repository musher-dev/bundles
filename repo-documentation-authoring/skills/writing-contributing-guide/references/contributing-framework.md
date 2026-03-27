# Contributing Guide Framework

A section-by-section authoring guide for CONTRIBUTING.md files. Each section defines its purpose, required content, quality test, and an example snippet.

## Section-by-Section Guide

### 1. Welcome

**Purpose:** Set the tone and signal that contributions are genuinely wanted.

**What to include:**
- A direct statement that contributions are welcome
- Specific types of contributions the project needs (code, docs, bug reports, design, translations)
- Link to CODE_OF_CONDUCT.md

**Quality test:** Does it name at least three specific contribution types? Is the tone welcoming without being patronizing or overly effusive?

**Example snippet:**
```markdown
# Contributing to ProjectName

Thank you for considering a contribution to ProjectName. We welcome bug reports,
documentation improvements, feature implementations, and test coverage expansion.

Please read our [Code of Conduct](CODE_OF_CONDUCT.md) before participating.
```

---

### 2. Ways to Contribute

**Purpose:** Help contributors self-select the right type of contribution for their skill level and available time.

**What to include:**
- A list of contribution types ranked by complexity
- Labels or tags that correspond to each type (e.g., `good first issue`, `help wanted`)
- Estimated time commitment for each type

**Quality test:** Could a newcomer scan this section and identify something they can do today?

**Example snippet:**
```markdown
## Ways to Contribute

| Type | Difficulty | Time | Where to Start |
|---|---|---|---|
| Fix a typo in docs | Beginner | 15 min | Files in `docs/` |
| Report a bug | Beginner | 20 min | [Issue template](.github/ISSUE_TEMPLATE/bug_report.md) |
| Add a test case | Intermediate | 1-2 hrs | Issues labeled `needs-tests` |
| Implement a feature | Advanced | 2-8 hrs | Issues labeled `help wanted` |
```

---

### 3. Prerequisites

**Purpose:** List every tool the contributor needs before they can begin, with exact versions and install links.

**What to include:**
- Language runtime and version (exact, not "latest")
- Package manager and version
- Any system-level dependencies (databases, compilers, native libraries)
- Links to installation instructions for each tool

**Quality test:** Does every tool have an exact version number? Does every tool have an install link? Could someone on a fresh machine follow this list?

**Example snippet:**
```markdown
## Prerequisites

- [Node.js](https://nodejs.org/) v20.11.0 or later (we recommend [nvm](https://github.com/nvm-sh/nvm))
- [pnpm](https://pnpm.io/) v8.15.0 or later
- [Docker](https://docs.docker.com/get-docker/) v24.0 or later (for integration tests)
```

---

### 4. Development Setup

**Purpose:** Take the contributor from a cloned repo to a working development environment with verified output.

**What to include:**
- Fork and clone instructions
- Dependency installation command
- Environment configuration (e.g., `.env` setup)
- Build command (if applicable)
- A verification step that proves the setup works

**Quality test:** Is every command copy-pasteable? Does the section end with a verification step that produces observable output? Are there no hidden assumptions?

**Example snippet:**
```markdown
## Development Setup

1. Fork and clone the repository:
   ```bash
   git clone https://github.com/<your-username>/projectname.git
   cd projectname
   ```

2. Install dependencies:
   ```bash
   pnpm install
   ```

3. Copy the environment template:
   ```bash
   cp .env.example .env
   ```

4. Verify your setup:
   ```bash
   pnpm test
   ```
   You should see all tests passing with output similar to:
   ```
   Tests: 142 passed, 142 total
   Time:  4.2s
   ```
```

---

### 5. Workflow (Branching, Commits, PR)

**Purpose:** Define the exact sequence from starting work to opening a pull request.

**What to include:**
- Branch naming convention with examples
- Commit message format (Conventional Commits, imperative mood, etc.)
- When and how to rebase or merge
- How to open a pull request

**Quality test:** Is the branch naming convention specified with a pattern and at least two examples? Is the commit message format defined with a template?

**Example snippet:**
```markdown
## Workflow

### Branch Naming

Use the pattern `type/short-description`:
- `fix/null-pointer-in-parser`
- `feat/add-csv-export`
- `docs/update-api-reference`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
feat: add CSV export endpoint
fix: handle null pointer in parser when input is empty
docs: update API reference for v2 endpoints
```

### Opening a Pull Request

1. Push your branch to your fork
2. Open a PR against `main`
3. Fill out the PR template completely
4. Ensure all CI checks pass
```

---

### 6. Coding Standards

**Purpose:** Specify exactly how code should be formatted and structured, referencing automated tooling rather than subjective guidelines.

**What to include:**
- Linter and formatter names with configuration file paths
- How to run linting and formatting locally
- Any pre-commit hooks that enforce standards automatically
- Language-specific conventions (naming, file structure, import ordering)

**Quality test:** Does this section reference at least one automated tool? Are there commands to run the tools locally? Is "follow best practices" absent from this section?

**Example snippet:**
```markdown
## Coding Standards

We use automated tooling to enforce style consistency:

- **Linting**: ESLint with the config in `.eslintrc.cjs`
  ```bash
  pnpm lint
  ```
- **Formatting**: Prettier with the config in `.prettierrc`
  ```bash
  pnpm format
  ```

Pre-commit hooks run both automatically. If a hook fails, fix the issues
before committing.
```

---

### 7. Testing

**Purpose:** Explain how to run the test suite and what coverage expectations exist.

**What to include:**
- Command to run the full test suite
- Command to run a subset of tests (single file, single test)
- Coverage expectations (percentage threshold or "no decrease" policy)
- How to write a new test (which directory, naming convention, what a test should assert)

**Quality test:** Can a contributor run the tests with a single command? Are coverage expectations stated as a number or policy?

**Example snippet:**
```markdown
## Testing

Run the full test suite:
```bash
pnpm test
```

Run a specific test file:
```bash
pnpm test -- src/parser.test.ts
```

### Coverage

We maintain a minimum of 80% line coverage. PRs that decrease coverage
will not be merged. Check coverage locally:
```bash
pnpm test:coverage
```
```

---

### 8. PR Process

**Purpose:** Set clear expectations for what happens after a PR is opened.

**What to include:**
- Who reviews PRs and expected response time
- What "approved" means (number of approvals required)
- Merge strategy (squash, rebase, merge commit)
- What to do if CI fails
- How to handle requested changes

**Quality test:** Is the review timeline stated as a specific number of business days? Is the merge strategy named explicitly?

**Example snippet:**
```markdown
## Pull Request Process

1. A maintainer will review your PR within **3 business days**
2. PRs require **1 approval** from a maintainer before merge
3. We use **squash merge** -- your commits will be combined into one
4. If CI fails, check the logs and push a fix. No need to open a new PR
5. If changes are requested, push new commits to the same branch
```

---

### 9. Issue Labels

**Purpose:** Help contributors understand the project's triage system so they can find work that matches their skill level.

**What to include:**
- A table of labels with descriptions
- Which labels indicate contributor-friendly work
- How to request a label if one is missing

**Quality test:** Are there at least two labels that signal "good for new contributors"?

**Example snippet:**
```markdown
## Issue Labels

| Label | Meaning |
|---|---|
| `good first issue` | Scoped, well-defined, suitable for first-time contributors |
| `help wanted` | Open for community contribution, may require project familiarity |
| `bug` | Confirmed defect |
| `enhancement` | Approved feature request |
| `needs-triage` | Not yet reviewed by a maintainer |
```

---

### 10. Recognition

**Purpose:** Acknowledge that contributors deserve credit and explain how recognition works.

**What to include:**
- How contributors are credited (CHANGELOG, README, release notes)
- Whether the project uses the All Contributors specification or a similar system
- Any other forms of recognition (shoutouts, swag, contributor badge)

**Quality test:** Is there at least one concrete mechanism for recognizing contributions?

**Example snippet:**
```markdown
## Recognition

We use the [All Contributors](https://allcontributors.org/) specification
to recognize every contribution, not just code. After your PR is merged, a
maintainer will add you to the contributors table in the README.

Contributors who make sustained contributions may be invited to join the
project as a maintainer.
```

---

## The First-PR Test

A well-written contributing guide passes this benchmark:

- **Documentation change**: A new contributor should go from reading the guide to submitting their first PR in **under 30 minutes**. This means the guide clearly explains where docs live, how to edit them, and how to submit without requiring a full development environment.

- **Code change**: A new contributor should go from reading the guide to submitting their first PR in **under 60 minutes**. This includes cloning, installing dependencies, making a change, running tests, and opening a PR.

If your guide cannot meet these thresholds, the setup instructions are too complex, the process has too many steps, or the prerequisites are not sufficiently documented.

### How to validate

1. Find someone unfamiliar with the project
2. Ask them to follow the guide from scratch on a clean machine
3. Time them
4. Note every point where they get stuck or confused
5. Fix those points in the guide

---

## All Contributors Specification

The [All Contributors](https://allcontributors.org/) specification defines a standard for recognizing all forms of contribution to open source projects, not just code. Contribution types include:

- Code, documentation, design, testing
- Bug reports, ideas, mentoring, review
- Infrastructure, translation, tutorials
- Financial support, project management

Adopting this specification signals that the project values every form of contribution equally. Integration is handled via the `all-contributors` CLI or GitHub bot, which updates a contributors table in the README automatically.
