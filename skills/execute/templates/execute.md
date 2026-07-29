# Execute Output Format

The execute stage writes `.konveyor/execute.json` with the overall run status,
per-step results, per-phase results, the final build status, and test results.

## Schema

```json
{
  "status": "completed|aborted",
  "steps": [
    {
      "step": 1,
      "title": "<step title>",
      "phase": "<domain-skill-phase-name>",
      "file": "<file path>",
      "status": "applied",
      "commit": "<short commit hash>"
    },
    {
      "step": 2,
      "title": "<step title>",
      "phase": "<domain-skill-phase-name>",
      "file": "<file path>",
      "status": "skipped",
      "reason": "<why skipped>"
    },
    {
      "step": 3,
      "title": "<step title>",
      "phase": "<domain-skill-phase-name>",
      "file": "<file path>",
      "status": "failed",
      "error": "<error description>"
    }
  ],
  "phases": [
    {
      "name": "<phase-1-name>",
      "status": "success",
      "fix_iterations": 0
    },
    {
      "name": "<phase-2-name>",
      "status": "success",
      "fix_iterations": 2,
      "errors_fixed": 5
    },
    {
      "name": "<phase-3-name>",
      "status": "partial",
      "fix_iterations": 3,
      "remaining_errors": ["<file>:<line> — <error description>"]
    }
  ],
  "build": {
    "status": "pass|fail",
    "command": "<build command from domain skill>",
    "errors": []
  },
  "smoke": {
    "status": "pass|fail|skipped",
    "command": "<smoke command from domain skill, if provided>"
  },
  "tests": {
    "status": "pass|fail|skipped",
    "passed": 10,
    "failed": 2,
    "total": 12,
    "failures": ["<TestName>.<method> — <error> at line <N>"]
  }
}
```

## Fields

### status

| Value | Description |
|---|---|
| `completed` | All phases finished (build may still have failed on the last phase if `CONTINUE_ON_BUILD_FAIL` is set) |
| `aborted` | Execution halted due to a build or smoke gate failure |

### steps

One entry per step from `.konveyor/implementation.md`, in execution order.
On abort, all unrun steps are recorded with `status: "skipped"` and
`reason: "build gate halt"`.

| Field | Description |
|---|---|
| `step` | Step number from the implementation plan |
| `title` | Step title from the implementation plan |
| `phase` | Domain skill phase this step belongs to |
| `file` | File path this step operates on |
| `status` | `applied`, `skipped`, or `failed` |
| `reason` | Why the step was skipped (present when `status` is `skipped`) |
| `error` | Error description (present when `status` is `failed`) |
| `commit` | Short commit hash (present when `status` is `applied`) |

### phases

One entry per domain skill phase, in execution order.

| Field | Description |
|---|---|
| `name` | Phase name from the domain skill |
| `status` | `success` if the build gate passed, `partial` if errors remain after max fix iterations |
| `fix_iterations` | Number of fix-and-rebuild cycles run for this phase |
| `errors_fixed` | Count of compiler errors resolved (omit if zero) |
| `remaining_errors` | Unresolved errors, only present when `status` is `partial` |

### build

Final build status after all phases complete.

| Field | Description |
|---|---|
| `status` | `pass` or `fail` |
| `command` | The build command used, from the domain skill's `metadata.build_command` |
| `errors` | Any errors still outstanding at the end of execution |

### smoke

Result of the smoke gate. `skipped` if the domain skill does not provide `metadata.smoke_command`.

| Field | Description |
|---|---|
| `status` | `pass`, `fail`, or `skipped` |
| `command` | The smoke command used, from the domain skill's `metadata.smoke_command` |

### tests

Result of the post-execution test run. `skipped` if no test command was available.

| Field | Description |
|---|---|
| `status` | `pass`, `fail`, or `skipped` |
| `passed` / `failed` / `total` | Test counts |
| `failures` | Failing test names with error and line number |
