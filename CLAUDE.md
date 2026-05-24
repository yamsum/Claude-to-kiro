# Claude-to-Kiro: Maintainer Guide

This repo is maintained with Claude Code. This file describes how the repo is structured and how to use Claude Code to add or update skills.

## What This Repo Does

Translates Claude Code skills into Kiro steering documents, Kiro hooks, VS Code tasks, and GitHub Copilot instructions. Each adapted skill lives in `skills/<name>/kiro/` and/or `skills/<name>/vscode/`.

## Adding a New Skill

1. Read the source skill file (usually a `SKILL.md` with YAML frontmatter).
2. Follow the checklist in `templates/NEW-SKILL.md`.
3. Use `templates/kiro-steering-template.md` and `templates/kiro-hook-template.yaml` as starting points.
4. Update `README.md` with the new row in the skills catalog table.
5. Update `skills/README.md` with the new entry.
6. If the skill has a VS Code Copilot component, append it to `.github/copilot-instructions.md`.
7. If the skill has a VS Code task, merge it into `.vscode/tasks.json`.

## Format Rules

- Kiro steering docs: YAML frontmatter (`inclusion`, `fileMatchPattern` if relevant), then plain markdown.
- Kiro hooks: YAML only, no markdown. See `docs/kiro-format-reference.md` for schema.
- Copilot instructions: Plain markdown, no YAML. Keep each skill section under a `##` heading.
- VS Code tasks: JSON, must be valid against the VS Code tasks schema.

## Updating an Existing Skill

When the source repo releases a new version:
1. Diff the old and new `SKILL.md` from the source.
2. Apply the same changes to the Kiro/VS Code adaptation.
3. Bump the `# Version:` comment at the top of the adapted file.

## Style

- No emojis in file content unless the source skill uses them for severity markers.
- Keep steering docs concise — Kiro reads them on every agent turn; long docs waste tokens.
- Hooks are for triggered automation; don't put reference material in hooks.
- Every adapted file must have a comment linking to the source repo and original skill name.
