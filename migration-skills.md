# Migration Stage Skills Design


**Pipeline:**
```
Questionnaire → Plan → Execute → Eval → Report
```

## Skill Discovery

Skills use `tags` in their SKILL.md frontmatter (per skillimage.io spec) to signal their role:

- `tags: [stage]` — process skill. Follow this. One per stage.
- `tags: [domain]` — migration knowledge. Consult this.
- `tags: [rule]` — always-on constraints. Respect this.

The harness discovers all skills at `/opt/skills/*/SKILL.md`. The agent finds the skill tagged `stage` and follows it, consulting any skills tagged `domain` for migration knowledge. No hardcoded paths or names.

---

## Stage 1: Questionnaire 

Brainstorming-style skill. Detect the app surface, then ask migration decisions one at a time.

```yaml
---
name: questionnaire
tags: [stage]
description: >
  Scans a project to detect its tech stack and gathers migration decisions
  through structured questions. Outputs questionnaire.json. Use as the first
  stage before planning.
---
```

### Phase 1 — Detect

Lightweight automated scan. No graphify, no analysis.json.

1. Read build manifest (pom.xml, package.json, go.mod, *.csproj, etc.)
2. Scan source files for framework markers (imports, annotations, config files)
3. Summarize: language, frameworks, build tool, app server, source file count

### Phase 2 — Gather Decisions

Based on what you detected, identify the decisions that need to be made before planning can begin. Ask them one at a time, with options and a recommendation.

In non-interactive mode: choose the best option for each and record your reasoning.

### Output

Write `.konveyor/questionnaire.json` 

```json
{
  "detection": { ... },
  "decisions": [ ... ],
  "mode": "interactive|non-interactive"
}
```

---

## Stage 2: Plan 

Analyzes the project deeply, produces a spec for approval, then writes the implementation plan. `KONVEYOR_INSTRUCTIONS` is the primary goal; questionnaire decisions supplement it.

```yaml
---
name: plan
tags: [stage]
description: >
  Analyzes a project using graphify, reads questionnaire decisions and
  analysis results, produces a migration spec for approval, then generates
  a detailed implementation plan.
---
```

### Inputs

- `KONVEYOR_INSTRUCTIONS` — migration goal (primary)
- `.konveyor/questionnaire.json` — decisions from prior stage
- `.konveyor/analysis.json` — Kantra rule violations and patterns (if present, written by harness from Hub)
- Domain skills (`tags: [domain]`) — migration knowledge (optional)

### Phase 1 — Analyze

1. Read `questionnaire.json` for decisions
2. Read `analysis.json` if present for rule violations
3. Run `graphify update` to produce `graph.json`
4. Read loaded domain skills if present (scan `/opt/skills`, find `tags: [domain]`)
5. Selectively read complex source files (max 5-8)

### Phase 2 — Spec

Write `.konveyor/spec.md` — what will be migrated, the approach, key decisions applied.

Interactive (`KONVEYOR_PARAM_INTERACTIVE=true`): present for approval. If rejected, revise and re-present until approved. Non-interactive (default): proceed.

### Phase 3 — Implementation Plan

Write `.konveyor/implementation.md` — step-by-step migration instructions. One step per file, dependency order, complex steps flagged. Every step must include a `Phase:` field (from domain skill if present, or a logical grouping).



---

## Stage 3: Execute 

Executes the implementation plan phase by phase. If domain skills are available, follows their phases and modules. Runs the build gate after each phase and fixes errors before moving on. Absorbs the verify stage — no separate verify skill.

```yaml
---
name: execute
tags: [stage]
description: >
  Reads the implementation plan and executes migrations phase by phase.
  If domain skills are available, follows their phases and modules. Runs
  the build gate after each phase, fixing compiler errors before proceeding.
---
```

### Process

1. Read `.konveyor/implementation.md`
2. If domain skills are present: read their phases, modules, and references; validate Phase: fields
3. Follow the phase order (from domain skill or the plan's grouping)
4. For each phase, select steps by `Phase:` field:
   - Apply transformations (using domain skill modules/references if available)
   - Commit after each step — do not push
   - Run the build gate (build command from domain skill, implementation.md, or auto-detected)
   - If build fails: fix errors iteratively (up to 3 attempts)
   - If build still fails after 3 attempts: **halt**
6. After all phases: run tests, report results (do not fix test failures)
7. Write `.konveyor/execute.json` — per-step status, per-phase results, build/test results

### Output

`.konveyor/execute.json` — structured execution summary for eval and report.

### Changes from current

- Combines execute + verify into one stage
- Reads `.konveyor/implementation.md` instead of `PLAN.md`
- Follows phases with build gates (domain skill phases if present, or plan's grouping)
- Discovers domain skills via `tags: [domain]` frontmatter (optional)
- `tags: [stage]` in frontmatter

---

## Stage 4: Eval 

Pure assessment. Reads `.konveyor/` artifacts. Does not run builds or tests.

```yaml
---
name: eval
tags: [stage]
description: >
  Evaluates migration quality, scores questionnaire decisions,
  and extracts learned patterns for future runs.
---
```

### Inputs

- `.konveyor/questionnaire.json` — decisions made
- `.konveyor/execute.json` — phase outcomes, fix iterations, build/test results
- `.konveyor/implementation.md` — what was planned

### Process

1. Score the migration (build pass/fail, test pass rate, completeness from steps applied/total, fix effort)
2. Score questionnaire decisions against phase outcomes (correlational — what outcomes are associated with each decision, not what each decision caused)
3. Extract learned patterns (what worked, what struggled, what failed)

### Output

`.konveyor/eval.json` — quality scores, decision scores, learned patterns.

---

## Stage 5: Report 

Reads everything under `.konveyor/` and produces a well-structured, human-readable migration summary. Replaces the current `handoff.md` the harness writes.

```yaml
---
name: report
tags: [stage]
description: >
  Generates a final migration report summarizing what was done,
  what remains, and quality assessment.
---
```

### Inputs

- `.konveyor/questionnaire.json` — what was detected, what was decided
- `.konveyor/execute.json` — phase outcomes, fix iterations
- `.konveyor/eval.json` — quality scores, learned patterns
- `.konveyor/spec.md`, `.konveyor/implementation.md` — what was planned

### Process

1. Summarize: source → target, scope, key decisions
2. What was done: phases completed, files changed
3. What remains: failed phases, remaining errors, failing tests
4. Quality score from eval

### Output

`.konveyor/report.md` — human-readable migration report. 

---

## Domain Skill Contract

Domain skills provide language-specific migration knowledge. Mounted at `/opt/skills/{name}/` via OCI ImageVolumes.

```yaml
---
name: javaee-to-quarkus
tags: [domain]
description: Migrates Java EE 7/8 applications to Quarkus 3.
metadata:
  source: java-ee-7
  target: quarkus-3
  language: java
  build_command: mvn clean compile -DskipTests
  smoke_command: mvn quarkus:dev &; sleep 15; curl -sf http://localhost:8080/q/health; kill %1
---
```

Structure:
```
{name}/
  SKILL.md           # tags: [domain], frontmatter with source/target/language
  modules/           # Phase-by-phase execution instructions
  references/        # Mapping tables (dependency-map, api-map, config-map, pattern-map)
```

Stage skills find domain skills by reading frontmatter (`tags: [domain]`). No hardcoded paths.

---

## Cross-Stage Data Flow

| Stage | Reads | Writes |
|-------|-------|--------|
| Questionnaire | source repo | `.konveyor/questionnaire.json` |
| Plan | questionnaire.json, analysis.json, domain skills (optional) | `.konveyor/spec.md`, `.konveyor/implementation.md`, `graph.json` |
| Execute | .konveyor/implementation.md, domain skills (optional) | migrated source files + `.konveyor/execute.json` |
| Eval | questionnaire.json, execute.json, .konveyor/implementation.md | `.konveyor/eval.json` |
| Report | questionnaire.json, execute.json, eval.json, .konveyor/spec.md, .konveyor/implementation.md | `.konveyor/report.md` |
