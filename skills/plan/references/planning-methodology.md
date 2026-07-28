---
name: planning-methodology
description: >
  Detailed planning methodology — graph-powered discovery, selective reading
  strategy, and reference-driven plan generation. Used by the plan stage skill.
---

# Planning Methodology

## Graph-Powered Discovery

The code graph (`graph.json`) produced by `graphify update` is the foundation
of the planning process. It lets you understand a project's architecture without
reading every file.

### What the graph provides

| Graph Feature | Planning Use |
|---|---|
| Communities | Architectural layers (build, models, services, API) |
| Edges | Dependency flow — who depends on what |
| God nodes (degree > 20) | High-risk files that affect many others |
| Node attributes: imports | Which namespaces/packages are used |
| Node attributes: annotations | Which patterns are used (DI, persistence, messaging) |
| Node attributes: file path | Exact paths for plan steps |

### Reading the graph

1. Start with communities — each is an architectural layer
2. Identify god nodes — these are COMPLEX and need source reading
3. Check node imports/annotations against domain skill patterns
4. Use edges to determine dependency order (migrate dependencies first)

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

| Graph tells you... | Action |
|---|---|
| Only namespace imports to change | Don't read — plan from graph |
| Only annotation swaps needed | Don't read — plan from graph |
| God node with complex patterns | READ — need to see the structure |
| File matches structural pattern | READ — before/after differ significantly |
| Simple entity/model class | Don't read — graph has the imports |
| Config file referenced by domain skill | READ — need exact current state |

### Reading rules

1. Read ONE file at a time
2. Read ONLY when graph + domain skill isn't enough
3. If uncertain, mark step COMPLEX and move on — the execute stage will read it
4. Prefer reading god nodes over leaf nodes (higher impact)

---

## Plan Quality Checklist

Before finalizing `.konveyor/implementation.md`, verify:

- [ ] Every step has a `Phase:` matching a domain skill phase
- [ ] Every file path is real (from graph.json), not a placeholder
- [ ] Steps follow the domain skill's phase order
- [ ] Dependencies between steps are explicit
- [ ] God nodes are marked COMPLEX
- [ ] DELETE steps come after MODIFY steps for the same area
- [ ] Build command is included from domain skill metadata (`metadata.build_command`)
- [ ] One file per step — no combined steps
