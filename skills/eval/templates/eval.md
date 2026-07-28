# Eval Output Format

The eval stage writes `.konveyor/eval.json` with build/test scores, a
per-decision effectiveness breakdown, and pattern observations.

## Schema

```json
{
  "run_status": "completed",
  "scores": {
    "build": "pass",
    "test_pass_rate": 0.83,
    "completeness": 0.85,
    "steps_applied": 17,
    "steps_skipped": 3,
    "steps_failed": 0,
    "total_fix_iterations": 5,
    "remaining_errors": 0
  },
  "decision_scores": [
    {
      "decision_id": 1,
      "question": "<question from questionnaire>",
      "chosen": "<chosen option>",
      "outcome": "effective",
      "notes": "<why this outcome>"
    },
    {
      "decision_id": 2,
      "question": "<question from questionnaire>",
      "chosen": "<chosen option>",
      "outcome": "struggled",
      "notes": "<why this outcome>"
    }
  ],
  "patterns": {
    "worked": ["<phase/pattern that completed cleanly>"],
    "struggled": ["<phase/pattern that needed multiple fix iterations>"],
    "failed": []
  }
}
```

## Fields

### scores

| Field | Description |
|---|---|
| `run_status` | `completed` or `aborted`, from `execute.json` top-level `status` |
| `build` | Final build status from `execute.json` (`pass` or `fail`) |
| `test_pass_rate` | Passed tests / total tests, from `execute.json`. `null` if tests were skipped (e.g. aborted run) |
| `completeness` | `steps_applied / (steps_applied + steps_failed)`. Steps skipped due to build gate halt are excluded — they were never attempted |
| `steps_applied` | Count of steps with `status: "applied"` |
| `steps_skipped` | Count of steps with `status: "skipped"` (not counted against completeness) |
| `steps_failed` | Count of steps with `status: "failed"` |
| `total_fix_iterations` | Sum of `fix_iterations` across all phases |
| `remaining_errors` | Count of unresolved compiler errors across all phases |

### decision_scores

One entry per decision in `questionnaire.json`. Outcomes are **correlational** —
they describe what execution results are associated with each decision, not
what each decision caused.

| Field | Description |
|---|---|
| `decision_id` | Matches the `id` in `questionnaire.json` |
| `question` | The decision as stated in `questionnaire.json` |
| `chosen` | The option that was selected |
| `outcome` | `effective`, `struggled`, or `failed` |
| `notes` | What phase outcomes are associated with this decision and why |

### patterns

Free-form observations about which phases or approaches went smoothly versus
which needed rework, for use in the final report.

| Field | Description |
|---|---|
| `worked` | Phases or patterns that completed without fix iterations |
| `struggled` | Phases or patterns that needed multiple fix iterations |
| `failed` | Phases or patterns that never reached a passing build |
