## 7. Tactical Anti-Patterns & Code Smells

> These are class/module-level problems. Architectural anti-patterns (Distributed Monolith, Big Ball of Mud) are in `software-design.md`.

| Anti-Pattern | Description | Symptoms | Fix |
|---|---|---|---|
| **God Class** | One class knows/does too much | 1000+ lines, touches everything | SRP decomposition |
| **Anemic Domain Model** | Domain objects are data bags; all logic in services | POJOs with only getters/setters; fat `@Service` | Move behavior to domain objects |
| **Service Locator** | Objects pull dependencies from a registry | `context.getBean(...)` scattered in code | DIP + constructor injection |
| **Singleton Overuse** | Everything is a static singleton | Untestable, hidden state, contention | Inject as Spring bean |
| **Primitive Obsession** | Domain concepts expressed as primitives | `String email`, `int money`, `String status` | Value Objects, enums |
| **Shotgun Surgery** | One change requires editing many files | Single feature, 10 file changes | Consolidate; SRP |
| **Feature Envy** | Method accesses another class's data excessively | Method calls 5+ getters on another object | Move method to that class |
| **Inappropriate Intimacy** | Two classes too tightly coupled | Access to private internals, circular imports | Interface + DIP |
| **Data Clumps** | Same group of fields always together | `firstName, lastName, email` in 5 classes | Extract to `ContactInfo` VO |
| **Parallel Inheritance Hierarchies** | Every new subclass in A requires one in B | `SmsNotification` + `SmsService` always together | Bridge pattern |
| **Speculative Generality** | Abstractions built for imaginary future needs | Unused interfaces, generic parameters | YAGNI; delete it |
| **Magic Numbers / Strings** | Unexplained literals in logic | `if (status == 3)`, `Thread.sleep(86400000)` | Named constants, enums |
| **N+1 Query Problem** | ORM lazy-loads in a loop | 100 orders → 101 SQL queries | `@EntityGraph`, JOIN FETCH |
| **Circular Dependency** | A → B → A | Spring `BeanCurrentlyInCreationException` | Refactor; use events |
| **Lava Flow** | Dead code never removed | Commented blocks, `@Deprecated` never deleted | Delete it |

**Exercises / Problems:**
- [ ] Identify 5 anti-patterns in a provided Spring Boot codebase (set up a code review exercise)
- [ ] Explain why Anemic Domain Model is considered an anti-pattern in DDD but may be acceptable in CRUD apps
- [ ] `Service Locator` vs `Dependency Injection` — are they equivalent? What does Spring's container use?
