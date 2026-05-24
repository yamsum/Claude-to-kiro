---
inclusion: manual
---
<!-- Source: https://github.com/garrytan/gstack — ship/SKILL.md -->
<!-- Version: 1.0.0 -->

# Release Management (Ship)

Triggered when the user asks to ship, release, or create a PR for production. Follow this checklist in order. Do not proceed past a failing gate without explicit user confirmation.

## Gate 1: Pre-Ship Checks

```
git status                          # Must be clean
ng build --configuration=production # Must succeed
ng test --no-watch                  # Must pass
ng lint                             # Must pass (zero errors)
npm audit --audit-level=high        # Must show no HIGH or CRITICAL
```

If any check fails, stop and report what failed. Do not continue until fixed.

## Gate 2: Review Confirmation

Ask the user: "Has a pre-landing code review been completed for this branch?" If no:
- Offer to run the review now (see review steering doc).
- Do not proceed until review is confirmed done.

## Gate 3: Security Check (Banking)

Ask: "Has a security review been completed for this branch?" If this is a change to authentication, payments, authorization, or data handling — a security review is mandatory. Do not proceed without confirmation.

Ask: "Do you have a change management ticket or approval reference?" Record the ticket number in the commit message and PR description.

## Gate 4: Prepare the Branch

1. Check for `WIP:` commits: `git log --oneline | grep "^[a-f0-9]* WIP:"`. If found, squash them:
   - Identify the base: `git merge-base HEAD origin/main`
   - Squash into one commit: `git rebase -i <base-sha>` — the user will need to complete the interactive rebase.
   - Or ask the user to handle the squash.
2. Ensure the branch is up to date: `git fetch origin && git log HEAD..origin/main --oneline`. If behind, rebase or merge.

## Gate 5: Pre-PR Summary

Produce a PR description for the user to review and use:

```
## Summary
[2-3 bullet points: what changed and why]

## Changes
[Key files changed and what each does]

## Test plan
- [ ] Unit tests pass (ng test)
- [ ] Linting clean (ng lint)
- [ ] Built successfully for production (ng build --configuration=production)
- [ ] [Feature-specific manual test steps]

## Security & Compliance
- Review completed: [yes/no]
- Security review: [yes/no/N/A]
- Change ticket: [ticket number or N/A]
- Rollback plan: [describe or "revert this commit"]
```

Do not create the PR or push to the remote. Present the description for the user to confirm.

## Gate 6: Post-Deploy Verification

After the user confirms deployment:
1. Ask for the deployed environment URL.
2. Check the application loads without console errors.
3. Verify the key user flows affected by this change work correctly.
4. Confirm the health endpoint (if one exists) returns healthy.

If anything fails post-deploy, immediately note the rollback plan from Gate 5.

## Constraints

- Do not force-push to main/master/develop.
- Do not skip Gates 1-3 for any reason.
- Always present the PR description to the user before the branch is pushed.
