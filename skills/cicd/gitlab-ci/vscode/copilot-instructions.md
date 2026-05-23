## GitLab CI Review

When reviewing `.gitlab-ci.yml` or GitLab CI template files:

**Stage/job validity:** Every job's `stage:` must exist in the top-level `stages:` list. No duplicate job names (later definitions silently override). Every `needs:` reference must name a real job in an earlier or same stage.

**DAG correctness (`needs:`):** `needs: []` runs a job immediately, ignoring stage order. `needs: [job]` with default `artifacts: true` requires that job to define `artifacts:` or downstream gets empty results. Flag any `needs:` pointing to a `when: manual` job — the downstream job waits indefinitely.

**Deprecated syntax:** Flag all `only:/except:` usage and provide the `rules:` equivalent. `rules:` and `only:/except:` cannot coexist in one job. First matching `rules:` entry wins — unreachable rules below a catch-all are silently ignored.

**Variable security:** No plaintext secrets in `variables:`. Protected variables are invisible to MR pipelines and unprotected branches — flag if a variable is expected there. Masked variables only mask if the value is base64-only, 8+ chars, no whitespace.

**Artifacts vs cache:** `artifacts:` passes files between jobs; `cache:` speeds up builds. Cache without `cache:key:` pollutes across branches. Artifacts without `expire_in:` use the instance default and waste storage. `artifacts:when: always` needed to capture failure evidence.

**Images and runners:** Images pinned to version tag or digest — not `latest`. Runner `tags:` are case-sensitive and must match registered runners exactly.

**YAML composition:** Do not mix YAML anchors and `extends:` on the same key. Use `!reference [job, script]` for script reuse — more composable than anchors.

Format: `[CRITICAL|IMPORTANT|NIT] job-name: problem. Fix: specific YAML change.`
