
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
