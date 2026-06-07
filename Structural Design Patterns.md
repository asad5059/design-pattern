# Structural Design Patterns

Structural patterns deal with **how objects and classes are composed** to form larger structures. They simplify relationships between entities while keeping the system flexible and efficient.

---

## Core Characteristics

- Focuses on **composition over inheritance**
- Defines how objects **relate and interact** structurally
- Makes it easier to **extend and modify** parts of a system without breaking the whole
- Reduces **coupling** between components

---

## The 7 Structural Patterns

| Pattern | Intent | Key Mechanism |
|---|---|---|
| Adapter | Make incompatible interfaces work together | Wraps an existing class |
| Bridge | Separate abstraction from implementation | Composition via reference |
| Composite | Treat single objects and groups uniformly | Tree structure |
| Decorator | Add behavior dynamically without subclassing | Wraps an object of same interface |
| Facade | Simplify a complex subsystem | Single unified interface |
| Flyweight | Share data to reduce memory usage | Intrinsic vs extrinsic state |
| Proxy | Control access to another object | Same interface as real object |

---

## 1. Adapter

Converts one interface into another that the client expects. Lets incompatible classes work together.

```python
class OldPrinter:
    def old_print(self, text):
        print(f"[Old] {text}")

class NewPrinter:
    def print(self, text): pass

class PrinterAdapter(NewPrinter):
    def __init__(self, old):
        self.old = old
    def print(self, text):
        self.old.old_print(text)   # delegates to old interface

adapter = PrinterAdapter(OldPrinter())
adapter.print("Hello")  # [Old] Hello
```

**When to use:** Integrating legacy code or third-party libraries with a mismatched interface.

---

## 2. Bridge

Decouples an abstraction from its implementation so both can vary independently.

```python
# Implementation side
class DarkTheme:
    def color(self): return "Dark"

class LightTheme:
    def color(self): return "Light"

# Abstraction side (holds a reference — the bridge)
class Button:
    def __init__(self, theme):
        self.theme = theme
    def render(self):
        print(f"Button in {self.theme.color()} theme")

Button(DarkTheme()).render()   # Button in Dark theme
Button(LightTheme()).render()  # Button in Light theme
```

**When to use:** When you want to avoid a permanent binding between abstraction and implementation, and both should grow independently.

---

## 3. Composite

Composes objects into tree structures. Clients treat individual objects and compositions the same way.

```python
class Item:
    def __init__(self, name, price):
        self.name, self.price = name, price
    def total(self): return self.price

class Box:
    def __init__(self):
        self.contents = []
    def add(self, item): self.contents.append(item)
    def total(self): return sum(i.total() for i in self.contents)

box = Box()
box.add(Item("Book", 200))
box.add(Item("Pen", 50))
print(box.total())  # 250  — same call whether it's Item or Box
```

**When to use:** Representing hierarchies — file systems, UI trees, org charts, nested menus.

---

## 4. Decorator

Attaches additional responsibilities to an object dynamically. A flexible alternative to subclassing for extending behavior.

```python
class Coffee:
    def cost(self): return 30
    def desc(self): return "Coffee"

class Milk:
    def __init__(self, base):
        self._base = base
    def cost(self): return self._base.cost() + 10
    def desc(self): return self._base.desc() + " + Milk"

class Sugar:
    def __init__(self, base):
        self._base = base
    def cost(self): return self._base.cost() + 5
    def desc(self): return self._base.desc() + " + Sugar"

drink = Sugar(Milk(Coffee()))
print(drink.desc())  # Coffee + Milk + Sugar
print(drink.cost())  # 45
```

**When to use:** Adding optional features at runtime — logging, auth middleware, UI styling, I/O streams.

---

## 5. Facade

Provides a single simplified interface to a complex subsystem. Hides the complexity from the client.

```python
class CPU:
    def start(self): print("CPU started")

class Memory:
    def load(self): print("Memory loaded")

class Disk:
    def read(self): print("Disk read")

# Facade
class Computer:
    def __init__(self):
        self.cpu, self.mem, self.disk = CPU(), Memory(), Disk()
    def start(self):                  # one clean call
        self.cpu.start()
        self.mem.load()
        self.disk.read()
        print("Computer ready!")

Computer().start()
```

**When to use:** Wrapping complex libraries, providing a simple API over multiple subsystems, SDK design.

---

## 6. Flyweight

Uses sharing to support a large number of fine-grained objects efficiently by separating shared state from unique state.

```python
class TreeType:               # Intrinsic (shared) state
    def __init__(self, name, color):
        self.name, self.color = name, color

class TreeFactory:
    _cache = {}
    @classmethod
    def get(cls, name, color):
        key = f"{name}-{color}"
        if key not in cls._cache:
            cls._cache[key] = TreeType(name, color)
        return cls._cache[key]

# Each Tree holds extrinsic (unique) state: x, y
trees = [(TreeFactory.get("Oak", "Green"), x, y)
         for x, y in [(1,2),(3,4),(5,6)]]   # Only 1 TreeType object shared
```

**When to use:** Rendering thousands of similar objects (game characters, particles, text characters, map tiles).

---

## 7. Proxy

Provides a surrogate for another object to control access, add caching, or defer creation.

```python
class RealImage:
    def __init__(self, file):
        self.file = file
        print(f"Loading {file}...")   # expensive
    def display(self): print(f"Displaying {self.file}")

class ImageProxy:
    def __init__(self, file):
        self.file = file
        self._real = None             # not loaded yet
    def display(self):
        if not self._real:
            self._real = RealImage(self.file)   # lazy load
        self._real.display()

img = ImageProxy("photo.jpg")
img.display()   # loads now
img.display()   # uses cached instance
```

**When to use:** Lazy initialization, access control, logging, remote object representation, caching.

---

## Quick Reference: How They Differ

```
Adapter  →  Changes the interface of an existing object
Bridge   →  Splits one abstraction into two independent hierarchies
Composite→  Lets you work with trees of objects uniformly
Decorator→  Adds behavior by wrapping (same interface, new features)
Facade   →  Hides complexity behind one simple interface
Flyweight→  Shares common state across many small objects
Proxy    →  Same interface as real object, but controls access
```

> **Rule of thumb:** If two things don't fit together → **Adapter**.
> If things are getting too complex → **Facade**.
> If you want extra behavior without changing code → **Decorator**.
> If you need to control access → **Proxy**.
