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


