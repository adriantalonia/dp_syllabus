## 1. Foundations & Vocabulary

### 1.1 What Is a Design Pattern?

A design pattern is a **reusable, named solution to a recurring design problem** within a given context. Patterns are not code you copy — they are templates for thinking.

Origin path: Christopher Alexander (architecture) → Gang of Four book (1994) → Martin Fowler's enterprise patterns → Cloud/microservice patterns. Each layer builds on the previous.

A complete pattern description has four parts:
- **Name** — shared vocabulary; enables concise communication
- **Problem** — when to apply it; the context and forces
- **Solution** — the arrangement of classes, objects, and responsibilities
- **Consequences** — trade-offs, results, and known uses

**🎤 Interview Quick-Fire**
- *Q: "Pattern vs. Principle?"* → Principles are guiding rules (SOLID, "encapsulate what varies"). Patterns are concrete solutions that embody those rules.
- *Q: "When NOT to use a pattern?"* → When the problem is simple/stable. YAGNI first; refactor toward a pattern when change becomes frequent.

---

### 1.2 Pattern Classification

| Category | Focus | GoF Count | Examples |
|---|---|---|---|
| **Creational** | Object instantiation | 5 | Factory Method, Builder, Singleton |
| **Structural** | Object composition | 7 | Adapter, Decorator, Proxy |
| **Behavioral** | Object communication & algorithms | 11 | Strategy, Observer, Command |

### 1.3 Key Vocabulary — Interview Required

- **Coupling** — degree to which one class depends on another. Goal: *low coupling*
- **Cohesion** — degree to which elements of a class belong together. Goal: *high cohesion*
- **Composition over Inheritance** — prefer *has-a* over *is-a*; runtime flexibility vs compile-time rigidity
- **Program to an interface, not an implementation** — depend on abstractions, not concrete types
- **Encapsulate what varies** — isolate the moving parts behind a stable interface
- **Hollywood Principle** — "don't call us, we'll call you" — inversion of control
- **Invariant** — constraint that must always hold on an object's state
- **Contract** — preconditions, postconditions, invariants (Design by Contract, Liskov)
- **Double dispatch** — method resolution based on the runtime type of two objects (used in Visitor)

### 1.4 UML Essentials for Interviews

You must be able to sketch these on a whiteboard:

**Class diagram relationships:**
- `──────>` Dependency (uses)
- `──────` Association (has-a, field reference)
- `◇─────` Aggregation (has-a, lifecycle independent)
- `◆─────` Composition (has-a, lifecycle dependent)
- `──────▷` Inheritance / Generalization
- `- - - -▷` Realization / Implements

**Sequence diagram elements:** lifelines, activation bars, synchronous message (`─►`), asynchronous message (`─>>`), return (`- - ►`), self-call

### 1.5 Exercises / Problems

- [ ] Given a class diagram with 4 classes, identify which GoF pattern is implemented
- [ ] Draw the class diagram for Strategy, Observer, and Decorator from memory
- [ ] Explain the difference between design patterns and design principles in one paragraph
- [ ] Name three patterns that primarily enable OCP, three that enable DIP
