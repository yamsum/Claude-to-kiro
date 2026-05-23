# Ship Skill Adaptation

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) — `/ship` skill (MIT)
**Original invocation:** `/ship` in Claude Code

## What the Ship Skill Does

Release automation: runs tests, verifies the review was done, builds, creates the PR, monitors deployment, and verifies the release. Squashes `WIP:` commits before the PR.

## What Was Adapted

- Multi-step release checklist preserved as Kiro steering doc
- PR creation adapted: Kiro instructed to describe the PR rather than call `gh pr create` directly (the user retains control of the git push and PR creation)
- Deployment verification retained as post-deploy check instructions

## Banking Additions

- Mandatory security review confirmation before ship
- Change management: prompt for change ticket/approval reference
- Rollback plan documented before deploy
