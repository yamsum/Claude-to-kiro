---
inclusion: fileMatch
fileMatchPattern: "{**/Jenkinsfile,**/Jenkinsfile.*,**/*.groovy}"
---
<!-- Source: https://github.com/yamsum/claude-to-kiro — cicd/groovy-review skill -->
<!-- Version: 1.0.0 -->

# Jenkins Groovy Review

Triggered when reviewing Jenkinsfiles or shared library Groovy. Covers CPS safety, credential exposure, sandbox restrictions, serialization, and pipeline structure. Run after the general `gstack/review` checklist.

## Step 1: Identify File Type

Determine what you're reviewing — this changes which checks apply:

| File Pattern | Type | Key Risks |
|---|---|---|
| `Jenkinsfile` | Declarative or scripted pipeline | All categories below |
| `vars/*.groovy` | Shared library step | CPS, sandbox, naming |
| `src/**/*.groovy` | Shared library class | Serialization, imports |
| `*.Jenkinsfile` | Named pipeline | Same as Jenkinsfile |

## Step 2: CPS (Continuation Passing Style) Check

The Jenkins pipeline engine rewrites pipeline code into a resumable state machine. Certain Groovy constructs break this.

**Patterns that MUST have `@NonCPS`:**
- Methods using Java streams: `.stream()`, `.filter()`, `.map()`, `.collect()`
- Methods using Groovy closures passed to non-pipeline methods
- Methods using `Iterator` directly
- Complex `for` loops over non-list types

**CPS-safe alternatives:**
```groovy
// BAD — breaks CPS
def names = items.stream().filter { it.active }.collect { it.name }

// GOOD — CPS-safe, or annotate the enclosing method with @NonCPS
@NonCPS
def filterActiveNames(items) {
    return items.stream().filter { it.active }.collect { it.name }
}
```

**Flag each violation as:**
```
[CRITICAL] CPS Violation
File: vars/myStep.groovy ~line 42
Problem: .stream().collect() used in CPS context — will fail at runtime with
         NotSerializableException or CpsCallableInvocation
Fix: Extract into a @NonCPS-annotated helper method
```

## Step 3: Credential and Secret Exposure

**CRITICAL: String interpolation of secrets**

```groovy
// BAD — secret written to build log
sh "curl -H 'Authorization: Bearer ${env.API_TOKEN}' ..."
sh "echo ${env.DB_PASSWORD} | psql ..."

// GOOD — credentials() binding keeps secrets out of logs
withCredentials([string(credentialsId: 'api-token', variable: 'API_TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $API_TOKEN" ...'  // single quotes!
}
```

Check for:
- `"${env.ANYTHING}"` inside `sh()`, `bat()`, or string concatenation — flag every instance
- `env.ANYTHING` used directly in a GString in any command context
- `withCredentials` blocks that use double-quoted strings inside `sh()` — the binding is correct but the interpolation still leaks
- Credentials stored in `params.ANYTHING` passed to shell commands
- Passwords, tokens, or keys hardcoded as literal strings

## Step 4: Sandbox Restrictions

Untrusted pipelines run in a Groovy sandbox. Unapproved method calls cause `RejectedAccessException` at runtime.

Check for calls to:
- `System.*` — almost always sandbox-blocked
- `Runtime.exec()`, `ProcessBuilder` — blocked; use `sh()` step instead
- Reflection: `getClass()`, `getDeclaredMethod()`, `.invoke()`
- File I/O outside of `readFile`/`writeFile` steps: `new File(...)`, `FileInputStream`
- `Thread`, `Executor`, `CompletableFuture` — not allowed in sandbox
- Third-party library imports not on the approved classpath

If any of these appear in a **trusted** shared library (`src/` directory), they may be allowed — but flag them for review with the Jenkins admin to confirm the library is on the approved classpath.

## Step 5: Serialization

Objects assigned to variables at the top-level pipeline scope must be Java-serializable (they are checkpointed between stages).

**Non-serializable types frequently misused:**
- Groovy closures assigned to pipeline-scope variables
- `JsonSlurper` results (use `JsonSlurperClassic` instead — it returns serializable maps)
- `XmlSlurper` / `XmlParser` results
- Custom Groovy objects without `implements Serializable`

```groovy
// BAD — JsonSlurper result not serializable across stage boundary
def config = new groovy.json.JsonSlurper().parseText(readFile('config.json'))

// GOOD — JsonSlurperClassic returns serializable LinkedHashMap
def config = new groovy.json.JsonSlurperClassic().parseText(readFile('config.json'))
```

## Step 6: Declarative Pipeline Structure

For declarative pipelines (`pipeline { ... }`):

- [ ] `agent` defined at top level or per stage; not missing
- [ ] `environment {}` block used for env var declarations, not variable assignment in `script {}`
- [ ] `options {}` includes `timeout(time: N, unit: 'MINUTES')` — no timeout = runaway pipeline
- [ ] `post { always { ... } }` block cleans up workspace and sends notification
- [ ] `when { ... }` conditions on stages use supported directives (not arbitrary Groovy)
- [ ] No `node {}` blocks inside `pipeline {}` — use `agent` instead
- [ ] `parallel {}` stages have independent workspaces if they write files

## Step 7: Scripted Pipeline Structure

For scripted pipelines (`node { ... }`):

- [ ] `try/catch/finally` wraps the main body to ensure cleanup always runs
- [ ] `timeout()` wraps the `node {}` block
- [ ] `parallel()` calls handle exceptions per-branch (exceptions in one branch don't silently suppress others)
- [ ] `stash`/`unstash` used for cross-node file transfer (not shared filesystem paths)

## Step 8: Shared Library Specific

For `vars/*.groovy` (global step definitions):

- [ ] The file defines a `call()` method — that's what `myStep(args)` calls
- [ ] No `def` at file scope outside a method — file-scope vars are not CPS-safe
- [ ] Method name matches the filename (convention: `buildAngular.groovy` defines `buildAngular()`)
- [ ] No circular imports between shared library files

For `src/**/*.groovy` (shared library classes):

- [ ] Class implements `Serializable` if instances are passed across stage boundaries
- [ ] Constructor does not call pipeline steps (`sh`, `echo`, `readFile`) — those only work in step context
- [ ] `@Grab` annotations not used (blocked in Jenkins class loader)

## Step 9: Blast Radius Assessment

Before finishing, assess impact:

1. Is this change to a `vars/` step or `src/` class? If yes: how many pipelines call it? (`grep -r "myStep(" pipelines/`)
2. Is the change backward-compatible? (New optional params, not removed params)
3. Does the shared library have a version pin in consuming Jenkinsfiles? (`@Library('mylib@v1.2')`) If pinned to a tag, changes only affect new tags. If pinned to a branch, all consumers are affected immediately.
4. Are there tests? (`src/test/` with JenkinsPipelineUnit)

Report blast radius as:
```
Blast Radius: [HIGH|MEDIUM|LOW]
Consumers: [list or "unknown — grep required"]
Backward-compatible: [yes|no|partial — explain]
Version pin: [tag-pinned|branch-pinned|none]
Tests: [yes|no]
```

## Output Format

```
[CRITICAL|IMPORTANT|NIT] Category
File: path/to/file.groovy ~line N
Problem: What is wrong and why it will fail
Fix: Specific fix with corrected code snippet if applicable
```

Present CRITICAL (runtime failures) first, then IMPORTANT (security/correctness), then NITs.
End with the Blast Radius Assessment.
