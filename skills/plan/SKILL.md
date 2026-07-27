---
name: plan
tags: [stage]
description: >
  Analyzes a project using graphify, reads questionnaire decisions and
  analysis results, produces a migration spec for approval, then generates
  a detailed implementation plan.
---

# Plan Stage

Analyzes the project deeply, produces a spec for approval, then writes the
implementation plan. `KONVEYOR_INSTRUCTIONS` is the primary goal; questionnaire
decisions supplement it. Does NOT modify any source files — planning only.

## Inputs

- `KONVEYOR_INSTRUCTIONS` — migration goal (primary)
- `.konveyor/questionnaire.json` — decisions from prior stage
- `.konveyor/analysis.json` — Kantra rule violations and patterns (if present)
- Domain skills (`tags: [domain]`) — migration knowledge (phases, modules, references)

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

This produces `graph.json` in the repo root.

### 1c. Discover Domain Skills

Scan `/opt/skills/` for skills with `tags: [domain]` in their frontmatter. Read
their SKILL.md, modules, and references to understand:

- Phase order (which transformations come first)
- What patterns to look for in the graph
- Mapping tables (dependencies, APIs, config, patterns)
- Build gate commands

### 1d. Understand the Project Architecture

Read `graph.json` to understand the project:

1. **Communities (architectural layers)**:
   - Community 0 might be build files (pom.xml, package.json)
   - Smaller communities often = data models (few dependencies)
   - Medium communities = services, business logic
   - Large, high-degree communities = API/controllers

2. **God nodes (high-risk abstractions)**:
   - Nodes with degree > 20 are central to the system
   - Mark these as COMPLEX in the plan
   - Changes here ripple across many files

3. **Dependency flow**:
   - Use edges to understand: who depends on what?
   - Models → Services → Controllers (typical layering)

### 1e. Match Patterns to Graph

Use the domain skill's patterns to identify which graph nodes need migration.
Check node attributes (imports, annotations) against the domain skill's
transformation rules to classify each file:

- Simple (import/annotation replacement only)
- Complex (structural changes needed)
- Delete (file will be removed)
- Create (new file needed)

### 1f. Build Migration Order

Map graph communities to the domain skill's phase order:

```
Community 0  (1 file: build manifest)    → Phase 1: Build config
Community 28 (5 files: data models)      → Phase 2: Models
Community 91 (8 files: services)         → Phase 3: Services
Community 164 (12 files: controllers)    → Phase 4: API
```

This gives you the migration sequence WITHOUT reading every file.

### 1g. Selectively Read Complex Source Files (max 5-8)

Read files where the graph alone isn't enough — structural changes, god nodes,
complex patterns from the domain skill. Don't read files that only need
import or annotation changes.

---

## Phase 2 — Spec

Write `spec.md` — a summary of what will be migrated, the approach, and key
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
<phase-by-phase summary from domain skill>

## Domain Skill
<name and description of the domain skill being used, or "none">
```

### Interactive mode

If `KONVEYOR_PARAM_INTERACTIVE` is `true`: present `spec.md` for approval.
Wait for the user to confirm before proceeding to Phase 3.

### Non-interactive mode

Proceed directly to Phase 3.

---

## Phase 3 — Implementation Plan

Write `implementation.md` — step-by-step migration instructions.

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
- File: <exact path from repo root>
- Action: CREATE | MODIFY | DELETE
- What to do: <specific instructions>
- Why: <what pattern is being changed>
- Depends on: <step numbers, or "none">
- Verify: <how to know this step is done>

...

## Verification
<build command from domain skill metadata>

## Notes
<gotchas, special cases>
```

### Rules for writing steps

1. **One file per step** — never combine two files in one step
2. **Exact paths** — use real paths from graph.json, not placeholders
3. **Dependency order** — steps that others depend on come first
4. **Phase order** — follow the domain skill's phase ordering
5. **Hard steps flagged** — add `COMPLEX:` prefix for structural changes
6. **DELETE steps last** — after all modifications are done

### Step detail levels

**Mechanical** (simple find-replace):
```markdown
### Step 5: Migrate imports in Order.java
- File: src/main/java/com/example/model/Order.java
- Action: MODIFY
- What to do: Replace all old namespace imports with new namespace imports
- Why: Target framework uses different namespace
- Depends on: Step 1
- Verify: No old namespace imports remain
```

**Complex** (structural/architectural changes — use domain skill patterns):
```markdown
### Step 14: COMPLEX — Convert message listener
- File: <path>
- Action: MODIFY
- What to do:
    - BEFORE: <old pattern from domain skill>
    - AFTER: <new pattern from domain skill>
    - Specific changes:
        1. Remove: <old imports/annotations/methods>
        2. Add: <new imports/annotations>
        3. Replace: <method signatures, configuration>
    - Affected files: <config files that also need updates>
- Why: <from domain skill — why the old pattern isn't supported>
- Depends on: Step X, Step Y
- Verify: <from domain skill — grep checks, compile commands>
```

---

## Output

Write `spec.md` and `implementation.md`.

---

## Rules

- Do NOT modify source files — planning only
- Do NOT execute any migration steps
- Do NOT skip graphify — the graph is essential for later stages
- Follow the domain skill's phase order when structuring steps
