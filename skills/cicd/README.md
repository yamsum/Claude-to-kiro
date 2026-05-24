# CI/CD Skill Adaptations

Skills adapted for Jenkins (scripted + declarative pipelines) and GitLab CI. These replace the generic gstack skills for teams whose primary work is pipeline development and maintenance.

## Included Skills

| Skill | Description | Kiro | VS Code |
|-------|-------------|------|---------|
| [groovy-review](groovy-review/) | Jenkins Groovy review: CPS, credentials, sandbox, serialization | ✅ steering + hook | ✅ copilot |
| [pipeline-debug](pipeline-debug/) | Systematic failing-stage diagnosis | ✅ steering | — |
| [gitlab-ci](gitlab-ci/) | GitLab CI YAML review and validation | ✅ steering + hook | ✅ copilot + task |
| [shared-library](shared-library/) | Shared library impact analysis before changes | ✅ steering | — |

## Relationship to gstack Skills

These CI/CD skills extend and replace the generic `gstack/review` skill for pipeline work:
- Use `gstack/review` for general code review on any language
- Use `cicd/groovy-review` when reviewing Jenkinsfiles or shared library Groovy
- Use `cicd/gitlab-ci` when reviewing `.gitlab-ci.yml` templates or job configs
- Both CI/CD review skills share the safety categories from `gstack/review/checklist.md` and extend it

## Jenkins vs GitLab CI Scope

**Jenkins** (scripted and declarative):
- `Jenkinsfile` — declarative or scripted pipeline definition
- `vars/*.groovy` — shared library global step definitions
- `src/org/*/` — shared library class implementations
- `resources/` — non-Groovy files available to pipelines

**GitLab CI**:
- `.gitlab-ci.yml` — root pipeline config
- `gitlab-ci/templates/*.yml` — reusable template includes
- Component catalog entries

## Setup

Copy the skills for your project:
```bash
# Jenkins focus
cp skills/cicd/groovy-review/kiro/steering.md .kiro/steering/40-groovy-review.md
cp skills/cicd/groovy-review/kiro/hook.yaml .kiro/hooks/groovy-review.yaml
cp skills/cicd/pipeline-debug/kiro/steering.md .kiro/steering/41-pipeline-debug.md
cp skills/cicd/shared-library/kiro/steering.md .kiro/steering/42-shared-library.md

# GitLab CI focus
cp skills/cicd/gitlab-ci/kiro/steering.md .kiro/steering/43-gitlab-ci.md
cp skills/cicd/gitlab-ci/kiro/hook.yaml .kiro/hooks/gitlab-ci.yaml
```
