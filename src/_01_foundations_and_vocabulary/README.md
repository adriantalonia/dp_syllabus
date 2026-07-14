## 1. Foundations & Vocabulary

<!-- TOC -->
  * [1. Foundations & Vocabulary](#1-foundations--vocabulary)
    * [1.1 What Is a Design Pattern?](#11-what-is-a-design-pattern)
    * [1.2 Pattern Classification](#12-pattern-classification)
    * [1.3 Key Vocabulary — Interview Required](#13-key-vocabulary--interview-required)
    * [1.4 UML Essentials for Interviews](#14-uml-essentials-for-interviews)
    * [1.5 Exercises / Problems](#15-exercises--problems)
    * [1.6 Object Relationships in OOP (Java)](#16-object-relationships-in-oop-java)
* [Table of Contents](#table-of-contents)
* [Introduction](#introduction)
* [Step 1 - Objects Without Relationships](#step-1---objects-without-relationships)
* [Association](#association)
  * [Definition](#definition)
  * [UML](#uml)
  * [Java Example](#java-example)
  * [Another Example](#another-example)
  * [Dependency Injection Example](#dependency-injection-example)
  * [Key Idea](#key-idea)
* [Aggregation](#aggregation)
  * [Definition](#definition-1)
  * [University Example](#university-example)
  * [Java Example](#java-example-1)
  * [More Aggregation Examples](#more-aggregation-examples)
  * [Important Note](#important-note)
* [Composition](#composition)
  * [Definition](#definition-2)
  * [House Example](#house-example)
  * [Java Example](#java-example-2)
  * [More Composition Examples](#more-composition-examples)
* [Aggregation vs Composition](#aggregation-vs-composition)
  * [Scenario 1](#scenario-1)
  * [Scenario 2](#scenario-2)
* [Visual Comparison](#visual-comparison)
  * [Association](#association-1)
  * [Aggregation](#aggregation-1)
  * [Composition](#composition-1)
* [Spring Boot Examples](#spring-boot-examples)
  * [Association](#association-2)
  * [Aggregation](#aggregation-2)
  * [Composition](#composition-2)
* [Database Perspective](#database-perspective)
  * [Composition](#composition-3)
  * [Aggregation](#aggregation-3)
* [Decision Tree](#decision-tree)
* [Comparison Table](#comparison-table)
* [Common Interview Questions](#common-interview-questions)
  * [1. What is Association?](#1-what-is-association)
  * [2. What is Aggregation?](#2-what-is-aggregation)
  * [3. What is Composition?](#3-what-is-composition)
  * [4. What is the main difference between Aggregation and Composition?](#4-what-is-the-main-difference-between-aggregation-and-composition)
  * [5. Is Dependency Injection Composition?](#5-is-dependency-injection-composition)
  * [6. Is @OneToMany always Composition?](#6-is-onetomany-always-composition)
  * [7. Does using `new` automatically mean Composition?](#7-does-using-new-automatically-mean-composition)
  * [8. Can the same child belong to multiple Compositions?](#8-can-the-same-child-belong-to-multiple-compositions)
* [Cheat Sheet](#cheat-sheet)
* [Final Takeaways](#final-takeaways)
  * [The Golden Rule (Interview Tip)](#the-golden-rule-interview-tip)
<!-- TOC -->

### 1.1 What Is a Design Pattern?

A design pattern is a **reusable, named solution to a recurring design problem** within a given context. Patterns are
not code you copy — they are templates for thinking.

Origin path: Christopher Alexander (architecture) → Gang of Four book (1994) → Martin Fowler's enterprise patterns →
Cloud/microservice patterns. Each layer builds on the previous.

A complete pattern description has four parts:

- **Name** — shared vocabulary; enables concise communication
- **Problem** — when to apply it; the context and forces
- **Solution** — the arrangement of classes, objects, and responsibilities
- **Consequences** — trade-offs, results, and known uses

**🎤 Interview Quick-Fire**

- *Q: "Pattern vs. Principle?"* → Principles are guiding rules (SOLID, "encapsulate what varies"). Patterns are concrete
  solutions that embody those rules.
- *Q: "When NOT to use a pattern?"* → When the problem is simple/stable. YAGNI first; refactor toward a pattern when
  change becomes frequent.

---

### 1.2 Pattern Classification

| Category       | Focus                             | GoF Count | Examples                           |
|----------------|-----------------------------------|-----------|------------------------------------|
| **Creational** | Object instantiation              | 5         | Factory Method, Builder, Singleton |
| **Structural** | Object composition                | 7         | Adapter, Decorator, Proxy          |
| **Behavioral** | Object communication & algorithms | 11        | Strategy, Observer, Command        |

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

**Sequence diagram elements:** lifelines, activation bars, synchronous message (`─►`), asynchronous message (`─>>`),
return (`- - ►`), self-call

### 1.5 Exercises / Problems

- [ ] Given a class diagram with 4 classes, identify which GoF pattern is implemented
- [ ] Draw the class diagram for Strategy, Observer, and Decorator from memory
- [ ] Explain the difference between design patterns and design principles in one paragraph
- [ ] Name three patterns that primarily enable OCP, three that enable DIP

### 1.6 Object Relationships in OOP (Java)

> A complete guide to **Association**, **Aggregation**, and **Composition** with Java examples, UML diagrams, lifecycle
> explanations, and common interview questions.

---

# Table of Contents

1. Introduction
2. Association
3. Aggregation
4. Composition
5. Comparison
6. UML Diagrams
7. Java Examples
8. Spring Boot Examples
9. JPA Examples
10. Decision Tree
11. Interview Questions
12. Cheat Sheet

---

# Introduction

One of the most common Object-Oriented Programming interview topics is understanding the relationships between objects.

Many developers memorize definitions like:

- Association = uses
- Aggregation = has-a
- Composition = strong has-a

While partially correct, these definitions don't explain **why** each relationship exists.

The real difference lies in **ownership** and **object lifecycle**.

---

# Step 1 - Objects Without Relationships

Suppose we have two simple classes.

```java
class Person {
}

class Car {
}
```

At this point:

```
Person

Car
```

There is **no relationship** between them.

---

# Association

## Definition

Association means:

> Two independent objects collaborate or know about each other.

Neither object owns the other.

Neither controls the other's lifecycle.

They simply interact.

Think of it like a friendship.

---

## UML

```
Person ---------------- Car
```

The person can drive the car.

The car still exists if the person disappears.

The person still exists if the car is destroyed.

They are completely independent.

---

## Java Example

```java
class Car {

    private String model;

    public Car(String model) {
        this.model = model;
    }

    public void start() {
        System.out.println(model + " started");
    }
}
```

```java
class Driver {

    public void drive(Car car) {
        car.start();
    }
}
```

Usage

```java
Car car = new Car("Mazda");

Driver driver = new Driver();

driver.

drive(car);
```

Notice:

The Driver does **not own** the Car.

The Car simply participates in the method.

```
Driver -------> Car
```

This is Association.

---

## Another Example

```java
class Doctor {

    public void examine(Patient patient) {

    }

}
```

The doctor does not own the patient.

The patient exists independently.

---

## Dependency Injection Example

Spring Boot constructor injection is also Association.

```java

@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

}
```

Spring owns the PaymentService.

OrderService only collaborates with it.

---

## Key Idea

Association answers:

> Can these objects collaborate?

NOT

> Who owns whom?

---

# Aggregation

## Definition

Aggregation is a specialized Association.

It means:

> One object contains another object, but the contained object has an independent lifecycle.

Ownership is **weak**.

---

## University Example

```
        University
            ◇
            |
    -------------------
    |        |        |
 Student  Student  Student
```

The University has Students.

If the University closes...

Students still exist.

They simply enroll somewhere else.

---

## Java Example

```java
class Student {

    private String name;

    Student(String name) {
        this.name = name;
    }

}
```

```java
class University {

    private List<Student> students;

    University(List<Student> students) {
        this.students = students;
    }

}
```

Usage

```java
Student john = new Student("John");
Student alice = new Student("Alice");

List<Student> students = List.of(john, alice);

University mit = new University(students);
```

Important:

Students already existed before the University.

```
Student
    ↓
Student
    ↓
University
```

Destroying the University does NOT destroy Students.

---

## More Aggregation Examples

- Company → Employees
- Team → Players
- Hospital → Doctors
- Library → Books

Each child object can exist independently.

---

## Important Note

This does NOT automatically mean Aggregation.

```java
List<Student> students;
```

Using a List does **not** determine the relationship.

Only lifecycle determines it.

---

# Composition

## Definition

Composition means:

> The parent owns the child.

The child cannot meaningfully exist without the parent.

Ownership is **strong**.

---

## House Example

```
         House
           ◆
           |
     ----------------
     |      |       |
   Room   Room   Room
```

Destroy the House.

The Rooms disappear.

---

## Java Example

```java
class Engine {

    public void start() {
        System.out.println("Engine started");
    }

}
```

```java
class Car {

    private Engine engine;

    public Car() {
        this.engine = new Engine();
    }

}
```

Usage

```java
Car car = new Car();
```

Nobody creates the Engine directly.

The Car owns it.

```
Car

↓

creates

↓

Engine
```

Destroy the Car.

The Engine disappears too.

---

## More Composition Examples

- Order → OrderItems
- Invoice → InvoiceItems
- House → Rooms
- Human → Heart

---

# Aggregation vs Composition

Suppose we have

```java
class Team {

    private List<Player> players;

}
```

Can we determine the relationship?

No.

Java syntax alone cannot determine ownership.

Business rules decide.

---

## Scenario 1

Players can leave the Team.

The Team can disappear.

Players continue existing.

Aggregation.

---

## Scenario 2

Players only exist inside a tournament.

Tournament deleted.

Players disappear.

Composition.

---

# Visual Comparison

## Association

```
Person -------- Car
```

Independent collaboration.

---

## Aggregation

```
University
     ◇
     |
 Student
```

Weak ownership.

Student survives.

---

## Composition

```
Car
◆
|
Engine
```

Strong ownership.

Engine does not survive.

---

# Spring Boot Examples

## Association

```java

@Service
class UserService {

    private final EmailService emailService;

}
```

EmailService exists independently.

---

## Aggregation

```java
class Company {

    private List<Employee> employees;

}
```

Employees continue existing if Company disappears.

---

## Composition

```java

@Entity
class Order {

    @OneToMany(
            cascade = CascadeType.ALL,
            orphanRemoval = true
    )
    private List<OrderItem> items;

}
```

Deleting the Order deletes its OrderItems.

---

# Database Perspective

## Composition

```
DELETE ORDER

↓

DELETE ORDER_ITEMS
```

---

## Aggregation

```
DELETE DEPARTMENT

↓

EMPLOYEE.department_id = NULL
```

or

Employee moves to another department.

---

# Decision Tree

```
Do objects collaborate?

        Yes
         |
         |
 Association
         |
         |
Does one object own another?
      /               \
    No                Yes
                      |
Can child exist alone?
      /           \
    Yes            No
     |             |
Aggregation    Composition
```

---

# Comparison Table

| Feature               | Association  | Aggregation     | Composition      |
|-----------------------|--------------|-----------------|------------------|
| Objects collaborate   | ✅            | ✅               | ✅                |
| Ownership             | None         | Weak            | Strong           |
| Child survives parent | ✅            | ✅               | ❌                |
| Lifecycle independent | ✅            | ✅               | ❌                |
| Parent creates child  | Not required | Not required    | Usually          |
| UML Symbol            | Line         | Empty Diamond ◇ | Filled Diamond ◆ |

---

# Common Interview Questions

## 1. What is Association?

Association is a relationship where two independent objects collaborate without owning each other.

---

## 2. What is Aggregation?

Aggregation is a weak ownership relationship where the child object has an independent lifecycle.

---

## 3. What is Composition?

Composition is a strong ownership relationship where the child object's lifecycle depends on the parent.

---

## 4. What is the main difference between Aggregation and Composition?

Ownership and lifecycle.

Aggregation:

- Child survives.

Composition:

- Child dies with the parent.

---

## 5. Is Dependency Injection Composition?

No.

Dependency Injection is Association.

The Spring container owns the dependency.

---

## 6. Is @OneToMany always Composition?

No.

JPA mappings describe persistence relationships.

Business rules determine whether the relationship is Aggregation or Composition.

---

## 7. Does using `new` automatically mean Composition?

No.

Creating an object with `new` is common in Composition, but the defining factor is lifecycle ownership, not object
creation syntax.

---

## 8. Can the same child belong to multiple Compositions?

Generally, no.

Composition implies exclusive ownership.

If multiple parents share the same child, the relationship is usually Aggregation or Association.

---

# Cheat Sheet

| Relationship | Remember This                 |
|--------------|-------------------------------|
| Association  | "Works with"                  |
| Aggregation  | "Has, but doesn't own"        |
| Composition  | "Owns and controls lifecycle" |

---

# Final Takeaways

Always remember:

Association answers:

> Can these objects work together?

Aggregation answers:

> Does one object contain another while allowing it to exist independently?

Composition answers:

> Does one object own another and control its lifecycle?

---

## The Golden Rule (Interview Tip)

Many developers try to identify these relationships by looking only at Java code or UML diagrams.

Experienced engineers know that **the business domain determines the relationship**, not the syntax.

When in doubt, ask yourself one question:

> **Can the child object meaningfully exist without the parent?**

- **Yes** → Aggregation
- **No** → Composition

If there is no ownership at all, and the objects simply collaborate:

→ Association.