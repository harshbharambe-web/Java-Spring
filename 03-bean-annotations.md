# Bean Annotations — `@Component`, `@Bean`, Stereotypes

## 1. What & Why

A **bean** is just an object that the Spring IoC container creates and manages, instead of you creating it with `new`. There are two ways to tell the container "make this a bean":

**A. Stereotype annotations on your own classes** — you write the class, slap an annotation on it, and `@ComponentScan` finds it during startup.

| Annotation | Meaning |
|---|---|
| `@Component` | Generic — "this is a bean," no specific role |
| `@Service` | A `@Component` that holds business logic |
| `@Repository` | A `@Component` that talks to the database (also auto-translates DB exceptions) |
| `@Controller` / `@RestController` | A `@Component` that handles web requests |

These are all the *same mechanism* underneath (`@Component`) — the specific ones exist for readability and a few extra framework behaviors (e.g. `@Repository` wraps exceptions).

**B. `@Bean` methods inside a `@Configuration` class** — for when you *don't* own the class (a third-party library class, or anything you can't put `@Component` on because you didn't write it).

> Analogy: `@Component` is like putting a "hire me" sticker on your own resume — the container finds you during its scan. `@Bean` is for hiring someone who already has a different job elsewhere (a third-party class) — you personally introduce them to the container.

