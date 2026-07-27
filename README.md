# Harness Skills

Stage skills for the Konveyor migration pipeline. Each skill is a language-agnostic process that the harness runs as a separate stage. Language-specific migration knowledge comes from domain skills (`tags: [domain]`) loaded alongside these at runtime.

## Pipeline

```
Questionnaire → Plan → Execute → Eval → Report
```

## Skills

| Skill | Description |
|---|---|
| [questionnaire](skills/questionnaire/SKILL.md) | Detects tech stack and gathers migration decisions |
| [plan](skills/plan/SKILL.md) | Analyzes the project, produces spec and implementation plan |
| [execute](skills/execute/SKILL.md) | Executes the plan phase by phase with build gates |
| [eval](skills/eval/SKILL.md) | Scores migration quality and decision outcomes |
| [report](skills/report/SKILL.md) | Generates a human-readable migration report |

## How It Works

The harness discovers all skills at `/opt/skills/*/SKILL.md`. Skills use `tags` in their frontmatter (per [skillimage.io](https://github.com/redhat-et/skillimage) spec) to signal their role:

- `tags: [stage]` — process skill. The harness follows this.
- `tags: [domain]` — migration knowledge. Stage skills consult these at runtime.
- `tags: [rule]` — always-on constraints.

Each stage reads artifacts from `.konveyor/` written by prior stages. Git is the source of truth for cross-stage data flow.

## Cross-Stage Data Flow

| Stage | Reads | Writes |
|---|---|---|
| Questionnaire | source repo | `.konveyor/questionnaire.json` |
| Plan | questionnaire.json, analysis.json, domain skills | `spec.md`, `implementation.md`, `graph.json` |
| Execute | implementation.md, domain skills | source files + `.konveyor/execute.json` |
| Eval | questionnaire.json, execute.json, implementation.md, git log | `.konveyor/eval.json` |
| Report | questionnaire.json, execute.json, eval.json, spec.md, implementation.md | `.konveyor/report.md` |

## Design

See [migration-skills.md](migration-skills.md) for the full design document.
