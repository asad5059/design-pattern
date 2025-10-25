>The singleton pattern is a design pattern that ensures a class has only one instance and provides a global point of access to that instance. This means that regardless of how many times the class is instantiated, there will only ever be one instance of it throughout the application.
<br>

## Purpose of the Singleton Pattern:

1. Restricting Instantiation <br>
   The singleton pattern restricts the instantiation of a class to ensure that there is only one instance created.

2. Global Access <br>
    It provides a global point of access to the single instance, allowing other parts of the application to easily access and use it.

>So, that means it breaks the Single Responsibility Principle (SRP)

<br>

## Example

```python
class Singleton:
    _instance = None

    @staticmethod
    def get_instance():
        if Singleton._instance is None:
            Singleton._instance = Singleton()
        return Singleton._instance

# Usage
singleton_instance = Singleton.get_instance()
```

The problem with the above implementation is that, it is not thread safe.  In a multithreaded environment, multiple threads could simultaneously check if the instance exists, leading to the creation of multiple instances.

To make the singleton pattern thread-safe, we can use synchronization mechanisms such as locks to ensure that only one thread can create the instance at a time. 

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()

    @staticmethod
    def get_instance():
        if Singleton._instance is None:
            with Singleton._lock:
                if Singleton._instance is None:
                    Singleton._instance = Singleton()
        return Singleton._instance

# Usage
singleton_instance = Singleton.get_instance()
```

## Pros and Cons
|      Pros                                                                                      | Cons                                                                                                |
| ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Guarantees a **single instance** across the application.                                       | Often **violates Single Responsibility Principle (SRP)** by mixing logic with lifecycle management. |
| Provides a **global access point** to shared resources like loggers or configuration managers. | Creates **tight coupling** and **hidden dependencies** between components.                          |
| Enables **lazy initialization** — instance created only when needed.                           | **Not thread-safe** by default; requires locks or special handling.                                 |
| Ensures **consistent state** across the app (e.g., shared configuration).                      | **Hard to test and mock** in unit tests — can’t easily reset or replace the instance.               |
| Simplifies **resource management** (e.g., DB connections, caches).                             | Can act like a **global variable**, making debugging and maintenance harder.                        |


# Fixing the problem with SRP and Dependency Injection

Imagine we have a logger that is used by an app.

```python
class Logger:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(Logger, cls).__new__(cls)
        return cls._instance

    def log(self, msg):
        print(msg)

class UserService:
    def __init__(self):
        self.logger = Logger()  # Directly accessing Singleton

    def create_user(self):
        self.logger.log("User created.")
```

**Problems**
1. Hidden Dependency → UserService depends on Logger, but it’s not clear from outside.
2. Tight Coupling → If we want to use a different logger (e.g., mock for tests), we can’t easily replace it.
3. SRP Violation → Logger handles both logging and its own lifecycle (Singleton logic).
4. Hard to Test → Every test run shares the same logger instance — no isolation.

**What Is Dependency Injection (DI)?**
> Instead of a class creating its own dependencies, they are given (injected) to it from the outside.
> The class doesn’t create what it needs — it receives what it needs.

**Utilizing Singleton and DI together to fix SRP breaking issue**

We let someone else (a container or setup code) create and manage the logger — and then “inject” it into the classes that need it.

```python
class Logger:
    def log(self, msg):
        print(msg)
```

Now instead of embedding Singleton logic here, we manage it externally:

```python
# Singleton manager or DI container
class Container:
    _logger_instance = None

    @staticmethod
    def get_logger():
        if Container._logger_instance is None:
            Container._logger_instance = Logger()
        return Container._logger_instance
```

Then your service just *asks* for what it needs:

```python
class UserService:
    def __init__(self, logger):
        self.logger = logger  # Dependency injected

    def create_user(self):
        self.logger.log("User created.")
```

Usage:

```python
# The DI container handles instance management
logger = Container.get_logger()
user_service = UserService(logger)
user_service.create_user()
```

---

###  What We Achieved

1. **Still Singleton** — Only one `Logger` instance exists (the container ensures that).
2. **SRP Fixed** — `Logger` is only responsible for logging; the container manages the instance.
3. **Loose Coupling** — `UserService` depends only on the *interface* (logger), not on its creation.
4. **Testability** — You can easily inject a fake or mock logger in tests.

---
