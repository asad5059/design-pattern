
## 📖 1. Composition

### 📝 Definition

**Composition** is an object-oriented design principle where you build complex objects by combining (or "composing") simpler objects.
Instead of inheritance (`is-a` relationship), composition favors a `has-a` relationship.

---

### ✅ Benefits

* **Loose coupling** – easy to swap components.
* **Better flexibility** – behavior can be changed at runtime.
* **Avoids deep inheritance trees** – easier to maintain and reason about.

---

### 📌 Example: Car and Engine

```python
# Component
class Engine:
    def start(self):
        print("Engine starting...")

# Composing Class
class Car:
    def __init__(self, engine: Engine):
        self.engine = engine  # Composition

    def drive(self):
        self.engine.start()
        print("Car is moving.")

# Usage
petrol_engine = Engine()
my_car = Car(petrol_engine)
my_car.drive()
```

Here:

* `Car` **has an** `Engine`.
* You could swap `Engine` with `ElectricEngine` or `HybridEngine` without changing `Car`’s code.

---

### 🏗 Composition in a Nutshell

```
Car ---- has-a ----> Engine
```

Composition by itself **does not define a pattern** — it just gives you the ability to plug in and swap objects.

---

## 🎯 2. Strategy Design Pattern

### 📝 Definition

The **Strategy Pattern** is a **behavioral design pattern** that:

> Defines a family of algorithms, encapsulates each one, and makes them interchangeable.
> It lets the algorithm vary independently from the clients that use it.

In short: **use composition to delegate a behavior** (or algorithm) that can be swapped at runtime.

---

### 🧩 Building Blocks

1. **Strategy Interface** – defines a common method signature for the behavior.
2. **Concrete Strategies** – implement the behavior in different ways.
3. **Context Class** – holds a reference to a Strategy and delegates execution to it.

---

### ✅ Benefits

* **Removes big if-else/switch statements** by delegating behavior.
* **Open for extension** – add new behaviors without modifying existing code.
* **Behavior can change at runtime** by swapping strategies.

---

### 📌 Example: Duck Simulator

```python
from abc import ABC, abstractmethod

# --- Strategy Interfaces ---
class FlyBehavior(ABC):
    @abstractmethod
    def fly(self):
        pass

# --- Concrete Strategies ---
class FlyWithWings(FlyBehavior):
    def fly(self):
        print("I'm flying with wings!")

class FlyNoWay(FlyBehavior):
    def fly(self):
        print("I can't fly...")

# --- Context (Duck) ---
class Duck:
    def __init__(self, fly_behavior: FlyBehavior):
        self.fly_behavior = fly_behavior  # Composition

    def perform_fly(self):
        self.fly_behavior.fly()

    def set_fly_behavior(self, fly_behavior: FlyBehavior):
        self.fly_behavior = fly_behavior  # Swap at runtime!

# --- Usage ---
mallard = Duck(FlyWithWings())
mallard.perform_fly()  # "I'm flying with wings!"

print("Oops, broke a wing...")
mallard.set_fly_behavior(FlyNoWay())
mallard.perform_fly()  # "I can't fly..."
```

---

### 🏗 Strategy Pattern Diagram

```
+----------------+      +-----------------+
|     Duck       |----->|   FlyBehavior   |<---+
| (Context)      |      | (Interface)    |    |
+----------------+      +-----------------+    |
| fly_behavior   |      ^                      |
| perform_fly()  |      |                      |
+----------------+   +----------------+   +----------------+
                     | FlyWithWings   |   |   FlyNoWay     |
                     | (Concrete)     |   |  (Concrete)    |
                     +----------------+   +----------------+
```

---

### 🎯 Key Principle

> **Favor Composition over Inheritance**
> Use composition (a "has-a" relationship) to swap behaviors dynamically, instead of hardcoding them in a class hierarchy.

---

## 🔑 Composition vs. Strategy Pattern (Side-by-Side)

| Aspect               | Composition (General)     | Strategy Pattern (Specific)                        |
| -------------------- | ------------------------- | -------------------------------------------------- |
| **Definition**       | Combine objects (“has-a”) | Encapsulate interchangeable behaviors              |
| **Scope**            | General OOP principle     | Behavioral design pattern                          |
| **Formal Structure** | None (freeform)           | Context + Strategy Interface + Concrete Strategies |
| **Problem Solved**   | Reuse, modularity         | Runtime swapping of algorithms/behaviors           |
| **Examples**         | Car has an Engine         | Duck can fly with wings or not fly                 |

---

## 🏁 Summary

* **Composition** is the **Lego block** — it lets you attach one object to another.
* **Strategy Pattern** is a **Lego build** — it uses composition + interfaces + polymorphism to solve a specific problem: **making behavior interchangeable at runtime**.
* Both are crucial tools for writing flexible, maintainable, and extensible code.

---

## QnA
> I still didn't get it. We are injecting fly behavior in a same manner as we are injecting engine in a car. So, what's the catch here?

💡 **You’ve hit the core of the confusion** — and this is where most people get stuck when first learning patterns.
You’re right: injecting `fly_behavior` into `Duck` **feels exactly the same** as injecting `engine` into `Car`.

The **difference is not in the mechanics** (yes, both use composition!) —
the **difference is in the *intent*** and the *problem being solved*.

Lets go this step by step:

---

## 🔑 1. Both Use the Same Technique

Yes — in both cases:

* We **compose** objects together (Car has Engine, Duck has FlyBehavior)
* We **can swap** the component (change engine, change fly behavior)
* We may even use **interfaces + polymorphism** to make them interchangeable

So mechanically, they’re the same.
But patterns are not about syntax — they’re about **design intent**.

---

## 🎯 2. The Catch: *Why* We’re Doing It

| **Car + Engine** (General Composition)                                      | **Duck + FlyBehavior** (Strategy Pattern)                                          |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Purpose: represent a **real-world object** accurately (a car has an engine) | Purpose: represent a **family of algorithms/behaviors** that can vary              |
| Behavior is usually **intrinsic** (car must have one engine)                | Behavior is **optional/dynamic** (duck can fly with wings, jetpack, or not at all) |
| Swapping engine is **rare** (not common in runtime logic)                   | Swapping behavior is **expected** (may happen often at runtime)                    |
| No formal “pattern” — just good OOP                                         | Follows a **pattern recipe**: Context + Strategy Interface + Concrete Strategies   |
| Main goal: **object modeling**                                              | Main goal: **algorithm/behavior flexibility**                                      |

---

### 👀 The Key Catch

> **Strategy Pattern is composition with a specific goal:**
> **Encapsulating a family of interchangeable behaviors and allowing them to vary independently from the object that uses them.**

A car doesn’t really need a *family of interchangeable engines at runtime*.
But a Duck Simulator **needs to be able to switch fly behaviors** dynamically (and easily add new ones without modifying Duck).

---

## 🧠 Another Perspective

Let’s step away from ducks and cars for a second and look at a clearer case:

### Example: Sorting Algorithms

```python
class SortStrategy:
    def sort(self, data): pass

class BubbleSort(SortStrategy):
    def sort(self, data): print("Bubble sort")

class QuickSort(SortStrategy):
    def sort(self, data): print("Quick sort")

class SortContext:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy

    def set_strategy(self, strategy: SortStrategy):
        self.strategy = strategy

    def sort(self, data):
        self.strategy.sort(data)

# Usage
data = [5, 3, 1]
ctx = SortContext(BubbleSort())
ctx.sort(data)  # Bubble sort
ctx.set_strategy(QuickSort())  # Switch algorithm
ctx.sort(data)  # Quick sort
```

Here, **the entire purpose** of the pattern is to **swap sorting algorithms** dynamically —
that’s the *catch*: **it’s solving the problem of interchangeable algorithms.**

---

## 🔍 TL;DR

* **Composition = HOW** we inject a part into another class.
* **Strategy Pattern = WHY** we do it (to make behaviors interchangeable, extensible, and swappable at runtime).

Think of it like this:

* Injecting an engine → just modeling reality (composition).
* Injecting a fly behavior → making behavior flexible by design (strategy pattern).

---
