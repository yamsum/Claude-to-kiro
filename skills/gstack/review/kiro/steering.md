---
inclusion: manual
---
<!-- Source: https://github.com/garrytan/gstack — review/SKILL.md -->
<!-- Version: 1.0.0 -->

# Pre-Landing PR Review

Triggered when the user asks to review a PR, check a diff, or run a pre-landing review. Follow these steps in order.

## Step 0: Verify You Are on a Feature Branch

Run `git status` and `git branch`. If on the main/master/develop branch with no diff, report: "No changes to review — you appear to be on the base branch." and stop.

## Step 1: Detect Scope Drift

1. Read `.kiro/specs/` (if present), `TODOS.md`, or recent commit messages to identify the stated intent of these changes.
2. Run `git diff --name-only $(git merge-base HEAD origin/main) HEAD` (replace `origin/main` with the actual base branch if different).
3. Compare the list of changed files against the stated intent.
4. Flag as **SCOPE DRIFT** any files changed that are unrelated to the stated intent.
5. Flag as **MISSING** any files the intent implies should be changed but aren't.

## Step 2: Load the Review Checklist

Read `.kiro/review-checklist.md` if it exists, otherwise read `skills/gstack/review/checklist.md`. This defines the categories to check.

## Step 3: Compute the Diff

```
git diff $(git merge-base HEAD origin/main) HEAD
```

Also check for uncommitted changes: `git diff HEAD`.

If the diff is larger than 500 lines, focus the review on:
1. Security and data safety categories first
2. New files and new public APIs second
3. Logic changes to existing functions third

## Step 4: Run the Checklist

For each category in the checklist, scan the diff and report findings. Format each finding as:

```
[SEVERITY] Category: Description
File: path/to/file.ts, line ~42
Problem: What is wrong and why it matters
Fix: What to do (specific, not vague)
Action: AUTO-FIX | ASK
```

**SEVERITY levels:**
- `CRITICAL` — Security vulnerability, data corruption risk, or production outage potential
- `IMPORTANT` — Logic error, missing validation, or policy violation
- `NIT` — Style, naming, or minor best-practice issue

**AUTO-FIX** = mechanical change with no ambiguity (add missing null check, fix typo in log message).
**ASK** = requires a decision (architectural change, behavior change, security trade-off).

## Step 5: Plan Completion Audit

If a plan file exists in `.kiro/specs/`, `.plan/`, or `TODOS.md`:
1. Extract each actionable item.
2. Check it against the diff.
3. Report: `DONE`, `PARTIAL`, `NOT DONE`, or `UNVERIFIABLE` for each item.

## Step 6: Present Findings

1. List CRITICAL findings first, then IMPORTANT, then NITs.
2. Group AUTO-FIX items together and apply them (after confirming with the user if more than 3 files are affected).
3. Present ASK items as a numbered decision list — one question per item, not grouped.
4. If no findings: "Review complete. No issues found against the checklist."

## Step 7: Record the Review

Append a one-line summary to `.kiro/memory/session-log.md` (if it exists):
```
## [date] Review: [branch-name] — [N critical, N important, N nits] findings
```

## Constraints

- Do not commit, push, or create a pull request.
- Do not auto-fix CRITICAL findings without explicit user confirmation.
- Suppress findings that have been previously reviewed and explicitly dismissed (check the session log).
