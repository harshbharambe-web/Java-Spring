# Spring Boot Configuration — Properties, YAML & Profiles

---

## 1. Why Configuration Is Needed

In real applications, values like server port, DB URL, username, password, app name, and environment-specific settings **change depending on where the app runs** (local machine, testing server, staging, production).

If these values are hardcoded in Java code:
- Every environment change requires a **code change + recompilation + redeployment**
- Sensitive data (passwords, keys) ends up **committed to source control**
- The same `.jar`/`.war` **cannot be reused** across dev/test/prod — you'd need a separate build per environment

**Core idea:** Separate *what the app does* (code) from *how/where it runs* (configuration). This is the **externalized configuration** principle — configuration lives outside the compiled code so it can change without touching the code.

---

## 2. Problems With Hardcoding Values

| Problem | Explanation |
|---|---|
| Not portable | Code tied to one environment (e.g., localhost DB) breaks on another machine |
| Security risk | Passwords/API keys visible in source code, pushed to Git |
| Redeployment needed | Any config change forces a full rebuild + redeploy |
| No environment flexibility | Can't run same build in dev/test/prod with different settings |
| Violates separation of concerns | Business logic mixed with environment-specific detail |

**Interview one-liner:** *"Hardcoding breaks the principle of externalized configuration — Spring Boot lets us inject values at runtime instead of compile time."*

---

## 3. `application.properties`

Default configuration file Spring Boot looks for at `src/main/resources/application.properties`. Spring Boot **auto-loads** it at startup — no extra setup needed.

```properties
server.port=8081
spring.application.name=demo-app
app.welcome.message=Welcome to Spring Boot!
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=root123
```

Spring Boot's auto-configuration classes read these standard `spring.*` keys automatically (e.g., `server.port` configures the embedded Tomcat port).

---

## 4. `@Value` Annotation

Used to inject a property value from `application.properties`/`.yml` into a field.

```java
@RestController
public class WelcomeController {

    @Value("${app.welcome.message}")
    private String welcomeMessage;

    @Value("${app.retry.count:3}")   // default value = 3 if key missing
    private int retryCount;

    @GetMapping("/welcome")
    public String getMessage() {
        return welcomeMessage;
    }
}
```

Key points:
- `${key}` — SpEL-style placeholder syntax
- `${key:defaultValue}` — provides a fallback if property is missing (avoids startup failure)
- Injected **after** bean construction but **before** it's ready for use (works with field/setter injection, not with constructor param easily unless combined properly)

---

## 5. Limitations of `@Value` / `.properties`

| Limitation | Why it matters |
|---|---|
| Scattered across classes | Each `@Value` field is a separate injection point — hard to track/refactor |
| No type-safety / grouping | Can't easily bind a group of related properties to one object |
| No validation | Typos in property keys fail silently (empty/null) or throw at runtime |
| Flat key-value structure | `.properties` files get repetitive for nested/related config: |

```properties
app.db.host=localhost
app.db.port=3306
app.db.name=mydb
```
vs the cleaner alternative → **YAML**.

> Interview tip: The industry-preferred alternative to scattered `@Value` is `@ConfigurationProperties`, which binds a whole prefix to a POJO — good follow-up point even though today's lesson focused on `@Value`.

---

## 6. Introduction to YAML (`application.yml`)

YAML = "YAML Ain't Markup Language" — a **hierarchical**, human-readable data format used as an alternative to `.properties`.

```yaml
server:
  port: 8081

spring:
  application:
    name: demo-app
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: root123

app:
  welcome:
    message: Welcome to Spring Boot!
```

Spring Boot auto-detects `application.yml` on the classpath the same way it detects `.properties` — **no extra configuration required**. (If both exist, `.properties` takes precedence.)

---

## 7. `.properties` vs `.yml`

| Aspect | `.properties` | `.yml` |
|---|---|---|
| Structure | Flat, dot-separated keys | Hierarchical (nested indentation) |
| Readability | Repetitive for nested config | Cleaner, groups related keys visually |
| Lists/Arrays | Verbose (`app.servers[0]=a`) | Native, concise (`- a`) |
| Multiple docs in one file | Not supported | Supported via `---` separator (used heavily for profiles) |
| Comment syntax | `#` | `#` |
| Syntax strictness | Lenient | Strict — indentation errors break parsing |
| Industry preference | Simple apps | Preferred for microservices / multi-env apps |

---

## 8. YAML Indentation Rules

YAML structure is defined **purely by indentation** — this is the #1 source of bugs for beginners.

Rules:
1. Use **spaces only, never tabs** (tabs cause parse errors)
2. Consistent indent size (commonly 2 spaces) at each nesting level
3. Child keys must be indented **further right** than their parent
4. Sibling keys must align at the **same indentation level**
5. Colon `:` must be followed by a space before the value (`port: 8081`, not `port:8081`)

```yaml
# Correct
server:
  port: 8081
  servlet:
    context-path: /api

# Wrong (misaligned siblings) — will parse incorrectly
server:
  port: 8081
   servlet:      # extra space breaks the hierarchy
    context-path: /api
```

**Interview one-liner:** *"YAML is indentation-sensitive — a single misplaced space changes the object hierarchy silently, whereas .properties fails loudly (wrong key = no value found)."*

---

## 9. Converting `.properties` → `.yml`

`.properties`:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=root123
```

Equivalent `.yml` (common prefix `spring.datasource` becomes a nested block):
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: root123
```

**Method:** group keys by shared dot-prefix → each dot becomes one indentation level → last segment becomes the leaf key with its value.

---

## 10. Lists & Nested Configuration in YAML

```yaml
app:
  servers:
    - server1.example.com
    - server2.example.com
    - server3.example.com

  team:
    lead: Harsh
    members:
      - Alice
      - Bob

  cors:
    allowed-origins:
      - http://localhost:3000
      - https://myapp.com
```

Equivalent `.properties` (much clunkier, index-based):
```properties
app.servers[0]=server1.example.com
app.servers[1]=server2.example.com
app.servers[2]=server3.example.com
```

Binding a list in Java (typically via `@ConfigurationProperties`):
```java
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private List<String> servers;
    // getters/setters
}
```

---

## 11. Why Spring Profiles Are Needed

An application behaves differently across environments:

| Environment | DB | Logging | External APIs |
|---|---|---|---|
| dev | local H2/MySQL | verbose (DEBUG) | mock/sandbox |
| test | test DB | moderate | stubbed |
| staging | staging DB | INFO | staging endpoints |
| prod | production DB | minimal (WARN/ERROR) | live/production |

**Spring Profiles** let you maintain **separate configuration sets** and switch between them **without changing code** — just by activating a different profile name at startup.

---

## 12. Profile-Specific Files

Spring Boot auto-recognizes files named `application-{profile}.yml` (or `.properties`):

```
src/main/resources/
 ├── application.yml            # common/default config
 ├── application-dev.yml        # dev-only overrides
 ├── application-prod.yml       # prod-only overrides
 └── application-test.yml
```

`application.yml` (common):
```yaml
spring:
  application:
    name: demo-app
```

`application-dev.yml`:
```yaml
server:
  port: 8081
logging:
  level:
    root: DEBUG
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db
```

`application-prod.yml`:
```yaml
server:
  port: 80
logging:
  level:
    root: WARN
spring:
  datasource:
    url: jdbc:mysql://prod-server:3306/prod_db
```

**Merge behavior:** `application.yml` (base/common) always loads first; the active profile's file is loaded on top and **overrides matching keys**.

You can also do this in a **single file** using `---` document separators (YAML multi-doc):
```yaml
spring:
  application:
    name: demo-app
---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8081
---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
```

---

## 13. Activating Profiles

**a) Inside `application.properties`/`.yml`:**
```properties
spring.profiles.active=dev
```

**b) Command line (when running jar):**
```bash
java -jar app.jar --spring.profiles.active=prod
```

**c) Environment variable:**
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

**d) IDE run configuration** — VM options or program arguments: `-Dspring.profiles.active=dev`

**Precedence (highest wins):** command-line args > environment variable > `application.properties` value > default.

Multiple profiles can be active together: `spring.profiles.active=dev,debug`

---

## 14. Profile-Specific Files vs `@Profile` Annotation

| Aspect | Profile-specific files (`application-dev.yml`) | `@Profile` annotation |
|---|---|---|
| Purpose | Environment-specific **property values** | Environment-specific **bean creation** |
| Applies to | Configuration keys (ports, URLs, logging levels) | Java classes/methods (`@Component`, `@Bean`) |
| Granularity | File-level | Class/method-level, conditional logic |
| Example use | Different DB URL per environment | Load a mock payment service in dev, real one in prod |

---

## 15. `@Profile` — Conditional Bean Creation

`@Profile` tells Spring to **only register a bean** if a given profile is active.

```java
public interface PaymentService {
    void pay();
}

@Component
@Profile("dev")
public class MockPaymentService implements PaymentService {
    public void pay() {
        System.out.println("Mock payment processed (dev)");
    }
}

@Component
@Profile("prod")
public class RealPaymentService implements PaymentService {
    public void pay() {
        System.out.println("Real payment gateway called (prod)");
    }
}
```

If `spring.profiles.active=dev`, Spring's container **only creates** `MockPaymentService` as a bean; `RealPaymentService` is skipped entirely (not just unused — never instantiated).

Also works on `@Bean` methods inside `@Configuration` classes:
```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() { ... }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() { ... }
}
```

`@Profile("!prod")` — negation syntax to mean "active in every profile except prod".

---

## 16. Conceptual Diagram (Mental Model)

```
                ┌────────────────────────────┐
                │      application.yml        │   ← common/default config
                └──────────────┬──────────────┘
                               │ loaded first
                               ▼
        spring.profiles.active = dev / prod / test
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                       ▼
application-dev.yml   application-prod.yml    application-test.yml
   (overrides)             (overrides)             (overrides)
        │                      │                       │
        └──────────────────────┴───────────────────────┘
                               │
                               ▼
                 Final merged Environment
                               │
                               ▼
         @Value / @ConfigurationProperties inject values
         @Profile decides WHICH beans get created
```

---

## 17. Common Pitfalls

- Using **tabs** instead of spaces in YAML → parsing errors
- Forgetting the space after `:` in YAML (`port:8081` is invalid)
- Assuming `application-dev.yml` fully **replaces** `application.yml` — it only **overrides overlapping keys**; non-overlapping keys from the base file still apply
- Hardcoding `spring.profiles.active` inside `application.yml` and then forgetting it overrides your command-line flag intention (actually command-line wins, but many beginners get confused about precedence order)
- Forgetting `@Profile` beans are **not created at all** for inactive profiles — code referencing them via `@Autowired` on an interface with no matching-profile bean active will fail with `NoSuchBeanDefinitionException` if no profile matches
- Mixing `.properties` and `.yml` for the same keys — `.properties` silently wins, causing confusion about "why is my YAML not applying?"

---

## 18. Interview Q&A Quick-Fire

**Q: Why avoid hardcoding configuration values?**
A: Breaks portability, exposes secrets in source control, forces rebuild/redeploy for every environment change.

**Q: What does `@Value("${key:default}")` do?**
A: Injects the property `key`; if not found, falls back to `default` instead of failing at startup.

**Q: Why did YAML replace `.properties` in many projects?**
A: Hierarchical structure avoids repeating common prefixes, native list support, and multi-document (`---`) syntax supports profile blocks in a single file.

**Q: What's the #1 gotcha with YAML?**
A: It's whitespace-sensitive — misaligned indentation silently changes structure instead of throwing a clear error.

**Q: How do Spring Profiles solve the multi-environment problem?**
A: They let you define environment-specific property overrides (`application-{profile}.yml`) and swap between them at runtime via `spring.profiles.active`, without touching code.

**Q: Three ways to activate a profile?**
A: In the properties file itself, via `--spring.profiles.active=` command-line arg, or via `SPRING_PROFILES_ACTIVE` environment variable — command-line has highest precedence.

**Q: Difference between profile-specific files and `@Profile`?**
A: Files control *property values* per environment; `@Profile` controls *which beans get instantiated* per environment — they solve config and object-creation problems respectively, and are often used together.

**Q: What happens if no bean matches the active profile for an `@Autowired` dependency?**
A: Spring throws `NoSuchBeanDefinitionException` at startup since no bean was ever registered for the container to inject.

---

## 19. Practice Questions

1. Convert this `.properties` block into equivalent YAML:
   ```properties
   app.mail.host=smtp.gmail.com
   app.mail.port=587
   app.mail.auth=true
   ```
2. Write a `@ConfigurationProperties` class that binds a `app.servers` YAML list into a `List<String>` field.
3. Create two `@Profile`-annotated implementations of a `NotificationService` (`EmailNotification` for prod, `ConsoleNotification` for dev) and wire them via constructor injection into a controller.
4. What would happen if `spring.profiles.active=dev` is set in `application.yml` AND `--spring.profiles.active=prod` is passed on the command line? Which wins and why?
5. Write an `application-staging.yml` that inherits DB config from `application.yml` but overrides only the logging level to `INFO`.
