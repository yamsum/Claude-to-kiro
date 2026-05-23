## Security Review

When asked to perform a security audit or check for vulnerabilities:

**Run through these categories in order. For each issue, report: severity (CRITICAL/HIGH/MEDIUM/LOW), file + line, what is wrong, and the specific fix.**

**Access Control:** All routes have auth guards. Authorization enforced server-side. Users cannot access other users' data by changing IDs in requests.

**Injection:** No `innerHTML` or `bypassSecurityTrust*` with user data. Angular templates do not use `eval()`. SQL queries parameterized.

**Cryptography:** No sensitive data in localStorage unencrypted. No credentials in URLs. HTTPS enforced. Tokens short-lived with proper refresh.

**Configuration:** CORS policy is explicit (not `*`). CSP header configured without `unsafe-inline`. Security headers present: HSTS, X-Frame-Options, X-Content-Type-Options.

**Dependencies:** `npm audit` shows no HIGH or CRITICAL issues. `package-lock.json` committed and up to date.

**Auth failures:** Session tokens invalidated on logout (client and server). JWT signature and expiry validated on every request. Account lockout after repeated failures.

**Audit logging:** Auth events, authorization failures, and financial transactions logged with full context. Logs do not contain passwords, card numbers, or SSNs.

**Banking-specific:** Account numbers and PANs masked in UI (last 4 digits only). Financial arithmetic uses integer cents — no floating point. Transaction amounts validated server-side.

**STRIDE for high-risk features:** Can an attacker spoof a user? Tamper with data in transit? Deny performing an action? Access information they shouldn't? Exhaust resources? Elevate privileges?

Present findings grouped by category, CRITICAL first.
