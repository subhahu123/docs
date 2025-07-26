# Composition and Polymorphism

## Composition

Composition means building complex behavior by combining simpler components, rather than inheriting from a base class.

“Has-a” relationship instead of “is-a.”

```python
class Engine:
    def start(self):
        print("Engine started")

class Car:
    def __init__(self):
        self.engine = Engine()  # Composition

    def start(self):
        self.engine.start()
        print("Car is ready to go!")
```

Car "has-a" engine, not "is-a"

Instead of inheriting from Engine, the Car contains it.

This decouples the classes and makes reuse easier.

## Polymorphism

Polymorphism allows you to use different objects interchangeably if they follow the same interface or behavior.

“Many forms” — the same interface can be implemented in multiple ways.

```python
class Shape:
    def draw(self):
        raise NotImplementedError()

class Circle(Shape):
    def draw(self):
        print("Drawing a circle")

class Square(Shape):
    def draw(self):
        print("Drawing a square")

def render(shape: Shape):
    shape.draw()

# Polymorphic use
render(Circle())  # Drawing a circle
render(Square())  # Drawing a square
```

`render()` doesn't care what kind of Shape it's drawing.

As long as the object implements `draw()`, it works.

This enables extensibility without modification - supports Open-Closed Principle.


