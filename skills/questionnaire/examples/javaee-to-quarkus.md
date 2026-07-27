# Example: Java EE to Quarkus Questionnaire (Interactive)

Interactive run against [coolstore](https://github.com/savitharaghunathan/coolstore) —
a Java EE 7 monolith on JBoss EAP. The user answers each decision one at a time
and overrides several recommendations.

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
- `lib/audit-logging-library-1.0.0.jar` → custom audit library (v2.0.0 also in `lib/`)

Detection summary:
```json
{
  "language": "java",
  "version": "1.8",
  "frameworks": ["java-ee-7", "ejb-3.1", "jms-2.0", "jpa-2.1", "jax-rs-2.0", "cdi-1.1", "json-p-1.0"],
  "build_tool": "maven",
  "app_server": "jboss-eap",
  "source_file_count": 25
}
```

## Decisions (interactive — user answers one at a time)

**Decision 1: Migration goal**
- A) Modernization demo — reference app for migration tooling
- B) Production-readiness — fully functional with tests, health checks, proper config
- C) Minimal viable migration — compile and run on Quarkus with least change

Recommendation: C — smallest behavioral change, preserves existing behavior.
**Chosen: C** (user agreed)

---

**Decision 2: How to handle JMS messaging (order topic)?**
- A) Replace with direct method calls — simplest, no broker dependency
- B) SmallRye Reactive Messaging — Quarkus-native, in-memory channel preserves async pub/sub
- C) Keep JMS via Quarkus JMS extension — closest to original but requires external broker

Recommendation: A — eliminates broker dependency, simplest for minimal migration.
**Chosen: B** (user overrode — wants to preserve async decoupling)

---

**Decision 3: What to do with WebLogic code?**
- A) Delete all WebLogic code entirely — cleanest, but loses inventory notification behavior
- B) Delete stubs, convert InventoryNotificationMDB to CDI observer — preserves behavior
- C) Delete stubs, keep InventoryNotificationMDB as dead code with TODOs

Recommendation: A — the MDB was already non-functional (fake WebLogic JNDI).
**Chosen: B** (user overrode — wants to preserve inventory notification logic)

---

**Decision 4: How to handle @SessionScoped cart and @Stateful ShoppingCartService?**
- A) Keep `@SessionScoped` endpoint, convert ShoppingCartService to `@SessionScoped` CDI bean
- B) Convert to `@ApplicationScoped` with in-memory Map keyed by cartId
- C) Convert to `@ApplicationScoped` and persist cart to database

Recommendation: A — smallest behavioral change.
**Chosen: A** (user agreed)

---

**Decision 5: How to handle the custom audit-logging-library?**
- A) Keep v1.0.0 as-is — update Maven coordinates only
- B) Upgrade to v2.0.0 per Konveyor rules — switch to StreamableAuditLogger
- C) Remove audit logging entirely

Recommendation: A — minimal change.
**Chosen: B** (user overrode — v2 JAR already exists, Konveyor rules describe the upgrade)

---

**Decision 6: How to handle the frontend (AngularJS + JSPs)?**
- A) Convert JSPs to static files, serve from META-INF/resources/
- B) Use Quarkus Qute templating to replace JSPs
- C) Strip the frontend entirely, keep only REST API

Recommendation: A — JSPs do almost nothing server-side, static serving is simplest.
**Chosen: B** (user overrode — wants Qute for session creation on first request)

---

**Decision 7: Target Java version**
- A) Java 17 — Quarkus minimum, smallest jump from Java 8
- B) Java 21 — current LTS, better performance

Recommendation: B — current LTS, Konveyor rules reference it.
**Chosen: B** (user agreed)

---

**Decision 8: Migration strategy**
- A) Incremental in-place migration — modify existing files, preserves git history
- B) Scaffold new Quarkus project, port code into it — clean structure

Recommendation: A — keeps git history meaningful, more natural for minimal migration.
**Chosen: A** (user agreed)

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
      "question": "Migration goal",
      "options": [
        "A) Modernization demo — reference app for migration tooling",
        "B) Production-readiness — fully functional with tests, health checks",
        "C) Minimal viable migration — compile and run with least change"
      ],
      "recommendation": "C",
      "chosen": "C",
      "reasoning": "Smallest behavioral change, preserves existing behavior"
    },
    {
      "id": 2,
      "question": "How to handle JMS messaging (order topic)?",
      "options": [
        "A) Replace with direct method calls",
        "B) SmallRye Reactive Messaging with in-memory connector",
        "C) Keep JMS via Quarkus JMS extension"
      ],
      "recommendation": "A",
      "chosen": "B",
      "reasoning": "Preserve async decoupling between order processing and notification"
    },
    {
      "id": 3,
      "question": "What to do with WebLogic code?",
      "options": [
        "A) Delete all WebLogic code entirely",
        "B) Delete stubs, convert InventoryNotificationMDB to CDI observer",
        "C) Delete stubs, keep InventoryNotificationMDB as dead code"
      ],
      "recommendation": "A",
      "chosen": "B",
      "reasoning": "Preserve inventory notification logic via reactive messaging"
    },
    {
      "id": 4,
      "question": "How to handle @SessionScoped cart and @Stateful ShoppingCartService?",
      "options": [
        "A) Keep @SessionScoped, convert to @SessionScoped CDI bean",
        "B) Convert to @ApplicationScoped with in-memory Map",
        "C) Convert to @ApplicationScoped and persist to database"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Smallest behavioral change, works with existing AngularJS frontend"
    },
    {
      "id": 5,
      "question": "How to handle the custom audit-logging-library?",
      "options": [
        "A) Keep v1.0.0 as-is",
        "B) Upgrade to v2.0.0 per Konveyor rules",
        "C) Remove audit logging entirely"
      ],
      "recommendation": "A",
      "chosen": "B",
      "reasoning": "v2 JAR already available, Konveyor rules describe the upgrade path"
    },
    {
      "id": 6,
      "question": "How to handle the frontend (AngularJS + JSPs)?",
      "options": [
        "A) Convert JSPs to static files",
        "B) Use Quarkus Qute templating",
        "C) Strip frontend, keep only REST API"
      ],
      "recommendation": "A",
      "chosen": "B",
      "reasoning": "Qute creates HTTP session on first request for @SessionScoped beans"
    },
    {
      "id": 7,
      "question": "Target Java version",
      "options": [
        "A) Java 17 — Quarkus minimum",
        "B) Java 21 — current LTS"
      ],
      "recommendation": "B",
      "chosen": "B",
      "reasoning": "Current LTS, Konveyor rules reference it, better performance"
    },
    {
      "id": 8,
      "question": "Migration strategy",
      "options": [
        "A) Incremental in-place migration",
        "B) Scaffold new Quarkus project, port code"
      ],
      "recommendation": "A",
      "chosen": "A",
      "reasoning": "Preserves git history, more natural for minimal migration"
    }
  ],
  "mode": "interactive"
}
```
