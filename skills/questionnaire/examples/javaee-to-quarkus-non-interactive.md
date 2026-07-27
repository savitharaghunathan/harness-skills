# Example: Java EE to Quarkus Questionnaire (Non-Interactive)

Non-interactive run against [coolstore](https://github.com/savitharaghunathan/coolstore) —
a Java EE 7 monolith on JBoss EAP. The agent detects the stack and makes all
decisions autonomously, recording reasoning for each.

## Detection

Build manifest: `pom.xml` (Maven, WAR packaging, Java 8)

Framework markers found:
- `javax.ejb.*` → EJB 3.1 (`@Stateless`, `@Stateful`, `@Singleton`, `@MessageDriven`)
- `javax.persistence.*` → JPA 2.1
- `javax.ws.rs.*` → JAX-RS 2.0
- `javax.enterprise.*` → CDI 1.1
- `javax.jms.*` → JMS 2.0
- `javax.json.*` → JSON-P 1.0
- `persistence.xml`, `web.xml`, `beans.xml` → Java EE deployment descriptors
- `weblogic/` package → WebLogic vendor stubs
- Flyway migration scripts in `db/migration/`

## Decisions (autonomous)

**Decision 1: Target platform**
- A) Quarkus 3.x — Red Hat successor, first-class CDI/JAX-RS/JPA support
- B) Spring Boot 3.x — requires rewriting CDI→Spring, JAX-RS→Spring MVC
- C) Jakarta EE 10 on WildFly — minimal code changes but same app server model

Recommendation: A
Chosen: A
Reasoning: App uses CDI, JAX-RS, JPA — all map 1:1 onto Quarkus extensions.
Spring Boot would require rewriting into Spring idioms. Quarkus is the natural
Red Hat successor platform.

**Decision 2: Java version**
- A) Java 17 (Quarkus minimum)
- B) Java 21 (current LTS, recommended)

Recommendation: B
Chosen: B
Reasoning: Java 21 is the current LTS. Quarkus 3.x requires minimum 17, and 21
is the recommended target.

**Decision 3: How to handle EJB session beans?**
- A) Convert to CDI beans (`@ApplicationScoped` / `@SessionScoped`)
- B) Use quarkus-ejb compatibility extension

Recommendation: A
Chosen: A
Reasoning: CDI is Quarkus-native. `@Stateless` → `@ApplicationScoped`,
`@Stateful` → `@SessionScoped`. Direct mapping, no compatibility layer needed.

**Decision 4: How to handle JMS messaging?**
- A) SmallRye Reactive Messaging with in-memory connector
- B) Qpid JMS client (keeps JMS API)

Recommendation: A
Chosen: A
Reasoning: JMS requires a broker. The original used JBoss embedded HornetQ.
SmallRye with `smallrye-in-memory` connector preserves in-process message flow
without external infrastructure.

**Decision 5: How to handle persistence.xml?**
- A) Convert to application.properties (Quarkus-native config)
- B) Keep persistence.xml

Recommendation: A
Chosen: A
Reasoning: Quarkus configures JPA through application.properties. JNDI datasource
`java:jboss/datasources/CoolstoreDS` becomes `quarkus.datasource.jdbc.url`.
Centralizes config and enables Quarkus dev services.

**Decision 6: How to handle WebLogic/JBoss vendor code?**
- A) Delete vendor-specific code, replace with Quarkus equivalents
- B) Keep as compatibility stubs

Recommendation: A
Chosen: A
Reasoning: `weblogic/` package (3 files) are stubs with no purpose in Quarkus.
`StartupListener` (WebLogic lifecycle) replaced by `@Observes StartupEvent`.
`DataBaseMigrationStartup` (manual Flyway) replaced by `quarkus-flyway` extension.

**Decision 7: How to serve the frontend SPA?**
- A) Qute template serving index.html via JAX-RS resource
- B) Static resource serving from META-INF/resources

Recommendation: A
Chosen: A
Reasoning: App needs an HTTP session for `@SessionScoped` beans. A JAX-RS resource
that serves via Qute creates the session on first request. Pure static serving
wouldn't establish a session.

## Output

```json
{
  "detection": {
    "language": "java",
    "version": "1.8",
    "frameworks": ["java-ee-7", "ejb-3.1", "jms-2.0", "jpa-2.1", "jax-rs-2.0", "cdi-1.1", "json-p-1.0"],
    "build_tool": "maven",
    "app_server": "jboss-eap",
    "source_file_count": 25
  },
  "decisions": [
    {
      "id": 1,
      "question": "Target platform",
      "options": [
        "A) Quarkus 3.x — first-class CDI/JAX-RS/JPA, Red Hat successor",
        "B) Spring Boot 3.x — requires rewriting to Spring idioms",
        "C) Jakarta EE 10 on WildFly — minimal changes, same app server model"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "App uses CDI, JAX-RS, JPA — all map 1:1 onto Quarkus extensions"
    },
    {
      "id": 2,
      "question": "Java version",
      "options": [
        "A) Java 17 — Quarkus minimum",
        "B) Java 21 — current LTS, recommended"
      ],
      "recommendation": "B",
      "chosen": "B",
      "reasoning": "Java 21 is the current LTS, Quarkus 3.x recommends it"
    },
    {
      "id": 3,
      "question": "How to handle EJB session beans?",
      "options": [
        "A) Convert to CDI beans",
        "B) Use quarkus-ejb compatibility extension"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "CDI is Quarkus-native, direct mapping for all EJB types"
    },
    {
      "id": 4,
      "question": "How to handle JMS messaging?",
      "options": [
        "A) SmallRye Reactive Messaging with in-memory connector",
        "B) Qpid JMS client"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Preserves in-process message flow without external broker"
    },
    {
      "id": 5,
      "question": "How to handle persistence.xml?",
      "options": [
        "A) Convert to application.properties",
        "B) Keep persistence.xml"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Centralizes config, enables Quarkus dev services"
    },
    {
      "id": 6,
      "question": "How to handle WebLogic/JBoss vendor code?",
      "options": [
        "A) Delete and replace with Quarkus equivalents",
        "B) Keep as compatibility stubs"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Vendor stubs serve no purpose in Quarkus, clean equivalents exist"
    },
    {
      "id": 7,
      "question": "How to serve the frontend SPA?",
      "options": [
        "A) Qute template via JAX-RS resource",
        "B) Static resource serving from META-INF/resources"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Need HTTP session for @SessionScoped beans, JAX-RS resource creates it"
    }
  ],
  "mode": "non-interactive"
}
```
