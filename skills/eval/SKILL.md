---
name: eval
tags: [stage]
description: >
  Evaluates migration quality, scores questionnaire decisions,
  and extracts learned patterns for future runs.
---

# Eval Stage

Pure assessment. Reads `.konveyor/` artifacts and git log to evaluate how
the migration went. Does not run builds or tests — trusts the execute stage's
results.

---

## Inputs

- `.konveyor/questionnaire.json` — decisions made before planning
- `.konveyor/execute.json` — phase outcomes, fix iterations, build/test results
- `.konveyor/implementation.md` — what was planned
- Git log — what actually changed

---

## Process

### 1. Score the Migration

Evaluate overall migration quality from `execute.json`:

- **Build**: pass or fail
- **Tests**: pass rate (passed / total)
- **Completeness**: steps applied vs total steps (from `execute.json` `steps` array)
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

From the git log and execute.json, identify:

- **What worked**: phases that completed cleanly (0-1 fix iterations)
- **What struggled**: phases that needed multiple fix iterations
- **What failed**: phases that hit max iterations with remaining errors
- **Common error patterns**: recurring compiler errors across phases

---

## Output

Write `.konveyor/eval.json`. See [templates/eval.md](templates/eval.md)
for the full schema and field descriptions.


---

## Rules

- Do NOT run builds or tests — trust `execute.json`
- Do NOT modify any source files
- Do NOT re-execute any migration steps
- Read `.konveyor/` artifacts as the source of truth
