## 2. SOLID Principles

> **Depth in this file:** Full — theory, violations, Java/Spring examples, exercises, interview angles  
> **In `software-design.md`:** How SOLID manifests at architectural level (Hexagonal, Clean Architecture)

---

### 2.1 Single Responsibility Principle (SRP)

**Theory:**  
A class should have **one, and only one, reason to change**. "Reason to change" refers to the *actor* or *stakeholder* who could demand a change — not the number of methods or lines of code.

**Key Points:**
- SRP ≠ "one method per class" — a large class can have SRP if all its logic serves the same actor
- Symptoms of violation: God Class, mixed concerns (persistence + business logic + formatting)
- SRP drives decomposition into services, layers, and modules
- A class with two responsibilities has *two reasons to change* — changes for one can break the other

**Java/Spring Example Contexts:**
- `UserService` handling authentication, email sending, and DB persistence → violates SRP → split into `AuthService`, `NotificationService`, `UserRepository`
- MVC itself is an expression of SRP: Model (data), View (rendering), Controller (flow)
- Spring's `@Service`, `@Repository`, `@Controller` stereotypes are SRP enforced by convention

**Common Violation Patterns:**
```java
// VIOLATION: Three reasons to change
class ReportService {
    public Report fetchFromDB() { ... }       // reason 1: DB schema changes
    public String formatAsHtml(Report r) { ... } // reason 2: HTML format changes
    public void sendByEmail(String html) { ... } // reason 3: email provider changes
}

// FIXED: Each class has one reason to change
class ReportRepository   { public Report fetch() { ... } }
class ReportFormatter    { public String toHtml(Report r) { ... } }
class ReportEmailSender  { public void send(String html) { ... } }
```

**Exercises / Problems:**
- [ ] Refactor a `UserManager` class that handles CRUD, password hashing, session tokens, and audit logging
- [ ] Does a `400-line` class necessarily violate SRP? Make the case for both answers
- [ ] Identify SRP violations in a real Spring Boot `@Service` you've written

**Interview Angle:**  
Interviewers often ask: *"How small should a class be?"* — The answer is SRP-driven, not line-count-driven. Demonstrate you understand "reason to change" as the measure.

---

### 2.2 Open/Closed Principle (OCP)

**Theory:**  
Software entities should be **open for extension but closed for modification**. Once a module is working and tested, adding new behavior should not require editing it — only extending it.

**Key Points:**
- Achieved via abstraction (interfaces/abstract classes) + polymorphism
- Parameterization by behavior: Strategy and Template Method are the primary OCP enablers
- Not "never modify" — OCP applies after the design has stabilized; early code changes freely
- Classic violation: `if-else` / `switch` on type codes that grows with every new type
- "The shape of change" — OCP forces you to predict which dimension will vary and abstract it

**Java/Spring Example Contexts:**
- Adding a new payment method without touching `PaymentProcessor` — Strategy enables OCP
- Spring `BeanPostProcessor` — extend Spring's lifecycle without modifying its core
- `Comparator<T>` — open for any comparison logic, closed for modification
- Spring MVC `HandlerMapping` implementations — add new mapping strategies without changing DispatcherServlet

**Common Violation & Fix:**
```java
// VIOLATION: Every new discount type requires modifying this class
class DiscountCalculator {
    public double calculate(Order order, String type) {
        if (type.equals("SEASONAL"))    return order.total() * 0.10;
        if (type.equals("LOYALTY"))     return order.total() * 0.15;
        if (type.equals("EMPLOYEE"))    return order.total() * 0.30;
        return 0;
    }
}

// FIXED: New discount types extend without modifying DiscountCalculator
interface DiscountPolicy {
    double calculate(Order order);
}
class SeasonalDiscount implements DiscountPolicy { ... }
class LoyaltyDiscount  implements DiscountPolicy { ... }
class EmployeeDiscount implements DiscountPolicy { ... }

class DiscountCalculator {
    public double calculate(Order order, DiscountPolicy policy) {
        return policy.calculate(order);
    }
}
```

**Exercises / Problems:**
- [ ] Extend a report exporter to support JSON, CSV, XML, and PDF without modifying the exporter class
- [ ] Show how an `enum + switch` violates OCP; refactor to a polymorphic solution
- [ ] Where does OCP conflict with YAGNI? Articulate the trade-off — when do you abstract early vs late?

**Interview Angle:**  
Expect: *"Doesn't OCP lead to over-engineering?"* — Show you can apply it selectively: abstract the axis of variation that is *known* to vary; leave stable parts un-abstracted.

---

### 2.3 Liskov Substitution Principle (LSP)

**Theory:**  
If `S` is a subtype of `T`, then objects of type `T` may be replaced with objects of type `S` without altering the correctness of the program. Subtypes must be **behaviorally** substitutable — not just syntactically compatible.

**Key Points:**
- **Preconditions** cannot be strengthened in a subtype (subtype must accept at least what supertype accepts)
- **Postconditions** cannot be weakened in a subtype (subtype must deliver at least what supertype promises)
- **Invariants** of the supertype must be preserved
- Classic violations: `Square extends Rectangle`, throwing `UnsupportedOperationException` in a subtype, narrowing parameter types
- LSP is about *behavioral subtyping*, not just inheritance syntax — an interface can be violated too

**Java/Spring Example Contexts:**
- `Arrays.asList()` returns a fixed-size `List` — calling `add()` throws `UnsupportedOperationException` — LSP violation
- `Stack extends Vector` in Java — Stack inherits `add(index, element)` which breaks stack semantics
- Spring `ApplicationContext extends BeanFactory` — properly adheres to LSP; ApplicationContext is a valid BeanFactory

**Classic Violation Explained:**
```java
class Rectangle {
    void setWidth(int w)  { this.width = w; }
    void setHeight(int h) { this.height = h; }
    int area()            { return width * height; }
}

class Square extends Rectangle {
    @Override void setWidth(int w)  { this.width = w;  this.height = w; } // breaks postcondition
    @Override void setHeight(int h) { this.width = h;  this.height = h; } // breaks postcondition
}

// Client code — correct for Rectangle, BREAKS with Square
void resize(Rectangle r) {
    r.setWidth(5);
    r.setHeight(3);
    assert r.area() == 15; // fails for Square — area is 9
}
```

**Fix:** `Shape` interface with `area()` only — Square and Rectangle are not in an inheritance relationship.

**Exercises / Problems:**
- [ ] Analyze `Stack extends Vector` — list every LSP violation
- [ ] Design a `Bird` hierarchy where `Penguin` (cannot fly) does not violate LSP
- [ ] Explain why `Collections.unmodifiableList()` is an LSP violation at the interface contract level

**Interview Angle:**  
*"Can you give a real example of an LSP violation you've seen?"* — Prepare a concrete story. The `Arrays.asList()` example is a great quick answer.

---

### 2.4 Interface Segregation Principle (ISP)

**Theory:**  
Clients should **not be forced to depend on interfaces they do not use**. Many client-specific interfaces are better than one general-purpose interface.

**Key Points:**
- Fat interfaces force implementors to stub methods or throw `UnsupportedOperationException`
- Split interfaces *by client* — ask "who uses this interface and what do they need?"
- **Role interfaces** (narrow, client-specific) vs **header interfaces** (mirroring a class's full API)
- Java 8+ `default` methods can mitigate some ISP issues — but also mask violations
- ISP and SRP are related but different: SRP is about classes, ISP is about interfaces

**Java/Spring Example Contexts:**
- Spring Data's layered hierarchy: `CrudRepository → PagingAndSortingRepository → JpaRepository` — ISP in action
- JDBC `ResultSet` — 100+ methods, arguably a fat interface; most clients use 5
- Custom `ReadableRepository<T>` + `WritableRepository<T>` instead of one combined interface

**Common Violation & Fix:**
```java
// VIOLATION: MultifunctionDevice forces all implementors to implement everything
interface MultifunctionDevice {
    void print(Document d);
    void scan(Document d);
    void fax(Document d);
    void copy(Document d);
}

// SimplePrinter is forced to throw on scan/fax/copy — ISP violation
class SimplePrinter implements MultifunctionDevice {
    public void print(Document d) { ... }
    public void scan(Document d)  { throw new UnsupportedOperationException(); } // violation
    ...
}

// FIXED: Segregated interfaces
interface Printable { void print(Document d); }
interface Scannable { void scan(Document d); }
interface Faxable   { void fax(Document d); }

class SimplePrinter     implements Printable { ... }
class OfficePrinter     implements Printable, Scannable { ... }
class MultifunctionUnit implements Printable, Scannable, Faxable { ... }
```

**Exercises / Problems:**
- [ ] Design repository interfaces for a read-heavy system where writes happen only in admin contexts
- [ ] Split a `UserManager` interface used by 3 different clients (Admin UI, Public API, Audit Service)
- [ ] When does ISP overlap with SRP? Give a concrete example where applying one requires the other

**Interview Angle:**  
Expect: *"How small should an interface be?"* — Answer: defined by its smallest coherent client need, not by lines of code.

---

### 2.5 Dependency Inversion Principle (DIP)

**Theory:**  
- High-level modules should not depend on low-level modules. **Both should depend on abstractions.**
- Abstractions should not depend on details. Details should depend on abstractions.

**Key Points:**
- DIP is the **goal**; Dependency Injection (DI) is a **technique** that achieves it; IoC is the **broader pattern**
- The "inversion": traditional code has business logic depending on infrastructure. DIP inverts this.
- Spring IoC container is the industrial-strength implementation of DIP
- Without DIP: unit tests require real databases, real email servers, real everything
- Layer rule: business logic must not import persistence or infrastructure classes — only interfaces

**Vocabulary Clarity (Frequently Confused):**
- **DIP** — design principle about abstraction direction
- **IoC** — broader principle: framework calls your code, not the other way around
- **DI** — technique: constructor injection, setter injection, field injection
- **IoC Container** — Spring `ApplicationContext` — manages DI for you

**Java/Spring Example Contexts:**
- `@Autowired` against interface types, not concrete classes — enforces DIP
- Hexagonal Architecture is DIP applied at system level (see `software-design.md`)
- `@Qualifier`, `@Primary` — when multiple implementations exist for one interface

**Common Violation & Fix:**
```java
// VIOLATION: High-level OrderService depends on low-level MySqlOrderRepository (concrete)
class OrderService {
    private MySqlOrderRepository repo = new MySqlOrderRepository(); // direct dependency on detail
    public void place(Order o) { repo.save(o); }
}

// FIXED: Both depend on abstraction
interface OrderRepository { void save(Order o); }

class OrderService {
    private final OrderRepository repo; // depends on abstraction
    public OrderService(OrderRepository repo) { this.repo = repo; } // injected
    public void place(Order o) { repo.save(o); }
}

class MySqlOrderRepository  implements OrderRepository { ... }
class InMemoryOrderRepository implements OrderRepository { ... } // usable in tests
```

**Exercises / Problems:**
- [ ] Refactor a service that `new`-s its own `SmtpEmailSender` — apply DIP + constructor injection
- [ ] Explain DIP, IoC, and DI in one concise answer suitable for an interview
- [ ] How does Spring's `ApplicationContext` embody DIP? Trace a `@Service` bean from annotation to injection

**Interview Angle:**  
*"What's the difference between DIP and DI?"* — This is a classic trap. Many candidates confuse the principle (DIP) with the technique (DI). Know the distinction cold.

---

### 2.6 SOLID — Interview Synthesis

**How SOLID violations compound:**
SRP violation → class has two reasons to change → new requirements force modification → OCP violated → subclass to patch the violation → subclass can't properly substitute parent → LSP violated → all this exposed through a fat interface → ISP violated → and the whole thing is untestable because of concrete dependencies → DIP violated.

**SOLID and Testability:**  
SOLID code is almost always easier to unit test. If a class is hard to test, it's usually violating at least one principle. Testability is a practical proxy for SOLID compliance.

**When to break SOLID (maturity signal):**
- **YAGNI** — don't abstract axes of variation that don't actually vary yet
- **Simple scripts / prototypes** — premature abstraction is waste
- **Performance-critical paths** — indirection (DIP) has a cost in hot paths
- The key is *intentional* trade-offs, not ignorance

**SOLID → GoF mapping:**

| Principle | Patterns That Enforce It |
|---|---|
| SRP | Facade (isolates subsystem), Command (encapsulates request) |
| OCP | Strategy, Template Method, Decorator, Observer |
| LSP | All patterns using interface hierarchies correctly |
| ISP | Adapter (wraps fat interface), role-based interfaces |
| DIP | Abstract Factory, Factory Method, all DI-based patterns |

**Exercises / Problems:**
- [ ] Review a real Spring Boot service you own — score it against all 5 principles with evidence
- [ ] *"SOLID makes code over-engineered"* — argue both sides, then give your position
- [ ] For each GoF pattern you know, state which SOLID principle it primarily supports
