---
inclusion: always
---
<!-- Source: https://github.com/thedotmack/claude-mem — memory skill (lightweight adaptation) -->
<!-- Version: 1.0.0 -->

# Session Memory

This project maintains a session memory log at `.kiro/memory/session-log.md`. Use it to preserve context, decisions, and findings across Kiro sessions.

## At Session Start

1. Check if `.kiro/memory/session-log.md` exists and read the last 50 lines.
2. Note any unresolved decisions, in-progress work, or relevant prior findings.
3. If the user's request relates to something in the log, surface the relevant context before starting.

## During a Session

Record observations to the memory log when:
- A non-obvious architectural decision is made (and why)
- A bug is found and its root cause identified
- A pattern or pitfall specific to this codebase is discovered
- A task is left incomplete and needs to be continued next session

Format for log entries:
```
## [YYYY-MM-DD] Session Note
**Context:** Brief description of what was being worked on.
**Finding:** What was discovered or decided.
**Action taken:** What was done, if anything.
**Unresolved:** Anything left open.
```

## At Session End

When wrapping up (user says "done", "that's it", "stop", or similar):
1. Append a session summary to `.kiro/memory/session-log.md`.
2. Include: what was worked on, key decisions made, anything left unresolved.
3. Keep it under 10 lines — this is a reference, not a full report.

## Privacy

Mark lines that contain sensitive data (credentials, PII, internal URLs) with `<!-- private -->`. These lines should not be shared or referenced in future sessions.

Example:
```
**API endpoint:** https://internal.bank.example/api/v2 <!-- private -->
```

## Log Maintenance

If the log exceeds 500 lines, summarize old entries into an archive section at the top, keeping the last 100 lines as-is. Label the archive: `## Archive (summarized)`.
