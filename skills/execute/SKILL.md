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

1. Read `.konveyor/implementation.md`
2. Scan `/opt/skills/` for skills with `tags: [domain]` in their frontmatter
3. Read the domain skill's phases, modules, and references
4. Read `.konveyor/questionnaire.json` for context on decisions made
5. Validate that every step in `.konveyor/implementation.md` has a `Phase:` field
   matching a domain skill phase. If any step has no match (orphan), halt with
   an error listing the orphan steps.

---

## Execution Loop

Follow the domain skill's phase order. For each phase, select only the steps
from `.konveyor/implementation.md` whose `Phase:` field matches this phase.

### Step 1 — Apply transformations

- Read the domain skill's module for this phase
- Read the domain skill's reference tables (dependency-map, api-map, config-map, pattern-map), if they exist
- For each step belonging to this phase:
  1. Read the target file
  2. Apply transformations per the module instructions and reference tables
  3. Write the modified file
  4. Git commit: `git commit -m "Step <N>: <step title>"`
  5. Record step status as `applied` in the execution log

### Step 2 — Build gate

Run the build command from the domain skill's metadata (`metadata.build_command`).

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

After fixing, commit the fixes (`git commit -m "Fix: <phase> iteration <N>"`),
then re-run the build (Step 2). Repeat up to
`KONVEYOR_PARAM_MAX_FIX_ITERATIONS` times (read from environment, default 3).

If the build still fails after max iterations: record remaining errors,
mark all unrun steps in later phases as `skipped` (reason: `"build gate halt"`),
commit the fix-iteration edits (`git commit -m "Fix: <phase> iteration <N>"`),
and **halt execution**. Write `.konveyor/execute.json` with `status: "aborted"`
before stopping.

To override this behavior and continue despite build failures, set
`KONVEYOR_PARAM_CONTINUE_ON_BUILD_FAIL=true` in the environment.

---

## After All Phases

1. Run the test command (if available from domain skill metadata or `.konveyor/implementation.md`)
2. Report test results (passed/failed/total counts)
3. Do NOT attempt to fix failing tests — document them

---

## Resume

If `.konveyor/execute.json` already exists with `status: "aborted"`, resume
from where execution stopped:

1. Read the existing `execute.json` to find the last `applied` step
2. Skip all steps already marked `applied` (their commits are in git history)
3. Continue from the next step

## Output

Write `.konveyor/execute.json`. See [templates/execute.md](templates/execute.md)
for the full schema and field descriptions.

Top-level `status` is `completed` when all phases finish, or `aborted` when
halted by a build gate failure.

Create the `.konveyor/` directory if it does not exist.

---

## Rules

- Work through all steps in the current phase before running the build gate
- Follow the domain skill's phase order exactly
- Select steps by `Phase:` field — only run steps matching the current phase
- Halt on orphan steps (no matching domain phase) during startup validation
- Run the build gate after EVERY phase — do not skip
- Fix only compiler errors, not warnings or style issues
- Halt after max fix iterations unless `KONVEYOR_PARAM_CONTINUE_ON_BUILD_FAIL=true`
- Git commit after every step
- Do NOT modify `.konveyor/implementation.md` or `.konveyor/spec.md`
- Do NOT re-read `.konveyor/implementation.md` after every step — read it once
- If no domain skill is loaded, treat `.konveyor/implementation.md` as a flat step list
  and run the build after all steps are complete
