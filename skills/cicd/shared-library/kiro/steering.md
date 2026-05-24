---
inclusion: fileMatch
fileMatchPattern: "{**/vars/*.groovy,**/src/**/*.groovy}"
---
<!-- Source: https://github.com/yamsum/claude-to-kiro — cicd/shared-library skill -->
<!-- Version: 1.0.0 -->

# Shared Library Analysis

Triggered when working on Jenkins shared library code (`vars/` or `src/`). Use this before making changes to assess blast radius, and after changes to validate safe deployment.

## Step 1: Identify the Change Scope

Determine what is changing:
- **New step in `vars/`** — additive, low risk unless it conflicts with an existing name
- **Modification to existing step in `vars/`** — check all consumers
- **Change to `src/` class** — check which `vars/` steps import it, then check their consumers
- **Resource file change** — check which steps call `libraryResource('filename')`
- **New parameter added** — assess backward compatibility (is it optional with a default?)
- **Parameter removed or renamed** — breaking change; requires consumer migration

## Step 2: Find All Consumers

Search for all Jenkinsfiles calling the modified step:

```bash
# Find all consumers of a vars/ step named 'buildAngular'
grep -r "buildAngular(" pipelines/ jobs/ --include="Jenkinsfile" --include="*.groovy" -l

# Find all @Library declarations (shows which repos use this library at all)
grep -r "@Library" pipelines/ --include="Jenkinsfile" -h | sort | uniq

# Find all version pins to understand rollout blast radius
grep -r "@Library('mylib@" pipelines/ --include="Jenkinsfile" -h | sort | uniq
```

Report:
- Total number of consuming Jenkinsfiles
- Which ones are tag-pinned vs. branch-pinned
- Whether the consuming pipelines are production/staging/development (if categorizable)

## Step 3: Assess Version Pin Strategy

Shared libraries can be referenced in two ways:

```groovy
@Library('my-shared-lib@main') _            // branch-pinned: always gets latest
@Library('my-shared-lib@v1.3.2') _          // tag-pinned: frozen at this version
@Library('my-shared-lib@a1b2c3d') _         // commit-pinned: frozen at exact commit
```

**Impact by pin type:**

| Pin Type | Deployment Behavior | Risk |
|----------|-------------------|------|
| Branch (main/develop) | Change goes live immediately on next pipeline run | Highest — all consumers affected simultaneously |
| Tag | Consumers must update their `@Library` reference to see the change | Low — staged rollout possible |
| Commit SHA | Same as tag — consumers must update | Low |
| No pin (uses Jenkins global default) | Change goes live immediately based on Jenkins admin config | High — often hidden |

Report all consumers grouped by pin type. Highlight branch-pinned consumers as highest risk.

## Step 4: Backward Compatibility Check

For modifications to existing steps, verify:

- [ ] All existing parameters still accepted (new optional params only — never remove or rename)
- [ ] Return type unchanged (if the step returns a value and callers use it)
- [ ] Behavior unchanged for existing inputs (no silent behavior changes)
- [ ] If the step creates files/directories: same paths, same names

For breaking changes, document a migration plan:
1. Create the new behavior under a new step name (or new parameter)
2. Keep the old behavior with a deprecation warning (`echo "WARNING: old param deprecated..."`)
3. Notify consumers via commit message and changelog
4. Remove old behavior only after all consumers have migrated

## Step 5: Testing Coverage

Check for tests in `src/test/groovy/` (JenkinsPipelineUnit):

```bash
ls src/test/groovy/
./gradlew test           # run tests
./gradlew test --tests "*MyStepSpec*"  # run specific test class
```

If the modified step has no tests, a test should be written before the change. A minimal JenkinsPipelineUnit test:

```groovy
class BuildAngularSpec extends Specification {
    def "buildAngular calls ng build with correct args"() {
        given:
        def script = loadScript('vars/buildAngular.groovy')
        
        when:
        script.call(configuration: 'production')
        
        then:
        // Verify the sh() call was made with expected args
        assertJobStatusSuccess()
    }
}
```

If no test framework is configured, note it as a gap and suggest setup steps.

## Step 6: Safe Deployment Plan

Based on the blast radius and pin strategy, recommend a deployment approach:

**For branch-pinned consumers (high blast radius):**
1. Create a feature branch in the shared library repo
2. Update one non-production consumer's `@Library` pin to the feature branch: `@Library('mylib@feat/my-change')`
3. Validate in that pipeline
4. Merge to main/develop — all branch-pinned consumers now get the change
5. Monitor for failures across consumers for 24h

**For tag-pinned consumers (staged rollout possible):**
1. Merge change to main/develop
2. Create a new semantic version tag: `git tag v1.3.3 && git push origin v1.3.3`
3. Update consumers one-by-one or team-by-team
4. Old tag-pinned consumers are unaffected until they choose to update

**For shared library with no tests (high risk):**
1. Add at least one smoke test before any change
2. Consider requiring `-DskipTests=false` or `./gradlew test` in the shared library's own pipeline

## Step 7: Report

Produce a summary:

```
Shared Library Change Report
============================
Modified: vars/buildAngular.groovy
Change type: [new step|modified step|new parameter|breaking change]

Consumers: N Jenkinsfiles
  - Branch-pinned (immediate rollout): [list or count]
  - Tag-pinned (staged rollout): [list or count]
  - Unknown/no @Library pin: [count]

Backward compatible: [yes|no|partial — explain]
Tests exist: [yes|no — describe gap]
Breaking change migration plan: [if applicable]

Recommended deployment: [branch-pin staged test|tag release|immediate]
Estimated blast radius: [HIGH|MEDIUM|LOW]
```
