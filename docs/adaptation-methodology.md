# Skill Adaptation Methodology

How Claude Code skills are translated into Kiro steering documents, Kiro hooks, and VS Code integrations.

## What a Claude Code Skill Is

A Claude Code skill is a `SKILL.md` file that Claude reads as high-priority instructions. It has two parts:

**YAML frontmatter** — metadata:
```yaml
---
name: review
preamble-tier: 4
version: 1.0.0
description: "Pre-landing PR review..."
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, Agent, AskUserQuestion
triggers: review this pr, code review, check my diff
---
```

**Markdown body** — step-by-step workflow instructions written for Claude to follow. These reference Claude Code-specific primitives:
- Tool calls: `Bash(...)`, `Read(...)`, `Agent(...)`, `AskUserQuestion(...)`
- Global paths: `~/.claude/`, `~/.gstack/`
- Claude Code concepts: subagents, preamble tiers, confidence scoring

## Adaptation Process

### Step 1: Strip Claude Code Primitives

Replace every Claude Code-specific tool call or concept with a natural language equivalent that any capable AI agent can interpret.

| Claude Code | Adaptation |
|-------------|-----------|
| `Bash("git diff HEAD~1")` | "Run `git diff HEAD~1` in the terminal" |
| `Read("src/app.ts")` | "Read `src/app.ts`" |
| `Agent({ prompt: "..." })` | "Analyze the following in parallel: ..." |
| `AskUserQuestion(...)` | "Ask the user: ..." |
| `~/.claude/skills/` | `.kiro/steering/` or the project root |
| `preamble-tier: 4` | Use `inclusion: always` in Kiro frontmatter |

### Step 2: Decide: Steering Doc or Hook?

**Use a steering doc** when the skill provides:
- Reference knowledge the AI should always have (checklists, workflow steps, definitions)
- Context that informs how the AI responds to related questions
- Instructions the AI invokes manually on user request

**Use a hook** when the skill should:
- Trigger automatically on a file change, git event, or agent lifecycle event
- Run a shell command and optionally follow up with AI analysis
- Require no user prompt — it fires and acts autonomously

Many skills need both: a steering doc (the full workflow) and a hook (the trigger mechanism).

### Step 3: Write the Kiro Steering Doc

Structure:
```markdown
---
inclusion: always | fileMatch | manual
fileMatchPattern: "**/*.ts"  # only when inclusion: fileMatch
---
<!-- Source: https://github.com/source/repo — SkillName skill -->
<!-- Version: 1.0.0 -->

# Skill Name

One-line description.

## When to Use

Conditions that make this skill relevant.

## Workflow

Step-by-step instructions written in imperative language ("Read...", "Check...", "Report...").

## Checklist

- [ ] Item one
- [ ] Item two
```

Guidelines:
- Write for the AI, not the human. Imperative, concise, no hedging.
- Keep steering docs under 400 lines. If longer, split into focused sub-docs.
- `inclusion: always` for core workflow docs. `inclusion: fileMatch` for language/framework-specific ones. `inclusion: manual` for rarely-needed reference docs.
- Remove Claude Code confidence scoring, telemetry, and learnings system references — Kiro doesn't have these.

### Step 4: Write the Kiro Hook

```yaml
# Source: https://github.com/source/repo — SkillName hook
# Version: 1.0.0
name: skill-name-hook
description: One-line description of when this fires and what it does.
on:
  - fileChanged          # Options: fileChanged, agentHookStart, agentHookStop, agentTurnStart
trigger:
  command: npm test      # Optional: shell command to run before the agent prompt
  paths:
    - "**/*.ts"
    - "**/*.html"
instructions: |
  Concise instructions for what the agent should do when this hook fires.
  Reference the related steering doc for the full workflow.
```

### Step 5: Write the VS Code Copilot Instructions

Copilot reads `.github/copilot-instructions.md` as project context. This file should:
- Be additive — append a new `##` section per skill
- Use imperative language but avoid referencing Kiro-specific concepts
- Include the checklist and key decision criteria, not the full multi-phase workflow
- Be short — Copilot's context window is shared with the code it's reviewing

### Step 6: Write VS Code Tasks (where applicable)

For skills that run CLI tools (graphify, ng test, etc.):
```json
{
  "label": "Graphify: Update Knowledge Graph",
  "type": "shell",
  "command": "python -m graphify . --update",
  "group": "build",
  "presentation": { "reveal": "always", "panel": "new" }
}
```

## What to Drop

Some Claude Code skill features have no equivalent in Kiro/VS Code and should be omitted rather than approximated badly:

- **Subagent dispatch**: Claude Code launches parallel sub-agents. Kiro works sequentially. Rewrite as a sequential checklist instead.
- **Confidence scoring (1-10)**: Drop it. Kiro doesn't surface this.
- **Telemetry and learnings persistence**: Drop it. Use the memory skill instead for session-level persistence.
- **GBrain integration**: Drop it. No equivalent.
- **Cross-model synthesis (Claude + Codex)**: Drop it.
- **Preamble tiers**: Map tier 3-4 to `inclusion: always`; tier 1-2 to `inclusion: fileMatch` or `manual`.

## Angular/Banking Additions

Since this repo was built for Angular CI/CD in a banking environment, every adapted skill should include Angular-specific file patterns and banking-relevant checks where appropriate.

**Angular file patterns for hooks:**
```yaml
paths:
  - "**/*.ts"
  - "**/*.html"
  - "**/*.scss"
  - "angular.json"
  - "tsconfig*.json"
```

**Banking-relevant additions to security skills:**
- PII data in logs or API responses
- Audit trail completeness
- CORS policy on API calls
- JWT token handling and expiry
- HTTP security headers (CSP, HSTS, X-Frame-Options)
- Sensitive data in localStorage or sessionStorage
