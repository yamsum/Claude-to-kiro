# Adding a New Skill — Checklist

Use this checklist when adapting a new Claude Code skill for Kiro and VS Code.

## 1. Research the Source Skill

- [ ] Identify the source repo and skill name
- [ ] Read the source `SKILL.md` completely — note: name, version, allowed-tools, triggers, workflow steps
- [ ] Note which features are Claude Code-specific and cannot be ported (see [docs/adaptation-methodology.md](../docs/adaptation-methodology.md))
- [ ] Determine relevance to the target environment (Angular, CI/CD, banking)

## 2. Create the Skill Directory

```
skills/<skill-name>/
├── README.md
├── kiro/
│   ├── steering.md    (if the skill provides persistent context or on-demand workflow)
│   └── hook.yaml      (if the skill should trigger automatically)
└── vscode/
    ├── copilot-instructions.md  (if the skill guides AI review or analysis)
    └── tasks.json               (if the skill runs a CLI tool)
```

Not every skill needs all four files. Use this decision guide:

| Does the skill... | Create |
|-------------------|--------|
| Provide workflow the AI follows when asked? | `kiro/steering.md` |
| Need to trigger automatically on file change or agent event? | `kiro/hook.yaml` |
| Give checklist/criteria for code review or analysis? | `vscode/copilot-instructions.md` |
| Run a CLI command? | `vscode/tasks.json` |

## 3. Write the README

- [ ] Source repo link and license
- [ ] Original invocation command
- [ ] What the skill does (2-3 sentences)
- [ ] Table: original features → what was adapted, what was dropped, what was added
- [ ] Usage instructions for Kiro and VS Code

## 4. Write the Kiro Steering Doc (if applicable)

Use [kiro-steering-template.md](kiro-steering-template.md) as your starting point.

- [ ] YAML frontmatter: `inclusion: always | fileMatch | manual`
- [ ] Source comment: `<!-- Source: <url> — <skill-name> skill -->`
- [ ] Version comment: `<!-- Version: 1.0.0 -->`
- [ ] Workflow steps in imperative language
- [ ] No Claude Code tool calls (`Bash(...)`, `Agent(...)`, etc.)
- [ ] No confidence scoring, telemetry, or GBrain references
- [ ] Angular/banking additions where relevant

## 5. Write the Kiro Hook (if applicable)

Use [kiro-hook-template.yaml](kiro-hook-template.yaml) as your starting point.

- [ ] `name`: kebab-case, unique
- [ ] `description`: one sentence
- [ ] `on`: correct event(s)
- [ ] `trigger.paths`: Angular file patterns where relevant
- [ ] `instructions`: concise, references steering doc for full workflow

## 6. Write VS Code Files (if applicable)

- [ ] Copilot instructions: plain markdown, `##` heading, under 300 words
- [ ] Tasks: valid JSON, unique `label` values, correct `group` type

## 7. Update Catalogs

- [ ] Add row to `README.md` skills catalog table
- [ ] Add row to `skills/README.md` status table
- [ ] Add entry to `skills/README.md` skill summaries section
- [ ] If Copilot instructions: append section to `.github/copilot-instructions.md`
- [ ] If VS Code tasks: merge into `.vscode/tasks.json`

## 8. Self-Review

- [ ] Steering doc under 400 lines
- [ ] Hook YAML valid (no tabs, consistent indentation)
- [ ] No broken file references in README
- [ ] Source attribution present in every adapted file
- [ ] Tested by copying into a sample Angular project (or described how to verify)
