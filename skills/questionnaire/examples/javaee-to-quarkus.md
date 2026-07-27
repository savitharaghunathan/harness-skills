# Example: Java EE to Quarkus Questionnaire

This shows what a questionnaire run looks like for a Java EE 7 → Quarkus 3 migration.

## Detection

Build manifest: `pom.xml` (Maven)

Framework markers found:
- `javax.ejb.*` imports → EJB 3.1
- `javax.jms.*` imports → JMS 1.1
- `javax.persistence.*` imports → JPA 2.1
- `persistence.xml` → JPA config
- `web.xml` → Servlet config
- `weblogic-ejb-jar.xml` → WebLogic app server descriptor

Detection summary:
```json
{
  "language": "java",
  "version": "1.8",
  "frameworks": ["java-ee-7", "ejb-3.1", "jms-1.1", "jpa-2.1", "servlet-3.1"],
  "build_tool": "maven",
  "app_server": "weblogic",
  "source_file_count": 42
}
```

## Decisions

**Decision 1: Migration goal**
- A) Full migration — convert everything to Quarkus idioms
- B) Minimal viable — get it compiling on Quarkus with compatibility extensions
- C) Lift and shift — repackage with minimal code changes

Recommendation: B — gets a working baseline faster, can iterate from there.

**Decision 2: How to handle EJB session beans?**
- A) Convert to CDI beans (recommended for most cases)
- B) Use quarkus-ejb compatibility extension (less work, but limited)

Recommendation: A — CDI is the Quarkus-native approach and most EJB patterns map cleanly.

**Decision 3: How to handle JMS messaging?**
- A) SmallRye Reactive Messaging (Quarkus-native, reactive)
- B) Qpid JMS client (keeps JMS API, simpler migration)

Recommendation: depends on whether the team wants to adopt reactive patterns.

**Decision 4: How to handle persistence.xml?**
- A) Convert to application.properties (Quarkus-native config)
- B) Keep persistence.xml (supported but not idiomatic)

Recommendation: A — centralizes config and enables Quarkus dev services.

## Output

```json
{
  "detection": {
    "language": "java",
    "version": "1.8",
    "frameworks": ["java-ee-7", "ejb-3.1", "jms-1.1", "jpa-2.1", "servlet-3.1"],
    "build_tool": "maven",
    "app_server": "weblogic",
    "source_file_count": 42
  },
  "decisions": [
    {
      "id": 1,
      "question": "Migration goal",
      "options": [
        "A) Full migration — convert everything to Quarkus idioms",
        "B) Minimal viable — get it compiling with compatibility extensions",
        "C) Lift and shift — repackage with minimal code changes"
      ],
      "recommendation": "B",
      "chosen": "B",
      "reasoning": "Gets a working baseline faster, can iterate toward full migration"
    },
    {
      "id": 2,
      "question": "How to handle EJB session beans?",
      "options": [
        "A) Convert to CDI beans",
        "B) Use quarkus-ejb compatibility extension"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "CDI is Quarkus-native, most EJB patterns map cleanly"
    },
    {
      "id": 3,
      "question": "How to handle JMS messaging?",
      "options": [
        "A) SmallRye Reactive Messaging",
        "B) Qpid JMS client"
      ],
      "recommendation": "B",
      "chosen": "A",
      "reasoning": "Team wants to adopt reactive patterns going forward"
    },
    {
      "id": 4,
      "question": "How to handle persistence.xml?",
      "options": [
        "A) Convert to application.properties",
        "B) Keep persistence.xml"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Centralizes config, enables Quarkus dev services"
    }
  ],
  "mode": "interactive"
}
```
