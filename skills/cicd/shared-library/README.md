# Shared Library Skill

**Source:** Original — authored for this repo.
**Covers:** Jenkins shared library development, impact analysis, versioning, and testing.

## What This Covers

Jenkins shared libraries are the highest blast-radius code in a CI/CD repo. A bug in a shared library function can break every pipeline that calls it simultaneously. This skill provides:

- Impact analysis before making changes (who calls what)
- Versioning strategy (tag pins vs. branch tracking)
- Testing approach (JenkinsPipelineUnit)
- Safe deployment patterns (staged rollout via version pins)

## Usage

```bash
cp skills/cicd/shared-library/kiro/steering.md .kiro/steering/42-shared-library.md
```

Invoke in Kiro: "shared library impact analysis", "how many pipelines use this", "shared library change review".
