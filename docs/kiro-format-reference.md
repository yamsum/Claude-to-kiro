# Kiro Format Reference

Quick reference for Kiro's steering document and hook file formats.

## Steering Documents

**Location:** `.kiro/steering/*.md` in your project root.

**Frontmatter fields:**

| Field | Values | Description |
|-------|--------|-------------|
| `inclusion` | `always` \| `fileMatch` \| `manual` | When Kiro injects this doc into context |
| `fileMatchPattern` | glob string | Required when `inclusion: fileMatch` |

**`inclusion` semantics:**
- `always` — Injected on every agent turn. Use for core workflow docs that always apply.
- `fileMatch` — Injected when the active file matches the glob. Use for framework-specific docs.
- `manual` — Never auto-injected. User must explicitly reference it. Use for rarely-needed reference material.

**Full example:**
```markdown
---
inclusion: fileMatch
fileMatchPattern: "**/*.spec.ts"
---

# Angular Unit Test Standards

When working on spec files, follow these conventions...
```

**Naming convention used in this repo:** `NN-skill-name.md` where `NN` is a two-digit sort order (lower = higher priority). Core docs use 00-09, skill docs use 10-89, reference docs use 90-99.

---

## Hooks

**Location:** `.kiro/hooks/*.yaml` in your project root.

**Schema:**

```yaml
name: string                    # Unique identifier, kebab-case
description: string             # Human-readable summary

on:                             # List of trigger events (one or more)
  - fileChanged                 # Any watched file is saved
  - agentHookStart              # Before the agent starts a task
  - agentHookStop               # After the agent finishes a task
  - agentTurnStart              # Before each agent message turn

trigger:                        # Optional: shell command to run first
  command: string               # Shell command (runs before instructions)
  paths:                        # File patterns that must match for fileChanged
    - "**/*.ts"

instructions: string            # Prompt/instructions for the agent to follow
```

**Trigger event guide:**

| Event | Use for |
|-------|---------|
| `fileChanged` | Auto-analysis when source files change (linting, graph updates) |
| `agentHookStart` | Setup before a task begins (load context, check prerequisites) |
| `agentHookStop` | Cleanup after a task ends (save memory, generate summary) |
| `agentTurnStart` | Inject context at the start of each conversation turn |

**Full example:**
```yaml
name: post-commit-graph-rebuild
description: Rebuild the knowledge graph after TypeScript files change.

on:
  - fileChanged

trigger:
  command: python -m graphify . --update --quiet
  paths:
    - "**/*.ts"
    - "**/*.html"
    - "**/*.scss"

instructions: |
  The knowledge graph has been updated. If the user asks about codebase
  structure or relationships between components, read
  graphify-out/GRAPH_REPORT.md before answering.
```

---

## Specs

**Location:** `.kiro/specs/<feature-name>/` in your project root.

Specs are not covered by this repo's adaptations (they are Kiro-specific and have no direct Claude Code equivalent), but the structure for reference:

```
.kiro/specs/my-feature/
├── requirements.md    # What to build
├── design.md          # How to build it
└── tasks.md           # Breakdown of work items
```

---

## Directory Layout in This Repo

The steering docs and hooks in `.kiro/` at the root of this repo are meta-docs that describe this repository itself. When copying skills to your project, copy from `skills/<name>/kiro/` instead:

```
skills/
  graphify/
    kiro/
      steering.md   → copy to your-project/.kiro/steering/graphify.md
      hook.yaml     → copy to your-project/.kiro/hooks/graphify.yaml
```
