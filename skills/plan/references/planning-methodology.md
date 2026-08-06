---
name: planning-methodology
description: >
  Detailed planning methodology — graph-powered discovery, selective reading
  strategy, and reference-driven plan generation. Used by the plan stage skill.
---

# Planning Methodology

## Graph-Powered Discovery

The code graph produced by `graphify update` is the foundation of the planning
process. Query it using graphify CLI commands — do NOT read `graph.json` manually.

### Graphify query commands

| Command | Planning Use |
|---|---|
| `graphify query "<question>"` | BFS/DFS traversal — ask about layers, patterns, imports, annotations. Use `--dfs` for depth-first, `--budget N` to cap tokens. |
| `graphify path "A" "B"` | Shortest dependency path between two nodes — traces chains without reading files. |
| `graphify explain "X"` | Plain-language explanation of a node and its neighbors. |
| `graphify affected "X"` | Reverse traversal — all nodes impacted by X. Use `--depth N` and `--relation R` to filter. |

### Discovery workflow

1. **Understand layers** — query for architectural structure (packages, modules, namespaces)
2. **Find central nodes** — query for high-degree nodes with many dependencies
3. **Match patterns** — query for migration-relevant patterns (from domain skill transformation rules if present, or based on detected source/target technologies)
4. **Trace dependencies** — use `graphify path` for specific dependency chains between components
5. **Assess impact** — use `graphify affected` on central or complex nodes before ordering phases

### Community → Phase mapping

Communities cluster files that are tightly coupled. Map them to the domain
skill's phases based on what the files contain and how they relate.

General heuristics:

- **1-file communities** with build manifests → build config phase
- **Small communities** with data types → model/entity phase
- **Medium communities** with business logic → core logic phase
- **Large communities** with many edges → API/interface phase
- **Isolated nodes** → utilities, helpers, or config files

If domain skills are present, they define the phase names and order. Map
communities to those phases. Otherwise, define your own logical phases
based on the project structure.

---

## Selective Reading Strategy

### Budget

- Build manifest: 1 file (always read)
- Domain skill modules/references: as needed
- Complex source files: 5-8 files max
- Total reads: ~10 files

### Decision matrix

| Graphify query tells you... | Action |
|---|---|
| Only namespace imports to change | Don't read — plan from query output |
| Only annotation swaps needed | Don't read — plan from query output |
| God node with complex patterns | READ — run `graphify explain "Node"` first, then read source if needed |
| File matches structural pattern | READ — run `graphify affected "Node"` to scope impact first |
| Simple entity/model class | Don't read — `graphify query` has the imports |
| Config file needing migration | READ — need exact current state |

### Reading rules

1. Query graphify BEFORE reading any source file
2. Read ONE file at a time
3. Read ONLY when graphify queries (+ domain skill if present) aren't enough
4. If uncertain, mark step COMPLEX and move on — the execute stage will read it
5. Prefer reading god nodes over leaf nodes (higher impact)

---

## Plan Quality Checklist

Before finalizing `.konveyor/implementation.md`, verify:

- [ ] Every step has a `Phase:` (from domain skill if present, or your own grouping)
- [ ] Every file path is real (from graphify query output), not a placeholder
- [ ] Steps follow the phase order (domain skill's if present, or logical grouping)
- [ ] Dependencies between steps are explicit
- [ ] God nodes are marked COMPLEX
- [ ] DELETE steps come after MODIFY steps for the same area
- [ ] Build command is included (from domain skill `metadata.build_command` or detected from build manifest)
- [ ] One file per step — no combined steps
