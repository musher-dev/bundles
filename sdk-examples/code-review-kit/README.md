# Code Review Kit

Musher bundle with installable skills for code review workflows. Designed as the primary bundle for SDK skill installation, export, and multi-platform integration examples.

## Registry

- **Name:** `musher-examples/code-review-kit`
- **Version:** 1.2.0
- **Replaces:** `acme/code-review-kit:1.2.0` (SDK placeholder)

## Assets

| Logical Path | Type | Description |
|---|---|---|
| `skills/researching-repos/SKILL.md` | skill | Repository structure exploration and dependency mapping |
| `skills/drafting-release-notes/SKILL.md` | skill | Release notes generation from git history |
| `skills/reviewing-pull-requests/SKILL.md` | skill | Systematic pull request review |
| `skills/checking-test-coverage/SKILL.md` | skill | Test coverage gap analysis |
| `prompts/review-checklist.md` | prompt | Structured code review checklist |
| `prompts/severity-guidelines.md` | prompt | Review finding severity classification |

## Used By SDK Examples

### TypeScript SDK
- `examples/basics/pull-bundle.ts` — pulls and enumerates this bundle
- `examples/basics/resolve-bundle.ts` — resolves manifest metadata
- `examples/claude/install-project-skills.ts` — installs skills to `.claude/skills/`
- `examples/ide/install-vscode-skills.ts` — installs skills to VS Code

### Python SDK
- `examples/claude/install_project_skills.py` — installs skills to `.claude/skills/`
- `examples/claude/export_plugin.py` — exports skills as a Claude plugin
- `examples/openai/hosted_inline_skill.py` — converts skills to OpenAI-compatible tools
- `examples/openai/local_shell_skill.py` — exports skills for local shell execution
