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

1. Take a DarkRoast object
2. Decorate it with a Mocha object
3. Decorate it with a Whip object
4. Call the cost() method and rely on delegation to add on the condiment costs
