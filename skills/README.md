# Skills Catalog

All adapted skills, their status, and what was included vs. dropped.

## Status Table

### General Skills

| Skill | Source | Kiro Steering | Kiro Hook | VS Code Task | VS Code Copilot | Status |
|-------|--------|:-------------:|:---------:|:------------:|:---------------:|--------|
| [graphify](graphify/) | safishamsi/graphify | ✅ | ✅ | ✅ | — | Complete |
| [memory](memory/) | thedotmack/claude-mem | ✅ | ✅ | — | — | Complete |
| [review](gstack/review/) | garrytan/gstack | ✅ | ✅ | — | ✅ | Complete |
| [security](gstack/security/) | garrytan/gstack | ✅ | — | — | ✅ | Complete |
| [qa](gstack/qa/) | garrytan/gstack | ✅ | — | ✅ | — | Complete |
| [investigate](gstack/investigate/) | garrytan/gstack | ✅ | — | — | — | Complete |
| [ship](gstack/ship/) | garrytan/gstack | ✅ | — | — | — | Complete |

### CI/CD Skills

| Skill | Source | Kiro Steering | Kiro Hook | VS Code Task | VS Code Copilot | Status |
|-------|--------|:-------------:|:---------:|:------------:|:---------------:|--------|
| [cicd/groovy-review](cicd/groovy-review/) | Original | ✅ | ✅ | — | ✅ | Complete |
| [cicd/pipeline-debug](cicd/pipeline-debug/) | Original | ✅ | — | — | — | Complete |
| [cicd/gitlab-ci](cicd/gitlab-ci/) | Original | ✅ | ✅ | ✅ | ✅ | Complete |
| [cicd/shared-library](cicd/shared-library/) | Original | ✅ | — | — | — | Complete |

## CI/CD Skill Summaries

### cicd/groovy-review
Pre-review for Jenkins Groovy files. Catches the failure classes unique to the Jenkins pipeline engine: CPS violations (`.stream()` without `@NonCPS`), credential exposure via GString interpolation, sandbox-blocked method calls, non-serializable objects at pipeline scope, and missing timeouts. Includes shared library blast radius assessment.

### cicd/pipeline-debug
Failure classification before source-file diagnosis. Maps Jenkins and GitLab CI error patterns to their root cause class (CPS, sandbox, credential, agent, serialization, YAML structure) and prescribes the correct investigation path. Distinguishes between problems that can be reproduced locally and those that require live infrastructure.

### cicd/gitlab-ci
Full review of `.gitlab-ci.yml` and template files. Covers DAG correctness (`needs:`), deprecated `only:/except:` vs `rules:`, variable security (protected/masked scoping), artifact/cache semantics, image pinning, runner tag matching, and YAML anchor vs `extends:` composition.

### cicd/shared-library
Impact analysis before touching Jenkins shared library code. Finds all consumers via grep, categorizes them by version pin strategy (branch vs tag), assesses backward compatibility, checks for JenkinsPipelineUnit tests, and recommends a staged deployment plan proportional to blast radius.

## General Skill Summaries

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
