# GitLab CI Skill

**Source:** Original — authored for this repo.
**Covers:** `.gitlab-ci.yml`, template files, component catalog entries.

## What This Covers

GitLab CI has its own set of structural concerns distinct from Jenkins:
- YAML-based (not Groovy) — different failure modes
- `needs:` creates a DAG; mistakes here cause jobs to run out of order or be skipped
- `rules:` vs. `only:/except:` — the old syntax is deprecated but still common
- Variables have complex precedence (project > group > instance > job) and protected/masked behavior
- Artifacts and cache have different semantics and common misconfiguration patterns
- `extends:` and YAML anchors interact in non-obvious ways with `rules:`

## What Was Built

| Component | Purpose |
|-----------|---------|
| `kiro/steering.md` | Full review and validation workflow for GitLab CI YAML |
| `kiro/hook.yaml` | Auto-prompt on `.gitlab-ci.yml` or template file save |
| `vscode/copilot-instructions.md` | Condensed Copilot context |
| `vscode/tasks.json` | Tasks to lint YAML locally and via GitLab API |

## Usage

```bash
cp skills/cicd/gitlab-ci/kiro/steering.md .kiro/steering/43-gitlab-ci.md
cp skills/cicd/gitlab-ci/kiro/hook.yaml .kiro/hooks/gitlab-ci.yaml
```

Invoke in Kiro: "review this GitLab CI config", "validate my pipeline YAML", "check my gitlab-ci.yml".
