# Pipeline Debug Skill

**Source:** Original — authored for this repo, informed by gstack `/investigate`.
**Covers:** Jenkins (declarative + scripted) and GitLab CI failing stages.

## Why This Is Separate from Investigate

The generic `investigate` skill covers root-cause debugging for application code. Pipeline failures have a completely different failure taxonomy:

- Jenkins failures are often not code errors — they're environment issues, agent availability, credential problems, or CPS violations that only surface at runtime
- Build logs are the primary debugging artifact, not source files
- Reproduction is often impossible locally (requires live Jenkins/GitLab infrastructure)
- The fix is often in the pipeline definition, not in the application being built

This skill front-loads log analysis and failure classification before touching any source file.

## Usage

```bash
cp skills/cicd/pipeline-debug/kiro/steering.md .kiro/steering/41-pipeline-debug.md
```

Invoke in Kiro: "debug this pipeline failure", "why did this stage fail", "investigate Jenkins error", "diagnose GitLab CI".

Paste the failing build log (or key excerpt) into the chat when invoking.
