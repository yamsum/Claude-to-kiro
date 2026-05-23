# Claude-to-Kiro

A curated collection of Claude Code's most valuable skills and workflows, adapted to work natively with **Kiro** and **VS Code**. Built for teams that rely on those tools but want the productivity gains that Claude Code's best features offer.

## Background

Claude Code ships with a powerful skill system — markdown-based instruction documents that guide AI through multi-step engineering workflows (code review, security auditing, QA, codebase analysis, and more). This repo translates those skills into formats that Kiro and VS Code understand natively, without requiring access to the Claude Code plugin ecosystem.

## Skills Catalog

| Skill | Source | Kiro | VS Code | Description |
|-------|--------|------|---------|-------------|
| [graphify](skills/graphify/) | [safishamsi/graphify](https://github.com/safishamsi/graphify) | ✅ steering + hook | ✅ task | Build a queryable knowledge graph from your codebase |
| [memory](skills/memory/) | [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) | ✅ steering + hook | — | Persist context and decisions across AI sessions |
| [review](skills/gstack/review/) | [garrytan/gstack](https://github.com/garrytan/gstack) | ✅ steering + hook | ✅ copilot | Pre-landing PR review with specialist dispatch |
| [security](skills/gstack/security/) | [garrytan/gstack](https://github.com/garrytan/gstack) | ✅ steering | ✅ copilot | OWASP Top 10 + STRIDE security audit |
| [qa](skills/gstack/qa/) | [garrytan/gstack](https://github.com/garrytan/gstack) | ✅ steering | ✅ task | Systematic QA with bug triage and regression tests |
| [investigate](skills/gstack/investigate/) | [garrytan/gstack](https://github.com/garrytan/gstack) | ✅ steering | — | Root-cause debugging with scope isolation |
| [ship](skills/gstack/ship/) | [garrytan/gstack](https://github.com/garrytan/gstack) | ✅ steering | — | Release management: test → review → deploy → verify |

## Quick Start

### Kiro

Copy the steering documents and hooks you want into your project:

```bash
# Add a skill's steering doc to your project
cp skills/gstack/review/kiro/steering.md .kiro/steering/review.md

# Add the corresponding hook
cp skills/gstack/review/kiro/hook.yaml .kiro/hooks/review.yaml

# Add graphify for codebase analysis
cp skills/graphify/kiro/steering.md .kiro/steering/graphify.md
cp skills/graphify/kiro/hook.yaml .kiro/hooks/graphify.yaml
```

Kiro will automatically include steering documents marked `inclusion: always` and trigger hooks on the configured events.

### VS Code (GitHub Copilot)

Merge the relevant sections into your project's `.github/copilot-instructions.md`:

```bash
cat skills/gstack/review/vscode/copilot-instructions.md >> .github/copilot-instructions.md
cat skills/gstack/security/vscode/copilot-instructions.md >> .github/copilot-instructions.md
```

Add tasks to your `.vscode/tasks.json` from the individual skill task files, or copy the aggregated one:

```bash
cp .vscode/tasks.json your-project/.vscode/tasks.json
```

## Repository Layout

```
Claude-to-kiro/
├── docs/                          # Format references and methodology
│   ├── adaptation-methodology.md  # How Claude skills → Kiro/VS Code
│   ├── kiro-format-reference.md   # Kiro steering + hook schemas
│   └── vscode-format-reference.md # Copilot instructions + tasks
├── .kiro/steering/                # Kiro steering docs for this meta-repo
├── .github/copilot-instructions.md # Aggregated VS Code Copilot context
├── .vscode/tasks.json             # Aggregated VS Code tasks
├── skills/                        # One directory per adapted skill
│   ├── graphify/
│   ├── memory/
│   └── gstack/
│       ├── review/
│       ├── security/
│       ├── qa/
│       ├── investigate/
│       └── ship/
└── templates/                     # Add a new skill from scratch
```

## Adding New Skills

See [templates/NEW-SKILL.md](templates/NEW-SKILL.md) for the checklist and [docs/adaptation-methodology.md](docs/adaptation-methodology.md) for the full process.

The general pattern:
1. Identify the source skill and understand its workflow
2. Strip Claude Code-specific tool calls; rewrite as natural language steps
3. Create a Kiro steering doc and/or hook
4. Create VS Code tasks or Copilot instructions where relevant
5. Add an entry to this catalog

## Sources

- [garrytan/gstack](https://github.com/garrytan/gstack) — AI-powered software factory with 50+ Claude Code skills (MIT)
- [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) — Persistent memory for Claude Code (Apache 2.0)
- [safishamsi/graphify](https://github.com/safishamsi/graphify) — Knowledge graph builder for codebases (MIT)
- [Claude Code review documentation](https://code.claude.com/docs/en/code-review) — Multi-agent PR review reference
