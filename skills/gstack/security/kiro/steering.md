---
inclusion: manual
---
<!-- Source: https://github.com/garrytan/gstack — cso/SKILL.md -->
<!-- Version: 1.0.0 -->

# Security Audit

Triggered when the user asks for a security audit, vulnerability review, or threat assessment. Covers OWASP Top 10 and STRIDE threat modeling with Angular/banking-specific extensions.

## Scope

Before starting, identify the audit scope:
- Full codebase audit: scan all TypeScript, HTML, SCSS, and config files
- Diff audit: scan only changed files (`git diff $(git merge-base HEAD origin/main) HEAD`)
- Targeted: focus on files the user specifies

## OWASP Top 10 Checklist

### A01 — Broken Access Control
- [ ] All routes protected by auth guards (`canActivate`, `canActivateChild`)
- [ ] API calls include authorization headers (checked in HTTP interceptors)
- [ ] Role-based access enforced server-side, not just in UI routing
- [ ] Direct object references validated (user cannot access other users' data by changing IDs)
- [ ] Horizontal privilege escalation: can user A act on user B's records?

### A02 — Cryptographic Failures
- [ ] No sensitive data in localStorage or sessionStorage unencrypted
- [ ] No passwords or tokens in URLs (query params, hash)
- [ ] HTTPS enforced; HTTP Strict Transport Security (HSTS) header present
- [ ] Tokens have short expiry; refresh logic implemented correctly
- [ ] No MD5 or SHA1 used for security-relevant hashing

### A03 — Injection
- [ ] No `innerHTML` assignment with user-controlled data
- [ ] `DomSanitizer.bypassSecurityTrust*` not called with unsanitized user input
- [ ] Angular template expressions do not use `eval()` or `Function()`
- [ ] SQL queries parameterized (if any direct DB access in BFF/server)
- [ ] Shell commands not constructed from user input

### A04 — Insecure Design
- [ ] Threat model exists for high-risk features (payments, auth, admin functions)
- [ ] Business logic errors: can a user skip a required workflow step?
- [ ] Rate limiting on sensitive operations (login, OTP, money transfer)
- [ ] Negative amounts, zero amounts handled explicitly in financial operations

### A05 — Security Misconfiguration
- [ ] CORS policy explicit and restrictive (not `*`)
- [ ] CSP header configured; `unsafe-inline` not used in script-src
- [ ] `X-Frame-Options: DENY` or `SAMEORIGIN`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] No debug/admin endpoints exposed in production build
- [ ] `angular.json` build config uses `optimization: true` and `aot: true` for production

### A06 — Vulnerable and Outdated Components
- [ ] Run `npm audit` — report HIGH and CRITICAL findings
- [ ] Check that `package-lock.json` or `yarn.lock` is committed and up to date
- [ ] No packages with known CVEs pinned to vulnerable versions
- [ ] Third-party scripts loaded via CDN have integrity hashes (SRI)

### A07 — Identification and Authentication Failures
- [ ] Session tokens invalidated on logout (both client and server)
- [ ] JWT signature validated on every request (not just decoded)
- [ ] JWT expiry (`exp`) checked; clock skew tolerance is reasonable (< 5 min)
- [ ] Multi-factor authentication enforced for admin and high-privilege actions
- [ ] Account lockout after repeated failed logins

### A08 — Software and Data Integrity Failures
- [ ] CI/CD pipeline uses pinned dependency versions (no `latest` in production)
- [ ] Build artifacts signed or checksummed before deployment
- [ ] No untrusted data deserialized without schema validation
- [ ] Angular service worker (if used) configured to not cache sensitive responses

### A09 — Security Logging and Monitoring Failures
- [ ] Authentication events logged (success, failure, logout)
- [ ] Authorization failures logged (with user ID, resource, action)
- [ ] Financial transactions have full audit trail (before/after state, actor, timestamp)
- [ ] Logs do not contain sensitive data (passwords, full card numbers, SSN)
- [ ] Log format includes enough context for incident response

### A10 — Server-Side Request Forgery
- [ ] URLs constructed from user input are validated against an allowlist
- [ ] HTTP client not used to fetch user-supplied URLs without validation
- [ ] Redirects validate the target URL is within expected domains

## STRIDE Threat Model

For each high-risk feature in scope, evaluate:

| Threat | Question |
|--------|---------|
| Spoofing | Can an attacker impersonate a legitimate user or service? |
| Tampering | Can data be modified in transit or at rest without detection? |
| Repudiation | Can a user deny performing an action? Is there an audit trail? |
| Information Disclosure | Can sensitive data leak to unauthorized parties? |
| Denial of Service | Can an attacker make the system unavailable? |
| Elevation of Privilege | Can a lower-privilege user perform higher-privilege actions? |

## Banking-Specific Checks

- [ ] Account numbers, card PANs, SSNs masked in all UI displays (show last 4 only)
- [ ] Financial arithmetic uses integer cents or a decimal library — no floating point
- [ ] Transaction amounts have server-side validation (cannot be altered client-side)
- [ ] Regulatory data (transaction history, consent records) retained per policy
- [ ] Third-party analytics/tracking scripts do not receive financial data

## Output Format

Group findings by OWASP category. For each finding:

```
[CRITICAL|HIGH|MEDIUM|LOW] A03 — Injection
File: src/app/transfer/transfer.component.ts ~line 87
Issue: innerHTML set from user-provided account name without sanitization.
Fix: Use Angular's text binding `{{ accountName }}` instead of [innerHTML], or
     call DomSanitizer.sanitize(SecurityContext.HTML, value) before binding.
```

Present CRITICAL and HIGH findings first. Conclude with a summary table: category, findings count, highest severity.

Do not modify files during a security audit. Report findings only.
