# 🌱 Spring Bean Lifecycle — Complete Interview Notes

## 📌 What & Why

The **Bean Lifecycle** describes the complete journey of a Spring-managed object — from the moment Spring reads its definition to the moment it's destroyed. Understanding this is critical because Spring's core value proposition (Inversion of Control) depends on it: instead of *you* creating, wiring, and cleaning up objects manually, **Spring takes over that responsibility**. This topic shows up constantly in interviews because it tests whether you truly understand IoC/DI internals, not just annotation syntax.

---

## 1. From Plain Java to Spring-Managed Objects

In plain Java, when you create an object yourself, you own its **entire journey**:

```java
PaymentService paymentService = new PaymentService();
```

You are responsible for:
1. Creating the object
2. Setting required values
3. Calling required methods
4. Cleaning up resources when done

In Spring, we instead **declare** which classes/methods should be container-managed, and Spring takes over that full journey.

```java
@Component
public class PaymentService {
}
```

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

**Key takeaway:** Once a class is registered via `@Component` or `@Bean`, it becomes a Spring-managed object — a **bean** — and Spring handles discovery, creation, dependency injection, lifecycle callbacks, and destruction on your behalf.

---

## 2. Simple Definition

> **Bean lifecycle** = the complete journey of a Spring-managed object, from the moment Spring discovers its definition until the moment Spring destroys it.

A Spring-managed object is called a **bean**. So "bean lifecycle" really just means:

**How Spring creates, prepares, manages, and destroys a bean.**

---

## 3. Complete Bean Lifecycle Flow (High Level)

```
Spring container starts
        ↓
Reads configuration / annotations
        ↓
Creates BeanDefinition
        ↓
Instantiates bean object
        ↓
Injects dependencies
        ↓
Calls Aware interfaces
        ↓
Runs initialization callbacks
        ↓
Bean is ready to use
        ↓
Application uses the bean
        ↓
Runs destruction callbacks
        ↓
Bean is removed
```

---

## 4. Step 0 — Spring Container Starts

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

- We are **not** passing an object of `AppConfig` — we're passing its **class metadata**.
- Spring uses **reflection** to inspect the class and checks:
  - Does it have `@Configuration`?
  - Does it have `@ComponentScan`?
  - Does it have `@Bean` methods?
  - Which package should be scanned?
  - Which bean definitions should be registered?

This is where Spring starts understanding your application's structure.

---

## 5. Step 1 — BeanDefinition is Read

Before creating any object, Spring first collects **metadata** about each bean. This metadata object is called a **`BeanDefinition`**.

| BeanDefinition Field | Example Value |
|---|---|
| Bean name | `paymentService` |
| Bean class | `PaymentService` |
| Scope | `singleton` |
| Lazy | `false` |
| Dependencies | resolved later |
| Init method | resolved later |
| Destroy method | resolved later |

> ⚠️ At this stage, the object is **not necessarily created yet** — Spring is only registering information about the bean.

### Why does Spring need `BeanDefinition` first?

Because Spring isn't creating one isolated object — it's building a **complete object network**.

```
OrderService depends on PaymentService
PaymentService depends on PaymentGateway
PaymentGateway depends on ApiClient
```

Spring must first know: which classes to manage, bean names, scopes, dependencies, laziness, and init/destroy methods — **before** it starts creating actual objects.

### Where Bean Definitions Come From

| Source | Style | Example |
|---|---|---|
| **Annotation-based** | `@Component` + `@ComponentScan` | `@Component public class PaymentService {}` |
| **Java-based (`@Bean`)** | Explicit factory method | `@Bean public PaymentService paymentService() { return new PaymentService(); }` |
| **XML-based** | Legacy config | `<bean id="paymentService" class="in.strikes.PaymentService"/>` |

---

## 6. Step 2 — Bean Object is Instantiated

After the bean definition is read, Spring creates the actual object — this is called **instantiation**.

```java
@Component
public class PaymentService {
    public PaymentService() {
        System.out.println("PaymentService constructor called");
    }
}
```

> ⚠️ **Common Beginner Mistake:** Confusing *instantiation* with *initialization*.

| Term | Meaning |
|---|---|
| **Instantiation** | Object is created (constructor runs) |
| **Initialization** | Object is prepared **after** creation (callbacks run) |

These are **two distinct phases** in the Spring lifecycle — don't conflate them.

---

## 7. Step 3 — Dependencies are Injected

After (or during) object creation, Spring supplies required dependencies — called **Dependency Injection (DI)**. The exact timing depends on the injection type.

| Injection Type | When Dependency is Set | Example |
|---|---|---|
| **Constructor Injection** | While creating the object | `public OrderService(PaymentService paymentService)` |
| **Setter Injection** | After object creation, via setter call | `@Autowired public void setPaymentService(...)` |
| **Field Injection** | After object creation, via reflection | `@Autowired private PaymentService paymentService;` |

### Constructor Injection
```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
Flow: `Spring needs OrderService → OrderService needs PaymentService → Spring creates/finds PaymentService → Spring passes it into the constructor`

### Setter Injection
```java
@Component
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
Object created first → Spring calls the setter → dependency assigned.

### Field Injection
```java
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```
Object created first → Spring uses reflection → dependency injected directly into the field.

> ✅ **Best Practice:** Field injection is common in examples/tutorials, but **constructor injection is generally preferred** for clean, testable, immutable code (allows `final` fields, easier unit testing without reflection tricks).

---

## 8. Step 4 — Aware Interfaces are Called

Sometimes a bean needs information *about* the container itself — e.g., "What's my bean name?" or "Which `ApplicationContext` am I running in?"

Spring provides this via **`Aware` interfaces** — callback interfaces where **Spring calls a method on the bean at the right time** to hand it container info.

| Aware Interface | Purpose |
|---|---|
| `BeanNameAware` | Informs bean of its registered name |
| `ApplicationContextAware` | Informs bean of the enclosing `ApplicationContext` |

```java
@Component("myCustomBeanName")
public class MyBean implements BeanNameAware, ApplicationContextAware {

    public MyBean() {
        System.out.println("1. Constructor called");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("2. BeanNameAware called");
        System.out.println("Bean name: " + name);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        System.out.println("3. ApplicationContextAware called");
        System.out.println("ApplicationContext: " + applicationContext.getClass().getSimpleName());
    }
}
```

**Output:**
```
1. Constructor called
2. BeanNameAware called
Bean name: myCustomBeanName
3. ApplicationContextAware called
ApplicationContext: AnnotationConfigApplicationContext
```

### Why `setBeanName()` and not `getBeanName()`?

Because the **direction of communication** is: `Spring container → bean`. Spring is *giving* information to the bean, not asking for it. Calling `setBeanName()` manually would just behave like a normal Java method call — it wouldn't change anything registered inside the actual Spring container.

### Do we use `Aware` interfaces in normal business logic?

Usually **no**. They're mostly useful for framework-level code, logging utilities, custom libraries, and infrastructure components — not typical service classes. They're valuable for *learning* the lifecycle because they show how Spring communicates container info to beans.

---

## 9. Step 5 — Initialization Callbacks

At this point: the bean object is created, dependencies are injected, and Aware callbacks are done. Now Spring gives the bean a chance to run **startup logic** before it's used — this is the **initialization phase**.

Typical initialization logic: validating configuration, initializing internal caches, checking required setup, preparing resources, logging startup state.

### Three Ways to Write Initialization Logic

| Option | Mechanism | Coupling to Spring | Example Trigger |
|---|---|---|---|
| **`@PostConstruct`** | Annotation on a method | Low (just an annotation) | Called once after dependency injection |
| **`InitializingBean`** | Implement interface, override `afterPropertiesSet()` | High (implements Spring interface) | Called during initialization phase |
| **Custom `initMethod`** | Specify method name in `@Bean(initMethod = "...")` | None (POJO stays framework-free) | Called after dependency injection |

#### Option 1: `@PostConstruct`
```java
import jakarta.annotation.PostConstruct;

@Component
public class PaymentService {

    @PostConstruct
    public void init() {
        System.out.println("Bean initialized");
    }
}
```
`@PostConstruct` tells Spring: *"After dependencies are injected, call this method once."* This is clean and widely used.

> 📌 **Additional Note:** In Spring 6/7, use the `jakarta.annotation.PostConstruct` import (not the old `javax.annotation`). If you're using plain Spring Core/Context (not Spring Boot), you may need to add the dependency manually:
```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### Option 2: `InitializingBean`
```java
import org.springframework.beans.factory.InitializingBean;

@Component
public class PaymentService implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Bean initialized");
    }
}
```
Spring calls `afterPropertiesSet()` during initialization. Works, but **tightly couples** the class to Spring.

#### Option 3: Custom `initMethod`
```java
@Configuration
public class AppConfig {

    @Bean(initMethod = "start")
    public PaymentService paymentService() {
        return new PaymentService();
    }
}

public class PaymentService {
    public void start() {
        System.out.println("Bean initialized");
    }
}
```
You define your own method name; Spring calls it after DI. This keeps the class **completely free of Spring-specific interfaces** — best for framework-agnostic POJOs.

### Initialization Callback Order (if all three are used)
```
@PostConstruct
      ↓
InitializingBean.afterPropertiesSet()
      ↓
custom initMethod
```
> ⚠️ In real projects, **don't mix all three** — pick one clear approach for consistency.

---

## 10. Step 6 — Bean is Ready to Use

After initialization completes, the bean is fully ready and other beans can use it. For **singleton** beans, Spring stores the object and reuses the *same instance* every time.

```
Bean created → dependencies injected → initialization completed → bean ready
```

At this stage, the bean can safely handle business logic.

---

## 11. Step 7 — Destruction Callbacks

When the Spring container shuts down, beans get a chance to clean up resources — the **destruction phase**.

Typical cleanup logic: closing database connections, stopping background threads, releasing file handles, clearing resources, flushing pending data.

### Three Ways to Write Destruction Logic

| Option | Mechanism | Coupling to Spring |
|---|---|---|
| **`@PreDestroy`** | Annotation on a method | Low |
| **`DisposableBean`** | Implement interface, override `destroy()` | High |
| **Custom `destroyMethod`** | Specify method name in `@Bean(destroyMethod = "...")` | None |

#### Option 1: `@PreDestroy`
```java
import jakarta.annotation.PreDestroy;

@Component
public class PaymentService {

    @PreDestroy
    public void cleanup() {
        System.out.println("Cleaning up bean");
    }
}
```
Spring calls this method **before** destroying the bean.

#### Option 2: `DisposableBean`
```java
import org.springframework.beans.factory.DisposableBean;

@Component
public class PaymentService implements DisposableBean {

    @Override
    public void destroy() {
        System.out.println("Cleaning up bean");
    }
}
```
Works, but couples the class to Spring.

#### Option 3: Custom `destroyMethod`
```java
@Configuration
public class AppConfig {

    @Bean(destroyMethod = "stop")
    public PaymentService paymentService() {
        return new PaymentService();
    }
}

public class PaymentService {
    public void stop() {
        System.out.println("Cleaning up bean");
    }
}
```

### Destruction Callback Order (if all three are used)
```
@PreDestroy
      ↓
DisposableBean.destroy()
      ↓
custom destroyMethod
```
> ⚠️ Again — in real projects, choose **one** approach.

---

## 12. Full Lifecycle Summary

| # | Stage |
|---|---|
| 1 | Spring container starts |
| 2 | Spring reads configuration and annotations |
| 3 | Spring creates `BeanDefinition` |
| 4 | Spring instantiates the bean object |
| 5 | Spring injects dependencies |
| 6 | Spring calls `Aware` interfaces |
| 7 | Spring runs initialization callbacks (`@PostConstruct` → `InitializingBean` → custom `initMethod`) |
| 8 | Bean is ready to use |
| 9 | Application uses the bean |
| 10 | Spring runs destruction callbacks (`@PreDestroy` → `DisposableBean` → custom `destroyMethod`) |
| 11 | Bean is removed |

---

## 13. Singleton Bean Lifecycle

By default, Spring beans are **singleton-scoped** — Spring creates only **one object** for that bean, and manages its **complete lifecycle**: creation → dependency injection → initialization → usage → destruction.

```java
@Component
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor called");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct called");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("@PreDestroy called");
    }
}
```

**On context start:**
```
Constructor called
@PostConstruct called
```

**On context close:**
```
@PreDestroy called
```

> **Singleton summary:** Spring creates it. Spring stores it. Spring reuses it. Spring destroys it.

---

## 14. Prototype Bean Lifecycle

**Prototype** scope means a **new object is created every time the bean is requested**.

```java
@Component
@Scope("prototype")
public class PaymentService {
    public PaymentService() {
        System.out.println("Constructor called");
    }
}
```

```java
PaymentService p1 = context.getBean(PaymentService.class);
PaymentService p2 = context.getBean(PaymentService.class);
// p1 and p2 are different instances
```

### Prototype Lifecycle Flow
```
Container starts
        ↓
BeanDefinition is read
        ↓
No object is usually created at startup
        ↓
Client requests prototype bean
        ↓
Spring creates a new object
        ↓
Dependencies are injected
        ↓
Aware callbacks are called
        ↓
Initialization callbacks are called
        ↓
Spring hands object to the client
        ↓
Spring does not track it for normal destruction
```

### Singleton vs Prototype — Key Difference

| Aspect | Singleton | Prototype |
|---|---|---|
| Object count | One instance, reused | New instance per request |
| Who owns final destruction | Spring | The caller |
| Managed by container fully? | ✅ Yes | ❌ Only creation phase |
| `context.close()` triggers destroy? | ✅ Yes | ❌ No (usually) |

> **Prototype summary:** Spring creates it. Spring prepares it. Spring gives it to the caller. After that, **the caller is responsible for it.**

---

## 15. Prototype Destruction Does Not Happen Automatically

```java
@Component
@Scope("prototype")
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor called");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct called");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("@PreDestroy called");
    }
}
```

```java
AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

PaymentService p1 = context.getBean(PaymentService.class);

context.close();
```

**Expected output:**
```
Constructor called
@PostConstruct called
```
Notice: **`@PreDestroy called` will NOT print.**

### Why doesn't `@PreDestroy` run for prototype beans?

Because once Spring hands a prototype object to the caller, it **stops tracking that specific instance**.

| Scope | Spring's Mental Model |
|---|---|
| **Singleton** | "I created this one object. It's stored in my singleton cache. When the context closes, I can destroy it." |
| **Prototype** | "I created this object, prepared it, gave it to the caller. Now the caller may use it, store it, discard it, or make many more copies. I will not track all prototype instances for automatic destruction." |

### Why doesn't Spring just track every prototype object?

Technically it *could* — but prototype beans can be created **many times**. If Spring held a reference to every prototype instance forever, none of them could ever be garbage collected → **memory leak**. So Spring intentionally does **not** fully manage prototype destruction.

### Does `context.close()` destroy prototype beans?

**Usually no.** It destroys singleton beans (registered & tracked), but prototype instances already handed out are **not automatically destroyed**. If a prototype bean holds expensive resources:
- Close the resource manually
- Use try-with-resources where possible
- Avoid putting heavy cleanup responsibility inside prototype beans

> 📌 Java can still garbage-collect the prototype object once there are no references to it — but **garbage collection ≠ Spring destruction callbacks**. They are unrelated mechanisms.

---

## 16. Important Case: Singleton Depends on Prototype

Scenario: `OrderService` is **singleton**, `PaymentSession` is **prototype**, and `OrderService` depends on `PaymentSession`.

```java
@Component
public class OrderService {

    private final PaymentSession paymentSession;

    public OrderService(PaymentSession paymentSession) {
        this.paymentSession = paymentSession;
    }
}

@Component
@Scope("prototype")
public class PaymentSession {

    public PaymentSession() {
        System.out.println("PaymentSession created");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("PaymentSession destroyed");
    }
}
```

Since `OrderService` is a singleton, it's created **only once**. At that moment, Spring creates **one** `PaymentSession` and injects it — and the **same instance stays inside `OrderService` forever**.

> ⚠️ **Gotcha:** `PaymentSession` being prototype-scoped does **NOT** automatically mean a fresh instance is created every time a method runs inside `OrderService`. Scope only controls creation-time behavior, not runtime behavior.


---

## 17. `@Lazy` and Bean Lifecycle

By default, **singleton beans are eagerly created** — as soon as the Spring container starts, singleton beans are created immediately.

Marking a singleton `@Lazy` changes this: Spring reads its `BeanDefinition` but **delays object creation** until the bean is actually requested.

```java
@Component
@Lazy
public class PaymentService {
    public PaymentService() {
        System.out.println("PaymentService created");
    }
}
```

### `@Lazy` Flow
```
Container starts
        ↓
BeanDefinition is registered
        ↓
Object is not created yet
        ↓
Bean is created only when someone asks for it
```

| Aspect | Eager Singleton (default) | `@Lazy` Singleton |
|---|---|---|
| BeanDefinition read | At startup | At startup |
| Object creation | Immediately at startup | Delayed until first use |
| DI + initialization | At startup | Delayed until first use |

---

## 18. `@PostConstruct` and Circular Dependency

Circular dependency example that **fails with constructor injection**:

```java
@Component
public class A {
    B b;
    public A(B b) { }
}

@Component
public class B {
    A a;
    public B(A a) { }
}
```

Spring cannot create `A` without `B`, and cannot create `B` without `A` → **constructor circular dependency fails** (Spring throws `BeanCurrentlyInCreationException`).

### Where `@PostConstruct` can help

```java
@Component
public class A {
    B b;

    public A(B b) {
        this.b = b;
    }

    @PostConstruct
    public void setB() {
        b.setA(this);
    }
}

@Component
public class B {
    A a;

    public void setA(A a) {
        this.a = a;
    }
}
```
- `B` no longer requires `A` in its constructor — breaking the circular constructor chain.
- `A`'s `@PostConstruct` method wires itself into `B` **after** both objects already exist — sidestepping the circular creation problem.

> ⚠️ **Best Practice:** Circular dependencies (even when "fixed" this way) usually indicate a **design smell**. Prefer refactoring to remove the cycle over patching it with setter/`@PostConstruct` wiring.

---

## 19. Final Revision Summary

Bean lifecycle is the **complete journey** of a Spring bean. Main phases:

```
BeanDefinition reading
        ↓
object creation
        ↓
dependency injection
        ↓
Aware callbacks
        ↓
BeanPostProcessor (before initialization)
        ↓
initialization callbacks
        ↓
BeanPostProcessor (after initialization)
        ↓
bean ready
        ↓
destruction callbacks
```

> 📌 **Additional Note:** `BeanPostProcessor` wasn't detailed in the original notes but is a core extension point in the real lifecycle — it lets Spring (or custom code) modify/wrap beans immediately **before** and **after** initialization callbacks run. This is how features like AOP proxies and `@Autowired` field processing are implemented internally. Worth knowing for deeper interview questions: `postProcessBeforeInitialization()` and `postProcessAfterInitialization()`.

| Scope | Lifecycle Ownership |
|---|---|
| **Singleton** | Spring manages the **full** lifecycle — creation to destruction |
| **Prototype** | Spring creates, injects, initializes, and hands off — **no automatic destruction** |
| **`@Lazy` Singleton** | `BeanDefinition` read at startup; object creation **delayed** until actually needed |
| **`@PostConstruct`** | Runs logic **after** dependency injection; part of initialization phase |

---

## 🎯 Interview Questions & Answers

**Q1: What is the Spring Bean Lifecycle?**
It's the complete journey of a Spring-managed object — from BeanDefinition creation, through instantiation, dependency injection, Aware callbacks, initialization, active use, and finally destruction. Spring automates all of this instead of the developer manually managing object creation and cleanup.

**Q2: What's the difference between bean instantiation and initialization?**
Instantiation is when the constructor runs and the object is created in memory. Initialization happens *after* — it's when Spring runs callbacks (`@PostConstruct`, `InitializingBean`, custom init methods) to prepare the bean for use, often after dependencies are already injected. They are distinct, sequential phases.

**Q3: Why does a prototype-scoped bean not get destroyed when the ApplicationContext closes?**
Because Spring only creates and initializes prototype beans — once handed to the caller, Spring stops tracking that instance. Tracking every prototype instance forever would prevent garbage collection and cause memory leaks, so Spring deliberately does not manage the full prototype lifecycle, including destruction.



---
