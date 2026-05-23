# Review Skill Adaptation

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) — `/review` skill (MIT)
**Original invocation:** `/review` in Claude Code

## What the Review Skill Does

A pre-landing PR review that analyzes code changes against the base branch for structural issues automated tests miss: SQL safety, LLM trust boundary violations, race conditions, shell injection, and more. It dispatches specialist sub-agents for security, performance, testing, and maintainability, then presents findings with auto-fix vs. user-approval decisions.

## What Was Adapted

| gstack Feature | Kiro Adaptation |
|----------------|-----------------|
| 5-step review workflow | `steering.md`: Full workflow rewritten as sequential steps |
| Checklist from `checklist.md` | `checklist.md`: Standalone — reference from the steering doc |
| Parallel specialist dispatch | Sequential checklist (Kiro is single-agent) |
| Plan completion audit | Included: check TODOS.md or `.kiro/specs/` against the diff |
| Scope drift detection | Included: compare stated intent vs. changed files |
| Confidence scoring (1-10) | Dropped — not surfaced in Kiro |
| Auto-fix vs ASK classification | Included: tag findings as AUTO-FIX or ASK |
| Greptile integration | Dropped — no equivalent |
| Codex adversarial review | Dropped — single model |

## What Was Added (Angular/Banking)

- Angular-specific checks: lazy module boundaries, OnPush change detection, memory leaks in subscriptions
- Banking-specific: sensitive data in logs, HTTP interceptor security, audit trail completeness
- CI/CD focus: pipeline-breaking changes, environment config drift

## Usage

### Kiro
```bash
cp skills/gstack/review/kiro/steering.md .kiro/steering/30-review.md
cp skills/gstack/review/kiro/hook.yaml .kiro/hooks/review.yaml
cp skills/gstack/review/checklist.md .kiro/review-checklist.md
```

Invoke by telling Kiro: "review this PR", "check my diff", or "pre-landing review".

### VS Code (Copilot)
Append `skills/gstack/review/vscode/copilot-instructions.md` to your `.github/copilot-instructions.md`.

Then in Copilot Chat: "Review the changes in this PR against the checklist."
