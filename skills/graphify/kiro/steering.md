---
inclusion: always
---
<!-- Source: https://github.com/safishamsi/graphify — graphify skill -->
<!-- Version: 1.0.0 -->

# Graphify: Codebase Knowledge Graph

This project uses graphify to maintain a persistent knowledge graph of the codebase. The graph is stored in `graphify-out/` and rebuilt incrementally as files change.

## Before Answering Codebase Questions

When asked about component relationships, module structure, dependency chains, or which files are affected by a change:

1. Check if `graphify-out/GRAPH_REPORT.md` exists.
2. If it exists, read it first — it contains high-degree nodes, unexpected connections, and suggested queries for this codebase.
3. For specific path or relationship queries, run: `python -m graphify . query "<your question>"`
4. For shortest-path between two components: `python -m graphify . path "ComponentA" "ComponentB"`
5. For details on a specific node: `python -m graphify . explain "NodeName"`

## Building or Updating the Graph

If `graphify-out/` does not exist, or the user asks to rebuild the graph:

```
python -m graphify .
```

For incremental update after changes:

```
python -m graphify . --update
```

The graph processes: TypeScript (`.ts`), Angular templates (`.html`), SCSS (`.scss`), Python (`.py`), Markdown (`.md`), and PDFs. It uses AST parsing for code files and semantic extraction for documentation.

## Output Files

| File | Contents |
|------|----------|
| `graphify-out/graph.json` | Persistent graph — reuse across sessions |
| `graphify-out/GRAPH_REPORT.md` | Summary: key nodes, unexpected connections, query suggestions |
| `graphify-out/graph.html` | Interactive visualization (open in browser) |
| `graphify-out/obsidian/` | Obsidian vault format for manual exploration |

## Graph Honesty

Every edge in the graph is tagged: `EXTRACTED` (directly observed), `INFERRED` (reasoned from context), or `AMBIGUOUS` (uncertain). Do not present inferred connections as facts without noting the tag.

## Installation

```bash
pip install graphifyy   # three y's
```

Python 3.10+ required.
