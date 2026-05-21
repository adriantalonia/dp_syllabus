## 8. Interview-Specific Prep

### 8.1 Big Tech / Top Consultancy Question Archetypes

**Implementation questions:**
- Implement Singleton (all thread-safe variants)
- Implement Observer from scratch
- Implement Builder for a complex immutable object
- Implement a logging Proxy via JDK Dynamic Proxy
- Implement Strategy with lambdas vs traditional classes
- Implement undo/redo with Command pattern

**Design questions (pattern-driven):**
- "Design a plugin system" → Strategy + Factory + Registry
- "How would you add logging without modifying services?" → Decorator or AOP Proxy
- "Design a discount system extensible for new discount types" → Strategy + OCP
- "How does Spring's `@Transactional` work internally?" → Proxy (AOP)

**Theory questions:**
- "What's the difference between Strategy and State?"
- "Decorator vs Proxy — same structure, why are they different patterns?"
- "What problem does Composite solve that inheritance cannot?"
- "Why is Singleton considered an anti-pattern?"
- "Explain SOLID with a real example from your career"
- "When would you NOT use a design pattern?" ← maturity signal

### 8.2 Commonly Confused Pairs — Interview Landmines

| Pair | Key Distinction |
|---|---|
| **Strategy vs State** | Strategy: algorithm swapped *externally* by client; behavior same. State: behavior changes based on *internal* state transitions |
| **Decorator vs Proxy** | Decorator: *adds* new behavior. Proxy: *controls access* to existing behavior. Same structure; different intent |
| **Adapter vs Facade** | Adapter: makes *incompatible* interfaces work together. Facade: *simplifies* a complex subsystem interface |
| **Adapter vs Decorator** | Adapter: *changes* the interface. Decorator: keeps the *same* interface, adds behavior |
| **Factory Method vs Abstract Factory** | FM: one product, subclass decides which concrete class. AF: family of related products, enforce consistency |
| **Composite vs Decorator** | Composite: tree of same-interface objects (part-whole). Decorator: wraps *one* object to add behavior |
| **Observer vs Mediator** | Observer: decentralized; publishers broadcast; subscribers register themselves. Mediator: centralized; one object coordinates all interactions |
| **Command vs Strategy** | Command: encapsulate *what to do* as an object (with undo, queuing). Strategy: encapsulate *how* (algorithm) |
| **Template Method vs Strategy** | Template Method: inheritance-based, compile-time fixed skeleton. Strategy: composition-based, runtime-swappable algorithm |
| **Bridge vs Adapter** | Bridge: designed *upfront* to let two hierarchies vary independently. Adapter: retrofitted to make *existing* incompatible interfaces work |

### 8.3 Trade-off Cheat Sheet

| Pattern | Primary Benefit | Primary Cost |
|---|---|---|
| Singleton | One shared instance, globally accessible | Global state, hard to test, concurrency risks |
| Factory Method | Decouples client from concrete products | Creator hierarchy mirrors product hierarchy |
| Abstract Factory | Enforces product family consistency | Adding new product types requires changing factory interface |
| Builder | Clean construction of complex objects | More code; Lombok hides but doesn't eliminate it |
| Prototype | Cheap object copying | Deep copy complexity; `Cloneable` is broken |
| Adapter | Integrates incompatible APIs | Extra indirection layer |
| Decorator | Runtime behavior composition; OCP | Many small classes; deep stacks are hard to debug |
| Proxy (AOP) | Transparent cross-cutting concerns | Hidden behavior; self-invocation bypasses proxy |
| Composite | Uniform tree treatment | Overly general: may force unsafe operations on leaf nodes |
| Strategy | Eliminates conditionals; OCP; testable | Context must know which strategy to use |
| Observer | Loose coupling; extensible | Update ordering undefined; memory leaks if not unregistered |
| State | Eliminates state-based conditionals | Class proliferation; transition logic distributed across states |
| Command | Undo/redo, queuing, auditability | More classes per operation; overhead for simple cases |
| Template Method | Reuse of algorithm skeleton | Inheritance coupling; harder to test hooks in isolation |
| Visitor | Add operations without modifying hierarchy | Adding new element types requires changing all visitors |

### 8.4 Communication Tips

1. **Name it, then justify it** — *"I'd use Strategy here because the algorithm for pricing varies at runtime and I want to add new strategies without modifying the context"*
2. **State the alternative you rejected** — shows deeper understanding
3. **Acknowledge trade-offs** — every pattern has costs; naming them signals maturity
4. **Connect to your production experience** — *"We used this in production when..."*
5. **Know when NOT to use it** — YAGNI / over-engineering is a real interview trap
6. **Know the Spring equivalent** — *"This is essentially what `@Transactional` does under the hood"*
