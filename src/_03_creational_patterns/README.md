## 3. Creational Patterns

> **Intent:** Control *how* objects are created — decouple clients from the specifics of what gets created, when, and how.

---

### 3.1 Singleton

**Theory:**  
Ensure a class has **only one instance** and provide a global access point to it.

**Key Points:**
- Thread-safe variants (know all of them):
  1. **Eager initialization** — instance created at class load; thread-safe by JVM classloading
  2. **Synchronized `getInstance()`** — correct but serializes all access; performance cost
  3. **Double-checked locking** — requires `volatile` keyword on the field; broken before Java 5 memory model
  4. **Initialization-on-demand holder (Bill Pugh)** — lazy, thread-safe, no synchronization overhead
  5. **Enum Singleton** — recommended by Joshua Bloch; handles serialization and reflection attacks
- Problems with Singleton: hidden global state, coupling, testability, classloader issues in EE containers
- Spring beans are **Singleton-scoped by default** — the container manages it; you rarely need to implement Singleton yourself
- Singleton is often called an anti-pattern when misused as a global state dumping ground

**Implementation Reference:**
```java
// RECOMMENDED: Enum Singleton — serialization-safe, reflection-safe
public enum ConfigManager {
    INSTANCE;
    private final Properties props = loadProperties();
    public String get(String key) { return props.getProperty(key); }
}

// ALSO CORRECT: Initialization-on-demand holder
public class ConnectionPool {
    private ConnectionPool() {}
    private static class Holder {
        static final ConnectionPool INSTANCE = new ConnectionPool();
    }
    public static ConnectionPool getInstance() { return Holder.INSTANCE; }
}

// CORRECT but verbose: Double-checked locking
public class Logger {
    private static volatile Logger instance; // volatile is mandatory
    private Logger() {}
    public static Logger getInstance() {
        if (instance == null) {                     // first check (no lock)
            synchronized (Logger.class) {
                if (instance == null)               // second check (with lock)
                    instance = new Logger();
            }
        }
        return instance;
    }
}
```

**Exercises / Problems:**
- [ ] Implement all 5 variants and explain the thread-safety guarantee of each
- [ ] Explain why `volatile` is mandatory in double-checked locking (Java memory model)
- [ ] Why does Singleton break unit tests? Show the fix using a Spring `@Bean` instead

---

### 3.2 Factory Method

**Theory:**  
Define an **interface for creating an object**, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

**Key Points:**
- Also called "Virtual Constructor" — decouples client from concrete product classes
- The creator class defines `createProduct()` as a factory method; subclasses override it
- Differs from Simple Factory (static method in a helper class — not a GoF pattern)
- Spring's `FactoryBean<T>` interface is a direct implementation
- Product and Creator hierarchies grow together — adding a product requires a new creator subclass (OCP tension if there are many creators)

**Structure:**
```java
// Product hierarchy
interface Notification { void send(String msg); }
class EmailNotification implements Notification { ... }
class SmsNotification   implements Notification { ... }

// Creator hierarchy — factory method is createNotification()
abstract class NotificationService {
    protected abstract Notification createNotification(); // factory method
    public void notify(String msg) {
        Notification n = createNotification(); // delegates creation to subclass
        n.send(msg);
    }
}
class EmailNotificationService extends NotificationService {
    protected Notification createNotification() { return new EmailNotification(); }
}
class SmsNotificationService extends NotificationService {
    protected Notification createNotification() { return new SmsNotification(); }
}
```

**Java/Spring Examples:**
- `DocumentBuilderFactory.newDocumentBuilder()` — JDK standard
- `LoggerFactory.getLogger(Class)` — SLF4J
- Spring's `FactoryBean<T>` — `getObject()` is the factory method

**Exercises / Problems:**
- [ ] Implement a `TransportFactory` that creates `Truck`, `Ship`, or `Plane` based on configuration
- [ ] Distinguish Factory Method from Abstract Factory with a concrete example — draw the class diagram
- [ ] What problem does Factory Method solve that a constructor does not?

---

### 3.3 Abstract Factory

**Theory:**  
Provide an **interface for creating families of related or dependent objects** without specifying their concrete classes.

**Key Points:**
- "Factory of factories" — each concrete factory creates an entire product family
- Enforces consistency among products (you can't mix AWS storage with Azure queue)
- Adding a new *product family* is easy (new concrete factory). Adding a new *product type* requires changing the factory interface — OCP tension
- Common in: UI toolkits, database driver abstraction, cloud provider SDKs, test doubles

**Structure:**
```java
// Product interfaces
interface MessageQueue { void enqueue(String msg); }
interface ObjectStorage { void upload(String key, byte[] data); }

// Abstract Factory
interface CloudFactory {
    MessageQueue   createQueue();
    ObjectStorage  createStorage();
}

// Concrete Factories — product families
class AwsFactory   implements CloudFactory {
    public MessageQueue  createQueue()   { return new SqsQueue(); }
    public ObjectStorage createStorage() { return new S3Storage(); }
}
class AzureFactory implements CloudFactory {
    public MessageQueue  createQueue()   { return new ServiceBusQueue(); }
    public ObjectStorage createStorage() { return new BlobStorage(); }
}
```

**Java/Spring Examples:**
- JDBC: `Connection` creates `Statement` and `PreparedStatement` — a factory for related SQL objects
- Spring `AbstractFactoryBean`
- `javax.xml.parsers.DocumentBuilderFactory`

**Exercises / Problems:**
- [ ] Design an `InfrastructureFactory` creating `Queue`, `Storage`, and `Cache` for AWS vs Azure vs GCP
- [ ] Identify the Abstract Factory in JDBC — map `Connection`, `Statement`, `ResultSet` to the pattern
- [ ] Abstract Factory vs Factory Method: when does one become the other?

---

### 3.4 Builder

**Theory:**  
**Separate the construction of a complex object from its representation**, allowing the same construction process to create different representations.

**Key Points:**
- Solves the **telescoping constructor anti-pattern** (constructors with many parameters)
- Enables **immutable objects** — builder accumulates mutable state during construction; `build()` creates the immutable product
- Fluent interface / method chaining makes it readable
- Optional **Director** role encapsulates a specific construction algorithm
- `build()` is the correct place to validate invariants — throw on invalid state before the object exists
- Lombok `@Builder` generates this mechanically; know what it generates under the hood

**Structure:**
```java
// Immutable product
public final class Order {
    private final String customerId;
    private final List<LineItem> items;
    private final Address shippingAddress;
    private final DiscountPolicy discount;

    private Order(Builder b) { // private constructor — only Builder can create
        this.customerId      = b.customerId;
        this.items           = List.copyOf(b.items); // defensive copy for immutability
        this.shippingAddress = b.shippingAddress;
        this.discount        = b.discount;
    }

    public static Builder builder(String customerId) { return new Builder(customerId); }

    public static class Builder {
        private final String customerId;
        private final List<LineItem> items = new ArrayList<>();
        private Address shippingAddress;
        private DiscountPolicy discount = DiscountPolicy.NONE;

        private Builder(String customerId) { this.customerId = customerId; }

        public Builder addItem(LineItem item)          { items.add(item); return this; }
        public Builder shippingAddress(Address addr)   { this.shippingAddress = addr; return this; }
        public Builder discount(DiscountPolicy policy) { this.discount = policy; return this; }

        public Order build() {
            if (items.isEmpty()) throw new IllegalStateException("Order must have at least one item");
            if (shippingAddress == null) throw new IllegalStateException("Shipping address required");
            return new Order(this);
        }
    }
}

// Usage
Order order = Order.builder("C-001")
    .addItem(new LineItem("SKU-100", 2))
    .shippingAddress(new Address("123 Main St"))
    .discount(DiscountPolicy.SEASONAL)
    .build();
```

**Java/Spring Examples:**
- Lombok `@Builder` — generated builder
- `ResponseEntity.ok().header(...).body(...)` — Spring MVC
- `MockMvcRequestBuilders.get("/api/orders").header(...).accept(...)` — Spring Test
- `UriComponentsBuilder`, `HttpHeaders` builder

**Exercises / Problems:**
- [ ] Implement an immutable `HttpRequest` class with 10+ fields including optional ones
- [ ] Add a `validate()` method inside `build()` — what invariants should it check?
- [ ] Compare Lombok `@Builder` vs a handwritten builder — when does the manual approach win?

---

### 3.5 Prototype

**Theory:**  
Specify the kinds of objects to create using a **prototypical instance**, and create new objects by **copying** this prototype.

**Key Points:**
- Use when object creation is expensive (DB load, complex graph) and a copy is cheaper
- **Shallow copy** — copies field values; references point to the same objects
- **Deep copy** — recursively copies all referenced objects; more correct but more expensive
- Java's `Cloneable` interface and `clone()` method are widely considered a design mistake (Joshua Bloch)
  - `clone()` creates an object without calling a constructor — can bypass invariants
  - `Cloneable` is a marker interface that doesn't declare `clone()` — confusing contract
- Better alternatives: copy constructor, static factory `copy(T original)`, serialization round-trip, `ObjectMapper` deep copy
- Spring `prototype` scope: new bean instance per injection point — the container is the prototype manager

**Deep Copy Approaches:**
```java
// Option 1: Copy constructor (recommended)
public class Document {
    private final String title;
    private final List<Section> sections;

    public Document(Document original) {
        this.title    = original.title; // String is immutable — safe to share
        this.sections = original.sections.stream()
                                .map(Section::new)  // deep copy each section
                                .collect(toList());
    }
}

// Option 2: Serialization round-trip (quick but relies on Serializable)
@SuppressWarnings("unchecked")
public static <T extends Serializable> T deepCopy(T original) {
    try (var bos = new ByteArrayOutputStream();
         var oos = new ObjectOutputStream(bos)) {
        oos.writeObject(original);
        return (T) new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray())).readObject();
    } catch (Exception e) { throw new RuntimeException(e); }
}

// Option 3: Jackson deep copy (useful in Spring context)
T copy = objectMapper.readValue(objectMapper.writeValueAsString(original), original.getClass());
```

**Exercises / Problems:**
- [ ] Implement deep copy for a `Report` object with nested `Section`, `Chart`, and `Table` lists
- [ ] Explain three reasons why `Cloneable` / `clone()` is considered a design mistake in Java
- [ ] How does Spring's `prototype` scope relate to the Prototype pattern?
