---
inclusion: always
---

# Claude-to-Kiro Repository

This repository adapts Claude Code skills into Kiro steering documents, Kiro hooks, VS Code tasks, and GitHub Copilot instructions. It is a tooling repo, not an application.

## Structure

- `skills/` — one directory per adapted skill, with `kiro/` and `vscode/` subdirectories
- `docs/` — format references and methodology
- `templates/` — blank templates for adding new skills
- `.kiro/steering/` — steering docs for this meta-repo itself
- `.github/copilot-instructions.md` — aggregated VS Code Copilot context

## When Working in This Repo

- To add a new skill: follow `templates/NEW-SKILL.md`
- To update an existing skill: diff the source repo's SKILL.md against the current adaptation, apply changes, bump the `<!-- Version: -->` comment
- To test a steering doc or hook: copy it into a sample Angular project and verify behavior
- Do not modify `docs/` reference files unless the underlying format (Kiro or VS Code) has changed

## Key Files

| Task | File |
|------|------|
| Add a skill | `templates/NEW-SKILL.md` |
| Kiro format reference | `docs/kiro-format-reference.md` |
| VS Code format reference | `docs/vscode-format-reference.md` |
| Adaptation methodology | `docs/adaptation-methodology.md` |
| Skills catalog | `README.md`, `skills/README.md` |
