# VS Code Format Reference

Quick reference for VS Code AI integration formats used in this repo.

## GitHub Copilot Instructions

**Location:** `.github/copilot-instructions.md` in your project root.

Copilot reads this file as persistent project context. It is injected into Copilot Chat and inline suggestions for all users of the repo.

**Rules:**
- Plain markdown only — no YAML frontmatter
- Use `##` headings to separate skill sections
- Keep the total file under ~4000 tokens (roughly 3000 words) — it competes with your code for context
- Write in imperative language: "Check for...", "Flag...", "Verify..."
- Avoid tool-specific references (no "use Bash", "spawn Agent") — Copilot interprets instructions generically

**Structure used in this repo:**
```markdown
# Project AI Instructions

Brief one-paragraph project description.

## Code Review

Instructions for what Copilot should check during code review...

## Security

Security-specific instructions...

## Testing

QA and testing instructions...
```

**Aggregation:** The `.github/copilot-instructions.md` at the root of this repo is the aggregated file for this meta-repo. For your project, copy the relevant sections from each `skills/<name>/vscode/copilot-instructions.md`.

---

## VS Code Tasks

**Location:** `.vscode/tasks.json`

Tasks are runnable from the Command Palette (`Ctrl+Shift+P → Tasks: Run Task`) or keyboard shortcuts.

**Schema for a single task:**
```json
{
  "label": "Human-readable name",
  "type": "shell",
  "command": "the-cli-command --flags",
  "args": [],
  "options": {
    "cwd": "${workspaceFolder}"
  },
  "group": "build",
  "presentation": {
    "reveal": "always",
    "panel": "new",
    "clear": true
  },
  "problemMatcher": []
}
```

**Full tasks.json wrapper:**
```json
{
  "version": "2.0.0",
  "tasks": [
    { ... },
    { ... }
  ]
}
```

**Group values:** `"build"`, `"test"`, `"none"`. Use `"test"` for QA tasks so they appear under `Tasks: Run Test Task`.

**Panel values:** `"shared"` (reuse existing terminal), `"new"` (always open fresh terminal). Use `"new"` for long-running tasks.

**Variable substitutions:**
- `${workspaceFolder}` — absolute path to the opened folder
- `${file}` — currently active file
- `${relativeFile}` — active file relative to workspace

---

## Extensions Recommendations

**Location:** `.vscode/extensions.json`

```json
{
  "recommendations": [
    "github.copilot",
    "github.copilot-chat"
  ]
}
```

Users see a prompt to install recommended extensions when they open the workspace.

---

## Aggregation Notes

This repo ships an aggregated `.vscode/tasks.json` and `.github/copilot-instructions.md` that combine all adapted skills. When copying to your project:

1. If you already have a `.vscode/tasks.json`, merge the `"tasks"` arrays — don't overwrite.
2. If you already have `.github/copilot-instructions.md`, append skill sections — don't replace the whole file.
3. Check for duplicate task `"label"` values after merging.
