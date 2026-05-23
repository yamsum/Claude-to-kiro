---
inclusion: manual
---
<!-- Source: https://github.com/garrytan/gstack — investigate/SKILL.md -->
<!-- Version: 1.0.0 -->

# Root-Cause Investigation

Triggered when the user asks to investigate a bug, diagnose an issue, or find the root cause of a problem. Follow this systematic process. Do not modify any files until Phase 4.

## Phase 1: Scope the Problem

1. Ask (or infer from context): What is the symptom? When does it occur? What is the expected behavior?
2. Identify the affected area: which component, service, module, or API endpoint?
3. List the files most likely involved (max 10). Do not expand scope beyond this list without justification.

## Phase 2: Reproduce

1. Document the exact reproduction steps.
2. If tests exist, find the failing test or write a minimal reproduction in a test file (do not commit this test yet).
3. Confirm the reproduction is reliable (not intermittent) before proceeding.
4. If intermittent: look for timing dependencies, race conditions, or environment differences.

## Phase 3: Hypothesize

Form 2-3 distinct hypotheses about the root cause. For each:
- State the hypothesis clearly
- Identify the specific code location it implies
- Describe how you would verify or falsify it

Evaluate the most likely hypothesis first. Do not start fixing before identifying the root cause.

## Phase 4: Identify Root Cause

Read the specific code locations identified by the leading hypothesis. Trace the execution path:
1. Where does the incorrect value or behavior originate?
2. Is this a symptom or the root cause? (If a symptom, trace further back.)
3. Are there other call sites that have the same bug?

Confirm the root cause before proceeding. State it clearly: "Root cause: X because Y."

## Phase 5: Fix

Apply the minimal fix that addresses the root cause without changing unrelated behavior:
1. Read the file to understand the full context.
2. Make the targeted change.
3. Do not refactor, clean up, or improve unrelated code in this same commit.
4. Commit: `git commit -m "fix: <root cause in one line>"`

## Phase 6: Verify

1. Run the reproduction test (or the failing test from Phase 2). It should pass.
2. Run the full test suite for the affected module: `ng test --include=**/affected*.spec.ts --no-watch`
3. Confirm no regressions in related areas.

## Phase 7: Report

Produce a brief investigation report:
```
Root Cause: [one paragraph]
Location: [file:line]
Fix applied: [commit SHA + description]
Other affected call sites: [list, or "none found"]
Follow-up recommended: [any related issues worth tracking, or "none"]
```

## Constraints

- Do not modify files during Phases 1-4 (investigation only).
- Do not fix more than one bug per investigation session. Open new sessions for additional bugs.
- Do not bypass existing tests or test configuration to make the build pass.
- If the root cause is in a third-party library, document the workaround rather than patching the library.
