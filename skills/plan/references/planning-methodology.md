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

1. **Understand layers** — `graphify query "What are the main architectural layers?"`
2. **Find god nodes** — `graphify query "Which nodes have the highest degree?"`
3. **Match patterns** — `graphify query "Which classes use @Stateless?"` (adapt to domain skill patterns)
4. **Trace dependencies** — `graphify path "ServiceA" "Database"` for specific chains
5. **Assess impact** — `graphify affected "OrderService" --depth 2` before ordering phases

### Community → Layer mapping

Communities cluster files that are tightly coupled. Map them to migration layers:

- **1-file communities** with build manifests → Build config layer
- **Small communities** with data classes → Model/entity layer
- **Medium communities** with business logic → Service layer
- **Large communities** with many edges → API/controller layer
- **Isolated nodes** → Utilities, helpers, or config files

The domain skill defines the phase order. Map communities to phases to get
the migration sequence.

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
| Config file referenced by domain skill | READ — need exact current state |

### Reading rules

1. Query graphify BEFORE reading any source file
2. Read ONE file at a time
3. Read ONLY when graphify queries + domain skill isn't enough
4. If uncertain, mark step COMPLEX and move on — the execute stage will read it
5. Prefer reading god nodes over leaf nodes (higher impact)

---

## Plan Quality Checklist

Before finalizing `.konveyor/implementation.md`, verify:

- [ ] Every step has a `Phase:` matching a domain skill phase
- [ ] Every file path is real (from graphify query output), not a placeholder
- [ ] Steps follow the domain skill's phase order
- [ ] Dependencies between steps are explicit
- [ ] God nodes are marked COMPLEX
- [ ] DELETE steps come after MODIFY steps for the same area
- [ ] Build command is included from domain skill metadata (`metadata.build_command`)
- [ ] One file per step — no combined steps
