---
inclusion: always
# Options:
#   always     — injected on every agent turn
#   fileMatch  — injected when active file matches fileMatchPattern
#   manual     — never auto-injected; user must reference explicitly
# fileMatchPattern: "**/*.ts"   # required when inclusion: fileMatch
---
<!-- Source: https://github.com/<owner>/<repo> — <SkillName> skill -->
<!-- Version: 1.0.0 -->

# Skill Name

One-sentence description of what this skill does and when it applies.

## When to Use

Describe the conditions under which this skill is relevant. What does the user ask that should trigger it?

## Workflow

Step-by-step instructions for the AI. Use imperative language. Number the steps.

1. Read `<file>` to understand the current state.
2. Run `<command>` to gather information.
3. Check each item in the checklist below.
4. Report findings in the format specified below.

## Checklist

- [ ] Item one
- [ ] Item two
- [ ] Item three

## Output Format

Describe the expected output format:

```
[SEVERITY] Category: Description
File: path/to/file ~line
Problem: What is wrong and why it matters
Fix: Specific, actionable fix
```

## Constraints

List any hard limits or things the AI must not do:
- Do not modify files without explicit user confirmation.
- Do not proceed past failing checks without reporting them.
