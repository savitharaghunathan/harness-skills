---
name: report
tags: [stage]
description: >
  Generates a final migration report summarizing what was done,
  what remains, and quality assessment.
inputs:
  - .konveyor/questionnaire.json
  - .konveyor/execute.json
  - .konveyor/eval.json
  - .konveyor/spec.md
  - .konveyor/implementation.md
outputs:
  - .konveyor/report.md
---

# Report Stage

Reads everything under `.konveyor/` and produces a well-structured,
human-readable migration report.

---

## Inputs

- `.konveyor/questionnaire.json` — what was detected, what was decided
- `.konveyor/execute.json` — phase outcomes, fix iterations, build/test results
- `.konveyor/eval.json` — quality scores, learned patterns
- `.konveyor/spec.md` — migration spec
- `.konveyor/implementation.md` — what was planned

---

## Process

Read all inputs and produce `.konveyor/report.md` with these sections:

### 1. Summary

- Source: <language, framework, version>
- Target: <language, framework, version>
- Scope: <files affected, complexity>
- Key decisions: <top 3-5 decisions from questionnaire.json>

### 2. What Was Done

For each phase in `execute.json`:
- Phase name and status (success/partial/failed)
- Files changed
- Fix iterations needed (if any)

### 3. What Remains

From `execute.json`:
- Run status (`completed` or `aborted`) — if aborted, state which phase halted and why
- Skipped steps and their count
- Failed phases and their remaining errors
- Failing tests and their details

### 4. Quality Assessment

From `eval.json`:
- Build status
- Test pass rate
- Completeness score
- Decision outcomes — which choices are associated with clean phases vs struggles (correlational, not causal)

### 5. Learned Patterns

From `eval.json`:
- What worked well (replicate in future migrations)
- What struggled (areas to improve)
- Common error patterns encountered

---

## Output

Write `.konveyor/report.md` using clear markdown formatting — headers, tables,
bullet lists. This report is for the developer who will review and continue
the migration.

Create the `.konveyor/` directory if it does not exist.

---

## Commit

After writing the output, commit it:

```bash
git add .konveyor/report.md
git commit -m "Add migration report"
```

Do NOT push.

---

## Rules

- Do NOT run builds or tests
- Do NOT modify source files
- Do NOT re-execute any migration steps
- Trust `.konveyor/` artifacts as the source of truth
- Write a report that a developer can act on without re-reading all artifacts
- Commit report output when done — do NOT push
