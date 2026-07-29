---
name: eval
tags: [stage]
description: >
  Evaluates migration quality, scores questionnaire decisions,
  and extracts learned patterns for future runs.
inputs:
  - .konveyor/questionnaire.json
  - .konveyor/execute.json
  - .konveyor/implementation.md
outputs:
  - .konveyor/eval.json
---

# Eval Stage

Pure assessment. Reads `.konveyor/` artifacts to evaluate how the migration
went. Does not run builds or tests — trusts the execute stage's results.

---

## Inputs

- `.konveyor/questionnaire.json` — decisions made before planning
- `.konveyor/execute.json` — step outcomes, phase outcomes, build/test results
- `.konveyor/implementation.md` — what was planned

---

## Process

### 1. Score the Migration

Evaluate overall migration quality from `execute.json`:

- **Run status**: `completed` or `aborted` — if aborted, note which phase halted
- **Build**: pass or fail
- **Tests**: pass rate (passed / total); `skipped` if run was aborted before tests
- **Completeness**: steps `applied` vs total steps (from `execute.json` `steps` array).
  Steps with `status: "skipped"` due to `"build gate halt"` are not counted as
  failures — they were never attempted. Report them separately.
- **Fix effort**: total fix iterations across all phases
- **Remaining errors**: count of unresolved compiler errors

### 2. Score Questionnaire Decisions

For each decision in `questionnaire.json`, assess against the phase outcomes
in `execute.json`. Decision scoring is **correlational, not causal** — report
what outcomes are associated with each decision, not what each decision caused:

- Is the chosen approach associated with clean phases? (phase succeeded without excessive fix iterations)
- Is any decision associated with problems? (phase failed or required max fix iterations)
- Would a different option likely have produced better outcomes? (based on the errors encountered)

### 3. Extract Learned Patterns

From `execute.json`, identify:

- **What worked**: phases that completed cleanly (0-1 fix iterations)
- **What struggled**: phases that needed multiple fix iterations
- **What failed**: phases that hit max iterations with remaining errors
- **Common error patterns**: recurring compiler errors across phases

---

## Phase 4 — Write Output and Commit

You MUST complete this phase — evaluation is not done until the output is
written and committed.

1. Create the `.konveyor/` directory if it does not exist
2. Write `.konveyor/eval.json` — see [templates/eval.md](templates/eval.md)
   for the full schema and field descriptions
3. Commit the output:

```bash
git add .konveyor/eval.json
git commit -m "Add eval results"
```

Do NOT push.

The stage is NOT complete until `eval.json` is written and committed.

---

## Rules

- Do NOT run builds or tests — trust `execute.json`
- Do NOT modify any source files
- Do NOT re-execute any migration steps
- Read `.konveyor/` artifacts as the source of truth
- You MUST write `.konveyor/eval.json` before finishing
- Commit eval output when done — do NOT push
