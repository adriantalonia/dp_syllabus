## 5. Behavioral Patterns

> **Intent:** Concerned with algorithms and the assignment of responsibilities between objects.

<!-- TOC -->
  * [5. Behavioral Patterns](#5-behavioral-patterns)
    * [5.1 Chain of Responsibility](#51-chain-of-responsibility)
    * [5.2 Command](#52-command)
    * [5.3 Interpreter](#53-interpreter)
    * [5.4 Iterator](#54-iterator)
    * [5.5 Mediator](#55-mediator)
    * [5.6 Memento](#56-memento)
    * [5.7 Observer](#57-observer)
    * [5.8 State](#58-state)
    * [5.9 Strategy](#59-strategy)
    * [5.10 Template Method](#510-template-method)
    * [5.11 Visitor](#511-visitor)
<!-- TOC -->

---

### 5.1 Chain of Responsibility

**Theory:**  
**Pass a request along a chain of handlers**. Each handler decides to process the request or pass it to the next handler in the chain.

**Key Points:**
- Decouples senders from receivers; sender doesn't know which handler processes the request
- Chain assembled at runtime — flexible ordering
- No guarantee the request gets handled (unless you add a default/catch-all handler)
- Spring Security's `FilterChain`, Servlet `Filter` chain, Spring MVC `HandlerInterceptor` — all CoR

**Structure:**
```java
abstract class RequestHandler {
    private RequestHandler next;

    public RequestHandler setNext(RequestHandler next) { this.next = next; return next; }

    public final void handle(HttpRequest req) {
        if (canHandle(req)) process(req);
        else if (next != null) next.handle(req);
        else throw new UnhandledRequestException(req);
    }

    protected abstract boolean canHandle(HttpRequest req);
    protected abstract void process(HttpRequest req);
}

class AuthHandler       extends RequestHandler { ... }
class RateLimitHandler  extends RequestHandler { ... }
class LoggingHandler    extends RequestHandler { ... }
class BusinessHandler   extends RequestHandler { ... }

// Assembly
AuthHandler auth = new AuthHandler();
auth.setNext(new RateLimitHandler()).setNext(new LoggingHandler()).setNext(new BusinessHandler());
auth.handle(request);
```

**Exercises / Problems:**
- [ ] Implement an expense approval chain: Employee → Manager → Director → CFO, each with a spending limit
- [ ] Implement a Spring `OncePerRequestFilter` chain for: JWT validation → rate limiting → request logging
- [ ] How does Spring Security's filter chain implement CoR? Where is the chain assembled?

---

### 5.2 Command

**Theory:**  
**Encapsulate a request as an object**, allowing parameterization, queuing, logging, undo/redo, and transactional behavior.

**Key Points:**
- Decouples invoker (who calls) from receiver (who executes)
- Enables: undo/redo stacks, macro recording, job queues, transactional scripts, delayed execution
- CQRS: the "Command" in Command Query Responsibility Segregation
- Event sourcing: commands are the write side; persisted events represent their effect
- Command vs Strategy: Command encapsulates *what to do and with what data*; Strategy encapsulates *how to do it*

**Structure with Undo:**
```java
interface Command {
    void execute();
    void undo();
}

class TextEditor {
    private StringBuilder text = new StringBuilder();
    private final Deque<Command> history = new ArrayDeque<>();

    public void executeCommand(Command cmd) {
        cmd.execute();
        history.push(cmd);
    }
    public void undo() { if (!history.isEmpty()) history.pop().undo(); }
}

class InsertTextCommand implements Command {
    private final TextEditor editor;
    private final int position;
    private final String text;

    public void execute() { editor.insertAt(position, text); }
    public void undo()    { editor.deleteAt(position, text.length()); }
}
```

**Java/Spring Examples:**
- Spring Batch `Step` — encapsulates a unit of work as a command
- Spring `@Async` tasks submitted to `TaskExecutor`
- CQRS command bus implementations (Axon Framework)
- `Runnable`, `Callable` in Java — the simplest possible Command

**Exercises / Problems:**
- [ ] Implement a text editor with full undo/redo stack using Command pattern
- [ ] Design a distributed job queue where commands are serialized to JSON and executed by workers
- [ ] Implement a macro recorder: capture a sequence of commands and replay them

---

### 5.3 Interpreter

**Theory:**  
Given a language, **define a representation for its grammar** along with an Interpreter that uses it to interpret sentences.

**Key Points:**
- Represents grammar rules as classes; an AST is built from them
- Composite pattern is typically used to build the AST
- Used for: SQL parsers, regex engines, expression languages, scripting engines, rule engines
- Spring Expression Language (SpEL) is a production implementation
- Not frequently asked in general interviews; more relevant for platform/compiler/DSL roles

**Structure Sketch:**
```java
interface Expression { int interpret(Map<String, Integer> ctx); }

class NumberExpression  implements Expression { /* leaf */ }
class VariableExpression implements Expression { /* leaf */ }
class AddExpression     implements Expression { /* composite: left + right */ }
class MultiplyExpression implements Expression { /* composite: left * right */ }

// Parse "x + 3 * y" → build AST → call interpret({"x":2, "y":5}) → returns 17
```

**Exercises / Problems:**
- [ ] Implement a calculator supporting `+`, `-`, `*`, `/` and parentheses
- [ ] How does SpEL use the Interpreter pattern? Trace `#{order.total * 0.1}` through the parser

---

### 5.4 Iterator

**Theory:**  
Provide a way to **access elements of an aggregate sequentially** without exposing its underlying representation.

**Key Points:**
- Java `Iterator<T>`, `Iterable<T>`, enhanced `for-each` loop (desugars to `iterator()`)
- **External iterator** — client controls iteration (`for` loop, `hasNext()`/`next()`)
- **Internal iterator** — collection controls iteration; client provides behavior (Java `Stream`, `forEach`)
- `Spliterator` — parallel iteration; supports stream pipelines
- **Fail-fast** — detects concurrent modification, throws `ConcurrentModificationException` (most Java collections)
- **Fail-safe** — iterates over a snapshot, no exception (`ConcurrentHashMap`, `CopyOnWriteArrayList`)

**Structure:**
```java
// Custom iterator for a binary tree (in-order traversal)
class BinaryTreeIterator<T extends Comparable<T>> implements Iterator<T> {
    private final Deque<TreeNode<T>> stack = new ArrayDeque<>();

    BinaryTreeIterator(TreeNode<T> root) { pushLeft(root); }

    private void pushLeft(TreeNode<T> node) {
        while (node != null) { stack.push(node); node = node.left; }
    }

    public boolean hasNext() { return !stack.isEmpty(); }

    public T next() {
        TreeNode<T> node = stack.pop();
        pushLeft(node.right);
        return node.value;
    }
}
```

**Exercises / Problems:**
- [ ] Implement a `PagedIterator<T>` that fetches pages lazily from a REST API
- [ ] Explain `ConcurrentModificationException` — what causes it and four ways to avoid it
- [ ] Compare `Iterator` vs Java `Stream` API — when do you choose each?

---

### 5.5 Mediator

**Theory:**  
**Define an object that encapsulates how a set of objects interact**. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.

**Key Points:**
- Reduces many-to-many object dependencies to one-to-many (all go through mediator)
- Chat rooms, air traffic control, event bus, CQRS command/query bus — all Mediators
- Spring `ApplicationEventPublisher` is a built-in Mediator
- Risk: Mediator becomes a God Object — keep it routing, not deciding
- Mediator vs Observer: Mediator centralizes coordination (one knows all); Observer is decentralized (publishers don't know subscribers)
- Mediator vs Facade: Facade is unidirectional (client → subsystem); Mediator coordinates multiple peers bidirectionally

**Java/Spring Examples:**
- `ApplicationEvent` + `@EventListener` — publish domain events, decouple producers from consumers
- `@TransactionalEventListener` — fire domain events only after the transaction commits
- Apache Camel routes as Mediators between systems

**Exercises / Problems:**
- [ ] Implement a `DomainEventPublisher` mediator for a Spring Boot order service
- [ ] When does the Mediator pattern become a liability? What signals it's time to refactor it?

---

### 5.6 Memento

**Theory:**  
Without violating encapsulation, **capture and externalize an object's internal state** so it can be restored to that state later.

**Key Points:**
- Three roles: **Originator** (the object), **Memento** (snapshot), **Caretaker** (manages snapshots)
- Originator creates and restores from Mementos; Caretaker stores them without peeking inside
- Java serialization is the practical Memento for persistence — but carries versioning risks
- Event sourcing is Memento at architectural scale
- JPA `@Version` + optimistic locking is a limited form of Memento for concurrency control

**Structure:**
```java
// Originator
class TextEditor {
    private String content;

    public Memento save() { return new Memento(content); } // captures state

    public void restore(Memento m) { this.content = m.getContent(); } // restores state

    public record Memento(String content) {} // immutable snapshot
}

// Caretaker
class UndoManager {
    private final Deque<TextEditor.Memento> history = new ArrayDeque<>();

    public void save(TextEditor editor)   { history.push(editor.save()); }
    public void undo(TextEditor editor)   { if (!history.isEmpty()) editor.restore(history.pop()); }
}
```

**Exercises / Problems:**
- [ ] Implement undo for a graphics editor that supports `move`, `resize`, and `recolor` operations
- [ ] Design event sourcing for a bank account — what is the Memento in this context?

---

### 5.7 Observer

**Theory:**  
Define a **one-to-many dependency** between objects so that when one object (Subject) changes state, all dependents (Observers) are notified and updated automatically.

**Key Points:**
- **Push model** — Subject pushes data to Observers in notification
- **Pull model** — Subject notifies; Observers pull what they need
- Classic Java `Observer`/`Observable` — deprecated since Java 9; too tightly coupled
- Memory leak risk: Observers must unregister; `WeakReference` is one mitigation
- Spring `@EventListener` — synchronous, in-transaction notification
- Spring `@TransactionalEventListener` — fires *after* transaction commit — critical for consistency
- Reactive Streams (`Publisher`, `Subscriber`) — asynchronous Observer with backpressure
- Kafka consumers — Observer at infrastructure scale

**Structure:**
```java
// Using Spring Events — production-grade Observer
// Subject
@Service
class OrderService {
    private final ApplicationEventPublisher publisher;

    public Order place(PlaceOrderCommand cmd) {
        Order order = new Order(cmd);
        orderRepository.save(order);
        publisher.publishEvent(new OrderPlacedEvent(order.getId(), order.getCustomerId()));
        return order;
    }
}

// Observers — decoupled, independently deployable
@Component
class InventoryObserver {
    @TransactionalEventListener // fires after OrderService transaction commits
    public void on(OrderPlacedEvent e) { inventoryService.reserve(e.orderId()); }
}

@Component
class NotificationObserver {
    @TransactionalEventListener
    public void on(OrderPlacedEvent e) { emailService.sendConfirmation(e.customerId()); }
}
```

**Exercises / Problems:**
- [ ] Implement a stock alert system: multiple `PriceAlertObserver`s react to `StockPriceChangedEvent`
- [ ] Explain why `@TransactionalEventListener` is safer than `@EventListener` for side effects
- [ ] Implement the same observer using Project Reactor `Flux` — compare the two approaches

---

### 5.8 State

**Theory:**  
Allow an object to **alter its behavior when its internal state changes**. The object will appear to change its class.

**Key Points:**
- Eliminates complex `if-else` / `switch` on state fields — each state encapsulates its own behavior
- State machine: States + Transitions + Guards + Actions
- Spring State Machine (`spring-statemachine`) is production-grade for this
- State vs Strategy: State = behavior changes based on *internal* state that transitions; Strategy = algorithm swapped *externally* by client

**Structure:**
```java
// State interface
interface OrderState {
    void pay(OrderContext ctx);
    void ship(OrderContext ctx);
    void cancel(OrderContext ctx);
    String name();
}

// Context — holds current state
class OrderContext {
    private OrderState state = new PendingState();

    public void setState(OrderState s) { this.state = s; }
    public void pay()    { state.pay(this); }
    public void ship()   { state.ship(this); }
    public void cancel() { state.cancel(this); }
}

// Each state defines valid transitions
class PendingState implements OrderState {
    public void pay(OrderContext ctx) {
        ctx.setState(new PaidState()); // valid transition
        System.out.println("Payment processed");
    }
    public void ship(OrderContext ctx) { throw new IllegalStateException("Must pay first"); }
    public void cancel(OrderContext ctx) { ctx.setState(new CancelledState()); }
    public String name() { return "PENDING"; }
}

class PaidState implements OrderState {
    public void pay(OrderContext ctx) { throw new IllegalStateException("Already paid"); }
    public void ship(OrderContext ctx) { ctx.setState(new ShippedState()); }
    public void cancel(OrderContext ctx) { ctx.setState(new CancelledState()); /* trigger refund */ }
    public String name() { return "PAID"; }
}
```

**Exercises / Problems:**
- [ ] Implement `PENDING → PAID → SHIPPED → DELIVERED → CANCELLED` with full State pattern
- [ ] Implement the same using Spring State Machine — compare boilerplate and flexibility
- [ ] When is a simple `enum + switch` acceptable vs when do you need full State pattern?

---

### 5.9 Strategy

**Theory:**  
**Define a family of algorithms, encapsulate each one, and make them interchangeable**. Strategy lets the algorithm vary independently from clients that use it.

**Key Points:**
- Context holds a reference to a Strategy interface; delegates algorithm execution to it
- Eliminates conditional logic for algorithm selection
- Java 8+ lambdas make Strategy trivial — any `@FunctionalInterface` is a Strategy
- `Comparator<T>`, `Predicate<T>`, `Function<T,R>`, `Runnable` — all are Strategies
- Strategy vs Command: Strategy = encapsulate *how* (algorithm); Command = encapsulate *what* (request + undo/redo)
- Strategy vs Template Method: Strategy = composition, runtime; Template Method = inheritance, compile-time

**Structure (Pre-Java 8 and Post-Java 8):**
```java
// Strategy interface
interface PricingStrategy {
    Money calculate(Order order);
}

// Concrete strategies
class RegularPricing    implements PricingStrategy { ... }
class SeasonalPricing   implements PricingStrategy { ... }
class MembershipPricing implements PricingStrategy { ... }

// Context
class PricingService {
    private PricingStrategy strategy;
    public void setStrategy(PricingStrategy s) { this.strategy = s; }
    public Money price(Order order) { return strategy.calculate(order); }
}

// Java 8+: strategies as lambdas
PricingStrategy flat    = order -> Money.of(order.items().size() * 10);
PricingStrategy percent = order -> order.subtotal().multiply(0.90);

// Strategy registered in Spring context
@Bean PricingStrategy membershipPricing(MemberRepository repo) {
    return order -> repo.findById(order.customerId())
                        .map(m -> order.subtotal().multiply(1 - m.discountRate()))
                        .orElse(order.subtotal());
}
```

**Java/Spring Examples:**
- Spring Security `PasswordEncoder` — injectable Strategy
- Spring MVC `ViewResolver` — multiple rendering strategies
- Spring `PlatformTransactionManager` — pluggable transaction strategy
- `Collections.sort(list, Comparator)` — Comparator is a Strategy

**Exercises / Problems:**
- [ ] Design a shipping cost calculator with `Standard`, `Express`, and `Overnight` strategies
- [ ] Refactor an `if-else` chain for discount calculation into Strategy using Java lambdas
- [ ] Compare Strategy vs Template Method — state when to choose each with a concrete example

---

### 5.10 Template Method

**Theory:**  
**Define the skeleton of an algorithm in a base class**, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps without changing the algorithm's structure.

**Key Points:**
- Inversion of control — the base class calls the subclass ("Hollywood Principle")
- **Abstract methods** — mandatory override points; subclass *must* implement
- **Hook methods** — optional override points; base class provides empty or default implementation
- Spring's `JdbcTemplate`, `RestTemplate`, `AbstractController` all use Template Method
- Often replaced by **Strategy + lambdas** in modern Java — same flexibility without inheritance coupling

**Structure:**
```java
// Template in base class
abstract class DataMigrator {
    // Template method — the algorithm skeleton
    public final void migrate() {
        List<Record> data = readSource();       // abstract — mandatory
        List<Record> transformed = transform(data); // hook — optional
        validate(transformed);                  // hook — optional
        writeTarget(transformed);              // abstract — mandatory
        cleanup();                              // hook — optional
    }

    protected abstract List<Record> readSource();
    protected abstract void writeTarget(List<Record> data);

    // Hooks with default implementations
    protected List<Record> transform(List<Record> data) { return data; }
    protected void validate(List<Record> data) { /* default: no validation */ }
    protected void cleanup() { /* default: nothing */ }
}

class CsvToPostgresMigrator extends DataMigrator {
    protected List<Record> readSource()        { /* read CSV */ }
    protected void writeTarget(List<Record> d) { /* insert into Postgres */ }
    protected List<Record> transform(List<Record> d) { /* clean + map fields */ }
}
```

**Java/Spring Examples:**
- `HttpServlet.service()` calling `doGet()`/`doPost()` — canonical Java EE example
- `JdbcTemplate.query(sql, RowMapper)` — `RowMapper` is the hook you implement
- Spring Batch `AbstractItemReader.read()` — calls `doRead()` which you implement

**Exercises / Problems:**
- [ ] Implement a report generation framework: Template Method defines fetch → format → deliver; subclasses specialize each
- [ ] Refactor a Template Method hierarchy to use Strategy + lambdas — what changes and what stays the same?
- [ ] Explain the trade-off: Template Method (inheritance) vs Strategy (composition)

---

### 5.11 Visitor

**Theory:**  
Represent an **operation to be performed on elements of an object structure**, letting you define a new operation without changing the classes of the elements.

**Key Points:**
- **Double dispatch** — `element.accept(visitor)` routes to `visitor.visit(element)` using runtime types of *both* objects
- Allows adding operations to a hierarchy without modifying it — OCP for operations
- Tension: adding a new *element type* requires modifying all visitors — OCP violated in that direction
- Used for: AST traversal (compiler passes), serialization, pretty-printing, tax/discount calculations on heterogeneous collections
- Java `Files.walkFileTree()` uses a Visitor-like callback model

**Structure:**
```java
// Element hierarchy
interface CartItem { void accept(CartVisitor visitor); }
class PhysicalProduct  implements CartItem { public void accept(CartVisitor v) { v.visit(this); } }
class DigitalProduct   implements CartItem { public void accept(CartVisitor v) { v.visit(this); } }
class Subscription     implements CartItem { public void accept(CartVisitor v) { v.visit(this); } }

// Visitor interface — one method per concrete element type
interface CartVisitor {
    void visit(PhysicalProduct p);
    void visit(DigitalProduct d);
    void visit(Subscription s);
}

// Concrete Visitors — new operations without touching element classes
class TaxCalculator implements CartVisitor {
    private Money totalTax = Money.ZERO;
    public void visit(PhysicalProduct p) { totalTax = totalTax.add(p.price().multiply(0.08)); }
    public void visit(DigitalProduct d)  { totalTax = totalTax.add(d.price().multiply(0.05)); }
    public void visit(Subscription s)    { /* tax-exempt */ }
    public Money getTotalTax() { return totalTax; }
}

class ShippingCalculator implements CartVisitor { ... }

// Usage — double dispatch in action
TaxCalculator taxCalc = new TaxCalculator();
cart.items().forEach(item -> item.accept(taxCalc));
Money tax = taxCalc.getTotalTax();
```

**Exercises / Problems:**
- [ ] Add a `DiscountVisitor` and a `ReceiptPrinterVisitor` to the cart example above — note you didn't touch element classes
- [ ] Implement an AST Visitor with `evaluate()` and `prettyPrint()` for arithmetic expressions
- [ ] Explain the double-dispatch mechanism — why can't a simple method overload replace it?
