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

## 2. Code Demo

**Stereotype annotations (your own code):**

```java
@Repository
public class UserRepository {
    public String findUser(int id) { return "User#" + id; }
}

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String getUser(int id) { return userRepository.findUser(id); }
}

@RestController
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/user/{id}")
    public String getUser(@PathVariable int id) {
        return userService.getUser(id);
    }
}
```

**`@Bean` for a third-party class you don't control** — e.g. registering an `ObjectMapper` (Jackson) or a custom-configured `RestTemplate`:

```java
@Configuration
public class AppConfig {

    // ObjectMapper is a Jackson library class — you can't annotate it with @Component,
    // you don't own its source code.
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        return mapper;
    }

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .setConnectTimeout(Duration.ofSeconds(5))
            .build();
    }
}
```


```

