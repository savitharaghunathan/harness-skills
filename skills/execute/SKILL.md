---
name: execute
tags: [stage]
description: >
  Reads the implementation plan and executes migrations phase by phase,
  following the domain skill's phases and modules. Runs the build gate
  after each phase, fixing compiler errors before proceeding.
---

# Execute Stage

Executes the migration by following the implementation plan and domain skill
phases. Runs the build gate after each phase and fixes errors before moving on.

---

## Startup

1. Read `implementation.md`
2. Scan `/opt/skills/` for skills with `tags: [domain]` in their frontmatter
3. Read the domain skill's phases, modules, and references
4. Read `.konveyor/questionnaire.json` for context on decisions made

---

## Execution Loop

Follow the domain skill's phase order. For each phase:

### Step 1 — Apply transformations

- Read the domain skill's module for this phase
- Read the domain skill's reference tables (dependency-map, api-map, config-map, pattern-map), if they exist
- Work through the steps in `implementation.md` that belong to this phase
- For each file:
  1. Read the target file
  2. Apply transformations per the module instructions and reference tables
  3. Write the modified file
  4. Move to the next file immediately

### Step 2 — Build gate

Run the build command from the domain skill's metadata (`metadata.build_tool`).

- If the build **succeeds** (exit code 0): proceed to the next phase
- If the build **fails**: go to Step 3

### Step 3 — Fix errors

For each compiler error:

1. Read the error message to identify the file and issue
2. Read the source file
3. Apply a minimal, conservative fix
4. Consult the domain skill's references for common error-fix mappings

**Fix rules:**
- Fix ONLY compiler errors, not warnings
- Minimal changes — do not refactor working code
- Only touch the file reported in the error

After fixing, re-run the build (Step 2). Repeat up to
`KONVEYOR_PARAM_MAX_FIX_ITERATIONS` times (read from environment, default 3).

If the build still fails after max iterations: record remaining errors and
proceed to the next phase.

---

## After All Phases

1. Run the test command (if available from domain skill metadata or `implementation.md`)
2. Report test results (passed/failed/total counts)
3. Do NOT attempt to fix failing tests — document them

---

## Output

Write `.konveyor/execute.json`. See [templates/execute.md](templates/execute.md)
for the full schema and field descriptions.

Create the `.konveyor/` directory if it does not exist.

---

## Rules

- Work through ALL steps — completeness matters more than perfection
- Follow the domain skill's phase order exactly
- Run the build gate after EVERY phase — do not skip
- Fix only compiler errors, not warnings or style issues
- If you cannot fix an error after max iterations, record it and move on
- Do NOT modify `implementation.md` or `spec.md`
- Do NOT re-read `implementation.md` after every step — read it once
- If no domain skill is loaded, treat `implementation.md` as a flat step list
  and run the build after all steps are complete
