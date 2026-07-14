## 4. Structural Patterns

> **Intent:** Compose classes and objects into larger structures while keeping them flexible and efficient.

<!-- TOC -->
  * [4. Structural Patterns](#4-structural-patterns)
    * [4.1 Adapter](#41-adapter)
    * [4.2 Bridge](#42-bridge)
    * [4.3 Composite](#43-composite)
    * [4.4 Decorator](#44-decorator)
    * [4.5 Facade](#45-facade)
    * [4.6 Flyweight](#46-flyweight)
    * [4.7 Proxy](#47-proxy)
<!-- TOC -->

---

### 4.1 Adapter

**Theory:**  
**Convert the interface of a class into another interface** that clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Key Points:**
- **Object Adapter** (composition — preferred): Adapter holds a reference to the Adaptee; delegates calls
- **Class Adapter** (multiple inheritance — Java: not possible with classes, only interfaces): extends both Target and Adaptee
- Use when integrating third-party code, legacy systems, or two incompatible APIs
- Target interface is what the client expects; Adaptee is the existing interface; Adapter bridges them

**Structure:**
```java
// Target — what the client expects
interface PaymentPort {
    PaymentResult pay(Money amount, String currency);
}

// Adaptee — existing third-party / legacy code with incompatible interface
class LegacyPaymentGateway {
    public boolean processPayment(double amount, String currencyCode, String ref) { ... }
}

// Object Adapter — composes Adaptee, implements Target
class PaymentGatewayAdapter implements PaymentPort {
    private final LegacyPaymentGateway gateway;

    public PaymentGatewayAdapter(LegacyPaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public PaymentResult pay(Money amount, String currency) {
        String ref = UUID.randomUUID().toString();
        boolean ok = gateway.processPayment(amount.toDouble(), currency, ref);
        return ok ? PaymentResult.success(ref) : PaymentResult.failure();
    }
}
```

**Java/Spring Examples:**
- `Arrays.asList(array)` — adapts array to `List`
- Spring `HandlerAdapter` — adapts various controller types to a unified handler interface
- `InputStreamReader` — adapts `InputStream` (byte stream) to `Reader` (character stream)

**Exercises / Problems:**
- [ ] Adapt a legacy `XmlPaymentGateway` to a modern `JsonPaymentPort` interface
- [ ] Adapter vs Facade: state the key difference in one sentence (hint: intent)
- [ ] Adapter vs Decorator: same structure, different purpose — explain

---

### 4.2 Bridge

**Theory:**  
**Decouple an abstraction from its implementation** so that the two can vary independently.

**Key Points:**
- Solves "class explosion" when two *independent dimensions* of variation exist
  - Without Bridge: `RedCircle`, `BlueCircle`, `RedSquare`, `BlueSquare` → N×M classes
  - With Bridge: `Shape` + `Color` as separate hierarchies → N+M classes
- Abstraction holds a reference to Implementor (via composition, not inheritance)
- JDBC is the canonical Java example: `DriverManager` (abstraction) + `Driver` implementations
- Bridge vs Strategy: both use composition, but Bridge structures two hierarchies; Strategy replaces one algorithm

**Structure:**
```java
// Implementor hierarchy (varies independently)
interface NotificationChannel {
    void send(String recipient, String message);
}
class EmailChannel implements NotificationChannel { ... }
class SmsChannel   implements NotificationChannel { ... }
class PushChannel  implements NotificationChannel { ... }

// Abstraction hierarchy (varies independently)
abstract class Notification {
    protected final NotificationChannel channel; // Bridge — reference to implementor
    Notification(NotificationChannel channel) { this.channel = channel; }
    abstract void notifyUser(User user, String event);
}

class AlertNotification  extends Notification {
    void notifyUser(User u, String event) {
        channel.send(u.getContact(), "ALERT: " + event);
    }
}
class DigestNotification extends Notification {
    void notifyUser(User u, String event) {
        channel.send(u.getContact(), "Digest: " + event);
    }
}

// Usage — combine freely
Notification alert = new AlertNotification(new SmsChannel());
Notification digest = new DigestNotification(new EmailChannel());
```

**Exercises / Problems:**
- [ ] Design a reporting system where `ReportType` (Summary/Detail/Audit) and `OutputFormat` (PDF/CSV/HTML) vary independently
- [ ] Compare Bridge vs Adapter — which problem does each solve?

---

### 4.3 Composite

**Theory:**  
Compose objects into **tree structures to represent part-whole hierarchies**. Composite lets clients treat individual objects and compositions of objects uniformly.

**Key Points:**
- Leaf and Composite implement the same `Component` interface — clients don't need to distinguish
- Composite delegates operations to children; Leaf implements the actual work
- Recursive tree processing: file systems, UI component trees, expression trees, org charts
- Decisions: should `add()`/`remove()` be on `Component` (transparency) or only `Composite` (safety)?
- Java AWT/Swing: `Component` is the interface; `Panel`/`JFrame` are Composites; `Button`/`Label` are Leaves

**Structure:**
```java
interface FileSystemItem {
    String name();
    long size();
    void print(String indent);
}

class File implements FileSystemItem {
    private final String name;
    private final long size;
    public void print(String indent) { System.out.println(indent + name + " (" + size + "b)"); }
    // name(), size() implementations...
}

class Directory implements FileSystemItem {
    private final String name;
    private final List<FileSystemItem> children = new ArrayList<>();

    public void add(FileSystemItem item) { children.add(item); }

    public long size() { return children.stream().mapToLong(FileSystemItem::size).sum(); }

    public void print(String indent) {
        System.out.println(indent + name + "/");
        children.forEach(c -> c.print(indent + "  ")); // recursive
    }
}
```

**Exercises / Problems:**
- [ ] Implement an expression tree supporting `+`, `-`, `*` with `evaluate()` and `toString()` on every node
- [ ] Implement an organizational chart where `getHeadcount()` on a department sums all sub-departments and employees
- [ ] When does Composite violate LSP? (Hint: `add()` on Leaf)

---

### 4.4 Decorator

**Theory:**  
**Attach additional responsibilities to an object dynamically**. Decorators provide a flexible alternative to subclassing for extending functionality.

**Key Points:**
- Decorator implements the **same interface** as the wrapped object — transparent to clients
- Unlimited stacking: `D3(D2(D1(Core)))` — each layer adds behavior
- Core behavior unchanged — decorators are additive, not replacements
- Java I/O is the canonical example: `new BufferedReader(new InputStreamReader(new FileInputStream(...)))`
- Decorator vs Inheritance: Decorator = runtime behavior addition; Inheritance = compile-time
- Decorator vs Proxy: same structure, different intent — Decorator adds behavior, Proxy controls access
- Spring AOP is a dynamic Decorator at runtime

**Structure:**
```java
interface OrderRepository {
    Optional<Order> findById(String id);
    void save(Order order);
}

class JpaOrderRepository implements OrderRepository { ... } // base implementation

// Logging Decorator — transparent to clients
class LoggingOrderRepository implements OrderRepository {
    private final OrderRepository delegate;
    private final Logger log = LoggerFactory.getLogger(getClass());

    LoggingOrderRepository(OrderRepository delegate) { this.delegate = delegate; }

    public Optional<Order> findById(String id) {
        log.debug("findById({})", id);
        long start = System.currentTimeMillis();
        Optional<Order> result = delegate.findById(id);
        log.debug("findById({}) -> {} in {}ms", id, result.isPresent(), System.currentTimeMillis() - start);
        return result;
    }
    public void save(Order order) {
        log.debug("save({})", order.getId());
        delegate.save(order);
    }
}

// Caching Decorator — stackable on top of Logging Decorator
class CachingOrderRepository implements OrderRepository {
    private final OrderRepository delegate;
    private final Map<String, Order> cache = new ConcurrentHashMap<>();

    CachingOrderRepository(OrderRepository delegate) { this.delegate = delegate; }

    public Optional<Order> findById(String id) {
        return Optional.ofNullable(cache.computeIfAbsent(id,
            k -> delegate.findById(k).orElse(null)));
    }
    public void save(Order order) {
        cache.put(order.getId(), order);
        delegate.save(order);
    }
}
```

**Java/Spring Examples:**
- `Collections.unmodifiableList()`, `Collections.synchronizedList()` — Decorators
- `HttpServletRequestWrapper` — Decorator in Spring Security filter chain
- `@Transactional` proxy — Decorator via Spring AOP
- `@Cacheable` — Caching Decorator via Spring AOP

**Exercises / Problems:**
- [ ] Implement a rate-limiting Decorator for a `PaymentGateway` interface
- [ ] Stack logging + caching + rate-limiting decorators — demonstrate the order matters
- [ ] Explain why `@Transactional` doesn't work when calling a transactional method from within the same bean (self-invocation limitation of the Proxy/Decorator mechanism)

---

### 4.5 Facade

**Theory:**  
Provide a **simplified, unified interface to a complex subsystem**. Facade defines a higher-level interface that makes the subsystem easier to use.

**Key Points:**
- Does **not** restrict access to subsystem (unlike Proxy which controls access) — Facade is convenience, not enforcement
- Reduces coupling between clients and complex subsystem internals
- Facade vs Adapter: Adapter makes incompatible interfaces work together; Facade simplifies a complex system
- Facade vs Mediator: Facade is one-way (client → subsystem); Mediator coordinates multiple systems bidirectionally
- Risk: Facade grows into a God Class if not kept disciplined
- Spring's `JdbcTemplate` is one of the best examples of Facade in any framework

**Structure:**
```java
// Complex subsystem classes
class InventoryService  { boolean reserve(String sku, int qty) { ... } }
class PaymentService    { PaymentResult charge(String cardToken, Money amount) { ... } }
class ShippingService   { TrackingNumber schedule(Address addr, List<Item> items) { ... } }
class NotificationSvc   { void sendConfirmation(String email, Order order) { ... } }

// Facade — hides the complexity
@Service
class OrderFacade {
    private final InventoryService inventory;
    private final PaymentService payment;
    private final ShippingService shipping;
    private final NotificationSvc notification;

    public OrderConfirmation placeOrder(PlaceOrderRequest req) {
        inventory.reserve(req.sku(), req.quantity());      // step 1
        var result = payment.charge(req.cardToken(), req.total()); // step 2
        var tracking = shipping.schedule(req.address(), req.items()); // step 3
        notification.sendConfirmation(req.email(), buildOrder(req, tracking)); // step 4
        return new OrderConfirmation(result.transactionId(), tracking);
    }
}
```

**Java/Spring Examples:**
- `JdbcTemplate` — Facade over raw JDBC (Connection, PreparedStatement, ResultSet lifecycle)
- `RestTemplate` / `WebClient` — Facade over HTTP client mechanics
- Spring's `TransactionTemplate` — Facade over programmatic transaction management

**Exercises / Problems:**
- [ ] Design a `ShippingFacade` coordinating `InventoryService`, `PaymentService`, and `CourierService`
- [ ] How does `JdbcTemplate` act as a Facade + Template Method simultaneously?
- [ ] What's the line between a well-designed Facade and a God Class?

---

### 4.6 Flyweight

**Theory:**  
Use **sharing to support a large number of fine-grained objects** efficiently. Extract shared state and reuse it across many instances.

**Key Points:**
- **Intrinsic state** — shared, immutable, context-independent (stored in Flyweight)
- **Extrinsic state** — unique, context-dependent, passed in by the client (not stored)
- A Factory is required to manage the pool of shared Flyweights (cache by key)
- Java examples: `Integer.valueOf()` caches -128 to 127; `String.intern()` pool; character glyph caches in document editors
- Use when: creating a huge number of similar objects; memory is a concern; most state can be made intrinsic

**Structure:**
```java
// Intrinsic state — shared
class TreeType {
    private final String species;
    private final Color color;
    private final Texture texture; // expensive to load

    TreeType(String species, Color color, Texture texture) { ... }
    public void draw(Canvas c, int x, int y) { /* renders texture at position */ }
}

// Flyweight Factory — ensures sharing
class TreeTypeFactory {
    private static final Map<String, TreeType> cache = new HashMap<>();

    public static TreeType get(String species, Color color, Texture texture) {
        String key = species + color;
        return cache.computeIfAbsent(key, k -> new TreeType(species, color, texture));
    }
}

// Extrinsic state — unique per instance
class Tree {
    private final int x, y;          // extrinsic: position
    private final TreeType type;     // intrinsic: shared reference

    public void draw(Canvas c) { type.draw(c, x, y); }
}
```

**Exercises / Problems:**
- [ ] Explain how `Integer.valueOf(127) == Integer.valueOf(127)` but `Integer.valueOf(200) != Integer.valueOf(200)` relates to Flyweight
- [ ] Design a Flyweight for a text editor where `CharacterGlyph` (font, size, bold) is shared but position is extrinsic
- [ ] When does Flyweight trade CPU time for memory savings? What's the trade-off?

---

### 4.7 Proxy

**Theory:**  
Provide a **surrogate or placeholder for another object** to control access to it.

**Key Points:**
- Same interface as the real subject — transparent substitution
- Types of Proxy:
  - **Virtual Proxy** — lazy initialization; creates expensive object on first use
  - **Remote Proxy** — local representative for a remote object (gRPC stub, RMI)
  - **Protection Proxy** — access control checks before delegating (Spring Security)
  - **Caching Proxy** — memoizes expensive operations (`@Cacheable`)
  - **Logging/Instrumentation Proxy** — cross-cutting concerns (`@Transactional`, AOP)
- Spring AOP creates **JDK Dynamic Proxy** (interface-based) or **CGLIB Proxy** (class-based, no interface needed)
- Critical interview topic: **self-invocation limitation** — calling `this.method()` bypasses the proxy

**JDK Dynamic Proxy Example:**
```java
// Logging proxy — works for any service interface
public class LoggingProxyFactory {
    @SuppressWarnings("unchecked")
    public static <T> T wrap(T target, Class<T> iface) {
        return (T) Proxy.newProxyInstance(
            iface.getClassLoader(),
            new Class<?>[]{ iface },
            (proxy, method, args) -> {
                long start = System.nanoTime();
                try {
                    Object result = method.invoke(target, args);
                    log.info("{} completed in {}ns", method.getName(), System.nanoTime() - start);
                    return result;
                } catch (InvocationTargetException e) {
                    log.error("{} threw {}", method.getName(), e.getCause().getMessage());
                    throw e.getCause();
                }
            }
        );
    }
}

// Usage
OrderService proxy = LoggingProxyFactory.wrap(new OrderServiceImpl(), OrderService.class);
```

**Java/Spring Examples:**
- Spring `@Transactional` — AOP proxy wraps the bean; transaction begins before, commits/rolls back after
- Spring Data JPA repository interfaces — implemented at runtime via dynamic proxy
- `@Cacheable` — caching proxy intercepts calls, checks cache before delegating
- Mockito mocks — proxy-based test doubles implementing the interface

**Exercises / Problems:**
- [ ] Implement a JDK Dynamic Proxy that adds latency logging and exception masking to any service
- [ ] Explain the **self-invocation problem** with `@Transactional` — why does calling a `@Transactional` method from within the same class not start a transaction?
- [ ] Compare Decorator vs Proxy: same structure, different intent — give a crisp one-sentence distinction
