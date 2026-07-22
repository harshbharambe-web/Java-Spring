🍃 Spring Core & Boot Learning GuideA comprehensive, all-in-one documentation guide and code reference covering fundamental Spring Framework concepts: Inversion of Control (IoC), Dependency Injection (DI), Annotations, Bean Scopes, Circular Dependencies, and the Bean Lifecycle.📌 Topic OverviewIDTopicKey ConceptsPrimary Annotations01IoC & Dependency InjectionDecoupling creation from execution, Constructor vs Setter injection@Autowired, @Component, @Service02Annotations & BeansComponent scanning vs Java-based configuration@Component, @Configuration, @Bean03Bean ScopesInstance lifecycle management across application contexts@Scope("singleton"), @Scope("prototype")04Circular DependencyCircular reference resolution and breaking cycles@Lazy, Setter Injection05Bean LifecyclePhase hooks from instantiation to destruction@PostConstruct, @PreDestroy, BeanNameAware🏗️ Core Architecture & IoC ConceptThe Inversion of Control (IoC) Container is the core component of the Spring Framework. Instead of manually instantiating classes using the new operator, the container accepts POJOs (Plain Old Java Objects) alongside configuration metadata to assemble, wire, and manage object instances.Plaintext       +-----------------------+
       | Configuration Metadata|
       | (@Component / @Bean)  |
       +-----------+-----------+
                   |
                   v
+------------------+------------------+
|          POJO Classes               |
+------------------+------------------+
                   |
                   v
       +-----------+-----------+
       |   Spring IoC Container|  <--- Manages Object Lifecycles
       +-----------+-----------+
                   |
                   v
       +-----------+-----------+
       |   Fully Wired System  |
       +-----------------------+
💻 Deep-Dive Modules & Code Examples01. Inversion of Control (IoC) & Dependency Injection (DI)ConceptIoC: A design principle where object creation and lifecycle management are inverted from the application code to the framework.DI: The design pattern used to implement IoC, passing required dependencies into a class rather than allowing the class to construct them internally.Code ImplementationJava// Service Interface
public interface NotificationService {
    void send(String message);
}

// Implementation Class
@Service
public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Sending Email: " + message);
    }
}

// Client Class using Constructor Injection
@RestController
public class UserController {

    private final NotificationService notificationService;

    // Recommended: Constructor Injection ensures immutability & ease of testing
    public UserController(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    @GetMapping("/notify")
    public String notifyUser() {
        notificationService.send("Welcome to Spring Boot!");
        return "Notification Sent";
    }
}
02. Spring Annotations & Bean DeclarationsConceptSpring provides two main ways to declare beans:Stereotype Annotations (Component Scanning): Automatically detected by Spring via @ComponentScan (e.g., @Component, @Service, @Repository, @Controller).Explicit Java Configuration: Uses @Configuration classes with @Bean methods, ideal for configuring third-party libraries.Code ImplementationJava// 1. Stereotype Scanning
@Repository
public class UserRepository {
    public String findUserById(Long id) {
        return "User_" + id;
    }
}

// 2. Explicit Java Config (Third-party library bean creation)
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        // Custom setup or third-party instantiation
        return new RestTemplate();
    }
}
03. Spring Bean ScopesScope Comparison MatrixScopeInstances CreatedCommon Use CaseSingleton (Default)One shared instance per Spring container contextStateless beans (@Service, @Repository)PrototypeA new instance every time requested from containerStateful objects, temporary data processingRequest (Web)One instance per HTTP request lifecycleWeb form processing, request contextSession (Web)One instance per HTTP session lifecycleUser session state, shopping cartCode ImplementationJava// Singleton Scope (Default)
@Component
@Scope("singleton")
public class AppSettings {
    private String theme = "DARK";
    // Shared state across all threads
}

// Prototype Scope
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ShoppingCart {
    private final List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }
}
04. Circular Dependency ResolutionConceptA circular dependency occurs when two or more beans depend on each other (BeanA -> BeanB -> BeanA). When constructor injection is used strictly on both sides, Spring throws a BeanCurrentlyInCreationException during container startup.Plaintext    +------------+                  +------------+
    |  ServiceA  | ----(needs)--->  |  ServiceB  |
    +------------+                  +------------+
          ^                               |
          |---------(needs)---------------+
Code Implementation SolutionsJava// Problematic Cycle
@Service
public class ServiceA {
    private final ServiceB serviceB;
    public ServiceA(ServiceB serviceB) { this.serviceB = serviceB; }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;
    public ServiceB(ServiceA serviceA) { this.serviceA = serviceA; }
}
Fix 1: Using @Lazy AnnotationJava@Service
public class ServiceA {
    private final ServiceB serviceB;

    // @Lazy creates a proxy, delaying initialization until first usage
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
Fix 2: Setter / Field InjectionJava@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA; // Allows container to instantiate empty objects first
}
05. Spring Bean LifecycleLifecycle Execution SequencePlaintext [1. Instantiation] -> [2. Populate Properties] -> [3. Aware Hooks] 
                    -> [4. @PostConstruct]     -> [5. Bean Ready] 
                    -> [6. @PreDestroy]        -> [7. Container Shutdown]
Code ImplementationJava@Component
public class DatabaseConnectionPool implements BeanNameAware {

    public DatabaseConnectionPool() {
        System.out.println("Phase 1: Instantiation (Constructor executed)");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("Phase 3: Aware Hook - Bean Name is: " + name);
    }

    @PostConstruct
    public void init() {
        System.out.println("Phase 4: @PostConstruct - Initializing Database Connections");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Phase 6: @PreDestroy - Releasing Database Connections");
    }
}
🚀 Running the CodeBuild Project:Bash./mvnw clean compile
Execute Application:Bash./mvnw spring-boot:run
