### Problems with inheritance
* Class explosion
* Inheritance hierarchy
* Static polymorphism
* No runtime polymorphism

<img width="1211" height="752" alt="image" src="https://github.com/user-attachments/assets/d9dd725c-a381-4d6c-b59b-7996b2a7652f" />

<img width="1098" height="707" alt="image" src="https://github.com/user-attachments/assets/07f8cd69-1954-462c-9d85-29abe4a3f363" />

### Composition
Instead of creating a big class by extending another class (inheritance), you create a class that holds references to other objects and delegates work to them.

```python
class Engine:
    def start(self):
        print("Engine started!")

class Battery:
    def charge(self):
        print("Charging battery...")

class Car:
    def __init__(self, engine=None, battery=None):
        self.engine = engine
        self.battery = battery

    def start(self):
        if self.engine:
            self.engine.start()

    def charge(self):
        if self.battery:
            self.battery.charge()

# Usage
my_car = Car(engine=Engine(), battery=Battery())
my_car.start()
my_car.charge()

```

### Open Close Principle
It's a principle where we say Software entities (classes, modules, functions) should be open for extension but closed for modification. Composition is one way to achieve the Open-Closed Principle.

| **Open-Closed Principle (OCP)**                                | **Composition**                                           |
| -------------------------------------------------------------- | --------------------------------------------------------- |
| A **design principle** (a rule to follow)                      | A **technique** (a way to build classes)                  |
| Goal: Make code extensible without modifying it                | Goal: Reuse and combine behavior without deep inheritance |
| Can be implemented via inheritance, interfaces, or composition | Just one possible tool to help implement OCP              |


### Decorator Pattern
The Decorator pattern is a structural design pattern that allows us to enhance or modify the behavior of objects at runtime. It achieves this by creating a set of decorator classes that are used to wrap concrete components. Each decorator adds a specific feature or behavior to the component, and we can stack multiple decorators to create various combinations.

**When to Use the Decorator Pattern**
1. Adding New Features
2. Avoiding Messy Code (E.g.: Class Explotion)
3. Make program follow OCD

**Example**
1. Take a DarkRoast object
2. Decorate it with a Mocha object
3. Decorate it with a Whip object
4. Call the cost() method and rely on delegation to add on the condiment costs

**Code**
```python
from abc import ABC, abstractmethod

# --- Component (Base Class) ---
class Beverage(ABC):
    @abstractmethod
    def get_description(self):
        pass

    @abstractmethod
    def cost(self):
        pass


# --- Concrete Components ---
class Espresso(Beverage):
    def get_description(self):
        return "Espresso"

    def cost(self):
        return 1.99


class DarkRoast(Beverage):
    def get_description(self):
        return "Dark Roast Coffee"

    def cost(self):
        return 0.99


# --- Decorator (Abstract) ---
class CondimentDecorator(Beverage, ABC):
    def __init__(self, beverage):
        self.beverage = beverage


# --- Concrete Decorators ---
class Mocha(CondimentDecorator):
    def get_description(self):
        return self.beverage.get_description() + ", Mocha"

    def cost(self):
        return self.beverage.cost() + 0.20


class Soy(CondimentDecorator):
    def get_description(self):
        return self.beverage.get_description() + ", Soy"

    def cost(self):
        return self.beverage.cost() + 0.30


class Whip(CondimentDecorator):
    def get_description(self):
        return self.beverage.get_description() + ", Whip"

    def cost(self):
        return self.beverage.cost() + 0.10


# --- Client Code ---
if __name__ == "__main__":
    # Order 1: Just Espresso
    beverage = Espresso()
    print(f"{beverage.get_description()} ${beverage.cost():.2f}")

    # Order 2: DarkRoast + Mocha + Mocha + Whip
    beverage2 = DarkRoast()
    beverage2 = Mocha(beverage2)
    beverage2 = Mocha(beverage2)
    beverage2 = Whip(beverage2)
    print(f"{beverage2.get_description()} ${beverage2.cost():.2f}")

    # Order 3: Espresso + Soy + Mocha + Whip
    beverage3 = Espresso()
    beverage3 = Soy(beverage3)
    beverage3 = Mocha(beverage3)
    beverage3 = Whip(beverage3)
    print(f"{beverage3.get_description()} ${beverage3.cost():.2f}")

```

#### Output
```
Espresso $1.99
Dark Roast Coffee, Mocha, Mocha, Whip $1.49
Espresso, Soy, Mocha, Whip $2.59
```
