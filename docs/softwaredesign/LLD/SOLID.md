# SOLID

## Single Responsibility Principle (SRP)

A class or module should have one, and only one, reason to change.

When you write a software module, you want to make sure that when changes are requested, those changes can only originate from a single person, or rather, a single tightly coupled group of people representing a single narrowly defined business function.

## Open-closed Principle (OCP)

**Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.**

This means that existing code should not be modified to support new behavior. Instead, the code should be designed in a way that allows its behavior to be extended without altering its source.

In practice, this often involves defining abstractions (such as interfaces or abstract base classes) that other components can implement or extend. This allows new functionality to be introduced by adding new code, rather than changing existing, tested code.

For example, if you have a `Shape` interface with a `draw()` method, adding a new shape (e.g., `Triangle`) should involve **creating a new class**, not modifying existing ones like `Circle` or `Rectangle`.

> While inheritance can be used to achieve this, it can lead to tight coupling. Prefer **composition and interfaces** to maintain flexibility and decoupling.

## Liskov Substitution Principle (LSP)

The liskov substitution principle states that you should be able to replace a parent class with its child without interfering with program behavior.

For example say we define the following:

```python
class Car:
    def __init__(self):
        self.speed = 0

    def speedup(self):
        self.speed = self.speed + 1

    def slowdown(self):
        self.speed = self.speed - 1

class SportsCar(Car):
    def speedup(self):
        self.speed = self.speed + 5

    def slowdown(self):
        self.speed = self.speed - 5

class Truck(Car):
    def speedup(self):
        self.speed = self.speed + 2

    def slowdown(self):
        pass

    def stop(self):
        self.speed = 0


def main():
    cars = [Truck(), Car(), SportsCar()]

    for c in cars:
        c.speedup()

    for c in cars:
        if isinstance(c, Truck):
            c.stop()
            print("Stopped truck")
        else:
            while c.speed != 0:
                c.slowdown()
            print("Stopped car or sportscar")
```

This is problematic and we have a violation because our truck instance cannot replace a car instance. We have unexpected behavior where if we try to replace a car with a truck it would never be able to stop.

## Interface Segregation Principle (ISP)

The idea of interface segregation is to break up interfaces so that there are not many useless methods for a user.

For example, say an interface is defined to represent a PetOwner.

```java
public interface PetOwner {
    void walkDog();
    void emptyCatLitter();
    void cleanFishbowl();
}
```

If we want to implement this for a `DogOwner`, at that point we have two completely irrelevant methods that will just be left empty. Rather, this interface should have been broken up into three interfaces, one for each type of pet owner:

```java
public interface DogOwner {
    void walkDog();
}

// ...
```

## Dependency Inversion Principle (DIP)

In simple terms, the Dependency Inversion Principle states that high-level modules should not depend on low-level modules. Both should depend on abstractions. This means that instead of a high-level module directly using a specific implementation (a "child" or low-level module), it should rely on an interface or abstract class that the low-level module implements. This allows for decoupling, easier testing, and more flexible code.

For example, tight coupling caused by direct dependency:

```python
class FileLogger:
    def log(self, message: str):
        print(f"[File] {message}")

class Application:
    def __init__(self):
        self.logger = FileLogger()  # Direct dependency on a low-level module

    def run(self):
        self.logger.log("Application started")
```

And then following the dependency inversion principle:

```python
from abc import ABC, abstractmethod

# Abstraction
class Logger(ABC):
    @abstractmethod
    def log(self, message: str):
        pass

# Low-level implementation 1
class FileLogger(Logger):
    def log(self, message: str):
        print(f"[File] {message}")

# Low-level implementation 2
class ConsoleLogger(Logger):
    def log(self, message: str):
        print(f"[Console] {message}")

# High-level module depends on the abstraction
class Application:
    def __init__(self, logger: Logger):
        self.logger = logger

    def run(self):
        self.logger.log("Application started")
```
