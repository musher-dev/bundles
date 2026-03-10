---
name: generate-skill
version: 1.0.0
user-invocable: true
description: Generate a well-designed project-level skill for Claude Code. Use when creating new skills, designing auto-discovered capabilities, or setting up triggered expertise. Produces portable-core skills with optional vendor adapter configurations. Triggered by: generate skill, create skill, new skill, skill file, portable skill.
argument-hint: <intent and context>
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
---

# Skill Generator

Generate a portable, project-level skill based on the provided intent and context.

## Purpose

Skills are on-demand triggered expertise -- instructions that activate when an AI assistant recognizes relevant context. A well-designed skill has a precise description (for discovery), structured instructions, and framework-independent content.

This skill helps you:
- Create new skill files with proper frontmatter and portable content
- Choose the right primitive (Persistent Instructions vs On-Demand Skill vs Hooks)
- Craft descriptions that trigger accurate auto-discovery
- Generate vendor adapter configurations when needed

**Input:** `$ARGUMENTS`

---

## Mental Model: Portable Core + Explicit Adapter

Skills follow a two-layer architecture:

| Layer | What It Contains | Where It Lives |
|-------|-----------------|----------------|
| **Portable Core** | Name, description, markdown instructions | `SKILL.md` |
| **Vendor Adapter** | Harness-specific config (tools, context mode, agent delegation) | `adapter.yaml` |

### How It Works

1. **Portable Core** is a plain markdown file with minimal frontmatter (`name` + `description`). Any AI harness can read and follow these instructions.
2. **Vendor Adapter** contains harness-specific configuration that enhances the skill for a particular tool (Claude Code's `allowed-tools`, Copilot's `disable-model-invocation`, etc.).
3. **For Claude Code specifically**, the generated SKILL.md merges adapter fields into frontmatter since that's what Claude Code reads today. The adapter.yaml serves as documentation and future portability.

### Implications for Skill Design

- **Description is everything for discovery** -- If it's not in the description, the assistant won't find the skill
- **Instructions must be framework-independent** -- No proprietary variable interpolation in the portable core
- **One skill = one focused expertise** -- Like a book on one topic, not an encyclopedia

---

## Primitive Decision Framework

Before generating, ensure this content belongs in a skill:

| Primitive | What | When to Use |
|-----------|------|-------------|
| **Persistent Instructions** | AGENTS.md / CLAUDE.md / `.claude/rules/` | Always-on rules, project knowledge, coding conventions |
| **On-Demand Skills** | SKILL.md in canonical directory | Triggered expertise, multi-step workflows, domain knowledge |
| **Hooks** | Deterministic scripts in adapter layer | Pre/post tool execution, format-on-save, validation gates |

### Key Distinctions

**Skills vs Persistent Instructions:**
- Skills activate on demand when context matches. Persistent instructions are always loaded.
- If the content is static knowledge that always applies -> Persistent Instructions.
- If the content is triggered expertise for specific tasks -> Skill.

**Skills vs Hooks:**
- Skills provide guidance and workflows. Hooks are deterministic scripts.
- If it always runs the same way with no judgment -> Hook.
- If it requires reasoning and adaptation -> Skill.

**Explicit-Only Invocation:**
- A skill with `disable-model-invocation: true` in its adapter config behaves like an explicit command (must be invoked with `/skill-name`). This is not a separate primitive -- it's a skill configuration.

**Agent Delegation:**
- Some harnesses support spawning isolated sub-agents. This is a vendor adapter concern, not a core skill property.

---

## Phase 1: Validate Input

First, validate that the required input is provided.

If `$ARGUMENTS` is empty or contains only whitespace, **STOP** and ask:

> "Please describe the intent for your skill.
>
> **Usage:** `/generate-skill <intent and context>`
>
> **Examples:**
> - `/generate-skill Analyze code for security vulnerabilities and authentication issues`
> - `/generate-skill Generate OpenAPI documentation from route definitions`
> - `/generate-skill Create unit tests following our project patterns`
>
> **What to include:**
> - **Intent**: What should this skill help accomplish?
> - **Context**: When should this skill activate? Any specific patterns or constraints?

If input is provided, proceed to derive the skill name.

### Naming Guidelines

Skill names follow these rules:
- **Required:** Lowercase alphanumeric + hyphens only, <= 64 characters, no leading/trailing hyphens
- **Recommended convention:** Gerund-Noun pattern (e.g., `auditing-security`, `generating-clients`)
- **Description quality matters more than name syntax** -- a clear description drives discovery

#### Recommended Gerund Vocabulary

| Category | Recommended Gerunds | Use When... |
|----------|---------------------|-------------|
| **Creation** | `scaffolding-`, `generating-`, `writing-` | Creating new files, code, or content |
| **Modification** | `refactoring-`, `migrating-`, `optimizing-`, `fixing-` | Changing existing code or structure |
| **Analysis** | `auditing-`, `reviewing-`, `analyzing-`, `explaining-` | Examining code without changes |
| **Execution** | `deploying-`, `running-`, `testing-` | Executing workflows or scripts |
| **Data** | `querying-`, `formatting-`, `validating-` | Working with data transformations |

#### Gerunds to Avoid

These are too vague -- prefer specific alternatives:

| Avoid | Use Instead |
|-------|-------------|
| `managing-` | `auditing-`, `configuring-` |
| `handling-` | `validating-`, `formatting-` |
| `doing-`, `using-`, `working-` | The actual action verb |
| `helping-` | The actual help: `explaining-`, `fixing-` |

### Name Derivation

1. **Identify the primary action verb** -> map to a gerund
2. **Extract the primary noun** -> prefer specific over generic (`schemas` not `code`)
3. **Combine:** `[gerund]-[noun]`
4. **Add scope prefix** (optional): `frontend-`, `api-`, `infra-`, `docs-`, `db-`
5. **Validate:** lowercase + hyphens, <= 64 chars, specific noun

See [references/naming.md](references/naming.md) for complete examples by category.

### Confirmation Step

Before proceeding, present the derived name and alternatives:

> **Derived skill name:** `[derived-name]`
>
> **Alternatives considered:**
> - `[alternative-1]` -- [why it might fit]
> - `[alternative-2]` -- [why it might fit]
>
> **Would you like to:**
> 1. Use `[derived-name]` (recommended)
> 2. Use one of the alternatives
> 3. Specify a custom name

Wait for user confirmation before proceeding to Phase 2.

---

## Phase 2: Gather Requirements

Ask clarifying questions to ensure the skill is well-designed.

### Questions to Ask

1. **Primary Purpose**
   > "What specific task should this skill help accomplish? Be as concrete as possible."

2. **Trigger Scenarios**
   > "When should this skill automatically activate? List specific keywords, file types, or situations."
   >
   > Examples of good triggers:
   > - "When user mentions 'review', 'code quality', or 'best practices'"
   > - "When working with .py files and user asks about testing"
   > - "When creating or updating API documentation"

3. **Success Criteria**
   > "How will you know the skill worked correctly? What should the output look like?"

4. **Scope Boundaries** (optional, ask if intent seems broad)
   > "What should this skill NOT do? Are there related tasks it should leave to other skills?"

5. **Scope Prefix** (ask if project is a monorepo or has distinct areas)
   > "Does this skill target a specific area of your codebase?
   > - `frontend-` -- UI/browser code
   > - `api-` -- Backend/server code
   > - `infra-` -- Deployment/infrastructure
   > - `docs-` -- Documentation
   > - `db-` -- Database/migrations
   > - None -- General-purpose skill"

6. **Target Harnesses**
   > "Which AI harnesses should this skill support?
   > - **Claude Code only** (default) -- Generates Claude Code SKILL.md with full frontmatter
   > - **Multi-harness** -- Generates portable core + adapter.yaml with vendor-specific configs
   >
   > Supported adapters: Claude Code, Copilot, Codex, OpenHands, Gemini CLI"

---

## Phase 3: Analyze Requirements

Based on the gathered information, determine:

### 3.1 Complexity Level

**Simple (prompt-only)** if the skill:
- Provides guidance, analysis, or review
- Follows patterns or conventions
- Does not generate files or run scripts

**Complex (with supporting files)** if the skill:
- Needs helper scripts for automation
- Requires templates or reference documents
- Involves multi-step workflows with file generation

### 3.2 Portable Core Validation

Before generating, verify the skill content will be portable:
- [ ] No proprietary variable interpolation (`$ARGUMENTS`, `$1`, `!cmd`, `@file`) in instructions
- [ ] File references use relative paths (`./scripts/`, `./references/`)
- [ ] Instructions use natural language directives, not framework-specific syntax
- [ ] Description follows the WHAT + WHEN + KEYWORDS pattern

### 3.3 Adapter Requirements

Based on target harnesses, determine vendor-specific configuration:

**Claude Code adapter fields:**
- `allowed-tools` -- restrict available tools (least-privilege)
- `user-invocable` -- enable `/skill-name` invocation
- `context: fork` -- isolated context for long-running analysis
- `agent` -- delegate to Explore or Plan agent
- `disable-model-invocation` -- explicit-only invocation

| Skill Intent | Suggested `allowed-tools` |
|--------------|---------------------------|
| Code review, analysis | `Read, Grep, Glob` |
| Documentation generation | `Read, Write, Glob, Grep` |
| Read-only inspection | `Read, Grep, Glob` |
| File transformation | `Read, Write, Glob` |
| Full automation (scripts, commands) | No restriction (omit field) |

See [references/adapters.md](references/adapters.md) for all vendor adapter schemas.

---

## Phase 4: Generate Portable Core SKILL.md

Create the skill file with optimized structure.

### 4.1 Craft the Description

The description is **critical** for discovery. It MUST include:

1. **WHAT**: What the skill does (action verb + outcome)
2. **WHEN**: When the assistant should use it (trigger conditions)
3. **KEYWORDS**: Specific terms that should activate this skill

**Format:** `[Action verb] [outcome]. Use when [trigger conditions]. Triggered by: [keywords/file types/situations].`

**Examples:**

Good -- specific and trigger-rich:
```
Review code for security vulnerabilities, performance issues, and best practices violations. Use when analyzing code quality, reviewing PRs, or auditing codebases. Triggered by: code review, security audit, best practices, code quality.
```

Bad -- vague and missing triggers:
```
Helps with code stuff.
```

**Quality checks before finalizing:**
- [ ] Under 1024 characters
- [ ] Contains action verb (Review, Generate, Analyze, Create, etc.)
- [ ] Includes WHEN conditions
- [ ] Lists specific trigger keywords
- [ ] No vague phrases ("helps with", "assists in", "various")

### 4.2 Structure the Content

Use this template for the generated skill's content. Instructions should be imperative and framework-independent:

```
# [Skill Title - Human Readable]

## Purpose
[1-2 sentences explaining what this skill accomplishes and why it exists]

## Instructions
Follow these steps when this skill is activated:
1. **[First Action]**
   - [Specific sub-step if needed]
2. **[Second Action]**
   - [Details]
3. **[Continue as needed...]**

## Examples
### Example 1: [Scenario Name]
**User asks:** "[Example prompt that would trigger this skill]"
**Approach:** [Step-by-step how to respond]
**Output:** [What the result should look like]

## Guidelines
- **DO**: [Positive guidance]
- **DON'T**: [Anti-pattern to avoid]

## Edge Cases
- **[Situation]**: [How to handle it]
```

**Content rules for portability:**
- Use natural language directives ("Read the file at...", "Search for...")
- Reference supporting files with relative paths (`./references/schema.md`)
- Do not embed proprietary variables (`$ARGUMENTS`, `$1`, `!cmd`, `@file`)
- Do not reference vendor-specific features (agent spawning, context forking) in the core instructions

### 4.3 Generate Frontmatter

**Portable core frontmatter** (the universal minimum):

```yaml
---
name: [skill-name]
description: [crafted description following 4.1 guidelines]
---
```

**For Claude Code output** (merge adapter fields into frontmatter since Claude Code reads SKILL.md directly):

```yaml
---
name: [skill-name]
version: 1.0.0
description: [crafted description following 4.1 guidelines]
allowed-tools: [suggested tools from 3.3, or omit if no restrictions]
user-invocable: [true/false]
context: [fork, or omit for shared context]
agent: [Explore/Plan, or omit if Claude handles directly]
---
```

**Frontmatter field reference:**

| Field | Layer | Required | Default | Purpose |
|-------|-------|----------|---------|---------|
| `name` | Core | Yes | -- | Skill identifier |
| `description` | Core | Yes | -- | Discovery text |
| `version` | Adapter | No | `1.0.0` | Semantic version |
| `allowed-tools` | Adapter | No | All tools | Restrict available tools |
| `user-invocable` | Adapter | No | `false` | Enable `/skill-name` invocation |
| `context` | Adapter | No | Shared | Set to `fork` for isolation |
| `agent` | Adapter | No | None | Delegate to specialized agent |
| `disable-model-invocation` | Adapter | No | `false` | Prevent auto-discovery |

---

## Phase 4.5: Generate Adapter Configuration

If the user requested multi-harness support, generate `adapter.yaml`:

```yaml
# Adapter configuration for [skill-name]
# Vendor-specific overrides for the portable core SKILL.md

claude:
  version: 1.0.0
  user-invocable: true
  allowed-tools:
    - Read
    - Grep
    - Glob
  # context: fork        # Uncomment for isolated execution
  # agent: Explore       # Uncomment for codebase exploration delegation

# copilot:
#   user-invocable: true
#   disable-model-invocation: false

# codex:
#   allow_implicit_invocation: true
#   dependencies:
#     tools: []

# openhands:
#   triggers:
#     - keyword1
#     - keyword2
```

See [references/adapters.md](references/adapters.md) for the complete adapter schema.

**Important:** For Claude Code projects, always generate the SKILL.md with merged frontmatter (Claude Code reads frontmatter directly). The adapter.yaml serves as documentation and portability reference.

**Future note:** A canonical `.agents/skills/` directory with automatic vendor mirroring is planned but not yet implemented. For now, output to `.claude/skills/`.

---

## Phase 5: Create Files

### 5.1 Create Directory Structure

Create the skill directory: `.claude/skills/[skill-name]/`

### 5.2 Write SKILL.md

Write the generated content to `.claude/skills/[skill-name]/SKILL.md`

For Claude Code: include merged adapter fields in frontmatter.

### 5.3 Write adapter.yaml (Multi-Harness Only)

If the user requested multi-harness support, write `.claude/skills/[skill-name]/adapter.yaml`

### 5.4 Complex Skills Only

If the skill requires supporting files, create additional structure:

```
.claude/skills/[skill-name]/
+-- SKILL.md
+-- adapter.yaml        # Multi-harness only
+-- scripts/            # Helper scripts (if needed)
+-- templates/          # Template files (if needed)
+-- references/         # Reference documentation (if needed)
```

Update SKILL.md to reference these files in a Supporting Files section.

---

## Phase 6: Output Summary

After creating the skill, display:

### Created Skill Summary

```
Skill: [name]
Location: .claude/skills/[name]/SKILL.md
```

### Portable Core Frontmatter
```yaml
name: [name]
description: [description]
```

### Claude Code Adapter Fields
```yaml
version: 1.0.0
allowed-tools: [tools or "All (no restrictions)"]
user-invocable: [true/false]
context: [fork or "shared (default)"]
agent: [agent or "None"]
```

### Portability Notes
- Portable core: Instructions use natural language directives only
- Adapter config: [adapter.yaml generated / Claude Code only]

### How to Test

> Ask a question that matches your trigger terms. For example:
> "[Example prompt that should activate this skill]"

### How to Modify

> Edit the skill directly at: `.claude/skills/[name]/SKILL.md`
>
> Key sections to customize:
> - **Description**: Update trigger keywords if the skill isn't being discovered
> - **Instructions**: Add or refine steps based on usage
> - **Examples**: Add real scenarios as you use the skill

### Next Steps

1. Test the skill by asking a question that matches the triggers
2. If the skill isn't discovered, refine the description's trigger keywords
3. Add more examples as you discover common usage patterns
4. Consider splitting if the skill becomes too broad

---

## Supporting Files

- See [references/naming.md](references/naming.md) for naming convention examples
- See [references/adapters.md](references/adapters.md) for vendor adapter configuration reference

---

## Anti-Pattern Warnings

During generation, warn if detecting these issues:

| Issue | Warning | Suggestion |
|-------|---------|------------|
| Vague description | "Description lacks specific triggers" | Add keywords and WHEN conditions |
| Overly broad scope | "Skill covers too many tasks" | Split into focused skills |
| Missing examples | "No concrete examples provided" | Add at least 2 usage examples |
| No trigger terms | "Missing keywords for auto-discovery" | List specific trigger words |
| Static knowledge | "Content is always-on context, not triggered action" | Use CLAUDE.md or .claude/rules/ instead |
| Proprietary variables in core | "Portable core contains vendor-specific interpolation" | Use natural language directives instead |
| Hardcoded absolute paths | "Instructions reference absolute file paths" | Use relative paths from skill directory |
| Vendor frontmatter in core | "Portable core contains adapter-layer fields" | Move to adapter.yaml or Claude Code merged frontmatter |
| Excessive tools | "Full tool access for read-only task" | Suggest appropriate restrictions |
| Vague noun in name | "Noun '[noun]' is too generic" | Use specific noun (e.g., `components` not `code`) |
| Name too long | "Name exceeds 64 characters" | Shorten noun or remove unnecessary words |
