### Why factory design pattern?
* No tightly coupling client code to any other specific concrete classes
* Which class will be instantiated is generally decided in runtime
* Run time polymorphism
* Instead of directly instantiating objects using the ```new``` keyword, the Factory Pattern delegates the responsibility of object creation to a separate entity—a factory.

### When should we use Factory Design Pattern?
* The exact type of object to be created is determined at runtime.
* We want to encapsulate object creation logic to improve code flexibility.
* We need to create objects that belong to a family of related classes.

### Types
1. Simple Factory (Not really a design pattern. More of a programming paradigm)
2. Factory Method
3. Abstract Factory

   
### Simple Factory
A Simple Factory is not technically a design pattern but a programming idiom. It uses a static method to create and return instances of classes based on input parameters. This method acts as a centralized point for object creation, making it easy to manage but less extensible.

_Why it is not a design pattern?_
A design pattern is defined as:
> A general, reusable solution to a commonly occurring problem within a given context in software design.

Key points here: <br>
✅ It must solve a recurring problem <br>
✅ It must be general enough to be applied in different contexts <br>
✅ It must have named participants/structure (so developers can discuss it) <br>

A Simple Factory is just a single function or class that decides which concrete class to instantiate and return, based on some input (usually parameters). It does not solve the reusability in a true manner. So, it's not a true design pattern, but a programming paradigm.

**Example**
```python
from enum import Enum

# Enum for roles
class UserRole(Enum):
    STUDENT = "student"
    TEACHER = "teacher"
    ADMIN = "admin"

# Base class
class User:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name

class Student(User):
    def __init__(self, first_name, last_name):
        super().__init__(first_name, last_name)

class Teacher(User):
    def __init__(self, first_name, last_name):
        super().__init__(first_name, last_name)

class Admin(User):
    def __init__(self, first_name, last_name):
        super().__init__(first_name, last_name)

# Simple Factory
class UserFactory:
   # Static method is not a hard requirement. Since statice method belongs to class itself, and not to any class object
   # and we don't really need to create a UserFactory object, it's more logical to keep create_user function as static method

    @staticmethod
    def create_user(role: UserRole):
        if role == UserRole.STUDENT:
            return Student("John", "Doe")
        elif role == UserRole.TEACHER:
            return Teacher("John", "Doe")
        elif role == UserRole.ADMIN:
            return Admin("John", "Doe")
        else:
            raise ValueError("Unknown role")

# Example usage
if __name__ == "__main__":
    user = UserFactory.create_user(UserRole.TEACHER)
    print(f"Created user: {user.__class__.__name__} - {user.first_name} {user.last_name}")

```

**How it works?**
1. The client code calls UserFactory.create_user(UserRole.TEACHER) to get a Teacher object without knowing the Teacher class.
2. The factory uses a ```if-else``` statement to decide which class to instantiate.

**Pros:**
1. Simple to implement.
2. Centralizes object creation logic.

**Cons:**
1. Violates the Open-Closed Principle (we must modify the factory to add new types).
2. Not reusable for creating other types of objects.


### Factory Method: Extensible and Reusable
The Factory Method Pattern moves the responsibility of object creation to subclasses. Instead of a static method, it defines an abstract method in a base class (or interface) that concrete subclasses implement to create specific objects. This makes the pattern extensible and reusable.

**Example**
```python
from abc import ABC, abstractmethod

# Abstract Factory
class UserFactory(ABC):
    @abstractmethod
    def create_user(self, first_name, last_name):
        pass

# Concrete Factories
class StudentFactory(UserFactory):
    def create_user(self, first_name, last_name):
        return Student(first_name, last_name)

class TeacherFactory(UserFactory):
    def create_user(self, first_name, last_name):
        return Teacher(first_name, last_name)

# Usage
if __name__ == "__main__":
    factory = StudentFactory()
    user = factory.create_user("John", "Doe")
    print(f"Created user: {user.__class__.__name__} - {user.first_name} {user.last_name}")
```

**Pros:**
1. Follows the Open-Closed Principle (add new factories without modifying existing code).
2. Reusable across different types of objects.
3. Reduces coupling between client code and concrete classes.

**Cons:**
Requires creating new factory classes for each product type, which can increase complexity.

### Abstract Factory

**Difference between factory and abstract factory**
Factory Method is about defining a single method (usually abstract) for creating one kind of object — and letting subclasses decide which exact concrete class to instantiate.

```
UserFactory factory = new StudentFactory();
User user = factory.createUser("John", "Doe");
```

Abstract Factory is about creating families of related objects without specifying their concrete classes.

Instead of a single method, an Abstract Factory has multiple factory methods — one for each type of object in the family.
