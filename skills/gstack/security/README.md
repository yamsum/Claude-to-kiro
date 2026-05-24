# Security Audit Skill Adaptation

**Source:** [garrytan/gstack](https://github.com/garrytan/gstack) — `/cso` skill (MIT)
**Original invocation:** `/cso` in Claude Code

## What the Security Skill Does

A comprehensive security audit covering OWASP Top 10 and STRIDE threat modeling. Reviews the codebase for vulnerabilities, produces a prioritized findings report, and suggests remediation.

## What Was Adapted

| gstack Feature | Kiro Adaptation |
|----------------|-----------------|
| OWASP Top 10 audit | `steering.md`: Full OWASP checklist adapted for Angular/TypeScript |
| STRIDE threat model | `steering.md`: Threat model section |
| Parallel specialist agents | Sequential checklist |
| Auto-fix suggestions | Retained: specific, actionable fixes per finding |
| Banking-specific checks | Added: PII, audit trails, financial arithmetic safety |

## What Was Added (Angular/Banking)

This adaptation extends the base gstack CSO skill with checks specific to Angular front-end applications in regulated financial environments:

- Angular security model (DomSanitizer, CSP, template injection)
- HTTP interceptor security (auth header injection, response sanitization)
- Financial data handling (decimal precision, masking, tokenization)
- Regulatory compliance signals (GDPR data exposure, PCI-DSS card data)
- CI/CD supply chain (dependency audit, build artifact integrity)

## Usage

### Kiro
```bash
cp skills/gstack/security/kiro/steering.md .kiro/steering/31-security.md
```

Invoke by telling Kiro: "security audit", "run the security review", or "check for vulnerabilities".

### VS Code (Copilot)
Append `skills/gstack/security/vscode/copilot-instructions.md` to your `.github/copilot-instructions.md`.
