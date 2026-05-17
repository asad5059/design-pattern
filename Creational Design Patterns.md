# Creational Design Patterns

> *"The act of object creation is the most fundamental — and most dangerous — thing we do in software."*

---

## Table of Contents

1. [What Are Creational Patterns?](#what-are-creational-patterns)
2. [The Core Idea](#the-core-idea)
3. [The Five Patterns](#the-five-patterns)
   - [Singleton](#1-singleton)
   - [Factory Method](#2-factory-method)
   - [Abstract Factory](#3-abstract-factory)
   - [Builder](#4-builder)
   - [Prototype](#5-prototype)
4. [Quick Comparison](#quick-comparison)
5. [Final Thoughts](#final-thoughts)

---

## What Are Creational Patterns?

Design patterns are proven, reusable solutions to recurring problems in software design. The Gang of Four (GoF) catalogued 23 of them, grouped into three families: **Creational**, Structural, and Behavioral.

Creational patterns deal with **one specific concern: how objects are created**.

As systems grow, object creation stops being trivial. We start asking:

- Who should be responsible for creating this object?
- Which concrete type should we instantiate — and should the caller even know?
- How many instances should exist?
- Should we build it step by step, or clone an existing one?

Creational patterns answer these questions.

---

## The Core Idea

The central philosophy behind all five creational patterns is this:

> **Hide the `new` keyword from the caller.**

Instead of the calling code saying *"I want a `PostgresDatabase`"*, it says *"I want a `Database`"* — and the pattern decides what it actually gets. This decouples client code from concrete classes, making systems easier to extend, test, and maintain.

A pattern is **creational** if it:
1. Controls or delegates object construction
2. Abstracts which class gets instantiated
3. Manages object lifecycle, count, or initialization complexity

All five GoF creational patterns satisfy these three criteria.

---

## The Five Patterns

### 1. Singleton

**Ensures a class has exactly one instance and provides a global access point to it.**

```python
class Config:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.settings = {"debug": True, "db_url": "postgres://..."}
        return cls._instance

c1 = Config()
c2 = Config()
print(c1 is c2)  # True
```

No matter how many times we call `Config()`, we get the same object back.

**Why it's creational:** Singleton intercepts the normal instantiation process and enforces a constraint on quantity — *only one*. The caller never truly creates an object; it retrieves a controlled instance. This is the most direct form of managing object lifecycle.

---

### 2. Factory Method

**Defines a method for creating an object, but lets subclasses decide which class to instantiate.**

```python
from abc import ABC, abstractmethod

class Notifier(ABC):
    @abstractmethod
    def send(self, message: str): pass

class EmailNotifier(Notifier):
    def send(self, message): print(f"Email: {message}")

class SMSNotifier(Notifier):
    def send(self, message): print(f"SMS: {message}")

class NotifierFactory(ABC):
    @abstractmethod
    def create_notifier(self) -> Notifier: pass  # <-- Factory Method

    def notify(self, message):
        notifier = self.create_notifier()
        notifier.send(message)

class EmailFactory(NotifierFactory):
    def create_notifier(self): return EmailNotifier()

class SMSFactory(NotifierFactory):
    def create_notifier(self): return SMSNotifier()

# Usage
factory = SMSFactory()
factory.notify("Your OTP is 4821")  # SMS: Your OTP is 4821
```

The `notify()` method in the base class never mentions `EmailNotifier` or `SMSNotifier`. The subclass decides.

**Why it's creational:** The calling code works with the abstract `Notifier` interface, with no knowledge of which concrete type it gets. Adding a `PushNotifier` later requires zero changes to existing code — we just add a new subclass. The pattern delegates the *which* of object creation to child classes.

---

### 3. Abstract Factory

**Provides an interface for creating families of related objects, ensuring they stay compatible.**

```python
from abc import ABC, abstractmethod

class AuthService(ABC):
    @abstractmethod
    def authenticate(self, token: str) -> bool: pass

class StorageService(ABC):
    @abstractmethod
    def save(self, data: dict): pass

# AWS implementations
class AWSAuth(AuthService):
    def authenticate(self, token): return token.startswith("aws-")

class AWSS3Storage(StorageService):
    def save(self, data): print(f"Saving to S3: {data}")

# GCP implementations
class GCPAuth(AuthService):
    def authenticate(self, token): return token.startswith("gcp-")

class GCPStorage(StorageService):
    def save(self, data): print(f"Saving to GCS: {data}")

# Abstract Factory
class CloudFactory(ABC):
    @abstractmethod
    def create_auth(self) -> AuthService: pass
    @abstractmethod
    def create_storage(self) -> StorageService: pass

class AWSFactory(CloudFactory):
    def create_auth(self): return AWSAuth()
    def create_storage(self): return AWSS3Storage()

class GCPFactory(CloudFactory):
    def create_auth(self): return GCPAuth()
    def create_storage(self): return GCPStorage()

# Client
def upload(factory: CloudFactory, token: str, data: dict):
    auth = factory.create_auth()
    storage = factory.create_storage()
    if auth.authenticate(token):
        storage.save(data)

upload(AWSFactory(), "aws-xyz", {"file": "report.pdf"})
# Saving to S3: {'file': 'report.pdf'}
```

We will never accidentally mix GCP storage with AWS auth — each factory only produces its own family.

**Why it's creational:** Abstract Factory coordinates the creation of multiple related objects as a unit. The client is shielded from all concrete classes. If we switch from `AWSFactory` to `GCPFactory`, every dependent object changes with it — consistently and safely.

---

### 4. Builder

**Separates the construction of a complex object into explicit steps, producing it only when done.**

```python
class QueryBuilder:
    def __init__(self):
        self._table = None
        self._conditions = []
        self._limit = None
        self._order_by = None

    def from_table(self, table):
        self._table = table
        return self

    def where(self, condition):
        self._conditions.append(condition)
        return self

    def order_by(self, field):
        self._order_by = field
        return self

    def limit(self, n):
        self._limit = n
        return self

    def build(self):
        query = f"SELECT * FROM {self._table}"
        if self._conditions:
            query += " WHERE " + " AND ".join(self._conditions)
        if self._order_by:
            query += f" ORDER BY {self._order_by}"
        if self._limit:
            query += f" LIMIT {self._limit}"
        return query

# Usage
query = (
    QueryBuilder()
    .from_table("orders")
    .where("status = 'pending'")
    .where("amount > 100")
    .order_by("created_at")
    .limit(50)
    .build()
)
print(query)
# SELECT * FROM orders WHERE status = 'pending' AND amount > 100 ORDER BY created_at LIMIT 50
```

Each step is optional and explicit. We call `build()` only when we are ready.

**Why it's creational:** Builder controls the entire construction lifecycle of a complex object. The object is never partially handed off — it only exists in its final, valid form after `build()`. This is essential when objects have many optional configurations, because a constructor with 8 parameters is unreadable and error-prone.

---

### 5. Prototype

**Creates new objects by cloning an existing instance rather than constructing from scratch.**

```python
import copy

class ReportTemplate:
    def __init__(self, title, sections, styles):
        self.title = title
        self.sections = sections   # list — needs deep copy
        self.styles = styles

    def clone(self):
        return copy.deepcopy(self)

    def __str__(self):
        return f"[{self.title}] Sections: {self.sections}"

# Build the expensive template once
base = ReportTemplate(
    title="Monthly Report",
    sections=["Summary", "Details", "Charts"],
    styles={"font": "Arial", "color": "#333"}
)

# Clone and customize cheaply
q1 = base.clone()
q1.title = "Q1 Report"
q1.sections.append("Q1 Highlights")

q2 = base.clone()
q2.title = "Q2 Report"

print(base)  # [Monthly Report] Sections: ['Summary', 'Details', 'Charts']
print(q1)    # [Q1 Report] Sections: ['Summary', 'Details', 'Charts', 'Q1 Highlights']
print(q2)    # [Q2 Report] Sections: ['Summary', 'Details', 'Charts']
```

The base template is untouched. Each clone is independent.

**Why it's creational:** Prototype replaces traditional construction with duplication. The source of a new object is not a class definition — it is an existing instance. This is a creational decision: we are choosing *from where* an object's initial state originates. It is especially useful when construction is expensive and most new objects differ only slightly from an existing one.

---

## Quick Comparison

| Pattern | Core Question | Best For |
|---|---|---|
| **Singleton** | How many instances should exist? | Shared resources: config, logger, DB pool |
| **Factory Method** | Which subclass should be created? | When subclasses decide the concrete type |
| **Abstract Factory** | How do we create a compatible family of objects? | Multi-provider systems, platform-specific families |
| **Builder** | How do we assemble a complex object step by step? | Objects with many optional parameters |
| **Prototype** | How do we cheaply create objects from an existing one? | Expensive construction, template-based variants |

---

## Final Thoughts

All five patterns share one philosophy: **the code that uses an object should not be burdened with knowing how to create it.**

When we scatter `new ConcreteClass()` calls everywhere, we create tight coupling. Creational patterns cut that coupling and give us flexibility, testability, and cleaner code.

They are not competing options. A real system might use all five — a Singleton for the DB pool, a Factory Method for choosing the right repository, an Abstract Factory for switching cloud providers, a Builder for assembling complex queries, and a Prototype for cloning report templates.

The skill is not memorizing the structures. It is recognizing *which problem each one solves* — and reaching for the right tool at the right moment.
