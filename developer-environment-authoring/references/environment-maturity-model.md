# Environment Maturity Model

A 5-level maturity framework for assessing developer environment configuration quality. Consumed by `auditing-environment-completeness` for scoring and by `environment_auditor` for producing maturity assessments.

## Purpose

Not every project needs a fully containerized, CI-aligned development environment. A weekend side project has different needs than an enterprise monorepo. This maturity model provides a realistic progression path — each level builds on the previous one, and the right target level depends on team size, project complexity, and longevity.

---

## Maturity Levels

### Level 1: Ad Hoc (Score 0-6)

**Characteristics:**
- Development setup lives in tribal knowledge or scattered README notes
- No version pinning — developers install whatever version is current
- No task runner — commands are shared via chat or wiki
- No environment variable documentation — secrets passed out of band
- Setup takes hours and usually requires help from a teammate

**Diagnostic questions:**
- Can a new developer set up the project without asking someone?
- Do developers have the same runtime versions?
- Is there any shared editor configuration?

**Expected files:** `README.md` (partial), `.gitignore` (basic)

**Upgrade path → Level 2:**
1. Create `.env.example` documenting all environment variables
2. Add `.tool-versions` or `.nvmrc` for runtime version pinning
3. Write a "Getting Started" section in README

---

### Level 2: Documented (Score 7-12)

**Characteristics:**
- README has clear setup instructions
- Runtime versions pinned in version files
- Environment variables documented in `.env.example`
- `.editorconfig` provides basic formatting consistency
- Setup still requires 5-10 manual steps following README

**Diagnostic questions:**
- Can a developer set up the project from README alone (no help)?
- Are runtime versions consistent across the team?
- Do new developers know what environment variables to configure?

**Expected files:** All of Level 1 plus:
- `.tool-versions` or `.nvmrc`
- `.env.example`
- `.editorconfig`
- `.gitignore` (comprehensive)

**Upgrade path → Level 3:**
1. Create a task runner (Taskfile.yml or Makefile) with universal tasks
2. Add a `setup` task or script that automates the README steps
3. Add VS Code recommended extensions
4. Add pre-commit hooks for formatting/linting

---

### Level 3: Automated (Score 13-18)

**Characteristics:**
- Single-command setup (`task setup` or `./scripts/setup.sh`)
- Task runner defines all common workflows (dev, test, lint, build)
- Pre-commit hooks enforce formatting and linting
- Docker Compose provides infrastructure services (database, cache)
- VS Code settings ensure consistent editor behavior
- Setup takes under 10 minutes

**Diagnostic questions:**
- Is setup a single command?
- Can you run the full test suite without manual service installation?
- Are formatting and linting enforced automatically?

**Expected files:** All of Level 2 plus:
- `Taskfile.yml` or `Makefile` with universal tasks
- `docker-compose.yml` (if infrastructure services needed)
- `.vscode/settings.json`, `extensions.json`, `launch.json`
- `.pre-commit-config.yaml` or `.husky/`
- `scripts/setup.sh` or equivalent

**Upgrade path → Level 4:**
1. Create `.devcontainer/devcontainer.json` packaging the entire environment
2. Ensure devcontainer feature versions match `.tool-versions`
3. Multi-stage Dockerfile for dev and production
4. Named volumes for dependency directories

---

### Level 4: Containerized (Score 19-22)

**Characteristics:**
- Devcontainer provides the complete development environment
- Opening in VS Code's container mode gives a fully working setup
- All tools, runtimes, and extensions are pre-configured in the container
- Infrastructure services have health checks and proper dependency ordering
- Environment works identically on macOS, Windows, and Linux
- Setup takes under 5 minutes (mostly container build time)

**Diagnostic questions:**
- Can a developer go from clone to running app using only "Reopen in Container"?
- Does the environment work on all platforms without platform-specific instructions?
- Are infrastructure services reliable (health checks, proper startup ordering)?

**Expected files:** All of Level 3 plus:
- `.devcontainer/devcontainer.json`
- `.devcontainer/Dockerfile` (if custom image needed)
- `.dockerignore`
- Multi-stage `Dockerfile` with dev and production targets

**Upgrade path → Level 5:**
1. CI reads from the same version files as local development
2. CI uses frozen/immutable dependency installation
3. CI service containers match local Docker Compose versions
4. CI uses the same task runner commands as local development
5. Optionally reuse the devcontainer in CI

---

### Level 5: CI-Aligned (Score 23-24)

**Characteristics:**
- Local, container, and CI environments are provably equivalent
- CI reads runtime versions from the same version files
- CI uses frozen lock file installation (npm ci, --frozen-lockfile)
- CI service containers use the same image versions as docker-compose.yml
- CI runs the same task runner commands as local development
- "Works on my machine" failures are structurally prevented
- Automated drift detection catches divergence

**Diagnostic questions:**
- Does CI read versions from version files (not hardcoded)?
- Does CI use the same task runner commands as local development?
- Is there an automated check for environment drift?

**Expected files:** All of Level 4 plus:
- CI config reading from version files
- CI using frozen dependency installation
- CI service versions matching docker-compose.yml
- Optional: drift detection CI job

---

## Scoring Summary

| Level | Score | Files Expected | Key Differentiator |
|---|---|---|---|
| 1 — Ad Hoc | 0-6 | README, .gitignore | Nothing standardized |
| 2 — Documented | 7-12 | + .tool-versions, .env.example, .editorconfig | Written down but manual |
| 3 — Automated | 13-18 | + Taskfile, docker-compose, .vscode/, setup script | Single-command setup |
| 4 — Containerized | 19-22 | + .devcontainer/, multi-stage Dockerfile | Platform-independent |
| 5 — CI-Aligned | 23-24 | + CI version-file reads, frozen installs | Provable parity |

---

## Choosing a Target Level

| Project Context | Target Level | Rationale |
|---|---|---|
| Solo side project | 2 (Documented) | Version pinning and env docs prevent self-inflicted drift |
| Small team (2-5), single project | 3 (Automated) | Automation pays off quickly with multiple developers |
| Growing team (5-15) | 4 (Containerized) | Platform differences and onboarding frequency justify containers |
| Large team (15+) or enterprise | 5 (CI-Aligned) | Environment drift has organizational cost; parity must be enforced |
| Open source project | 4 (Containerized) | Contributors have diverse setups; containers normalize everything |

**Rule:** Don't skip levels. Each level builds on the previous one. A Level 4 project that skips Level 2 basics (no `.env.example`) will have gaps that undermine the container setup.
