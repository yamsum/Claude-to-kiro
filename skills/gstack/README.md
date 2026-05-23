# gstack Skill Adaptations

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) (MIT)

gstack ships 50+ Claude Code skills. This directory contains adaptations of the subset most relevant to Angular CI/CD in a banking environment. Each skill is in its own subdirectory.

## Included Skills

| Skill | gstack Command | Description |
|-------|---------------|-------------|
| [review](review/) | `/review` | Pre-landing PR review |
| [security](security/) | `/cso` | OWASP + STRIDE security audit |
| [qa](qa/) | `/qa` | QA testing with bug triage and fix loop |
| [investigate](investigate/) | `/investigate` | Root-cause debugging |
| [ship](ship/) | `/ship` | Release management checklist |

## gstack Skills Not Included (Yet)

These gstack skills exist and may be worth adapting for future iterations:

| gstack Command | Description | Why Deferred |
|---------------|-------------|--------------|
| `/office-hours` | Product design brainstorming | More relevant to product teams than DevOps |
| `/design-*` | UI design consultation and review | Frontend-design focus; may add later |
| `/document-release` | Auto-update docs after shipping | High value; planned next |
| `/canary` | Post-deploy monitoring | High value for CI/CD; planned next |
| `/autoplan` | Automated planning pipeline | Covers multiple skills; complex to adapt |
| `/browse` | Real browser control | Requires Playwright infrastructure |
| `/retro` | Engineering retrospectives | Process skill; low adaptation complexity |

## Notes on Adaptation

gstack skills use Claude Code's parallel subagent dispatch (`Agent(...)`) extensively. Since Kiro is single-agent, all parallel dispatch has been rewritten as sequential checklists. This trades speed (parallel analysis) for compatibility, but the coverage is equivalent.

The checklist at `review/checklist.md` is shared across the review and security skills and can be customized per project.
