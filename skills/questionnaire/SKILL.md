---
name: questionnaire
tags: [stage]
description: >
  Scan the project, detect its tech stack, gather migration decisions
  (interactive or non-interactive based on KONVEYOR_PARAM_INTERACTIVE),
  write .konveyor/questionnaire.json, and commit it.
  The stage is NOT complete until the commit succeeds.
inputs:
  - KONVEYOR_INSTRUCTIONS
  - source repo
outputs:
  - .konveyor/questionnaire.json
---

# Questionnaire Stage

Scans the project to understand what it is, then asks the migration decisions
that need to be made before planning can begin.

## Inputs

- `KONVEYOR_INSTRUCTIONS` — migration goal (read this to understand what kind
  of migration is being attempted, so detection and decisions are targeted)

---

## Phase 1 — Detect

Lightweight automated scan. No graphify, no deep analysis.

1. Read the build manifest (pom.xml, package.json, go.mod, Cargo.toml, *.csproj, etc.)
2. Scan source files for framework markers (imports, annotations, config files, app server descriptors)
3. Summarize what you found:
   - Language and version
   - Frameworks and libraries
   - Build tool
   - Application server (if any)
   - Approximate source file count

---

## Phase 2 — Gather Decisions

Based on what you detected, identify the decisions that need to be made before
planning can begin. These are choices where multiple valid approaches exist and
the right answer depends on the project's constraints.

Ask decisions **one at a time**. For each:
- State the decision clearly
- List 2-4 concrete options
- Explain trade-offs briefly
- Give your recommendation and why

Wait for the answer before asking the next question.

### Non-interactive mode

If `KONVEYOR_PARAM_INTERACTIVE` is `false` or not set:
choose the best option for each decision and record your reasoning.
Do not wait for answers — proceed through all decisions autonomously.

---

## Phase 3 — Write Output and Commit

You MUST complete this phase — detection alone is not enough.

1. Create the `.konveyor/` directory if it does not exist
2. Write `.konveyor/questionnaire.json` — see [templates/questionnaire.md](templates/questionnaire.md)
   for the full schema and field descriptions
3. Commit the output:

```bash
git add .konveyor/questionnaire.json
git commit -m "Add questionnaire results"
```

Do NOT push.

The stage is NOT complete until `questionnaire.json` is written and committed.

---

## Rules

- Keep detection lightweight — read manifests and scan imports, nothing more
- You MUST write `.konveyor/questionnaire.json` before finishing
- Commit questionnaire output when done — do NOT push
