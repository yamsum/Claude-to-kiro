# Project AI Instructions (Claude-to-Kiro Repository)

This is a tooling repository that adapts Claude Code skills for Kiro and VS Code. It contains markdown-based steering documents, YAML hooks, and JSON task configurations.

When working in this repo, prefer editing existing skill files over creating new ones. Follow the patterns in `docs/adaptation-methodology.md`. Use `#codebase` (not `@workspace`) to search across all roots in a multi-root workspace.

---

## Code Review

When asked to review code, a PR, or a diff:

**Step 1 — Scope check:** Compare changed files against the stated goal. Flag scope drift (unrelated changes) and missing implied changes.

**Step 2 — Safety checklist:** For each category below, report: `[SEVERITY] file ~line: problem. Fix: action.`

**Severity:** CRITICAL (security/data risk) → IMPORTANT (logic error/policy) → NIT (style).

**General categories:**
- SQL: parameterized queries only; no string interpolation
- XSS: no `innerHTML`/`bypassSecurityTrust*` with user data
- Auth: new endpoints guarded; JWT sig + expiry validated on every request
- Race conditions: observables use switchMap; subscriptions unsubscribed
- Secrets: no API keys, passwords, or internal URLs in source
- PII: sensitive fields not logged; masked in UI
- Types: switch on enums exhaustive; discriminated unions fully handled

---

## Jenkins Groovy Review

When reviewing Jenkinsfiles or shared library Groovy (`.groovy`, `Jenkinsfile`):

**CPS violations (runtime failures):** Methods using `.stream()`, `.filter()`, `.map()`, `.collect()`, or Groovy closures passed to non-pipeline methods must be annotated `@NonCPS`. Flag every occurrence without this annotation.

**Credential exposure (CRITICAL):** Any `"${env.VARIABLE}"` or `"${params.ANYTHING}"` inside `sh()` or string concatenation writes secrets to the build log. Must use `withCredentials([...])` binding with **single-quoted** strings in `sh()`.

**Sandbox blocks:** Flag `System.*`, `Runtime.exec()`, `new File(...)`, reflection calls, `Thread`, or unapproved third-party imports. These cause `RejectedAccessException` at runtime in untrusted pipelines.

**Serialization:** Top-level pipeline variables must be serializable. Flag `new groovy.json.JsonSlurper()` (use `JsonSlurperClassic`), Groovy closures at pipeline scope, and custom classes without `implements Serializable`.

**Declarative structure:** Every pipeline needs `timeout()` in `options {}`. No `node {}` inside `pipeline {}`. Parallel stages must not write to shared paths.

**Shared library blast radius:** For `vars/` or `src/` changes — report how many pipelines consume this step, whether the change is backward-compatible, and whether consumers are tag-pinned or branch-pinned.

---

## GitLab CI Review

When reviewing `.gitlab-ci.yml` or GitLab CI template files:

**Stage/job validity:** Every job's `stage:` must be in the `stages:` list. No duplicate job names. Every `needs:` must reference a real, earlier-stage job. `needs:` pointing to a `when: manual` job causes the downstream job to wait indefinitely.

**DAG (`needs:`):** `needs: []` ignores stage ordering and runs immediately. `needs: [job]` with `artifacts: true` (default) requires that job to define `artifacts:` or downstream receives nothing silently.

**Deprecated syntax:** Flag all `only:/except:` and provide the `rules:` equivalent. They cannot coexist in one job. First matching `rules:` entry wins — unreachable rules below a catch-all are silently ignored.

**Variable security:** No plaintext secrets in `variables:`. Protected variables are unavailable to MR pipelines and unprotected branches — flag if a protected variable is expected there. Masked variables only mask if the value is base64-only, 8+ chars, no whitespace.

**Artifacts and cache:** `cache:key:` must be defined to avoid cross-branch pollution. `artifacts:expire_in:` must be set. `artifacts:when: always` needed to capture failure evidence.

**Images and runners:** Images pinned to version tag or digest — not `latest`. Runner `tags:` are case-sensitive.

Format: `[CRITICAL|IMPORTANT|NIT] job-name: problem. Fix: specific YAML.`

---

## Security Review

When asked for a security audit or vulnerability check:

**OWASP Top 10 — Angular/Banking:**
- Access control: routes guarded, authorization server-side, no horizontal privilege escalation
- Injection: no `innerHTML`/`bypassSecurityTrust*`; no `eval()`; SQL parameterized
- Cryptography: no unencrypted sensitive data in localStorage; no credentials in URLs; short-lived tokens
- Configuration: CORS explicit (not `*`); CSP without `unsafe-inline`; HSTS, X-Frame-Options, X-Content-Type-Options present
- Dependencies: `npm audit` no HIGH/CRITICAL; lock file committed
- Auth failures: session invalidated on logout; JWT sig+exp validated; account lockout present
- Logging: auth events, authorization failures, transactions logged; logs contain no passwords/PANs/SSNs

**Banking-specific:** PANs masked (last 4 only); financial arithmetic uses integer cents; transaction amounts validated server-side; audit trail complete.

**CI/CD pipeline security:** Credentials only via binding (never interpolated). No shell injection via build parameters. Agent labels restricted to approved pools. No `@Library` from unreviewed external sources.

---

## Skill Adaptation (when modifying this repo)

When creating or updating a skill adaptation:

1. Read the source SKILL.md (linked in each README.md).
2. Strip Claude Code-specific tool calls. Replace with imperative natural language.
3. Steering docs: YAML frontmatter with `inclusion`, then markdown workflow steps.
4. Hooks: YAML only — `name`, `description`, `on`, optional `trigger`, `instructions`.
5. Do not port: subagent dispatch, confidence scoring, GBrain, telemetry.
6. Add CI/CD-specific checks for Groovy/YAML files; add banking-specific checks for security.
7. Update `README.md` catalog and `skills/README.md` status table.
