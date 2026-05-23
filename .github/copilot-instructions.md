# Project AI Instructions (Claude-to-Kiro Repository)

This is a tooling repository that adapts Claude Code skills for Kiro and VS Code. It contains markdown-based steering documents, YAML hooks, and JSON task configurations.

When working in this repo, prefer editing existing skill files over creating new ones. Follow the patterns established in `docs/adaptation-methodology.md`.

---

## Code Review

When asked to review code, a PR, or a diff, follow this process:

**Step 1 — Scope check:** Compare changed files against the stated goal. Flag scope drift (unrelated changes) and missing implied changes.

**Step 2 — Safety checklist:** Check each category below. Report: severity, file + line, problem, fix.

**Severity:** CRITICAL (security/data risk) → IMPORTANT (logic error/policy) → NIT (style).

**Categories:**
- SQL: no string interpolation in queries; parameterized only
- XSS: no `innerHTML` or `bypassSecurityTrust*` with user data; Angular templates no `eval()`
- Auth: new endpoints have auth guards; JWT validated signature and expiry on every request
- Race conditions: observables use switchMap for cancellable operations; state mutations synchronized; subscriptions unsubscribed
- Secrets: no API keys, passwords, or internal URLs in source
- PII: sensitive fields not logged; masked in UI displays
- Audit: state-changing operations write audit log entries
- Types: switch on enums exhaustive; discriminated unions fully handled
- Dependencies: new packages vetted for license and CVEs

Format: `[SEVERITY] file.ts ~line: problem. Fix: specific action.`

---

## Security Review

When asked for a security audit or vulnerability check:

**OWASP Top 10 — Angular/Banking:**
- Access control: routes guarded, authorization server-side, no horizontal privilege escalation
- Injection: no `innerHTML`/`bypassSecurityTrust*` with user data; no `eval()`; SQL parameterized
- Cryptography: no unencrypted sensitive data in localStorage; no credentials in URLs; short-lived tokens
- Configuration: CORS explicit (not `*`); CSP without `unsafe-inline`; HSTS, X-Frame-Options, X-Content-Type-Options present
- Dependencies: `npm audit` no HIGH/CRITICAL; lock file committed
- Auth failures: session invalidated on logout; JWT sig+exp validated; account lockout present
- Logging: auth events, authorization failures, transactions logged; logs contain no passwords/PANs/SSNs

**Banking-specific:** PANs masked (last 4 only); financial arithmetic uses integer cents; transaction amounts validated server-side; audit trail complete.

**STRIDE for high-risk features:** Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Privilege Escalation.

Present CRITICAL and HIGH findings first, grouped by category.

---

## Skill Adaptation (when modifying this repo)

When creating or updating a skill adaptation:

1. Read the source skill's SKILL.md (linked in each README.md).
2. Strip Claude Code-specific tool calls (`Bash(...)`, `Read(...)`, `Agent(...)`). Replace with imperative natural language.
3. Steering docs: YAML frontmatter with `inclusion`, then markdown workflow steps.
4. Hooks: YAML only, with `name`, `description`, `on` (event list), optional `trigger` (command + paths), and `instructions`.
5. Do not port features that have no Kiro/VS Code equivalent (subagent dispatch, confidence scoring, GBrain, telemetry).
6. Add Angular/banking-specific checks where the source skill is generic.
7. Update `README.md` catalog table and `skills/README.md` status table.

---

## Markdown and YAML Quality

For markdown files: use ATX headings (`#`, `##`), fenced code blocks with language tags, and tables for structured data. No trailing whitespace.

For YAML files: 2-space indentation, quoted strings when they contain colons or special characters, no trailing whitespace.

Flag any YAML that fails `yamllint` basic checks.
