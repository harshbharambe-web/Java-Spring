# Circular Dependency

## 1. What & Why

A circular dependency happens when Bean A needs Bean B to be constructed, and Bean B needs Bean A — a loop. The container gets stuck: to build A it needs a finished B, but to build B it needs a finished A.

> Analogy: two people each refusing to sit down until the other one sits first. Neither ever sits — unless someone breaks the deadlock by half-standing (setter injection) instead of demanding to be fully seated (constructor injection) before acting.

**With constructor injection, this fails at startup** with `BeanCurrentlyInCreationException`, because a constructor *requires* a fully-built object to be passed in — there's no way to hand over a "half-built" A while B is still being constructed.

**With setter/field injection, Spring can resolve it**, because the container can:
1. Create a raw, empty instance of A (no dependencies set yet).
2. Start creating B, and inject that raw A reference into B via setter.
3. Finish building A, then inject the now-complete B into A via setter.

This works because setters run *after* the object already exists — the container can hand over a reference before it's fully wired, then fill in the fields moments later.

## 2. Code Demo

**Broken — constructor injection, fails at startup:**

```java
@Component
public class AService {
    private final BService bService;
    public AService(BService bService) { this.bService = bService; }
}

@Component
public class BService {
    private final AService aService;
    public BService(AService aService) { this.aService = aService; }
}
// Startup error: 
// "Error creating bean with name 'aService': Requested bean is currently in creation:
//  Is there an unresolvable circular reference?"
```

**Fix 1 — Setter Injection (lets Spring resolve it):**

```java
@Component
public class AService {
    private BService bService;

    @Autowired
    public void setBService(BService bService) { this.bService = bService; }
}

@Component
public class BService {
    private AService aService;

    @Autowired
    public void setAService(AService aService) { this.aService = aService; }
}
// Works — but this is treating a symptom. A circular dependency
// usually means A and B are too tightly coupled and should be redesigned.
```

**Fix 2 — `@Lazy` (defers resolution of one side until it's actually used):**

```java
@Component
public class AService {
    private final BService bService;

    public AService(@Lazy BService bService) { this.bService = bService; }
    // Spring injects a proxy here instead of the real BService immediately.
    // The real BService is only fully resolved the first time a method is called on it.
}

@Component
public class BService {
    private final AService aService;
    public BService(AService aService) { this.aService = aService; }
}
```

**Fix 3 — the actual best fix: redesign.** If A needs B and B needs A, extract the shared behavior both need into a third class `C`, and have both A and B depend on `C` instead of each other. This removes the cycle entirely instead of working around it.

stance
    Container->>B: setAService(rawA reference)
    Container->>A: setBService(fully built B)
    Note over A,B: Both fully wired — deadlock avoided because setters run after construction
```

## 4. Interview Notes

- **"Why does constructor injection fail on circular deps but setter injection doesn't?"** → Constructors require the full argument to exist before the object itself exists; setters run *after* the object already exists as a raw instance, so a partially-built reference can be handed over and completed later.
- **"Is fixing it with setter injection or `@Lazy` a good idea?"** → It's a workaround, not a fix for the design smell. A genuine circular dependency between two services usually signals they should be split, or a shared piece of logic extracted into a third class.
- **"Can circular dependency happen with prototype beans?"** → It's worse — since the container can't cache a half-built prototype the way it does for singletons mid-creation, circular prototype dependencies are generally unresolvable and will throw regardless of injection style.
- **"How do you actually spot this bug?"** → The stack trace names `BeanCurrentlyInCreationException` explicitly and lists the cycle of bean names — read that list, it tells you the exact loop.
