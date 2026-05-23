## Code Review

When asked to review code, a PR, or a diff, follow this process:

**Step 1 — Scope check:** Compare the changed files against the stated goal of the task. Flag files changed that seem unrelated (scope drift) and flag implied changes that are missing.

**Step 2 — Safety checklist:** Check the diff against each category below. For each issue found, report: severity, file + approximate line, what is wrong, and what the fix is.

**Step 3 — Angular-specific checks:** For Angular projects, additionally check for unsubscribed observables, missing OnPush change detection on async components, and lazy module boundary violations.

**Severity levels:** CRITICAL (security/data risk) → IMPORTANT (logic error/policy violation) → NIT (style/minor).

**Safety categories to check:**
- SQL: no string interpolation in queries; parameterized only
- XSS: no `innerHTML` or `bypassSecurityTrust*` with user data
- Auth: new endpoints have auth guards; JWT validated on every request
- Race conditions: observables use switchMap (not mergeMap) for cancellable operations; state mutations synchronized
- Secrets: no API keys, passwords, or internal URLs committed
- PII: sensitive fields not logged; masked in UI
- Audit: state-changing operations write audit log entries
- Types: switch statements on enums are exhaustive; discriminated unions fully handled
- Dependencies: new packages are vetted for license and known vulnerabilities

**Format each finding as:**
```
[SEVERITY] file.ts ~line42: One-line problem description.
Fix: Specific, actionable fix.
```

Do not auto-apply fixes without confirmation. Present CRITICAL and IMPORTANT findings first.
