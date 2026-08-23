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
thod is called on it.
}


