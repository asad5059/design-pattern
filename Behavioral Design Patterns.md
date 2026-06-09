# Behavioral Design Patterns

Behavioral patterns deal with **how objects communicate and distribute responsibility** at runtime. They define the flow of control and data between objects to keep interactions flexible and loosely coupled.

---

## Core Characteristics

- Focuses on **communication between objects**, not their structure
- Assigns **responsibilities** clearly so no single class does too much
- Replaces rigid control flow (large `if-else`, `switch`) with flexible delegation
- Makes behaviors **interchangeable, extensible, and testable** in isolation

---

## The 11 Behavioral Patterns

| Pattern | Intent | Key Mechanism |
|---|---|---|
| Chain of Responsibility | Pass a request along a chain of handlers | Linked handler objects |
| Command | Encapsulate a request as an object | Invoker, Command, Receiver |
| Iterator | Traverse a collection without exposing its structure | `__iter__` / `__next__` |
| Mediator | Centralize communication between objects | Hub object |
| Memento | Save and restore an object's state | Snapshot object |
| Observer | Notify dependents when state changes | Subscribe / notify |
| State | Change behavior when internal state changes | State objects per state |
| Strategy | Swap algorithms at runtime | Inject strategy object |
| Template Method | Fix the algorithm skeleton, vary the steps | Abstract base class |
| Visitor | Add operations to objects without modifying them | `accept(visitor)` |
| Interpreter | Define a grammar and interpret sentences | Expression tree |

---

## 1. Chain of Responsibility

Passes a request along a chain of handlers. Each handler either processes it or forwards it to the next one. Sender has no knowledge of which handler will respond.

```python
class Handler:
    def __init__(self): self._next = None
    def set_next(self, h): self._next = h; return h
    def handle(self, req):
        if self._next: return self._next.handle(req)

class LowHandler(Handler):
    def handle(self, req):
        if req < 10: print(f"Low handled: {req}")
        else: super().handle(req)

class HighHandler(Handler):
    def handle(self, req):
        if req >= 10: print(f"High handled: {req}")
        else: super().handle(req)

low = LowHandler()
low.set_next(HighHandler())
low.handle(5)   # Low handled: 5
low.handle(15)  # High handled: 15
```

**When to use:** Multiple handlers might process a request and the handler isn't known upfront — middleware pipelines, event bubbling, approval workflows.

---

## 2. Command

Wraps a request as a standalone object. This lets you parameterize, queue, log, and undo operations.

```python
class Light:                        # Receiver
    def on(self):  print("Light ON")
    def off(self): print("Light OFF")

class TurnOnCommand:                # Command
    def __init__(self, light): self.light = light
    def execute(self): self.light.on()
    def undo(self):    self.light.off()

class RemoteControl:                # Invoker
    def __init__(self): self.history = []
    def press(self, cmd):
        cmd.execute()
        self.history.append(cmd)
    def undo(self):
        if self.history: self.history.pop().undo()

remote = RemoteControl()
remote.press(TurnOnCommand(Light()))  # Light ON
remote.undo()                         # Light OFF
```

**When to use:** Undo/redo functionality, job queues, transaction logging, GUI button actions.

---

## 3. Iterator

Provides a way to access elements of a collection sequentially without exposing its underlying structure.

```python
class NumberRange:
    def __init__(self, start, end):
        self.start, self.end = start, end
    def __iter__(self):
        current = self.start
        while current <= self.end:
            yield current           # generator-based iterator
            current += 1

for n in NumberRange(1, 5):
    print(n)                        # 1, 2, 3, 4, 5
```

**When to use:** You need to traverse different collection types (list, tree, graph) through a uniform interface without exposing internals.

---

## 4. Mediator

Defines a central object that handles all communication between components. Components only talk to the mediator, not to each other.

```python
class ChatRoom:                     # Mediator
    def send(self, msg, sender, receiver):
        print(f"[{sender}→{receiver}]: {msg}")

class User:
    def __init__(self, name, room):
        self.name, self.room = name, room
    def send(self, msg, to):
        self.room.send(msg, self.name, to)

room  = ChatRoom()
alice = User("Alice", room)
bob   = User("Bob", room)

alice.send("Hey!", "Bob")           # [Alice→Bob]: Hey!
bob.send("Hi Alice!", "Alice")      # [Bob→Alice]: Hi Alice!
```

**When to use:** Many components are tightly interdependent — chat apps, air traffic control, UI form coordination, Redux/Flux stores.

---

## 5. Memento

Captures an object's internal state in a snapshot so it can be restored later, without breaking encapsulation.

```python
class Memento:
    def __init__(self, state): self._state = state
    def get_state(self): return self._state

class Editor:                       # Originator
    def __init__(self): self.text = ""
    def write(self, t): self.text += t
    def save(self):     return Memento(self.text)
    def restore(self, m): self.text = m.get_state()
    def show(self):     print(f'> "{self.text}"')

editor = Editor()
editor.write("Hello ")
snap = editor.save()               # save checkpoint
editor.write("World")
editor.show()                      # > "Hello World"
editor.restore(snap)               # undo
editor.show()                      # > "Hello "
```

**When to use:** Undo/redo stacks, game save points, database transaction rollbacks.

---

## 6. Observer

Defines a one-to-many relationship: when the subject changes state, all registered observers are notified automatically.

```python
class EventEmitter:                 # Subject
    def __init__(self): self._subs = []
    def subscribe(self, fn):   self._subs.append(fn)
    def unsubscribe(self, fn): self._subs.remove(fn)
    def emit(self, data):
        for fn in self._subs: fn(data)

price_feed = EventEmitter()
price_feed.subscribe(lambda p: print(f"Trader A: price is {p}"))
price_feed.subscribe(lambda p: print(f"Trader B: {'BUY' if p < 50 else 'HOLD'} at {p}"))

price_feed.emit(60)   # Trader A: price is 60 | Trader B: HOLD at 60
price_feed.emit(40)   # Trader A: price is 40 | Trader B: BUY at 40
```

**When to use:** Event-driven systems, UI state updates, pub-sub messaging, React state/effects, Django signals.

---

## 7. State

Allows an object to alter its behavior when its internal state changes. Each state is its own class. Eliminates large `if-else` chains based on state.

```python
class TrafficLight:
    def __init__(self): self._state = RedState(self)
    def set_state(self, s): self._state = s
    def next(self):         self._state.next()

class RedState:
    def __init__(self, light): self.light = light
    def next(self):
        print("RED   → stop")
        self.light.set_state(GreenState(self.light))

class GreenState:
    def __init__(self, light): self.light = light
    def next(self):
        print("GREEN → go")
        self.light.set_state(YellowState(self.light))

class YellowState:
    def __init__(self, light): self.light = light
    def next(self):
        print("YELLOW→ slow down")
        self.light.set_state(RedState(self.light))

light = TrafficLight()
light.next()   # RED   → stop
light.next()   # GREEN → go
light.next()   # YELLOW→ slow down
light.next()   # RED   → stop
```

**When to use:** Object behavior depends heavily on its state — vending machines, order status, TCP connections, ATM flow.

---

## 8. Strategy

Defines a family of interchangeable algorithms, encapsulates each one, and lets the client swap them at runtime without changing the context.

```python
class Sorter:
    def __init__(self, strategy): self.strategy = strategy
    def sort(self, data): return self.strategy(data)

bubble = lambda d: sorted(d)              # stand-in for bubble sort
quick  = lambda d: sorted(d, reverse=False)   # stand-in for quick sort

data = [5, 2, 8, 1]

s = Sorter(bubble)
print(s.sort(data))           # [1, 2, 5, 8]

s.strategy = quick            # swap at runtime
print(s.sort(data))           # [1, 2, 5, 8]
```

**When to use:** Multiple variations of an algorithm are needed — sorting, payment methods, compression, routing, auth strategies.

---

## 9. Template Method

Defines the **skeleton of an algorithm** in a base class. Subclasses override specific steps without changing the overall structure.

```python
from abc import ABC, abstractmethod

class DataReport(ABC):
    def generate(self):         # template method
        self.fetch_data()
        self.process_data()
        self.format_output()

    def fetch_data(self): print("Fetching data...")
    def process_data(self): print("Processing...")

    @abstractmethod
    def format_output(self): pass   # subclass fills this in

class PDFReport(DataReport):
    def format_output(self): print("Exporting as PDF\n")

class CSVReport(DataReport):
    def format_output(self): print("Exporting as CSV\n")

PDFReport().generate()
CSVReport().generate()
```

**When to use:** Several classes share the same algorithm structure but differ in specific steps — report generators, data parsers, test lifecycle hooks (JUnit, Django TestCase).

---

## 10. Visitor

Lets you define a new operation on elements of a structure without modifying the element classes. Operations live in a separate Visitor object.

```python
class Circle:
    def __init__(self, r): self.r = r
    def accept(self, v): v.visit_circle(self)

class Rectangle:
    def __init__(self, w, h): self.w, self.h = w, h
    def accept(self, v): v.visit_rect(self)

class AreaCalculator:           # Visitor
    def visit_circle(self, c):  print(f"Circle area:    {3.14 * c.r**2:.2f}")
    def visit_rect(self, r):    print(f"Rectangle area: {r.w * r.h}")

class Exporter:                 # Another Visitor — no shape changes needed
    def visit_circle(self, c):  print(f"<circle r='{c.r}'/>")
    def visit_rect(self, r):    print(f"<rect w='{r.w}' h='{r.h}'/>")

shapes = [Circle(5), Rectangle(4, 6)]
calc   = AreaCalculator()
for s in shapes: s.accept(calc)
# Circle area:    78.50
# Rectangle area: 24
```

**When to use:** You need to add many unrelated operations to a stable object structure without polluting element classes — AST traversal, compilers, document exporters.

---

## 11. Interpreter

Defines a grammar for a language and provides an interpreter to evaluate sentences in that language. Each grammar rule maps to a class.

```python
# Grammar: expr = Number | Add(expr, expr) | Multiply(expr, expr)

class Number:
    def __init__(self, v): self.v = v
    def interpret(self): return self.v

class Add:
    def __init__(self, l, r): self.l, self.r = l, r
    def interpret(self): return self.l.interpret() + self.r.interpret()

class Multiply:
    def __init__(self, l, r): self.l, self.r = l, r
    def interpret(self): return self.l.interpret() * self.r.interpret()

# Represents: (3 + 4) * 2
expr = Multiply(Add(Number(3), Number(4)), Number(2))
print(expr.interpret())   # 14
```

**When to use:** Building small domain-specific languages (DSL), expression evaluators, SQL/regex parsers, configuration rule engines.

---

## Quick Reference: How They Differ

```
Chain of Resp.  →  Unknown handler; request walks the chain until handled
Command         →  Request as object; enables undo, queue, and logging
Iterator        →  Uniform traversal of any collection type
Mediator        →  Replaces many-to-many wiring with a central hub
Memento         →  Snapshot of state; restores without breaking encapsulation
Observer        →  Subject broadcasts changes; observers react independently
State           →  Object swaps its own behavior class as state transitions
Strategy        →  Caller injects the algorithm; context stays the same
Template Method →  Base class owns the flow; subclass fills in the blanks
Visitor         →  New operation injected into a structure; elements unchanged
Interpreter     →  Grammar rules as objects; evaluate by walking the tree
```

> **Rule of thumb:**
> Too many `if-else` on state? → **State**.
> Same steps, different implementations? → **Template Method** (inheritance) or **Strategy** (composition).
> Need undo/redo? → **Command** + **Memento**.
> Objects broadcasting changes? → **Observer**.
> Objects talking too much to each other? → **Mediator**.
> New operations on a fixed structure? → **Visitor**.
