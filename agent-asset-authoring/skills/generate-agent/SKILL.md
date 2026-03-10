---
name: generate-agent
version: 1.0.0
user-invocable: true
description: Generate a well-designed project-level agent for Claude Code. Use when creating new agents, designing delegation patterns, or setting up specialized task handlers with context isolation. Triggered by: generate agent, create agent, new agent, agent file, delegation, specialist agent, context isolation.
argument-hint: <agent-name> <purpose and context>
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
---

# Agent Generator

Generate an optimized, project-level agent for Claude Code based on the provided purpose and context.

## Purpose

Agents are Claude Code's delegation mechanism -- isolated specialists that handle complex tasks in their own context window. A well-designed agent has a precise description (for auto-delegation), appropriate tool restrictions, and a structured system prompt.

This skill helps you:
- Create new agent files with proper frontmatter and system prompts
- Choose the right mechanism (Agent vs Skill vs CLAUDE.md)
- Craft descriptions that trigger accurate auto-delegation
- Apply least-privilege tool restrictions

**Input:** `$ARGUMENTS`

---

## Phase 1: Validate Input

First, validate that the required input is provided.

If `$ARGUMENTS` is empty or contains only whitespace, **STOP** and ask:

> "Please provide an agent name and describe its purpose.
>
> **Usage:** `/generate-agent <agent-name> <purpose and context>`
>
> **Examples:**
> - `/generate-agent security-auditor Audit code for vulnerabilities and security issues`
> - `/generate-agent debugger Investigate errors and trace bugs through the codebase`
> - `/generate-agent code-reviewer Review code for quality, maintainability, and best practices`
>
> **What to include:**
> - **Name**: lowercase, hyphens only (e.g., `security-auditor`, `debugger`)
> - **Purpose**: What tasks should this agent handle?
> - **Context**: When should Claude delegate to this agent?"

If input is provided, extract:
- **Agent name** (`$1`): The first word/argument
- **Purpose & context**: Everything after the name

### Name Validation

Check that the agent name:
- Uses only lowercase letters, numbers, and hyphens
- Is 64 characters or fewer
- Does not start or end with a hyphen

If invalid, suggest corrections before proceeding.

---

## Understanding: Agents vs Skills vs CLAUDE.md

Before generating an agent, ensure this content belongs in an agent:

| Aspect | **CLAUDE.md** | **Skills** | **Agents** |
|--------|---------------|------------|------------|
| **What it is** | Project context & rules | Reusable capabilities | Task delegation specialists |
| **Analogy** | Employee handbook | Job procedures | Department specialists |
| **Loading** | Always active | Model-invoked (automatic) | User-invoked or auto-delegated |
| **Context** | Shared with main | Shared with main | **Isolated** (separate window) |
| **Best for** | What Claude should KNOW | What Claude can DO | What Claude should DELEGATE |
| **Model** | N/A | Inherits main | Can specify different |

### This Should Be an Agent If:
- Complex multi-step workflows needing delegation
- Tasks requiring **context isolation** (don't pollute main conversation)
- Specialized roles (security auditor, data scientist, debugger)
- Different model requirements (use Opus for thorough review)
- Permission control needed (bypassPermissions for CI, plan mode for exploration)
- Chaining agents together for complex orchestration

### This Should Be a Skill If:
- Reusable workflow triggered by keywords
- Single, focused capability
- Claude should automatically recognize and apply it
- Shared context is fine (no isolation needed)

### This Should Be CLAUDE.md If:
- Static project knowledge (architecture, philosophy)
- Coding conventions or standards
- Commands Claude should always know
- Constraints that always apply

**Decision Question**: Does this need context isolation or task delegation? If yes -> Agent. If no -> consider a skill or CLAUDE.md instead.

---

## Phase 2: Gather Requirements

Ask clarifying questions to ensure the agent is well-designed.

### Questions to Ask:

1. **Primary Purpose**
   > "What specific tasks should this agent handle? Be as concrete as possible about the domain expertise."

2. **Delegation Triggers**
   > "When should Claude delegate to this agent? List specific keywords, situations, or task types that should trigger delegation."
   >
   > Examples of good triggers:
   > - "When user asks for security review, vulnerability audit, or penetration testing"
   > - "When debugging errors, investigating failures, or tracing unexpected behavior"
   > - "When reviewing PRs, auditing code quality, or checking for anti-patterns"

3. **Tool Requirements**
   > "What tools does this agent need? Choose based on whether it needs to read only, write files, or execute commands."
   >
   > | Agent Intent | Suggested Tools |
   > |--------------|-----------------|
   > | Code review, security audit | `Read, Grep, Glob` (read-only) |
   > | Documentation generation | `Read, Write, Glob, Grep` |
   > | Debugging, investigation | `Read, Grep, Bash, Glob` |
   > | Full automation | No restriction (omit field) |

4. **Model Selection** (optional, ask if relevant)
   > "Should this agent use a specific model?"
   >
   > | Agent Task | Recommended Model |
   > |------------|-------------------|
   > | Thorough security/code review | `opus` |
   > | Quick analysis, simple tasks | `haiku` |
   > | General purpose | `sonnet` or `inherit` |

5. **Permission Mode** (optional, ask if relevant)
   > "What permission level should this agent have?"
   >
   > | Mode | Use Case |
   > |------|----------|
   > | `default` | Normal operation, ask for permissions |
   > | `acceptEdits` | Auto-approve file edits |
   > | `bypassPermissions` | CI/CD automation, trusted scripts |
   > | `plan` | Read-only exploration, planning |

6. **Skills to Load** (optional)
   > "Should any existing skills be auto-loaded into this agent?"

---

## Phase 3: Analyze Requirements

Based on the gathered information, determine:

### 3.1 Validate Agent Appropriateness

Confirm this should be an agent, not a skill or CLAUDE.md:

- **If it's static knowledge that Claude needs always** -> Recommend CLAUDE.md instead
- **If it's a triggered workflow without need for isolation** -> Recommend a skill instead
- **If it requires isolation, delegation, or specialized model** -> Proceed with Agent

### 3.2 Complexity Assessment

**Simple agent** if:
- Provides analysis, review, or investigation
- Follows established patterns
- Read-only operations

**Complex agent** if:
- Multi-step workflows with file generation
- Requires executing scripts or commands
- Chains to other agents

### 3.3 Recommend Configuration

Present recommendations and ask for confirmation:

> "Based on the agent's purpose, I recommend:
> - **Tools**: `[suggested tools]`
> - **Model**: `[suggested model]`
> - **Permission Mode**: `[suggested mode]`
>
> Would you like to adjust any of these?"

---

## Phase 4: Craft the Description

The description is **critical** for Claude's auto-delegation. It MUST include:

1. **WHAT**: What the agent does (action verb + domain)
2. **WHEN**: When Claude should delegate to it (trigger conditions)
3. **KEYWORDS**: Specific terms that should trigger delegation

**Format:**
```
[Action verb] [domain/outcome]. Use when [trigger conditions]. Triggered by: [keywords/situations].
```

**Examples:**
```yaml
# Good - specific and trigger-rich
description: Audit code for security vulnerabilities, authentication issues, and data exposure risks. Use when reviewing PRs for security, auditing codebases, or investigating potential vulnerabilities. Triggered by: security review, vulnerability audit, security check, penetration testing.

# Bad - vague and missing triggers
description: Helps with security stuff.
```

**Quality checks before finalizing:**
- [ ] Under 1024 characters
- [ ] Contains action verb (Audit, Review, Investigate, Analyze, Generate, etc.)
- [ ] Includes WHEN conditions
- [ ] Lists specific trigger keywords
- [ ] No vague phrases ("helps with", "assists in", "various")

---

## Phase 5: Generate Agent File

Create the agent file at `.claude/agents/[agent-name].md`

### 5.1 Generate Frontmatter

```yaml
---
name: [validated-agent-name]
description: [crafted description following Phase 4 guidelines]
tools: [comma-separated tools from Phase 3, or omit if no restrictions]
model: [selected model, or omit to inherit]
permissionMode: [selected mode, or omit for default]
skills: [comma-separated skill names, or omit if none]
---
```

### 5.2 Generate System Prompt

Structure the system prompt with these sections:

```markdown
# [Agent Title - Human Readable]

You are a [role description] specializing in [domain/expertise].

## Responsibilities

1. **[Primary duty]**: [Description]
2. **[Secondary duty]**: [Description]
3. **[Continue as needed...]**

## Process

When activated, follow these steps:

1. **[First step]**
   - [Specific details]
   - [Decision points if applicable]

2. **[Second step]**
   - [Details]

3. **[Continue as needed...]**

## Output Format

[Describe how the agent should structure its responses]

## Guardrails

- **DO NOT**: [Thing to avoid]
- **DO NOT**: [Another thing to avoid]
- **ALWAYS**: [Required behavior]
- **ALWAYS**: [Another required behavior]
```

### 5.3 Create the File

```bash
!mkdir -p .claude/agents
```

Write the generated content to `.claude/agents/[agent-name].md`

---

## Phase 6: Output Summary

After creating the agent, display:

### Created Agent Summary

```
Agent: [name]
Location: .claude/agents/[name].md
```

### Description Preview
> [The full description that was generated]

### Configuration
| Setting | Value |
|---------|-------|
| Tools | [list or "All (no restrictions)"] |
| Model | [model or "Inherit from parent"] |
| Permission Mode | [mode or "default"] |
| Skills | [list or "None"] |

### How to Invoke

You can invoke this agent in several ways:

```
# Explicit invocation
> Use the [name] agent to [task]

# Auto-delegation (if description has good triggers)
> [Natural request matching trigger keywords]
```

### How to Modify

> Edit the agent directly at: `.claude/agents/[name].md`
>
> Key sections to customize:
> - **Description**: Update trigger keywords if Claude isn't delegating appropriately
> - **Tools**: Add or remove based on what the agent needs to do
> - **System Prompt**: Refine the role, process, and output format based on usage

### Next Steps

1. Test the agent by asking Claude a question that matches the triggers
2. If Claude doesn't delegate, refine the description's trigger keywords
3. Monitor the agent's outputs and refine the system prompt as needed
4. Consider splitting if the agent becomes too broad

---

## Anti-Pattern Warnings

During generation, warn if detecting these issues:

| Issue | Warning | Suggestion |
|-------|---------|------------|
| Vague description | "Description lacks delegation triggers" | Add specific WHEN conditions and keywords |
| Too broad scope | "Agent covers too many domains" | Split into focused agents |
| Missing role definition | "System prompt lacks clear role" | Add "You are a..." opening |
| No isolation benefit | "This task doesn't need context isolation" | Consider using a Skill instead |
| Always-needed knowledge | "This is static context, not delegated tasks" | Use CLAUDE.md instead |
| Missing guardrails | "No boundaries defined for agent behavior" | Add DO NOT / ALWAYS rules |
| Read-only with write tools | "Agent is read-only but has write tools" | Restrict tools to Read, Grep, Glob |

---

## Reference Examples

See `references/examples.md` for complete agent examples (security auditor, debugger, code reviewer).
