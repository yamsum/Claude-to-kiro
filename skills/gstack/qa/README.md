# QA Skill Adaptation

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) — `/qa` skill (MIT)
**Original invocation:** `/qa` in Claude Code

## What the QA Skill Does

Systematically tests a web application using real browser automation, documents bugs found with severity scores, fixes them atomically (one commit per fix), and re-tests after each fix to confirm resolution. Uses a health scoring model across 8 categories.

## What Was Adapted

| gstack Feature | Kiro Adaptation |
|----------------|-----------------|
| Real Chromium browser (Playwright) | VS Code task: `ng e2e` via Cypress or Playwright |
| 11-phase test pipeline | `steering.md`: Simplified 6-phase workflow |
| Auto-commit per fix | Retained in steering doc |
| Health score (8 categories) | Retained as checklist |
| Angular-specific checks | Added: lazy routing, reactive forms, async pipes |
| Screenshot evidence | Retained: instruct Kiro to capture terminal output as evidence |

## What Was Dropped

- GStack Browser daemon (Playwright headless infrastructure)
- iOS testing (`/ios-qa`)
- Regression test auto-generation (retained as suggestion, not automated)
- WTF-likelihood self-regulation heuristic (simplified to "stop after 10 fixes")

## Usage

### Kiro
```bash
cp skills/gstack/qa/kiro/steering.md .kiro/steering/32-qa.md
```

Invoke by telling Kiro: "QA test this", "find and fix bugs", or "run QA".

### VS Code
Merge `skills/gstack/qa/vscode/tasks.json` into your `.vscode/tasks.json`.
Run Angular tests via `Ctrl+Shift+P → Tasks: Run Test Task`.
