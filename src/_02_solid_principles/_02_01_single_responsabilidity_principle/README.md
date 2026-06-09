# 2.1 Single Responsibility Principle (SRP)

<!-- TOC -->
* [2.1 Single Responsibility Principle (SRP)](#21-single-responsibility-principle-srp)
  * [Theory](#theory)
  * [Key Points](#key-points)
  * [The Canonical Example: The `Employee` God Class](#the-canonical-example-the-employee-god-class)
  * [SRP at Every Layer of the Stack](#srp-at-every-layer-of-the-stack)
    * [Method level](#method-level)
    * [Class level](#class-level)
    * [Module / package level](#module--package-level)
    * [Microservice / system level](#microservice--system-level)
  * [The Three Violation Patterns and Their Java Signatures](#the-three-violation-patterns-and-their-java-signatures)
    * [Pattern 1: Mixed abstraction levels](#pattern-1-mixed-abstraction-levels)
    * [Pattern 2: Cross-cutting concerns embedded in business logic](#pattern-2-cross-cutting-concerns-embedded-in-business-logic)
    * [Pattern 3: Multiple formats / representations](#pattern-3-multiple-formats--representations)
  * [SRP and the Open/Closed Principle](#srp-and-the-openclosed-principle)
  * [Spring Boot Violation Patterns and Fixes](#spring-boot-violation-patterns-and-fixes)
  * [Refactoring Exercise: `UserManager` — Unpacked](#refactoring-exercise-usermanager--unpacked)
  * [SRP + Facade: Avoiding Over-decomposition](#srp--facade-avoiding-over-decomposition)
  * [The Nuanced Interview Positions](#the-nuanced-interview-positions)
    * ["Does a 400-line class violate SRP?"](#does-a-400-line-class-violate-srp)
    * ["How small should a class be?"](#how-small-should-a-class-be)
    * ["Isn't SRP just common sense?"](#isnt-srp-just-common-sense)
    * ["When would you NOT split a class?"](#when-would-you-not-split-a-class)
  * [Common Violation Patterns (Code)](#common-violation-patterns-code)
  * [Exercises](#exercises)
  * [Interview Angle](#interview-angle)
  * [References](#references)
<!-- TOC -->

## Theory

A class should have **one, and only one, reason to change**. "Reason to change" refers to the *actor* or *stakeholder*
who could demand a change — not the number of methods or lines of code.

The definition most developers know — *"a class should do one thing"* — is actually misleading. It tempts you to count
methods or lines. The real definition, from Robert Martin's *Clean Architecture*, is about **actors**: a class should
serve one and only one actor (user, stakeholder, system) whose requirements could force a change.

Two methods in the same class are fine if they both exist to satisfy the same person. Two methods are a problem if they
exist to satisfy *different* people, because changing one risks breaking the other.

---

## Key Points

- **SRP ≠ "one method per class"** — a large class can have SRP if all its logic serves the same actor
- **Symptoms of violation:** God Class, mixed concerns (persistence + business logic + formatting)
- **SRP drives decomposition** into services, layers, and modules
- A class with two responsibilities has *two reasons to change* — changes for one can break the other
- **The `and` test:** if a method or class name contains "and", it likely violates SRP (`validateAndSave()`,
  `fetchAndFormat()`)
- **Spring stereotypes** (`@Service`, `@Repository`, `@Controller`) are SRP enforced by convention

---

## The Canonical Example: The `Employee` God Class

**The hidden danger of shared helpers:**

```java
// VIOLATION: Three actors, one class
class Employee {
    // Actor 1: Finance — cares about pay calculation
    public Money calculatePay() {
        return basePay().plus(overtimePay(regularHours()));
    }

    // Actor 2: HR — cares about hours reporting
    public void reportHours(HoursReport report) {
        report.add(regularHours()); // uses the SAME regularHours() helper!
    }

    // Actor 3: IT / Ops — cares about persistence
    public void save() {
        Database.insert(this);
    }

    // Shared helper — the ticking time bomb
    private int regularHours() { ...}
}
```

**Why this is dangerous:** If Finance requires a change to `regularHours()` for overtime policy, the HR report silently
breaks. No compile error. No test failure. The failure surfaces at runtime, in production, for a completely different
team's use case.

**Fixed — intentional duplication over accidental coupling:**

```java
// Each class serves one actor and owns its own helpers
class PayCalculator {          // Actor: Finance
    public Money calculatePay(Employee e) { ...}

    private int regularHours(Employee e) { ...} // Finance's version
}

class HoursReporter {          // Actor: HR
    public void reportHours(Employee e, HoursReport r) { ...}

    private int regularHours(Employee e) { ...} // HR's version — independent
}

class EmployeeRepository {     // Actor: IT / Ops
    public void save(Employee e) { ...}
}
```

> The duplication of `regularHours()` is **intentional**. Each actor now owns its own copy. Finance can change theirs
> without any risk to HR.

---

## SRP at Every Layer of the Stack

SRP isn't a class-level concept — it applies at every granularity. Recognizing this in an interview is the signal of
seniority.

### Method level

A method with a name like `validateAndSave()` is screaming an SRP violation. The `and` is the tell.

```java
// Violation
public void validateAndSave(User user) { ...}

// Fixed — two methods, each with one reason to change
public void validate(User user) { ...}   // changes when validation rules change

public void save(User user) { ...}       // changes when persistence mechanism changes
```

### Class level

```java
// Violation: @Service doing persistence directly
@Service
public class OrderService {
    @Autowired
    JdbcTemplate jdbc;  // wrong — persistence belongs in @Repository

    public Order findById(Long id) {
        return jdbc.queryForObject("SELECT * FROM orders WHERE id = ?", ...);
    }
}

// Fixed
@Service
public class OrderService {
    @Autowired
    OrderRepository orderRepository; // delegates to the right layer

    public Order findById(Long id) {
        return orderRepository.findById(id).orElseThrow();
    }
}
```

### Module / package level

A package `com.myapp.user` containing `UserController`, `UserService`, `UserRepository`, `UserEmailSender`,
`UserReportGenerator`, and `UserAuditLogger` is a **God Package**. Some of those belong in dedicated packages:

```
com.myapp.user         → UserController, UserService, UserRepository
com.myapp.notification → EmailSender, NotificationService
com.myapp.reporting    → ReportGenerator, ReportFormatter
com.myapp.audit        → AuditLogger, AuditEvent
```

### Microservice / system level

This is where SRP becomes the **Bounded Context** principle from DDD. A service that owns user profiles, authentication
tokens, billing subscriptions, and email preferences has multiple reasons to change and multiple teams wanting to modify
it. You decompose so the billing team can change the billing service without touching authentication.

---

## The Three Violation Patterns and Their Java Signatures

### Pattern 1: Mixed abstraction levels

One method does high-level orchestration, another does low-level detail. They feel like they're in different
conversations.

```java
// Violation: two different altitudes in one class
class OrderService {
    // High-level: orchestration
    public void placeOrder(Order order) {
        validateStock(order);
        reserveInventory(order);
        chargePayment(order);
        sendConfirmationEmail(order);
    }

    // Low-level: byte manipulation — wrong altitude for a service
    private byte[] serializeOrderForKafka(Order order) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        // ... manual serialization logic
        return baos.toByteArray();
    }
}

// Fixed: serialization has its own class with its own reason to change
class KafkaOrderSerializer {
    public byte[] serialize(Order order) { ...}
}
```

### Pattern 2: Cross-cutting concerns embedded in business logic

Logging, metrics, auditing, and caching shouldn't live inside business methods — they're infrastructure concerns.

```java
// Violation: business logic entangled with infrastructure
@Service
public class TransferService {
    public void transfer(Account from, Account to, BigDecimal amount) {
        log.info("Transfer started: {} -> {} for {}", from.getId(), to.getId(), amount);
        long start = System.currentTimeMillis();

        from.debit(amount);    // actual business logic — just these two lines
        to.credit(amount);

        metricsRegistry.timer("transfer.duration")
                .record(System.currentTimeMillis() - start, TimeUnit.MILLISECONDS);
        auditLog.record(new AuditEvent("TRANSFER", from.getId(), amount));
        log.info("Transfer complete in {}ms", System.currentTimeMillis() - start);
    }
}

// Fixed: Spring AOP handles cross-cutting concerns as aspects
@Service
public class TransferService {
    @Audited  // custom annotation, handled by AuditAspect
    @Timed    // Micrometer annotation, handled by MetricsAspect
    public void transfer(Account from, Account to, BigDecimal amount) {
        from.debit(amount);
        to.credit(amount);
        repository.saveAll(List.of(from, to));
    }
}

// The aspect — one reason to change (audit requirements)
@Aspect
@Component
public class AuditAspect {
    @AfterReturning("@annotation(Audited)")
    public void audit(JoinPoint jp) {
        auditLog.record(new AuditEvent(jp.getSignature().getName(), Instant.now()));
    }
}
```

### Pattern 3: Multiple formats / representations

A class that knows how to produce its data in multiple formats has a reason to change every time a new consumer appears.

```java
// Violation: the business object knows about its serialization formats
class Invoice {
    private List<LineItem> items;
    private BigDecimal total;

    public String toJson() { ...}   // reason 1: JSON consumer changes

    public String toCsv() { ...}    // reason 2: CSV consumer changes

    public String toHtml() { ...}   // reason 3: HTML consumer changes

    public void saveToDatabase() { ...} // reason 4: DB schema changes
}

// Fixed: each class has one axis of change
class Invoice { /* data only */
}

class InvoiceRepository {
    void save(Invoice i) { ...}
}

class InvoiceJsonView {
    String render(Invoice i) { ...}
}

class InvoiceCsvExporter {
    String export(Invoice i) { ...}
}

class InvoiceHtmlReport {
    String generate(Invoice i) { ...}
}
```

---

## SRP and the Open/Closed Principle

When a class violates SRP, it almost always also violates OCP. If `ReportService` handles three concerns, adding a new
report format requires *modifying* the class. But if `ReportFormatter` is its own interface, you add a new formatter by
*extending*, not modifying.

```java
// SRP + OCP working together
interface ReportFormatter {
    String format(Report report);
}

class HtmlReportFormatter implements ReportFormatter { ...
}

class PdfReportFormatter implements ReportFormatter { ...
}

class CsvReportFormatter implements ReportFormatter { ...
} // new: no existing class touched

class ReportService {
    private final ReportRepository repository;
    private final ReportFormatter formatter;  // injected via constructor — OCP

    public ReportService(ReportRepository repo, ReportFormatter formatter) {
        this.repository = repo;
        this.formatter = formatter;
    }

    public String generateReport(ReportQuery query) {
        Report report = repository.fetch(query);
        return formatter.format(report);  // SRP: one reason to change
    }
}
```

---

## Spring Boot Violation Patterns and Fixes

| Violation                                              | Spring-idiomatic fix                                                        |
|--------------------------------------------------------|-----------------------------------------------------------------------------|
| `@Service` calling `JdbcTemplate` directly             | Extract a `@Repository`; service only calls repository interface            |
| `@Controller` containing business logic                | Push logic to `@Service`; controller does request mapping + delegation only |
| `@Service` sending emails via `JavaMailSender`         | Extract `@Component NotificationService`; call it from the service          |
| `@Entity` with `toJson()` / `toCsv()` methods          | Use Jackson `@JsonView` or dedicated DTOs / assemblers                      |
| One `@KafkaListener` handling all message types        | One listener class per message type; shared logic in a helper               |
| Cross-cutting concerns inline (logging, timing, audit) | Spring AOP `@Aspect`; business code stays clean                             |

---

## Refactoring Exercise: `UserManager` — Unpacked

**The key is identifying the actors first:**

| Concern          | Actor                | Extracted class   |
|------------------|----------------------|-------------------|
| CRUD             | IT / Ops             | `UserRepository`  |
| Password hashing | Security / Auth team | `PasswordService` |
| Session tokens   | Platform team        | `SessionService`  |
| Audit logging    | Compliance team      | `AuditService`    |

```java
// Before: one class, four actors
class UserManager {
    public User create(CreateUserRequest req) {
        String hash = BCrypt.hashpw(req.password(), BCrypt.gensalt(12));
        User user = db.insert(new User(req.email(), hash));
        String token = JWT.create().withSubject(user.id()).sign(algorithm);
        sessionStore.put(token, user.id());
        auditLog.append(new AuditEntry("USER_CREATED", user.id(), Instant.now()));
        return user;
    }
}

// After: each class has one axis of change
@Service
public class UserRegistrationService {

    private final PasswordService passwordService;       // Security team owns this
    private final UserRepository userRepository;         // IT/Ops owns this
    private final SessionService sessionService;         // Platform team owns this
    private final AuditService auditService;             // Compliance owns this

    public UserRegistrationService(
            PasswordService passwordService,
            UserRepository userRepository,
            SessionService sessionService,
            AuditService auditService) {
        this.passwordService = passwordService;
        this.userRepository = userRepository;
        this.sessionService = sessionService;
        this.auditService = auditService;
    }

    // One reason to change: the registration WORKFLOW
    public User register(CreateUserRequest req) {
        String hash = passwordService.hash(req.password());
        User user = userRepository.save(req.toUser(hash));
        sessionService.createSession(user);
        auditService.log("USER_CREATED", user.id());
        return user;
    }
}
```

> `UserRegistrationService` now has exactly one reason to change: the registration workflow. If the hashing algorithm
> changes, only `PasswordService` changes. If the audit format changes, only `AuditService` changes.
`UserRegistrationService` doesn't even know.

---

## SRP + Facade: Avoiding Over-decomposition

SRP has a practical floor. If decomposition forces callers to instantiate and orchestrate five classes just to do one
logical operation, you've over-decomposed. The **Facade pattern** gives callers a single entry point into a subsystem of
SRP-compliant classes.

```java
// Five SRP classes, but callers shouldn't have to orchestrate all of them.
// The @Service IS the facade — callers see one method, internals are fully decomposed.
@Service
public class UserRegistrationFacade {

    private final UserRepository userRepository;
    private final PasswordHasher passwordHasher;
    private final EmailVerificationService emailService;
    private final AuditLogger auditLogger;
    private final WelcomeEmailSender welcomeEmail;

    public void register(RegistrationRequest req) {
        String hash = passwordHasher.hash(req.password());
        User user = userRepository.create(req.email(), hash);
        emailService.sendVerification(user);
        welcomeEmail.send(user);
        auditLogger.log("USER_REGISTERED", user.id());
    }
}
```

---

## The Nuanced Interview Positions

### "Does a 400-line class violate SRP?"

Not necessarily, but it's a smell worth investigating. A 400-line `PaymentProcessor` that does nothing but handle
payment state transitions — with many small private helpers all in service of that one goal — can be perfectly
SRP-compliant. A 40-line class that queries a database, formats output, and sends an email violates SRP badly.

**The metric is actors, not lines.**

How to investigate: look for the word "and" in method names, for imports from wildly different domains (e.g.,
`javax.mail` alongside `java.sql`), and for the answer to: *"if requirement X changed, what else in this class might
break?"*

### "How small should a class be?"

SRP-driven, not line-count-driven. There is no target line count. There is only: does this class have more than one
reason to change?

### "Isn't SRP just common sense?"

SRP is common sense in the same way that eating well is common sense. The hard part is applying it consistently under
deadline pressure when it's "just one more small thing" to add to an existing class. The discipline is resisting that —
and having a code structure that makes the right thing easy. That's why Spring enforces it via stereotypes: convention
removes the decision from the developer's plate.

### "When would you NOT split a class?"

When the split creates more coordination overhead than the coupling it removes. If two concerns always change together
and always for the same actor, splitting them is ceremony without benefit. Also: when a class is small and its scope is
already clear, premature decomposition adds indirection that makes the code harder to trace.

---

## Common Violation Patterns (Code)

```java
// VIOLATION: Three reasons to change
class ReportService {
    public Report fetchFromDB() { ...}           // reason 1: DB schema changes

    public String formatAsHtml(Report r) { ...}  // reason 2: HTML format changes

    public void sendByEmail(String html) { ...}  // reason 3: email provider changes
}

// FIXED: Each class has one reason to change
class ReportRepository {
    public Report fetch() { ...}
}

class ReportFormatter {
    public String toHtml(Report r) { ...}
}

class ReportEmailSender {
    public void send(String html) { ...}
}
```

---

## Exercises

- [ ] Refactor a `UserManager` class that handles CRUD, password hashing, session tokens, and audit logging
- [ ] Does a 400-line class necessarily violate SRP? Make the case for both answers
- [ ] Identify SRP violations in a real Spring Boot `@Service` you've written
- [ ] Take a class with `javax.mail` and `java.sql` imports and decompose it along actor lines
- [ ] Write a `TransferService` that uses Spring AOP to move logging and auditing out of the business method

---

## Interview Angle

Interviewers often ask: *"How small should a class be?"* — The answer is SRP-driven, not line-count-driven. Demonstrate
you understand **"reason to change"** as the measure, not method count or line count.

Key phrases to use in your answer:

- *"The question I ask is: who would ask for this change? If the answer is two different stakeholders, the class has two
  responsibilities."*
- *"In Spring, the stereotype annotations formalize SRP by convention — `@Service` for business logic, `@Repository` for
  persistence, `@Controller` for HTTP concerns."*
- *"I look for the word 'and' in method names and class names — it's almost always a signal of a mixed responsibility."*
- *"Over-decomposition is also a risk. The Facade pattern lets you apply SRP internally while still giving callers a
  simple, single entry point."*

---

## References

- Robert C. Martin — *Clean Architecture* (Chapter 7: SRP)
- Robert C. Martin — *Clean Code* (Chapter 3: Functions, Chapter 10: Classes)
- *Effective Java* by Joshua Bloch — Item 15 (minimize accessibility), Item 17 (design for inheritance)
- Spring Framework
  docs — [Stereotype Annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-stereotype-annotations)
- [Neetcode / SOLID Principles refresher](https://neetcode.io)