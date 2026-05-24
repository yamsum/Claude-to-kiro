---
inclusion: fileMatch
fileMatchPattern: "**/.gitlab-ci.yml"
---
<!-- Source: https://github.com/yamsum/claude-to-kiro — cicd/gitlab-ci skill -->
<!-- Version: 1.0.0 -->

# GitLab CI Review and Validation

Triggered when reviewing `.gitlab-ci.yml` files or GitLab CI template includes. Covers YAML structure, job DAG correctness, variable security, artifact/cache configuration, and deprecated syntax.

## Step 1: YAML Validity

Before anything else, verify the YAML is syntactically valid. If the user can run the API lint:
```bash
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "$GITLAB_URL/api/v4/ci/lint" \
  --form "content=$(cat .gitlab-ci.yml)" | jq '.valid, .errors'
```

Parse the YAML structure manually: check for tabs (GitLab CI requires spaces), duplicate keys, and incorrect indentation in multi-line strings.

## Step 2: Stages and Job Definition

- [ ] `stages:` list defined at top level
- [ ] Every job's `stage:` value exists in the `stages:` list — jobs with undefined stages silently default to `test` stage (GitLab 15+) or fail (older versions)
- [ ] No duplicate job names — later definitions silently override earlier ones
- [ ] `workflow:` rules defined if the pipeline should not run on every push

## Step 3: Job DAG (`needs:`)

`needs:` creates a directed acyclic graph, bypassing stage ordering. Mistakes cause jobs to run prematurely, be skipped, or deadlock.

- [ ] Every job listed in `needs:` exists in the pipeline
- [ ] `needs:` jobs are in earlier stages or the same stage — forward `needs:` requires `needs: [job, pipeline: true]`
- [ ] `needs:` with `artifacts: true` (default) — the upstream job must define `artifacts:` or downloads will silently produce empty results
- [ ] No circular `needs:` chains
- [ ] If `needs: []` (empty array), the job runs immediately in the first stage regardless of defined `stage:`

Flag any `needs:` referencing a job that has `when: manual` — manual jobs are not triggered automatically, so the downstream job may wait indefinitely.

## Step 4: Rules vs. Only/Except

`only:/except:` is deprecated. Flag any usage and provide the `rules:` equivalent.

```yaml
# DEPRECATED
only:
  - main
  - /^release-.*/

# PREFERRED
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
  - if: '$CI_COMMIT_BRANCH =~ /^release-.*/'
```

Common `rules:` mistakes:
- `rules:` and `only:/except:` cannot be mixed in the same job — the job silently uses `rules:` only
- `rules: [if: '...']` without a `when:` defaults to `when: on_success` — be explicit
- `changes:` in rules only works correctly for MR pipelines; on branch pipelines it compares against the previous commit
- Order matters: first matching rule wins; unreachable rules below a catch-all are silently ignored

## Step 5: Variables Security

- [ ] No plaintext secrets in `variables:` — all secrets via CI/CD → Variables (masked + protected)
- [ ] `variables:` at job level override group/project/instance variables — check for accidental overrides
- [ ] `GIT_STRATEGY: none` set for jobs that don't need the source code (speeds up agent-only jobs)
- [ ] Protected variables: available only in protected branches/tags and pipelines run against them. Flag if a variable is expected in MR pipelines but marked Protected.
- [ ] Masked variables: only masked if value matches the masking format (base64-only, min 8 chars, no whitespace). Check if the value would actually be masked.
- [ ] `CI_JOB_TOKEN` scope: by default allows access to all projects. If restricting, ensure `allowlist` is configured.

## Step 6: Artifacts and Cache

**Artifacts** — files passed between jobs and downloadable from the UI:
- [ ] `artifacts:paths:` defined for jobs whose output other jobs need via `needs:`
- [ ] `artifacts:expire_in:` set — without it, artifacts use the instance default (often 30 days); large artifacts waste storage
- [ ] `artifacts:when: always` used when you need artifacts even on failure (logs, test reports)
- [ ] `reports:junit:` path matches actual test output location

**Cache** — build acceleration across pipeline runs:
- [ ] `cache:key:` defined; without it all jobs share one cache (causes cross-branch pollution)
- [ ] `cache:key: $CI_COMMIT_REF_SLUG` for branch-isolated cache
- [ ] `cache:policy: pull` for jobs that only read cache; `pull-push` for jobs that update it
- [ ] `cache:paths:` does not include secrets, credentials, or large binaries

## Step 7: Image and Runner Tags

- [ ] Docker images pinned to a digest or version tag — not `latest`
- [ ] Runner `tags:` match registered runner tag names exactly (case-sensitive)
- [ ] Jobs that need Docker-in-Docker have `services: [docker:dind]` and correct `DOCKER_HOST`
- [ ] `interruptible: true` set for jobs that can safely be cancelled when a new pipeline starts

## Step 8: YAML Anchors and Extends

- [ ] YAML anchors (`&name`/`*name`) and `extends:` not mixed on the same key — `extends:` does deep merge; anchors do shallow copy
- [ ] `!reference [job, script]` used instead of anchors for script reuse — more composable
- [ ] Hidden jobs (`.name:`) used as templates when they should not run standalone
- [ ] `extends:` chains are not deeper than 3 levels — deep chains make effective config impossible to reason about

## Step 9: Pipeline Efficiency

Flag inefficiencies that waste runner minutes:
- Jobs cloning full repo when they only need one directory: `GIT_DEPTH: 1` for shallow clone where appropriate
- Sequential jobs that could use `needs:` to run in parallel
- Missing `interruptible: true` on long-running jobs when the branch gets updated
- `before_script:` reinstalling dependencies that are already cached

## Output Format

```
[CRITICAL|IMPORTANT|NIT] Category
Job: job-name (or "global")
Problem: What is wrong
Fix: Specific YAML fix
```

CRITICAL = pipeline won't run or will produce incorrect results silently.
IMPORTANT = security, correctness, or significant efficiency issue.
NIT = style, deprecation, or minor best practice.
