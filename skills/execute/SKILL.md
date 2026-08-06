---
name: execute
tags: [stage]
description: >
  Reads the implementation plan and executes migrations phase by phase.
  If domain skills are available, follows their phases and modules.
  Runs the build gate after each phase, fixing compiler errors before
  proceeding.
inputs:
  - .konveyor/implementation.md
  - .konveyor/questionnaire.json
  - domain skills (optional)
outputs:
  - modified source files
  - .konveyor/execute.json
---

# Execute Stage

Executes the migration by following the implementation plan. If domain skills
are available, follows their phases and modules. Runs the build gate after
each phase and fixes errors before moving on.

---

## Startup

1. Read `.konveyor/implementation.md`
2. Read `.konveyor/questionnaire.json` for context on decisions made
3. Scan `/opt/skills/` for skills with `tags: [domain]` in their frontmatter
4. If domain skills are found: read their phases, modules, and references.
   Validate that every step's `Phase:` field matches a domain skill phase —
   halt on orphans.
5. If no domain skills are found: group steps by their `Phase:` field (as
   set by the plan stage). Detect the build command from the build manifest
   if not specified in `implementation.md`.

---

## Execution Loop

Follow the phase order (from domain skill if present, or from the plan's
grouping). For each phase, select only the steps from
`.konveyor/implementation.md` whose `Phase:` field matches this phase.

### Step 1 — Apply transformations

- If domain skills are present: read the module and reference tables for this phase
- For each step belonging to this phase:
  1. Read the target file
  2. Apply transformations per the module instructions and reference tables
  3. Write the modified file
  4. Run: `git add -A && git commit -m "Step <N>: <short description of the migration change>"`
  5. Record the commit hash from the output
  6. Record step status as `applied` with the commit hash
  6. If the step cannot be applied (file missing, transformation impossible):
     record step status as `failed` with the error, and continue to the next step

### Step 2 — Build gate

Run the build command (`metadata.build_command` from domain skill, or from
`implementation.md`'s Verification section, or auto-detected from build manifest).

- If the build **succeeds** (exit code 0): go to Step 3
- If the build **fails**: go to Step 4

### Step 3 — Smoke gate (optional)

If a `smoke_command` is available (from domain skill `metadata.smoke_command`
or `implementation.md`), run it after the build gate passes. This verifies
runtime behavior — the app starts, dependencies resolve, and wiring works.

- If the smoke **succeeds** (exit code 0): proceed to the next phase
- If the smoke **fails**: go to Step 4

If no `smoke_command` is available, skip this step and proceed to the next phase.

### Step 4 — Fix errors

For each compiler error:

1. Read the error message to identify the file and issue
2. Read the source file
3. Apply a minimal, conservative fix
4. Consult the domain skill's references for common error-fix mappings (if available)
5. Run: `git add -A && git commit -m "Fix: <describe what was fixed>"`

**Fix rules:**
- Fix ONLY compiler errors, not warnings
- Minimal changes — do not refactor working code
- Only touch the file reported in the error

After fixing, re-run the build (Step 2) and smoke (Step 3). Repeat up to
3 times.

If the build still fails after 3 attempts: record remaining errors,
mark all unrun steps in later phases as `skipped` (reason: `"build gate halt"`),
and **halt execution**. Write `.konveyor/execute.json` with `status: "aborted"`
before stopping.

---

## After All Phases

1. Run the test command (if available from domain skill metadata or `.konveyor/implementation.md`)
2. Report test results (passed/failed/total counts)
3. Do NOT attempt to fix failing tests — document them

If execution was aborted, tests are skipped — record `tests.status: "skipped"`.

---

## Resume

If `.konveyor/execute.json` already exists with `status: "aborted"`, resume
from where execution stopped:

1. Read the existing `execute.json`
2. Find the phase that caused the abort (the last phase with `status: "partial"`)
3. Re-run that phase's build gate (Step 2) — the tree may have been fixed
   externally between runs
4. If the build gate passes: clear all `skipped` steps back to pending and
   continue the execution loop from the next phase
5. If the build gate fails: re-enter the fix loop (Step 3) with a fresh
   iteration count
6. Steps already marked `applied` are never re-run

## Write Output and Commit

You MUST complete this step — execution is not done until `execute.json` is
written and committed.

1. Create the `.konveyor/` directory if it does not exist
2. Write `.konveyor/execute.json` — see [templates/execute.md](templates/execute.md)
   for the full schema and field descriptions. Top-level `status` is `completed`
   when all phases finish, or `aborted` when halted by a build gate failure.
3. Commit the output:

```bash
git add .konveyor/execute.json
git commit -m "Add execute results"
```

Do NOT push.

The stage is NOT complete until `execute.json` is written and committed.

---

## Rules

- Work through all steps in the current phase before running the build gate
- Follow the phase order (from domain skill if present, or from the plan)
- Select steps by `Phase:` field — only run steps matching the current phase
- If domain skills are present: halt on orphan steps during startup validation
- Run the build gate after EVERY phase — do not skip
- Fix only compiler errors, not warnings or style issues
- Halt after 3 fix iterations — do not continue on build failure
- Commit after each step and each fix iteration — do NOT push
- You MUST write `.konveyor/execute.json` before finishing
- Do NOT modify `.konveyor/implementation.md` or `.konveyor/spec.md`
- Do NOT re-read `.konveyor/implementation.md` after every step — read it once
- If no domain skills are loaded, use the plan's phase grouping and detect
  the build command from `implementation.md` or the build manifest
