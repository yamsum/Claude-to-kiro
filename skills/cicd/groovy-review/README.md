# Groovy Review Skill

**Source:** Original — no direct Claude Code equivalent. Authored for this repo.
**Covers:** Jenkinsfile (declarative and scripted), `vars/*.groovy`, `src/org/**/*.groovy`

## Why Groovy Review is Separate from Generic Code Review

The general `gstack/review` skill checks for universal concerns (SQL, XSS, auth). Jenkins Groovy has an entirely different failure class driven by:

- **CPS (Continuation Passing Style)**: The Jenkins pipeline engine transforms Groovy bytecode into a resumable state machine. Certain Groovy/Java constructs break this transformation silently or with cryptic errors.
- **Sandbox restrictions**: Untrusted pipelines run in a Groovy sandbox. Calling unapproved methods causes `RejectedAccessException` at runtime, not compile time.
- **Credential exposure**: Groovy string interpolation (`"${env.SECRET}"`) writes secrets to the build log. The `credentials()` binding prevents this, but only if used correctly.
- **Serialization across stage boundaries**: Objects passed between stages must be serializable. Groovy closures, Iterators, and many Java objects are not.
- **Scripted vs. declarative context**: Scripted pipelines have different scoping rules than declarative; mixing them causes unexpected behavior.

## What Was Built

| Component | Purpose |
|-----------|---------|
| `kiro/steering.md` | Full review workflow for Jenkinsfiles and shared library Groovy |
| `kiro/hook.yaml` | Auto-trigger review when `.groovy` or `Jenkinsfile` files change |
| `vscode/copilot-instructions.md` | Condensed Copilot context for inline Groovy review |

## Usage

```bash
cp skills/cicd/groovy-review/kiro/steering.md .kiro/steering/40-groovy-review.md
cp skills/cicd/groovy-review/kiro/hook.yaml .kiro/hooks/groovy-review.yaml
```

Invoke in Kiro: "review this Jenkinsfile", "check this shared library change", "groovy review".
