# Investigate Skill Adaptation

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) — `/investigate` skill (MIT)
**Original invocation:** `/investigate` in Claude Code

## What the Investigate Skill Does

Systematic root-cause debugging. Scope-isolates the problem, forms hypotheses, reproduces the issue, identifies the root cause (not just the symptom), applies a minimal fix, and verifies resolution. The original auto-freezes to prevent accidental changes during investigation.

## What Was Adapted

- Full root-cause methodology preserved as a Kiro steering doc
- Scope isolation adapted: Kiro asked to read only relevant files rather than using Claude Code's `/freeze` mechanism
- `/careful` safety prompt adapted as an explicit instruction to confirm before modifying files
