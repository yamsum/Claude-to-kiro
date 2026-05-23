---
inclusion: manual
---
<!-- Source: https://github.com/garrytan/gstack — qa/SKILL.md -->
<!-- Version: 1.0.0 -->

# QA Testing

Triggered when the user asks to run QA, test the application, find bugs, or "test and fix". Follows a structured 6-phase process: baseline → triage → fix → verify → report → reflect.

## Prerequisites

Before starting:
1. Ensure the working tree is clean (`git status`). If dirty, ask: "Commit or stash your changes before QA, or I'll include them in the test scope?"
2. Confirm the app can be built: `ng build` (or equivalent). If the build fails, fix the build first.
3. Confirm tests run: `ng test --no-watch` for unit tests, `ng e2e` for e2e.

## Phase 1: Scope

Determine scope:
- **Diff-aware (PR mode):** `git diff --name-only $(git merge-base HEAD origin/main) HEAD` — test only changed components and their dependencies.
- **Full-scope:** Test the entire application.

Ask the user which scope if not clear from context.

## Phase 2: Baseline Health Check

Run the full test suite and record the baseline:
```
ng test --no-watch --code-coverage 2>&1 | tail -30
ng lint 2>&1 | tail -20
```

Score the baseline across these categories (0-10 each):
- Console errors (10 = zero errors)
- Functional correctness (tests passing %)
- UX completeness (forms validate, nav works, loading states present)
- Accessibility (ARIA labels, keyboard navigation, color contrast)
- Visual correctness (no layout breaks, responsive at 375px and 1440px)
- Performance (no unnecessary re-renders, lazy modules loading correctly)
- Content (no placeholder text, truncated strings, missing translations)
- Link health (no broken router links or dead HTTP endpoints)

## Phase 3: Bug Triage

List all failing tests, lint errors, and any manual observations. For each:
- Severity: CRITICAL (blocks core user flow) | HIGH (major feature broken) | MEDIUM (degraded UX) | LOW (cosmetic)
- File: most likely source file
- Reproduction: exact steps or test name

Sort by severity descending. Focus on CRITICAL and HIGH first.

## Phase 4: Fix Loop

For each bug (start with CRITICAL):
1. Locate the source: read the failing test, trace to the component/service.
2. Apply the minimal fix. Do not refactor unrelated code.
3. Commit: `git commit -m "fix: <one-line description of what was fixed>"`
4. Re-run the relevant test to verify: `ng test --include=**/affected.spec.ts --no-watch`
5. If the fix introduces a new failure, revert and re-approach.
6. Stop after 10 fixes maximum. Report remaining issues as "not addressed" with their severity.

## Angular-Specific Checks

During manual inspection, verify:
- [ ] Lazy-loaded routes load without console errors (navigate to each route)
- [ ] Reactive forms: validation messages appear on invalid submit; form submits on valid data
- [ ] `async` pipe teardowns: no "Expression changed after it was checked" errors
- [ ] HTTP error states: 4xx and 5xx responses show user-friendly messages
- [ ] Token expiry: expired token redirects to login without white screen
- [ ] OnPush components: UI updates when inputs change

## Phase 5: Report

After all fixes (or after hitting the 10-fix cap):
1. Report baseline health score → final health score.
2. List all bugs fixed (with commit SHA).
3. List all bugs not addressed (with severity and reason).
4. List any new issues discovered but out of scope.

Format:
```
QA Report — [date]
Scope: [diff-aware|full]
Baseline: [score]/80 | Final: [score]/80

Fixed:
- [sha] fix: subscription not unsubscribed in UserDashboard (CRITICAL)

Not addressed:
- Pagination breaks at 1000+ records (MEDIUM) — out of scope for this PR

New findings (deferred):
- Color contrast ratio below WCAG AA on secondary button (LOW)
```

## Phase 6: Update TODOs

If `TODOS.md` or `.kiro/specs/` tasks exist, mark completed items as done. Append unresolved findings as new items.
