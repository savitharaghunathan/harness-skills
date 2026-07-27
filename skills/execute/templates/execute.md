# Execute Output Format

The execute stage writes `.konveyor/execute.json` with per-phase results,
the final build status, and test results.

## Schema

```json
{
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
| `command` | The build command used, from the domain skill's `metadata.build_tool` |
| `errors` | Any errors still outstanding at the end of execution |

### tests

Result of the post-execution test run. `skipped` if no test command was available.

| Field | Description |
|---|---|
| `status` | `pass`, `fail`, or `skipped` |
| `passed` / `failed` / `total` | Test counts |
| `failures` | Failing test names with error and line number |
