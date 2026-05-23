# Graphify Skill Adaptation

**Source:** [safishamsi/graphify](https://github.com/safishamsi/graphify) (MIT)
**Original invocation:** `/graphify <path>` in Claude Code

## What Graphify Does

Graphify scans a codebase — TypeScript, Angular templates, SCSS, Python, PDFs, images — and builds a queryable knowledge graph from it. It outputs:

- `graphify-out/graph.json` — the persistent graph (survives sessions)
- `graphify-out/GRAPH_REPORT.md` — high-degree nodes, unexpected connections, suggested queries
- `graphify-out/graph.html` — interactive visualization with search and filtering

The key value: instead of re-reading entire source files on every question, the AI reads the compact graph report (71x fewer tokens according to the project's benchmarks) and traverses relationships.

## Requirements

```bash
pip install graphifyy   # note: three y's on PyPI
```

Python 3.10+ required.

## What Was Adapted

| Claude Code Feature | Kiro Adaptation | VS Code Adaptation |
|--------------------|-----------------|-------------------|
| `/graphify .` slash command | `steering.md`: instruct Kiro to read `GRAPH_REPORT.md` before answering codebase questions | `tasks.json`: "Graphify: Update Knowledge Graph" task |
| Auto-rebuild on file change | `hook.yaml`: `fileChanged` trigger on `.ts`, `.html`, `.scss` | — |
| Graph query (`/graphify query`) | Steering doc: instruct Kiro to use `python -m graphify . query` for path lookups | — |
| Git post-commit hook | `hook.yaml` covers this via `fileChanged` | — |

## What Was Dropped

- `--watch` daemon mode — hooks handle the trigger instead
- MCP server (`--mcp`) — no equivalent needed in Kiro
- Neo4j export — infrastructure concern outside scope
- Multi-language mode flags — use CLI directly if needed

## Usage

### Kiro
1. Install graphify: `pip install graphifyy`
2. Run once: `python -m graphify .`
3. Copy steering doc and hook into your project:
   ```bash
   cp skills/graphify/kiro/steering.md .kiro/steering/10-graphify.md
   cp skills/graphify/kiro/hook.yaml .kiro/hooks/graphify.yaml
   ```
4. Kiro will now read the graph report before answering codebase questions and rebuild the graph when source files change.

### VS Code
1. Install graphify: `pip install graphifyy`
2. Merge `skills/graphify/vscode/tasks.json` into your `.vscode/tasks.json`
3. Run via `Ctrl+Shift+P → Tasks: Run Task → Graphify: Update Knowledge Graph`
