---
inclusion: manual
---
<!-- Source: https://github.com/yamsum/claude-to-kiro — cicd/pipeline-debug skill -->
<!-- Version: 1.0.0 -->

# Pipeline Failure Diagnosis

Triggered when the user asks to debug or diagnose a failing Jenkins or GitLab CI pipeline. Always start with the build log — do not look at source files until you know the failure class.

## Phase 1: Classify the Failure

Read the build log excerpt the user provides. Identify the failure class before anything else — this determines what to look at next.

### Jenkins Failure Classes

| Error Pattern in Log | Class | Next Step |
|---|---|---|
| `org.codehaus.groovy.control.MultipleCompilationErrorsException` | Groovy syntax error | Read the Jenkinsfile/groovy file |
| `hudson.remoting.ProxyException: groovy.lang.MissingMethodException` | Called a non-existent method | Check shared library version, method name |
| `org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException` | Sandbox restriction | Identify blocked method; needs Script Approval |
| `NotSerializableException` or `java.io.NotSerializableException` | CPS serialization failure | Find non-serializable object; apply @NonCPS |
| `hudson.AbortException: script returned exit code N` | Shell command failed | Check the `sh` step output above the error |
| `java.lang.InterruptedException` or `FlowInterruptedException` | Timeout or user abort | Check timeout settings and build duration |
| `No agent is available` or `Waiting for next available executor` | Agent provisioning failure | Check node labels, agent pool availability |
| `Could not get unknown property '...'` | Missing env var or credential | Check credentials() binding and env vars |
| `org.codehaus.groovy.runtime.typehandling.GroovyCastException` | Type mismatch | Check return types from steps |
| `FATAL: Could not find 'Jenkinsfile'` | Wrong branch or path | Check SCM config |

### GitLab CI Failure Classes

| Error Pattern | Class | Next Step |
|---|---|---|
| `ERROR: Job failed: exit code N` | Script error in job | Check job `script:` block |
| `ERROR: Job failed (system failure)` | Runner infrastructure issue | Not a pipeline code problem; check runner health |
| `This job is stuck` | No runner with matching tags | Check `tags:` in job vs. registered runner tags |
| `Preparing the "docker" executor` hangs | Docker executor issue | Check runner's Docker daemon |
| `ERROR: Downloading artifacts from coordinator... 403` | Artifact permission issue | Check `artifacts:` scope and expiry |
| `yaml: line N: ...` | YAML syntax error | Lint the YAML |
| `jobs:job-name:needs` unknown job | `needs:` references nonexistent job | Check job names and stage order |
| `could not resolve variable '$VAR'` | Variable not defined for this context | Check protected/masked variable visibility |

## Phase 2: Gather Context

Based on the failure class, gather exactly what you need — do not read everything:

**For Groovy errors:** Read the specific file and line mentioned in the stack trace.

**For agent/executor issues:** Check the job's `agent {}` or `node {}` label against what the user tells you is available. Do not look at source files.

**For credential issues:** Check the `withCredentials()` block in the Jenkinsfile. Ask the user to confirm the credential ID exists in Jenkins credentials store — you cannot verify this from source.

**For shell exit code failures:** Read the `sh()` step that failed and the command it runs. Also check: does the command depend on a file that might not exist? Does it assume a specific working directory?

**For GitLab variable issues:** Check whether the variable is marked Protected — protected variables are not passed to pipelines triggered from unprotected branches or MR pipelines.

## Phase 3: Reproduce (or Explain Why You Cannot)

State clearly whether this failure can be reproduced locally:

**Can reproduce locally:**
- Groovy syntax errors: `groovy -c Jenkinsfile`
- Declarative lint: `java -jar jenkins-cli.jar declarative-linter < Jenkinsfile`
- GitLab YAML lint: `gitlab-runner exec shell <job-name>` or API lint endpoint
- Shell script failures: copy and run the `sh()` content in a local shell
- Shared library unit tests: `./gradlew test` (if JenkinsPipelineUnit is configured)

**Cannot reproduce locally (explain this to the user):**
- Agent availability issues — requires the live Jenkins environment
- Credential binding — credentials exist only in Jenkins/GitLab credential store
- Plugin-specific behavior — requires the exact plugin version on the server
- Race conditions in parallel stages — environment-dependent

## Phase 4: Identify Root Cause

State the root cause in one sentence: "The pipeline fails because `<X>` causes `<Y>` when `<Z>`."

Then explain:
- Why this happens (the mechanism)
- Why it happens now and not before (if the failure is new)
- Whether this is a pipeline code bug, a configuration issue, or an infrastructure issue

## Phase 5: Fix

Apply the minimal fix:
- For Jenkinsfile/shared library changes: edit the file, explain the change
- For configuration issues (credential IDs, node labels): tell the user exactly what to change in Jenkins/GitLab UI
- For infrastructure issues: escalate — this is outside pipeline code scope

Do not commit or push. Present the fix and let the user validate.

## Phase 6: Prevention

After fixing, note any prevention measures:
- Should this be caught by declarative linter in CI? (`java -jar jenkins-cli.jar declarative-linter`)
- Should a shared library test cover this case?
- Should the pipeline have a shorter timeout to fail faster?
- Is this a class of failure that the Groovy review skill should check for?

## Common Fixes Reference

**CPS serialization fix:**
```groovy
// Add @NonCPS annotation to any method using non-serializable operations
@NonCPS
def parseConfig(String json) {
    return new groovy.json.JsonSlurperClassic().parseText(json)
}
```

**Credential exposure fix:**
```groovy
// Replace GString interpolation with single-quoted string inside binding
withCredentials([string(credentialsId: 'my-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
}
```

**Timeout fix:**
```groovy
// Declarative
options {
    timeout(time: 30, unit: 'MINUTES')
}
// Scripted
timeout(time: 30, unit: 'MINUTES') {
    node('linux') { ... }
}
```

**JsonSlurper serialization fix:**
```groovy
// Replace JsonSlurper with JsonSlurperClassic
import groovy.json.JsonSlurperClassic
def data = new JsonSlurperClassic().parseText(readFile('config.json'))
```

**GitLab protected variable fix:**
Uncheck "Protected" in CI/CD → Variables if the variable must be available in MR pipelines or unprotected branches.
