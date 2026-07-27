---
name: questionnaire
tags: [stage]
description: >
  Scans a project to detect its tech stack and gathers migration decisions
  through structured questions. Outputs questionnaire.json. Use as the first
  stage before planning.
---

# Questionnaire Stage

Scans the project to understand what it is, then asks the migration decisions
that need to be made before planning can begin.

---

## Phase 1 — Detect

Lightweight automated scan. No graphify, no deep analysis.

1. Read the build manifest:
   - Java: `pom.xml` or `build.gradle`
   - JavaScript/TypeScript: `package.json`
   - Go: `go.mod`
   - .NET: `*.csproj` or `*.sln`
   - Python: `pyproject.toml`, `setup.py`, or `requirements.txt`
   - Rust: `Cargo.toml`

2. Scan source files for framework markers:
   - Imports and annotations (e.g. `javax.ejb`, `@SpringBootApplication`)
   - Config files (e.g. `persistence.xml`, `web.xml`, `application.yml`)
   - App server descriptors (e.g. `weblogic.xml`, `jboss-web.xml`)

3. Summarize what you found:
   - Language and version
   - Frameworks and libraries
   - Build tool
   - Application server (if any)
   - Approximate source file count

---

## Phase 2 — Gather Decisions

Based on what you detected, identify the decisions that need to be made before
planning can begin. These are choices where multiple valid approaches exist and
the right answer depends on the project's constraints.

Ask decisions **one at a time**. For each:
- State the decision clearly
- List 2-4 concrete options
- Explain trade-offs briefly
- Give your recommendation and why

Wait for the answer before asking the next question.

### Non-interactive mode

If `KONVEYOR_PARAM_INTERACTIVE` is `false` or not set:
choose the best option for each decision and record your reasoning.
Do not wait for answers — proceed through all decisions autonomously.

---

## Output

Write `.konveyor/questionnaire.json` with this structure:

```json
{
  "detection": {
    "language": "java",
    "version": "1.8",
    "frameworks": ["java-ee-7", "ejb-3.1", "jms-1.1"],
    "build_tool": "maven",
    "app_server": "weblogic",
    "source_file_count": 42
  },
  "decisions": [
    {
      "id": 1,
      "question": "...",
      "options": ["A) ...", "B) ...", "C) ..."],
      "recommendation": "A",
      "chosen": "B",
      "reasoning": "..."
    }
  ],
  "mode": "interactive"
}
```

Create the `.konveyor/` directory if it does not exist.

---

## Rules

- Do NOT run graphify or read analysis.json — that is the plan stage's job
- Do NOT modify any source files
- Do NOT start planning — only detect and gather decisions
- Keep detection lightweight — read manifests and scan imports, nothing more
