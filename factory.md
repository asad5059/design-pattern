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
