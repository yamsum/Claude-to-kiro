# Review Checklist

<!-- Source: https://github.com/garrytan/gstack — review/checklist.md -->
<!-- Version: 1.0.0 -->

Reference checklist used by the review steering doc and Copilot instructions. Edit this file to tune what gets flagged in your project.

## Safety Categories

### SQL & Data Safety
- [ ] No raw string interpolation in SQL queries (use parameterized queries)
- [ ] No unvalidated user input passed to database operations
- [ ] Sensitive fields (passwords, tokens) not logged or returned in API responses
- [ ] PII not written to application logs

### Race Conditions
- [ ] Shared mutable state accessed with proper synchronization
- [ ] No TOCTOU (time-of-check/time-of-use) patterns in critical paths
- [ ] Observable subscriptions properly unsubscribed (Angular: takeUntil, async pipe)
- [ ] No duplicate API calls from parallel triggers (Angular: switchMap not mergeMap for navigation)

### Injection Vectors
- [ ] No shell injection via unsanitized input in Bash/exec calls
- [ ] No XSS via direct DOM manipulation (`innerHTML`, `bypassSecurityTrust*`)
- [ ] Angular template expressions do not eval() user content
- [ ] Template literals with user data properly escaped before rendering

### LLM Trust Boundaries (if AI features are present)
- [ ] Prompt content from untrusted sources is wrapped or sanitized
- [ ] LLM output is not executed as code without validation
- [ ] Token costs are bounded (no unbounded context accumulation)

### API & Auth
- [ ] New endpoints have authentication and authorization checks
- [ ] JWT tokens validated on every protected route (not just at login)
- [ ] CORS policy is explicit and restrictive
- [ ] HTTP security headers present (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)

### Enum & Type Completeness
- [ ] Switch statements on enums have a default or exhaustive case
- [ ] TypeScript discriminated unions are exhaustively handled
- [ ] New enum values added to all relevant switch/if chains

### Banking-Specific
- [ ] Audit log entries written for all state-changing operations
- [ ] Sensitive data (account numbers, PAN, SSN) masked in logs and UI displays
- [ ] localStorage/sessionStorage does not hold unencrypted sensitive data
- [ ] Money/currency arithmetic uses decimal-safe types (no floating point)
- [ ] External API calls timeout and have fallback behavior

## Angular-Specific Checks

- [ ] Components using async data implement `OnPush` change detection or async pipe
- [ ] No memory leaks: observables unsubscribed, intervals cleared, event listeners removed
- [ ] Lazy-loaded modules do not cross-import eager modules
- [ ] Route guards applied to all protected routes
- [ ] Environment-specific config (`environment.ts`) not hardcoded in components
- [ ] Angular reactive forms validate at both client and server

## CI/CD Checks

- [ ] Changes to `angular.json`, `tsconfig*.json`, or pipeline configs are intentional
- [ ] New dependencies vetted (license, security, size impact on bundle)
- [ ] No secrets or API keys committed (even in comments)
- [ ] Build output (`dist/`) not committed to source control
