# Spring Boot Interview Questions & Answers

This reference collects 100 commonly asked Spring Boot interview questions along with concise, interview-ready answers. It also covers the essential Spring Framework concepts that Spring Boot builds on, because most interviews expect you to explain both. Questions are grouped into sections that mirror how real interviews flow: core Spring and the IoC container first, then Spring Boot fundamentals, configuration, web/REST, data access, security, actuator, testing, AOP, and finally microservices and production concerns. Numbering runs continuously from 1 to 100 across all sections, and a short quick-fire round at the end covers extra rapid-recall facts.

---

## Core Spring & IoC

**1. What is the Spring Framework and why is it widely used?**

Spring is a modular Java application framework that promotes loose coupling through dependency injection and provides infrastructure support so developers focus on business logic rather than plumbing.

Its core value comes from the IoC container, plus integrations for web (Spring MVC), data (Spring Data, transactions), security, messaging, and testing. It is non-invasive (works with plain POJOs), highly configurable, and has a huge ecosystem, which is why it became the de facto standard for enterprise Java.

**2. What is Inversion of Control (IoC)?**

IoC is a design principle where the control of creating and wiring objects is inverted from your code to a container. Instead of an object instantiating its own dependencies, the container creates them and supplies them.

This reduces coupling and improves testability, because dependencies can be swapped (for example, a mock in a test) without changing the dependent class.

**3. What is Dependency Injection (DI) and how does it relate to IoC?**

DI is the primary technique used to implement IoC. The container "injects" the collaborators a bean needs rather than the bean looking them up itself.

- IoC is the broad principle (don't call us, we'll call you).
- DI is the concrete mechanism (constructor, setter, or field injection).

**4. What are the types of dependency injection in Spring?**

- Constructor injection: dependencies passed through the constructor. Preferred because it supports immutability, mandatory dependencies, and easy testing.
- Setter injection: dependencies set through setters. Good for optional dependencies.
- Field injection: dependencies set directly on fields via reflection (`@Autowired` on a field). Convenient but discouraged since it hides dependencies and makes testing harder.

```java
@Service
public class OrderService {
    private final PaymentClient paymentClient;

    // constructor injection (preferred)
    public OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }
}
```

**5. Why is constructor injection preferred over field injection?**

Constructor injection makes dependencies explicit and final, guarantees the object is fully initialized before use, and prevents circular dependency surprises at construction time. It also allows you to instantiate the class in a unit test without a Spring context, simply by passing mocks to the constructor.

Field injection relies on reflection, cannot produce `final` fields, and hides how many dependencies a class really has, which often signals a class doing too much.

**6. What is the Spring IoC container?**

The IoC container is the part of Spring that instantiates, configures, assembles, and manages the lifecycle of beans based on configuration metadata (annotations, Java config, or XML). The two main container interfaces are `BeanFactory` and `ApplicationContext`.

**7. What is the difference between BeanFactory and ApplicationContext?**

`BeanFactory` is the basic container providing DI and lazy bean instantiation. `ApplicationContext` is a superset that adds enterprise features.

- `ApplicationContext` eagerly creates singletons at startup by default.
- It adds internationalization (i18n), event publishing, AOP integration, and easy access to resources.
- In practice you almost always use `ApplicationContext` (Spring Boot uses it internally).

**8. What is a Spring bean?**

A bean is simply an object that is instantiated, assembled, and managed by the Spring IoC container. Beans are defined by configuration metadata and the container controls their lifecycle and dependencies.

**9. Describe the Spring bean lifecycle.**

1. Instantiation of the bean.
2. Population of properties / dependency injection.
3. Aware callbacks (`BeanNameAware`, `ApplicationContextAware`, etc.).
4. `BeanPostProcessor.postProcessBeforeInitialization`.
5. Initialization callbacks (`@PostConstruct`, `InitializingBean.afterPropertiesSet`, custom init-method).
6. `BeanPostProcessor.postProcessAfterInitialization`.
7. Bean is ready and in use.
8. Destruction (`@PreDestroy`, `DisposableBean.destroy`, custom destroy-method) on container shutdown.

**10. What are the bean scopes in Spring?**

- `singleton` (default): one shared instance per container.
- `prototype`: a new instance every time the bean is requested.
- `request`: one instance per HTTP request (web only).
- `session`: one instance per HTTP session (web only).
- `application`: one instance per `ServletContext`.
- `websocket`: one instance per WebSocket session.

**11. What is the default bean scope and what does it imply?**

The default is `singleton`, meaning one instance is shared across the whole container. This implies beans should generally be stateless, since shared mutable state creates thread-safety problems in a concurrent web application.

**12. How do you inject a prototype bean into a singleton correctly?**

A plain injection would give the singleton one fixed prototype instance. To get a fresh instance each time, use a `Provider`/`ObjectProvider`, method injection with `@Lookup`, or a scoped proxy.

```java
@Autowired
private ObjectProvider<Task> taskProvider;

public void run() {
    Task task = taskProvider.getObject(); // new prototype each call
}
```

**13. What is the difference between @Component, @Service, @Repository, and @Controller?**

All four are stereotype annotations that mark a class as a Spring-managed component eligible for component scanning. Functionally they are specializations of `@Component`.

- `@Component`: generic stereotype.
- `@Service`: business/service layer (semantic marker).
- `@Repository`: persistence layer; also enables exception translation (converting persistence exceptions into Spring's `DataAccessException`).
- `@Controller`: web layer for MVC controllers.

**14. What does @Autowired do?**

`@Autowired` tells the container to resolve and inject a matching bean by type. It can be placed on constructors, setters, or fields. If exactly one candidate exists it is injected; if none exists it fails unless `required = false`.

Note: since Spring 4.3, if a class has a single constructor, `@Autowired` on it is optional.

**15. How does Spring resolve ambiguity when multiple beans of the same type exist?**

If more than one candidate matches by type, Spring throws `NoUniqueBeanDefinitionException` unless you disambiguate:

- `@Primary` marks one bean as the default choice.
- `@Qualifier("beanName")` selects a specific bean by name at the injection point.
- Matching the field/parameter name to a bean name can also resolve it.

**16. What is @Qualifier used for?**

`@Qualifier` narrows autowiring when multiple beans of the same type exist by naming the exact bean to inject.

```java
public NotificationService(@Qualifier("smsSender") MessageSender sender) {
    this.sender = sender;
}
```

**17. What is @Primary?**

`@Primary` designates a bean as the preferred candidate when several beans of the same type qualify for injection. Unlike `@Qualifier` (used at the injection point), `@Primary` is declared on the bean definition and acts as a global default that a `@Qualifier` can still override.

**18. What is a configuration class and what does @Bean do?**

A class annotated with `@Configuration` declares beans through methods annotated with `@Bean`. Each `@Bean` method's return value is registered as a bean, and its name defaults to the method name. This is the Java-based alternative to XML configuration.

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

**19. What is the difference between @Configuration and @Component for defining beans?**

Both can host `@Bean` methods, but `@Configuration` classes are enhanced by CGLIB proxying so that inter-bean method calls return the same singleton (this is called "full" mode). In a `@Component` (lite mode), calling one `@Bean` method from another creates a new object each time rather than returning the managed singleton.

**20. What is component scanning?**

Component scanning is the process by which Spring discovers classes annotated with stereotypes (`@Component`, `@Service`, etc.) within specified packages and registers them as beans automatically. In Spring Boot, `@SpringBootApplication` triggers scanning of the package of the main class and its sub-packages.

**21. What is @Lazy and when would you use it?**

`@Lazy` defers a bean's creation until it is first needed instead of at startup. Use it to speed up startup for rarely used heavy beans, or to break certain circular dependency chains. Overusing it can hide misconfiguration until runtime.

**22. How does Spring handle circular dependencies?**

For singleton beans wired with setter/field injection, Spring can resolve circular references using an early bean reference (a partially constructed proxy). With constructor injection on both sides it cannot, and startup fails with a `BeanCurrentlyInCreationException`. The clean fix is to redesign the dependency, or use `@Lazy` on one side. Since Spring Boot 2.6, circular references are prohibited by default.

**23. What are the ways to provide configuration metadata to the container?**

- Annotation-based configuration (stereotypes + `@Autowired`).
- Java-based configuration (`@Configuration` + `@Bean`).
- XML-based configuration (legacy).

Modern Spring Boot applications overwhelmingly use annotation and Java-based configuration.

---

## Spring Boot Fundamentals

**24. What is Spring Boot and what problem does it solve?**

Spring Boot is an opinionated layer on top of the Spring Framework that eliminates most boilerplate configuration. Before Boot, wiring a Spring app meant lots of XML, manual dependency version management, and configuring an external servlet container.

Spring Boot solves this with auto-configuration, starter dependencies, embedded servers, and production-ready features, so you can create a runnable, stand-alone application with minimal setup.

**25. What are the key features of Spring Boot?**

- Auto-configuration based on the classpath.
- Starter dependencies for curated, version-aligned libraries.
- Embedded servers (Tomcat, Jetty, Undertow).
- Production-ready features via Actuator.
- No code generation and no XML requirement.
- Externalized, environment-aware configuration.

**26. What does @SpringBootApplication do?**

It is a convenience annotation that combines three annotations:

- `@SpringBootConfiguration` (a specialized `@Configuration`).
- `@EnableAutoConfiguration` (turns on auto-configuration).
- `@ComponentScan` (scans the current package and sub-packages).

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**27. What is auto-configuration and how does it work?**

Auto-configuration attempts to automatically configure your Spring application based on the jars on the classpath and the beans you have already defined. For example, if H2 and Spring Data JPA are present, it configures a `DataSource` and an `EntityManagerFactory`.

It works through classes listed in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (formerly `spring.factories`), guarded by `@Conditional` annotations such as `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and `@ConditionalOnProperty`, so auto-config backs off whenever you provide your own configuration.

**28. How can you see what auto-configuration was applied?**

Run the app with `--debug` (or set `debug=true`) to print the auto-configuration report, which lists positive and negative matches. You can also expose the Actuator `conditions` endpoint. To exclude specific auto-configurations, use `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`.

**29. What are Spring Boot starters?**

Starters are curated dependency descriptors that bundle related libraries under one coordinate with compatible versions. Instead of adding many individual dependencies, you add one starter.

- `spring-boot-starter-web`: web + REST + embedded Tomcat.
- `spring-boot-starter-data-jpa`: JPA + Hibernate.
- `spring-boot-starter-security`: Spring Security.
- `spring-boot-starter-test`: JUnit, Mockito, AssertJ, Spring Test.

**30. What is the role of spring-boot-starter-parent?**

It is a special parent POM that provides sensible defaults: dependency management (so you omit versions), Java version, plugin configuration, resource filtering, and the Spring Boot Maven plugin setup. If you cannot use it as a parent, you can import `spring-boot-dependencies` as a BOM instead.

**31. What is an embedded server and why does Spring Boot use one?**

An embedded server (Tomcat by default) runs inside the application's own process, so the app is a self-contained runnable jar rather than a WAR deployed to an external container. This simplifies packaging, deployment, and scaling, and fits containerized/cloud environments well.

You can switch servers by excluding Tomcat and adding `spring-boot-starter-jetty` or `spring-boot-starter-undertow`.

**32. Can you still deploy a Spring Boot app as a WAR?**

Yes. Package it with `war` packaging, extend `SpringBootServletInitializer`, override `configure()`, and mark the embedded server dependency as `provided`. This lets you deploy to an external servlet container when required, though embedded jars are the modern default.

**33. Explain the SpringApplication.run() startup flow.**

At a high level:

1. Create a `SpringApplication` and determine the application type (servlet, reactive, or none).
2. Set up the `Environment` (load properties, profiles).
3. Create and prepare the `ApplicationContext`.
4. Run `ApplicationContextInitializers` and fire lifecycle events.
5. Perform auto-configuration and register/instantiate beans.
6. Start the embedded server.
7. Invoke `ApplicationRunner` and `CommandLineRunner` beans.

**34. What is the difference between CommandLineRunner and ApplicationRunner?**

Both run code once after the context is ready, just before the application is considered started. The difference is the argument type:

- `CommandLineRunner.run(String... args)` receives raw string arguments.
- `ApplicationRunner.run(ApplicationArguments args)` receives a parsed abstraction with option and non-option arguments.

```java
@Component
public class StartupTask implements CommandLineRunner {
    public void run(String... args) {
        System.out.println("App started with args: " + Arrays.toString(args));
    }
}
```

**35. What is Spring Initializr?**

Spring Initializr (start.spring.io) is a web tool and API that generates a ready-to-use Spring Boot project skeleton. You pick the build tool, language, Boot version, and dependencies, and it produces a zip with the correct build file and a main class.

**36. How do you create a fat/uber jar and run it?**

The `spring-boot-maven-plugin` (or Gradle plugin) repackages the application into an executable "fat jar" containing your code plus all dependencies and a launcher.

```bash
mvn clean package
java -jar target/myapp-0.0.1-SNAPSHOT.jar
```

**37. What is the difference between Spring and Spring Boot?**

Spring is the underlying framework providing IoC, MVC, data, and more, but it requires substantial manual configuration. Spring Boot is a productivity layer that auto-configures Spring, manages dependency versions, embeds a server, and adds production features. Boot does not replace Spring; it makes Spring easier to use.

---

## Configuration

**38. What is the difference between application.properties and application.yml?**

Both externalize configuration and Spring Boot supports either. `.properties` uses flat `key=value` lines; `.yml` uses a hierarchical, indentation-based structure that is more readable for nested config and lists.

```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/app
```

Note that YAML does not support the `@PropertySource` annotation and is whitespace-sensitive.

**39. What does @Value do?**

`@Value` injects a single property value (or an expression) into a field or parameter, with optional defaults.

```java
@Value("${app.timeout:30}")
private int timeoutSeconds; // defaults to 30 if property missing
```

It also supports SpEL, for example `@Value("#{systemProperties['user.region']}")`.

**40. What is @ConfigurationProperties and when is it better than @Value?**

`@ConfigurationProperties` binds a group of properties (by prefix) to the fields of a strongly typed POJO. It is preferred over many scattered `@Value` annotations because it supports relaxed binding, type-safety, validation, and IDE metadata.

```java
@ConfigurationProperties(prefix = "app.mail")
@Component
public class MailProperties {
    private String host;
    private int port;
    // getters and setters
}
```

**41. What are Spring profiles?**

Profiles let you register beans and configuration conditionally for different environments (dev, test, prod). Activate one with `spring.profiles.active`, and provide environment-specific files like `application-dev.yml`.

```java
@Bean
@Profile("dev")
public DataSource devDataSource() { ... }
```

**42. How do you activate a profile?**

Several ways, all subject to precedence:

```bash
# command line
java -jar app.jar --spring.profiles.active=prod

# environment variable
export SPRING_PROFILES_ACTIVE=prod
```

You can also set `spring.profiles.active` in a properties file, though externalizing it per environment is more common.

**43. What is externalized configuration?**

Externalized configuration means keeping settings outside the packaged code so the same artifact runs in every environment. Spring Boot reads config from many sources: properties/YAML files, environment variables, command-line arguments, config servers, and more, letting operators tune behavior without rebuilding.

**44. What is the order of precedence for externalized configuration?**

From highest to lowest (simplified):

1. Command-line arguments.
2. `SPRING_APPLICATION_JSON`.
3. OS environment variables / Java system properties.
4. Profile-specific application properties outside the jar.
5. Profile-specific application properties inside the jar.
6. Application properties outside the jar.
7. Application properties inside the jar.
8. `@PropertySource` and defaults.

The key idea: more specific and more external sources win.

**45. How do you keep secrets out of application.properties?**

Do not commit secrets. Inject them via environment variables, use a secrets manager (Vault, AWS Secrets Manager), or Spring Cloud Config with encryption. Spring Boot's relaxed binding maps an env var like `SPRING_DATASOURCE_PASSWORD` to `spring.datasource.password` automatically.

**46. What is relaxed binding?**

Relaxed binding lets a single property target be matched by several naming styles, so `app.myProperty`, `app.my-property`, `APP_MYPROPERTY` all bind to the same field. This is what allows environment variables (uppercase with underscores) to map cleanly to dotted property names.

---

## Web & REST

**47. What is the difference between @Controller and @RestController?**

`@Controller` is the classic MVC stereotype whose methods typically return view names. `@RestController` is `@Controller` + `@ResponseBody`, so every handler method returns data serialized directly into the response body (usually JSON) rather than resolving a view.

**48. What does @RequestMapping do and what are its HTTP-verb shortcuts?**

`@RequestMapping` maps HTTP requests to handler methods by path, method, headers, params, and content type. The shortcut annotations specialize it per verb:

- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping("/{id}")
    public User get(@PathVariable Long id) { ... }
}
```

**49. What is the difference between @PathVariable and @RequestParam?**

- `@PathVariable` binds a value from the URI path template, for example `/users/42`.
- `@RequestParam` binds a query parameter or form field, for example `/users?active=true`.

Use path variables for identifying a resource and request params for filtering, sorting, or optional inputs.

**50. What do @RequestBody and @ResponseBody do?**

`@RequestBody` binds and deserializes the HTTP request body (JSON, XML) into a method parameter object using an `HttpMessageConverter`. `@ResponseBody` serializes the returned object into the response body. In a `@RestController`, `@ResponseBody` is implied on every method.

**51. What is ResponseEntity and why use it?**

`ResponseEntity<T>` represents the full HTTP response: status code, headers, and body. Use it when you need control beyond the body alone, for example returning `201 Created` with a `Location` header or `404 Not Found`.

```java
@PostMapping
public ResponseEntity<User> create(@RequestBody User u) {
    User saved = service.save(u);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved);
}
```

**52. How do you handle exceptions globally in Spring MVC?**

Use `@ControllerAdvice` (or `@RestControllerAdvice`) with `@ExceptionHandler` methods to centralize exception-to-response mapping across all controllers.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ApiError(ex.getMessage()));
    }
}
```

**53. What is the difference between @ExceptionHandler and @ControllerAdvice?**

`@ExceptionHandler` on a controller handles exceptions only for that controller. `@ControllerAdvice` is a global component that applies its `@ExceptionHandler`, `@InitBinder`, and `@ModelAttribute` methods across many controllers, giving you one place for cross-cutting request handling.

**54. How do you validate request data in Spring Boot?**

Add `spring-boot-starter-validation`, annotate the DTO fields with Bean Validation constraints, and put `@Valid` on the `@RequestBody` parameter. Invalid input triggers a `MethodArgumentNotValidException` you can handle globally.

```java
public record CreateUser(@NotBlank String name, @Email String email) {}

@PostMapping
public User create(@Valid @RequestBody CreateUser body) { ... }
```

**55. What is the difference between @Valid and @Validated?**

`@Valid` is the standard Bean Validation annotation that triggers validation of an object. `@Validated` is Spring's variant that additionally supports validation groups and enables method-level validation on beans. Use `@Validated` on a class to validate method parameters/return values.

**56. What is content negotiation?**

Content negotiation is how Spring decides the response representation (JSON, XML, etc.) based on the request's `Accept` header, URL suffix, or a query parameter. With Jackson on the classpath, JSON is the default. Adding the XML converter lets the same endpoint serve XML when the client asks for it.

**57. How does JSON serialization work in Spring Boot?**

Spring Boot auto-configures Jackson's `ObjectMapper`. Handler return values are serialized to JSON via `MappingJackson2HttpMessageConverter`. You customize behavior with `@JsonProperty`, `@JsonIgnore`, `@JsonFormat`, or by defining a custom `ObjectMapper`/`Jackson2ObjectMapperBuilderCustomizer` bean.

**58. How do you enable CORS in a Spring Boot REST API?**

Per handler/controller with `@CrossOrigin`, or globally by implementing `WebMvcConfigurer` and overriding `addCorsMappings`.

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**").allowedOrigins("https://app.example.com");
    }
}
```

**59. What is DispatcherServlet?**

`DispatcherServlet` is the front controller of Spring MVC. It receives every request, consults handler mappings to find the right controller method, invokes it, applies message converters or view resolvers, and writes the response. Spring Boot registers and configures it automatically.

**60. How do you version a REST API?**

Common strategies:

- URI versioning: `/api/v1/users`.
- Request-parameter versioning: `/api/users?version=1`.
- Header versioning: a custom `X-API-Version` header.
- Media-type (content) versioning: `Accept: application/vnd.app.v1+json`.

URI versioning is the most common because it is simple and cache-friendly.

---

## Data Access

**61. What is Spring Data JPA?**

Spring Data JPA is a Spring module that dramatically reduces boilerplate for JPA-based data access. You declare repository interfaces, and Spring generates the implementation at runtime, providing CRUD operations, pagination, and query derivation without writing DAO code.

**62. What is the difference between CrudRepository, JpaRepository, and PagingAndSortingRepository?**

- `CrudRepository`: basic CRUD operations.
- `PagingAndSortingRepository`: adds pagination and sorting on top of CRUD.
- `JpaRepository`: adds JPA-specific features like batch operations, `flush()`, and returning `List` instead of `Iterable`.

`JpaRepository` extends the other two and is the most commonly used.

**63. What are derived query methods?**

Derived queries let Spring Data generate a query from the method name following a keyword grammar, so you do not write the query yourself.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByLastNameAndActiveTrue(String lastName);
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThanOrderByAgeDesc(int age);
}
```

**64. When and how do you use @Query?**

Use `@Query` when a derived method name would be unwieldy or when you need JPQL/native SQL control.

```java
@Query("select u from User u where u.email = :email")
Optional<User> lookup(@Param("email") String email);

@Query(value = "select * from users where active = true", nativeQuery = true)
List<User> findActiveNative();
```

**65. What is @Transactional and what does it do?**

`@Transactional` declaratively wraps a method (or all methods of a class) in a database transaction. Spring begins a transaction before the method, commits on success, and rolls back on a runtime exception. It is implemented via AOP proxies.

By default it rolls back on unchecked exceptions (`RuntimeException`, `Error`) but not checked exceptions unless you set `rollbackFor`.

**66. What are transaction propagation types?**

Propagation defines how a method participates in existing transactions. Common values:

- `REQUIRED` (default): join an existing transaction or start one.
- `REQUIRES_NEW`: always start a new transaction, suspending any current one.
- `SUPPORTS`: join if one exists, otherwise run non-transactionally.
- `MANDATORY`: must run within an existing transaction, else throw.
- `NESTED`: run in a nested transaction with savepoints.

**67. What are transaction isolation levels?**

Isolation controls visibility of concurrent changes and which anomalies can occur:

- `READ_UNCOMMITTED`: allows dirty reads.
- `READ_COMMITTED`: prevents dirty reads.
- `REPEATABLE_READ`: prevents dirty and non-repeatable reads.
- `SERIALIZABLE`: strictest, prevents phantom reads.

Higher isolation improves correctness but reduces concurrency.

**68. Why does @Transactional sometimes not work (self-invocation)?**

Because it relies on a proxy, calling a `@Transactional` method from another method of the same bean bypasses the proxy, so the transaction advice never runs. Fixes include moving the method to another bean, self-injecting the proxy, or using `AspectJ` weaving.

**69. What is the N+1 select problem and how do you fix it?**

The N+1 problem occurs when loading N parent entities triggers one query for the parents and then one additional query per parent to load a lazy association, resulting in N+1 queries.

Fixes:

- Use a `JOIN FETCH` in JPQL.
- Use an `@EntityGraph` on the repository method.
- Batch fetching via `@BatchSize` / `hibernate.default_batch_fetch_size`.

**70. What is the difference between FetchType.LAZY and FetchType.EAGER?**

`LAZY` loads an association only when it is first accessed; `EAGER` loads it immediately with the parent. `@ManyToOne` and `@OneToOne` default to EAGER, while `@OneToMany` and `@ManyToMany` default to LAZY. LAZY is generally preferred for performance, but accessing lazy data outside a session causes `LazyInitializationException`.

**71. How do you map entity relationships in JPA?**

Use association annotations with the owning side holding the foreign key:

```java
@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

**72. What is the difference between save, saveAndFlush, and saveAll?**

- `save`: persists or merges an entity; the SQL may be deferred until flush.
- `saveAndFlush`: persists and immediately flushes to the database, so changes are visible in the current transaction right away.
- `saveAll`: persists a collection of entities in one call.

**73. What is a connection pool and what does Spring Boot use by default?**

A connection pool reuses a bounded set of open database connections instead of opening a new one per request, which is expensive. Spring Boot uses HikariCP by default because it is fast and lightweight. You tune it under `spring.datasource.hikari.*`, for example `maximum-pool-size`.

**74. How do you configure database schema management in Spring Boot?**

For development, `spring.jpa.hibernate.ddl-auto` controls schema generation (`none`, `validate`, `update`, `create`, `create-drop`). For production, prefer versioned migrations with Flyway or Liquibase and set `ddl-auto` to `validate` or `none`, so schema changes are explicit and auditable.

**75. What is the difference between JPA, Hibernate, and Spring Data JPA?**

- JPA is the specification (interfaces and annotations) for ORM in Java.
- Hibernate is the most common JPA implementation (provider).
- Spring Data JPA is an abstraction on top of JPA/Hibernate that removes repository boilerplate.

You typically use all three together: Spring Data JPA repositories backed by Hibernate implementing JPA.

---

## Security

**76. What is Spring Security?**

Spring Security is a comprehensive authentication and authorization framework for Spring applications. It secures web endpoints and method calls through a chain of servlet filters, and it protects against common attacks such as CSRF, session fixation, and clickjacking out of the box.

**77. What is the difference between authentication and authorization?**

- Authentication verifies who you are (validating credentials).
- Authorization verifies what you are allowed to do (checking roles/permissions).

Authentication happens first; authorization then gates access to resources based on the authenticated principal's authorities.

**78. How does the Spring Security filter chain work?**

Incoming requests pass through an ordered chain of servlet filters (the `SecurityFilterChain`). Each filter handles a concern: extracting credentials, authenticating, storing the `SecurityContext`, checking authorization, and handling exceptions. If any filter rejects the request, it stops there.

**79. How do you configure security in modern Spring Boot?**

Define a `SecurityFilterChain` bean with the lambda DSL (the `WebSecurityConfigurerAdapter` class is removed in current versions).

```java
@Configuration
public class SecurityConfig {
    @Bean
    SecurityFilterChain chain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }
}
```

**80. What is JWT and how is it used for stateless authentication?**

A JSON Web Token is a signed, self-contained token carrying claims (subject, roles, expiry). After login, the server issues a JWT; the client sends it in the `Authorization: Bearer` header on each request. The server validates the signature and claims without server-side session state, which suits scalable/stateless APIs.

Key points: sign with a strong secret or key pair, keep expiries short, and never store sensitive secrets in the payload since it is only encoded, not encrypted.

**81. What is OAuth2 and how does Spring support it?**

OAuth2 is an authorization framework that lets a client access resources on behalf of a user without handling the user's credentials, using tokens issued by an authorization server. Spring provides `spring-boot-starter-oauth2-client` (login via Google/GitHub, etc.) and `spring-boot-starter-oauth2-resource-server` (validating bearer/JWT tokens on APIs).

**82. What is method-level security?**

Method security lets you secure service methods with annotations rather than only URLs. Enable it with `@EnableMethodSecurity`, then annotate methods.

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }

@PostAuthorize("returnObject.owner == authentication.name")
public Document load(Long id) { ... }
```

**83. What is CSRF and when should you enable or disable protection?**

Cross-Site Request Forgery tricks an authenticated browser into submitting unwanted state-changing requests. Spring Security enables CSRF protection by default for browser-based, session-cookie apps. For stateless token-based REST APIs (no cookies), CSRF is commonly disabled because the attack vector does not apply. Never blindly disable it for cookie-session web apps.

**84. How should passwords be stored, and what does Spring provide?**

Never store plaintext or reversibly encrypted passwords. Store a salted, one-way hash using an adaptive algorithm. Spring Security provides `PasswordEncoder` implementations; `BCryptPasswordEncoder` (or `DelegatingPasswordEncoder`) is the standard choice.

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

**85. What are UserDetailsService and UserDetails?**

`UserDetailsService` is the interface Spring Security calls to load user data by username during authentication, returning a `UserDetails` object (username, hashed password, authorities, account status flags). You implement `UserDetailsService` to plug in your own user store, typically backed by a database.

---

## Actuator & Observability

**86. What is Spring Boot Actuator?**

Actuator adds production-ready endpoints that expose operational information about a running application: health, metrics, environment, configuration, mappings, thread dumps, and more. Add `spring-boot-starter-actuator` and expose the endpoints you need.

**87. What are the most important Actuator endpoints?**

- `/actuator/health`: liveness/readiness and dependency health.
- `/actuator/metrics`: application and JVM metrics.
- `/actuator/info`: arbitrary app info (build, git).
- `/actuator/env`, `/actuator/loggers`, `/actuator/mappings`, `/actuator/threaddump`, `/actuator/httpexchanges`.

Only `health` is exposed over HTTP by default; expose others explicitly.

**88. How do you expose and secure Actuator endpoints?**

Control exposure with properties and protect sensitive endpoints with Spring Security.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when_authorized
```

**89. What is a custom health indicator?**

A custom `HealthIndicator` bean lets you report the health of a dependency Actuator does not know about (an external API, a queue). Its status contributes to the aggregate `/health`.

```java
@Component
public class QueueHealthIndicator implements HealthIndicator {
    public Health health() {
        return queueReachable() ? Health.up().build()
                                : Health.down().withDetail("queue", "unreachable").build();
    }
}
```

**90. What are liveness and readiness probes?**

These are health "groups" designed for orchestrators like Kubernetes. Liveness indicates the app is running (restart if it fails); readiness indicates the app can accept traffic (remove from load balancer if not). Spring Boot exposes `/actuator/health/liveness` and `/actuator/health/readiness` when enabled.

**91. What is Micrometer?**

Micrometer is a vendor-neutral metrics facade (like SLF4J but for metrics). Spring Boot uses it to collect metrics and export them to monitoring systems such as Prometheus, Datadog, or New Relic by simply adding the relevant registry dependency. You create custom metrics with `Counter`, `Timer`, and `Gauge`.

---

## Testing

**92. What does @SpringBootTest do?**

`@SpringBootTest` bootstraps the full application context for integration testing, wiring real beans together. You can control the web environment (`MOCK`, `RANDOM_PORT`, `DEFINED_PORT`, `NONE`). It is powerful but heavier than slice tests, so use it when you truly need the whole context.

**93. What are slice tests, and what are @WebMvcTest and @DataJpaTest?**

Slice tests load only a focused portion of the context for faster, targeted tests.

- `@WebMvcTest`: loads the web layer (controllers, `MockMvc`) without services or the database; collaborators are mocked.
- `@DataJpaTest`: loads JPA repositories and an in-memory database, and rolls back transactions after each test.

**94. What is MockMvc?**

`MockMvc` lets you test Spring MVC controllers by sending simulated HTTP requests through the dispatcher without starting a real server, then asserting on status, headers, and body.

```java
mockMvc.perform(get("/api/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("Ada"));
```

**95. What is @MockBean and how does it differ from @Mock?**

`@MockBean` creates a Mockito mock and registers it in the Spring context, replacing any existing bean of that type, so injected collaborators get the mock. Plain `@Mock` (from Mockito) creates a mock unaware of Spring. Use `@MockBean` in Spring integration/slice tests; use `@Mock` in pure unit tests without a context.

Note: in the newest Spring Boot versions `@MockBean` is superseded by `@MockitoBean`, but the concept is the same.

**96. What are Testcontainers and why use them?**

Testcontainers is a library that spins up real dependencies (PostgreSQL, Kafka, Redis) in throwaway Docker containers during tests. This gives you production-like integration tests instead of relying on in-memory substitutes, catching database-specific behavior. Spring Boot offers first-class integration via `@ServiceConnection`.

---

## AOP

**97. What is Aspect-Oriented Programming (AOP) and why is it useful?**

AOP modularizes cross-cutting concerns (logging, security, transactions, caching) that would otherwise be scattered across many classes. You define the behavior once in an aspect and apply it declaratively, keeping business code clean. Spring implements AOP with runtime proxies (JDK dynamic proxies or CGLIB).

**98. Explain the key AOP terms and advice types.**

- Aspect: a module of cross-cutting concern.
- Join point: a point in execution (in Spring, a method call).
- Pointcut: an expression selecting join points.
- Advice: action taken at a join point.

Advice types: `@Before`, `@After` (finally), `@AfterReturning`, `@AfterThrowing`, and `@Around` (the most powerful, wrapping the call).

```java
@Aspect
@Component
public class LoggingAspect {
    @Around("execution(* com.app.service..*(..))")
    public Object logTiming(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        log.info("{} took {} ms", pjp.getSignature(), System.currentTimeMillis() - start);
        return result;
    }
}
```

---

## Microservices & Production

**99. What is Spring Cloud and what problems does it solve?**

Spring Cloud is a set of tools for building distributed systems and microservices on top of Spring Boot. It addresses common distributed concerns:

- Centralized configuration: Spring Cloud Config server.
- Service discovery: Eureka (or Consul/Kubernetes).
- API gateway/routing: Spring Cloud Gateway.
- Resilience: Resilience4j (circuit breakers, retries, rate limiting).
- Distributed tracing: Micrometer Tracing / Sleuth with Zipkin.

**100. How do you prepare a Spring Boot app for production (packaging, Docker, graceful shutdown, performance)?**

Package the app as an executable jar and containerize it, ideally with layered jars or buildpacks so image rebuilds are efficient.

```dockerfile
FROM eclipse-temurin:21-jre
COPY target/app.jar /app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

For safe operations, enable graceful shutdown so in-flight requests complete before the process exits:

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

Additional production practices: externalize all config and secrets, use `validate`/`none` for `ddl-auto` with Flyway migrations, tune the HikariCP pool and JVM heap, expose health/readiness probes for orchestrators, ship metrics via Micrometer to Prometheus, add structured logging and distributed tracing, apply circuit breakers for downstream calls, and consider GraalVM native images or class-data sharing to cut startup time and memory.

---

## Quick-fire round

- **Default embedded server?** Tomcat.
- **Default port?** 8080 (set `server.port` to change).
- **Default connection pool?** HikariCP.
- **Default JSON library?** Jackson.
- **Turn off a specific auto-config?** `@SpringBootApplication(exclude = ...)`.
- **Make a field optional in `@Value`?** Provide a default: `${key:defaultValue}`.
- **Bean scope for stateless services?** Singleton (the default).
- **Rollback on checked exceptions?** Set `@Transactional(rollbackFor = Exception.class)`.
- **Read a list from YAML?** Bind with `@ConfigurationProperties` to a `List` field.
- **Fastest way to test one controller?** `@WebMvcTest` + `MockMvc`.
- **Endpoint exposed by Actuator over HTTP by default?** Only `health`.
- **Password encoder of choice?** `BCryptPasswordEncoder`.
- **Difference between `@RestController` and `@Controller`?** The former adds `@ResponseBody`.
- **Break a constructor circular dependency?** Redesign, or annotate one side `@Lazy`.
- **Run code once at startup?** Implement `CommandLineRunner` or `ApplicationRunner`.

Treat these questions as a map, not a script. In real interviews, back every definition with a short reason and, where possible, a concrete example from your own projects: a time auto-configuration surprised you, an N+1 you fixed with a fetch join, or a `@Transactional` bug caused by self-invocation. Interviewers care far more about whether you understand the "why" and the trade-offs than whether you can recite annotations. Build one small end-to-end Spring Boot app, break it deliberately, read the startup logs, and you will be able to answer most of these from genuine experience.
