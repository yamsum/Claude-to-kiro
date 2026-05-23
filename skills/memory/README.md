# Memory Skill Adaptation

**Source:** [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) (Apache 2.0)
**Original mechanism:** Five lifecycle hooks + SQLite/Chroma worker service

## What claude-mem Does

claude-mem gives Claude Code persistent memory across sessions. It captures observations, decisions, and tool usage during a session, stores them in a local database (SQLite + Chroma vector search), and retrieves relevant history at the start of future sessions.

## What Was Adapted

claude-mem's full architecture (Node.js worker, Chroma vector DB, BullMQ) cannot be dropped into Kiro without significant infrastructure setup. The adaptation takes a lighter approach: markdown-based session logging that Kiro can read and write natively, providing ~80% of the value without any additional infrastructure.

| claude-mem Feature | Kiro Adaptation |
|-------------------|-----------------|
| SessionStart hook: inject past context | `hook.yaml` (`agentHookStart`): read `session-log.md` |
| PostToolUse hook: capture observations | `steering.md`: instruct Kiro to record key decisions inline |
| Stop/SessionEnd hook: compress and save | `hook.yaml` (`agentHookStop`): write session summary |
| Chroma semantic search | Natural language: Kiro reads the log file directly |
| Web viewer UI | Not ported — view `session-log.md` directly |
| `<private>` tag filtering | Not ported — mark sensitive lines with `<!-- private -->` as convention |

## What Was Dropped

- The full Node.js/Bun worker service infrastructure
- SQLite + Chroma vector database
- BullMQ job queue
- Web viewer at localhost:37777
- Cross-session semantic search (replaced by Kiro reading the markdown log)
- Multi-account support

## Usage

### Kiro
1. Copy the steering doc and hook into your project:
   ```bash
   cp skills/memory/kiro/steering.md .kiro/steering/20-memory.md
   cp skills/memory/kiro/hook.yaml .kiro/hooks/memory.yaml
   ```
2. Create the memory log directory:
   ```bash
   mkdir -p .kiro/memory
   echo "# Session Log" > .kiro/memory/session-log.md
   ```
3. Add `.kiro/memory/session-log.md` to your `.gitignore` if the project is shared (it contains session-specific context).

### Upgrading to Full claude-mem

If you need semantic search across many sessions, install the full claude-mem package:
```bash
npx claude-mem install
```
Note: requires Node.js 20+ and Claude Code plugin support. Not available in Kiro directly.
