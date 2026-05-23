# Skills Catalog

All adapted skills, their status, and what was included vs. dropped.

## Status Table

| Skill | Source | Kiro Steering | Kiro Hook | VS Code Task | VS Code Copilot | Status |
|-------|--------|:-------------:|:---------:|:------------:|:---------------:|--------|
| [graphify](graphify/) | safishamsi/graphify | ✅ | ✅ | ✅ | — | Complete |
| [memory](memory/) | thedotmack/claude-mem | ✅ | ✅ | — | — | Complete |
| [review](gstack/review/) | garrytan/gstack | ✅ | ✅ | — | ✅ | Complete |
| [security](gstack/security/) | garrytan/gstack | ✅ | — | — | ✅ | Complete |
| [qa](gstack/qa/) | garrytan/gstack | ✅ | — | ✅ | — | Complete |
| [investigate](gstack/investigate/) | garrytan/gstack | ✅ | — | — | — | Complete |
| [ship](gstack/ship/) | garrytan/gstack | ✅ | — | — | — | Complete |

## Skill Summaries

### graphify
Builds a queryable knowledge graph from your codebase. Kiro reads the graph report before answering structure questions; the hook rebuilds the graph when TypeScript/HTML/SCSS files change.

### memory
Lightweight session memory using a markdown log. Kiro reads prior session notes at task start and appends a summary at task end. No external infrastructure needed.

### review
Pre-landing PR review: scope drift detection, safety checklist (SQL, XSS, auth, race conditions), plan completion audit, and auto-fix vs. ASK classification. Includes Angular- and banking-specific checks.

### security
OWASP Top 10 + STRIDE threat model adapted for Angular/TypeScript frontends in a banking context. Covers injection, access control, cryptography, logging, dependency vulnerabilities, and financial data handling.

### qa
Systematic 6-phase QA: baseline → triage → fix loop (one commit per fix) → verify → report. Angular-aware: lazy routes, reactive forms, async pipe teardowns, HTTP error states.

### investigate
Root-cause debugging methodology: reproduce → hypothesize → identify cause → minimal fix → verify. Read-only until Phase 4; no refactoring in fix commits.

### ship
Release checklist with hard gates: tests pass, review confirmed, security review confirmed (mandatory for auth/payment changes), change ticket recorded, PR description produced, post-deploy verification.

## Adding a New Skill

See [../templates/NEW-SKILL.md](../templates/NEW-SKILL.md).

Priority candidates for future additions (from gstack and other sources):
- `office-hours` — Product/design brainstorming (gstack)
- `document-release` — Auto-update docs after shipping (gstack)
- `canary` — Post-deploy monitoring (gstack)
- `claude-code-review` — Multi-agent PR review (claude.ai code review)
