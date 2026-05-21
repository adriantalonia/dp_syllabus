## 6. Java & Spring Boot Pattern Mapping

> Knowing *which pattern* underlies each Spring feature is a force multiplier in interviews.  
> "What design pattern does `@Transactional` use?" is a real interview question.

| Spring / Java Feature | Pattern(s) | Notes |
|---|---|---|
| `@Component` / `@Bean` | Factory Method, Registry | Container is the factory |
| `ApplicationContext` | IoC Container, Factory (Abstract), Facade | Also a Registry for beans |
| `@Transactional` | Proxy (AOP), Decorator, Unit of Work | JDK proxy or CGLIB proxy |
| `JdbcTemplate` | Template Method, Facade | `RowMapper` = hook method |
| `@EventListener` | Observer (Mediator) | Synchronous dispatch |
| `@TransactionalEventListener` | Observer | Fires post-commit |
| `@Async` + `TaskExecutor` | Command, Half-Sync/Half-Async | Runnable = Command |
| `@Cacheable` / `@CacheEvict` | Proxy (Caching), Decorator | AOP-based |
| Security `FilterChain` | Chain of Responsibility | Each filter is a handler |
| `DispatcherServlet` | Front Controller, Command, Strategy | Central entry point |
| Spring Data Repository | Repository (PoEAA), Proxy | Dynamic proxy at runtime |
| `RestTemplate` / `WebClient` | Facade, Builder | Fluent builder for requests |
| `@Qualifier` / `@Primary` | Strategy (selection) | Multiple impl disambiguation |
| Spring State Machine | State | Formal state machine |
| `@Retryable` | Proxy + Strategy | Retry policy = Strategy |
| `FactoryBean<T>` | Factory Method | `getObject()` is the factory method |
| `BeanPostProcessor` | Decorator, Chain of Responsibility | Post-process all beans |
| `HandlerInterceptor` | Chain of Responsibility, Decorator | Pre/post request handling |
| `Collections.unmodifiableList()` | Decorator, Protection Proxy | Access control |
| `Arrays.asList()` | Adapter | Array → List |
| `Integer.valueOf()` | Flyweight | Cache for -128..127 |
| Java `Iterator` / `Iterable` | Iterator | for-each desugars to this |
| `Comparator<T>` | Strategy | Algorithm = comparison |
| `Runnable` / `Callable` | Command | Encapsulated executable |
