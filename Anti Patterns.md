# Antipatterns

Antipatterns are **commonly used solutions that look reasonable but make things worse** — harder to read, test, extend, or maintain. Unlike mistakes, they are recognizable patterns with known, better alternatives.

---

## Core Characteristics

- Appear as a **quick fix** but create long-term technical debt
- Often emerge from **good intentions** applied in the wrong context
- Are **widespread and named** — which makes them easier to communicate and fix
- Every antipattern has a **refactored solution** that replaces it

---

## The 12 Antipatterns

★ = covered in detail below

| # | Antipattern | The Problem | The Fix |
|---|---|---|---|
| ★ 1 | God Object | One class knows and does everything | Split by Single Responsibility |
| ★ 2 | Spaghetti Code | Tangled, unstructured logic | Modularize and structure flow |
| ★ 3 | Copy-Paste Programming | Duplication instead of abstraction | Extract reusable functions/classes |
| ★ 4 | Magic Numbers | Unexplained hardcoded values | Named constants |
| 5 | Premature Optimization | Optimizing before profiling | Measure first, then optimize |
| 6 | Anemic Domain Model | Domain objects are just data bags | Move logic into the model |
| 7 | Refused Bequest | Subclass ignores what it inherits | Prefer composition over inheritance |
| ★ 8 | Circular Dependency | A depends on B, B depends on A | Invert or extract a shared abstraction |
| 9 | Yo-Yo Problem | Deep inheritance forces constant up/down navigation | Flatten hierarchy |
| 10 | Golden Hammer | Applying the same tool to every problem | Match tool to context |
| 11 | Lava Flow | Dead code nobody dares to remove | Delete it; version control is your safety net |
| 12 | Big Ball of Mud | No structure — everything is tangled with everything | Introduce layers and boundaries |

---

## 1. God Object

One class accumulates too many responsibilities over time. It knows everything and does everything — the entire system depends on it.

```python
# ❌ Antipattern
class Application:
    def get_user(self, id): ...
    def hash_password(self, pw): ...
    def send_email(self, to, body): ...
    def render_template(self, name): ...
    def connect_to_db(self): ...
    def write_log(self, msg): ...
    def calculate_tax(self, amount): ...
    # ... 40 more methods

# ✅ Better — split by responsibility
class UserRepository:
    def get_user(self, id): ...

class EmailService:
    def send(self, to, body): ...

class AuthService:
    def hash_password(self, pw): ...
```

**How to spot it:** A class with dozens of unrelated methods; every file imports it; it's impossible to test in isolation.

---

## 2. Spaghetti Code

Unstructured logic with deeply nested conditionals, no clear flow, and side effects scattered everywhere. Follows no discernible design.

```python
# ❌ Antipattern
def process(order):
    if order:
        if order['user']:
            if order['user']['active']:
                if order['items']:
                    for item in order['items']:
                        if item['stock'] > 0:
                            if item['price'] > 0:
                                # actual logic buried 6 levels deep
                                pass

# ✅ Better — early returns flatten the nesting (guard clauses)
def process(order):
    if not order or not order.get('user'):     return
    if not order['user'].get('active'):        return
    if not order.get('items'):                 return
    for item in order['items']:
        if item['stock'] > 0 and item['price'] > 0:
            process_item(item)                 # logic extracted
```

**How to spot it:** Functions you are afraid to touch; logic that can only be understood by running it; nesting deeper than 3 levels.

---

## 3. Copy-Paste Programming

Duplicating code instead of abstracting it. Works short-term, creates a maintenance nightmare — a bug must be fixed in every copy.

```python
# ❌ Antipattern
def get_user_discount(user):
    if user['purchases'] > 100:
        rate = 0.10
    elif user['purchases'] > 50:
        rate = 0.05
    else:
        rate = 0.0
    return rate

def get_member_discount(member):   # identical logic, different name
    if member['purchases'] > 100:
        rate = 0.10
    elif member['purchases'] > 50:
        rate = 0.05
    else:
        rate = 0.0
    return rate

# ✅ Better — one function, parameterized
def get_discount(purchases: int) -> float:
    if purchases > 100: return 0.10
    if purchases > 50:  return 0.05
    return 0.0
```

**How to spot it:** Searching the codebase for a bug fix reveals the same code block in 5 files; functions that are identical except for a variable name.

---

## 4. Magic Numbers / Magic Strings

Hardcoded literals scattered through the code with no explanation of what they mean or why that value was chosen.

```python
# ❌ Antipattern
def calculate_fee(amount):
    return amount * 0.035          # what is 0.035?

if status == 3:                    # what does 3 mean?
    send_alert()

time.sleep(86400)                  # why 86400?

# ✅ Better — named constants
TRANSACTION_FEE_RATE = 0.035
STATUS_OVERDUE       = 3
SECONDS_PER_DAY      = 86_400      # underscore for readability

def calculate_fee(amount):
    return amount * TRANSACTION_FEE_RATE

if status == STATUS_OVERDUE:
    send_alert()
```

**How to spot it:** Raw numbers or strings in logic with no comment; changing one value requires a grep across the whole codebase.

---

## 5 – 7 & 9 – 12. Also Worth Knowing

These antipatterns are not covered in depth here, but are worth being aware of.

| # | Antipattern | One-liner |
|---|---|---|
| 5 | **Premature Optimization** | Optimizing before you have profiling data. *"Make it work, make it right, make it fast — in that order."* |
| 6 | **Anemic Domain Model** | Domain objects that are just data bags with no behavior; all logic leaks into `*Service` classes. |
| 7 | **Refused Bequest** | A subclass inherits from a parent but overrides everything to raise errors — it shouldn't be a subclass at all. |
| 9 | **Yo-Yo Problem** | Inheritance so deep that understanding one method requires jumping up and down 5+ class levels. |
| 10 | **Golden Hammer** | Reaching for the same familiar tool regardless of fit. *"If all you have is a hammer, everything looks like a nail."* |
| 11 | **Lava Flow** | Dead code nobody dares remove — commented-out blocks, unused functions, flags that are always `True`. |
| 12 | **Big Ball of Mud** | No architecture whatsoever; everything depends on everything; nobody fully understands the system. |

---

## 8. Circular Dependency

Module A imports Module B, and Module B imports Module A. Creates tight coupling, makes testing and refactoring painful, and often causes import errors.

```python
# ❌ Antipattern
# user.py
from order import Order
class User:
    def place_order(self): return Order(self)

# order.py
from user import User           # circular!
class Order:
    def get_owner(self) -> User: ...

# ✅ Better — extract a shared abstraction both can depend on
# interfaces.py
class UserProtocol: ...         # abstract, no imports

# user.py
from interfaces import UserProtocol
class User(UserProtocol): ...

# order.py
from interfaces import UserProtocol
class Order:
    def __init__(self, owner: UserProtocol): ...
```

**How to spot it:** `ImportError: cannot import name X` at startup; tests that can't import a single class without pulling in half the codebase.

---

## Quick Reference: Antipattern Categories

```
Code-Level
  Magic Numbers          →  Replace with named constants
  Copy-Paste Programming →  Extract and reuse (DRY)
  Spaghetti Code         →  Guard clauses + extract methods
  Premature Optimization →  Profile first, optimize second

OOP-Level
  God Object             →  Single Responsibility Principle
  Anemic Domain Model    →  Move behavior into domain objects
  Refused Bequest        →  Prefer composition over deep inheritance
  Yo-Yo Problem          →  Flatten hierarchy with composition
  Circular Dependency    →  Introduce abstractions / invert dependencies

Architecture-Level
  Golden Hammer          →  Match tool to problem context
  Lava Flow              →  Delete dead code aggressively
  Big Ball of Mud        →  Introduce clear layers and boundaries
```

> **Rule of thumb:**
> Code hard to read? → **Spaghetti Code** or **Magic Numbers**.
> One class/file touched in every PR? → **God Object** or **Big Ball of Mud**.
> Inheritance feels wrong? → **Refused Bequest** or **Yo-Yo Problem**.
> Import errors or test setup pain? → **Circular Dependency**.
> Tech choice made before the problem was understood? → **Golden Hammer**.
> Afraid to delete code? → **Lava Flow**.
