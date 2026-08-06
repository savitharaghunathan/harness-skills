---
name: plan
tags: [stage]
description: >
  Analyzes a project using graphify, reads questionnaire decisions and
  analysis results, produces a migration spec for approval, then generates
  a detailed implementation plan.
inputs:
  - KONVEYOR_INSTRUCTIONS
  - .konveyor/questionnaire.json
  - .konveyor/analysis.json (optional)
  - domain skills (optional)
outputs:
  - .konveyor/spec.md
  - .konveyor/implementation.md
---

# Plan Stage

Analyzes the project deeply, produces a spec for approval, then writes the
implementation plan. `KONVEYOR_INSTRUCTIONS` is the primary goal; questionnaire
decisions supplement it. Does NOT modify any source files — planning only.

## Inputs

- `KONVEYOR_INSTRUCTIONS` — migration goal (primary)
- `.konveyor/questionnaire.json` — decisions from prior stage
- `.konveyor/analysis.json` — Kantra rule violations and patterns (if present)
- Domain skills (`tags: [domain]`) — migration knowledge (optional, see 1c)

---

## Phase 1 — Analyze

### 1a. Read Prior Stage Outputs

1. Read `.konveyor/questionnaire.json` for detection results and decisions
2. Read `.konveyor/analysis.json` if present for rule violations and patterns

### 1b. Generate Code Graph

Run graphify on the project:

```bash
graphify update
```

This produces `graph.json` in `graphify-out/`. All subsequent `graphify`
commands read from this graph automatically.

### 1c. Discover Domain Skills (optional)

Scan `/opt/skills/` for skills with `tags: [domain]` in their frontmatter. If
any are found, read their SKILL.md, modules, and references to understand:

- Phase order (which transformations come first)
- What patterns to look for in the graph
- Mapping tables (dependencies, APIs, config, patterns)
- Build command (`metadata.build_command`)

If no domain skills are found, proceed without them. You will structure the
plan based on your own analysis of the project, using graphify and
`questionnaire.json`.

### 1d. Understand the Project Architecture

Use graphify query commands to understand the project — do NOT read
`graph.json` manually.

Available commands:

| Command | Use |
|---|---|
| `graphify query "<question>"` | Ask about structure, layers, patterns, imports. Use `--dfs` for depth-first, `--budget N` to cap tokens. |
| `graphify path "A" "B"` | Shortest dependency path between two nodes |
| `graphify explain "X"` | Plain-language explanation of a node and its neighbors |
| `graphify affected "X"` | All nodes impacted by changing X. Use `--depth N` to limit. |

Use these to discover:

1. **Architectural layers** — how the project is organized (packages, modules, namespaces)
2. **Central nodes** — high-degree nodes whose changes ripple across the codebase — mark these COMPLEX
3. **Dependency chains** — ordering constraints between components
4. **Migration-relevant patterns** — which files match transformation patterns (from the domain skill if present, or from your own analysis)

Adapt your queries to what was detected in `questionnaire.json` and what
the domain skill says to look for (if present). Do not use hardcoded
queries — ask about the actual technologies, frameworks, and patterns in
this project.

### 1e. Match Patterns to Graph

Use graphify queries to identify which nodes need migration. If domain
skills are present, query for patterns from their transformation rules.
Otherwise, adapt queries to the source and target technologies detected in
`questionnaire.json`.

Classify each matched file:

- Simple (import/annotation replacement only)
- Complex (structural changes needed)
- Delete (file will be removed)
- Create (new file needed)

### 1f. Build Migration Order

Use `graphify affected` on central nodes to understand ripple effects.

Use impact analysis to:

- Determine which files must be migrated before others
- Identify hidden dependencies the community view doesn't show
- Validate that your ordering won't break downstream consumers

If domain skills are present, map each community to the domain skill phase
it belongs to. Otherwise, group steps into logical phases based on your
analysis (e.g. config, models, services, API). This gives you the migration
sequence WITHOUT reading every file.

### 1g. Selectively Read Complex Source Files (max 5-8)

Read files where the graph alone isn't enough — structural changes, central
nodes, complex patterns. Don't read files that only need import or
annotation changes.

---

## Phase 2 — Spec

Write `.konveyor/spec.md` — a summary of what will be migrated, the approach, and key
decisions applied. Structure:

```markdown
# Migration Spec

## Goal
<restate KONVEYOR_INSTRUCTIONS in one sentence>

## Source → Target
<source framework/version> → <target framework/version>

## Scope
- Files affected: <N>
- Estimated complexity: Low/Medium/High
- Hardest areas: <list 1-3 most complex>

## Key Decisions Applied
<from questionnaire.json — list each decision and chosen option>

## Approach
<phase-by-phase summary — from domain skill if present, or from your own analysis>

## Domain Skill
<name and description of the domain skill being used, or "none — plan is based on project analysis">
```

### Interactive mode

If `KONVEYOR_PARAM_INTERACTIVE` is `true`: present `.konveyor/spec.md` for
approval.

- If the user **approves**: proceed to Phase 3.
- If the user **rejects**: ask what needs to change, revise the spec, and
  re-present for approval. Repeat until approved.

### Non-interactive mode (default)

If `KONVEYOR_PARAM_INTERACTIVE` is unset or not `true`: proceed directly to
Phase 3. This is the default — interactive mode requires explicit opt-in.

---

## Phase 3 — Implementation Plan

Write `.konveyor/implementation.md` — step-by-step migration instructions.

```markdown
# Implementation Plan

## Goal
<one sentence>
- Domain skill: <name of domain skill used, or "none">

## Project Summary
- Type: <build tool / framework>
- Files affected: <N>
- Estimated complexity: Low/Medium/High
- Hardest steps: <list the 1-3 most complex items>

## Steps

### Step 1: <title>
- Phase: <phase-name> (from domain skill, or your own grouping)
- File: <exact path from repo root>
- Action: CREATE | MODIFY | DELETE
- What to do: <specific instructions>
- Why: <what pattern is being changed>
- Depends on: <step numbers, or "none">
- Verify: <how to know this step is done>

...

## Verification
<build command — from domain skill metadata.build_command, or detected from build manifest>

## Notes
<gotchas, special cases>
```

### Rules for writing steps

1. **Phase on every step** — every step must have a `Phase:` field (from domain skill phases if present, or your own logical grouping)
2. **One file per step** — never combine two files in one step
3. **Exact paths** — use real paths from graphify output, not placeholders
4. **Dependency order** — steps that others depend on come first
5. **Phase order** — follow the domain skill's phase ordering if present, or your own logical ordering
6. **Hard steps flagged** — add `COMPLEX:` prefix for structural changes
7. **DELETE steps last** — after all modifications are done

### Step detail levels

**Mechanical** (simple find-replace):
```markdown
### Step 5: Migrate imports in <file>
- Phase: <phase-name>
- File: <exact path from graphify output>
- Action: MODIFY
- What to do: Replace all old namespace imports with new namespace imports
- Why: Target framework uses different namespace
- Depends on: Step 1
- Verify: No old namespace imports remain
```

**Complex** (structural/architectural changes):
```markdown
### Step 14: COMPLEX — Convert message listener
- Phase: <phase-name>
- File: <path>
- Action: MODIFY
- What to do:
    - BEFORE: <old pattern>
    - AFTER: <new pattern>
    - Specific changes:
        1. Remove: <old imports/annotations/methods>
        2. Add: <new imports/annotations>
        3. Replace: <method signatures, configuration>
- Why: <why the old pattern isn't supported>
- Depends on: Step X, Step Y
- Verify: <grep checks, compile commands>
```

If a complex change also requires config file updates, create a separate step
for each config file rather than listing them as "Affected files" — one file
per step, always.

**CREATE** (new file):
```markdown
### Step 3: Create target config file
- Phase: <phase-name>
- File: <path>
- Action: CREATE
- What to do: Create file with <specific content>
- Why: <target framework requires this config file>
- Depends on: Step 1
- Verify: File exists with required properties
```

**DELETE** (remove file):
```markdown
### Step 20: Remove legacy config file
- Phase: <phase-name>
- File: <path>
- Action: DELETE
- What to do: Delete this file — no longer needed by target framework
- Why: <target framework does not use this file>
- Depends on: Step 14, Step 15
- Verify: File no longer exists
```

---

## Phase 4 — Write Output and Commit

You MUST complete this phase — planning is not done until outputs are written
and committed.

1. Create the `.konveyor/` directory if it does not exist
2. Write `.konveyor/spec.md`
3. Write `.konveyor/implementation.md`
4. Commit the outputs:

```bash
git add .konveyor/spec.md .konveyor/implementation.md
git commit -m "Add migration plan and spec"
```

Do NOT push.

The stage is NOT complete until both files are written and committed.

---

## Rules

- Do NOT modify source files — planning only
- Do NOT execute any migration steps
- Do NOT skip graphify — the graph is essential for later stages
- Follow the domain skill's phase order when structuring steps (if present)
- You MUST write `.konveyor/spec.md` and `.konveyor/implementation.md` before finishing
- Commit plan outputs when done — do NOT push
