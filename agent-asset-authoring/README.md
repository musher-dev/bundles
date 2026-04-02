# Agent Asset Authoring

Generate well-structured agent and skill files with portable-core architecture and harness adapters. This bundle takes the guesswork out of creating new agent components — it produces optimized system prompts, proper frontmatter, reference file organization, and vendor adapter configurations that work across harnesses.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/agent-asset-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Generate a new agent for reviewing database migrations
```

## What's Inside

The bundle ships two skills for generating the two most common agent asset types.

The `generate-agent` skill creates project-level agents with optimized system prompts, proper tool access declarations, model selection, and context isolation patterns. It includes reference examples showing complete agent files with different specialization patterns (read-only auditors, implementers, orchestrators).

The `generate-skill` skill creates portable skills following the two-layer architecture: a markdown core (SKILL.md) that works everywhere, plus optional vendor adapter configurations for harness-specific features. It covers naming conventions, trigger definitions, frontmatter structure, and reference file organization.

Both skills produce assets that follow the portable-core + adapter pattern, ensuring components work across all supported harnesses rather than being locked to a single vendor.

## Usage Examples

**Generate a new agent**
```
Generate an agent for auditing TypeScript code quality — it should be read-only with access to Read, Grep, and Glob tools
```
Creates an agent file with system prompt, tool restrictions, model selection, and delegation patterns.

**Generate a new skill**
```
Generate a skill for reviewing pull requests that triggers on "review PR", "code review", and "PR feedback"
```
Creates a SKILL.md with portable core content, trigger definitions, and adapter configuration.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `generate-agent` | Create project-level agents with system prompts, tool access, and context isolation |
| `generate-skill` | Create portable skills with markdown core, triggers, and vendor adapters |

</details>

---

**Domain:** agent-asset · **Technologies:** Harness-agnostic · **Aliases:** skill-authoring, agent-skills, agent-files
