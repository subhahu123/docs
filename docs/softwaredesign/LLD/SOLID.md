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


## Interface Segregation Principle (ISP)



## Dependency Inversion Principle (DIP)
