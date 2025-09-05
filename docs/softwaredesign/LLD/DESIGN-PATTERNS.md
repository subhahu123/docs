# Design Patterns

* https://github.com/ashishps1/awesome-low-level-design

## Creational

How to create objects.

### Singleton

Method for creating a single instance of a class.

(in python, `__new__` created the object, `__init__` initializes)

```python
# Straightforward implementation of the Singleton Pattern

class Logger(object):
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            print('Creating the object')
            cls._instance = super(Logger, cls).__new__(cls)
            # Put any initialization here.
        return cls._instance

print(Logger())
print(Logger())
```

```
Creating the object
<__main__.Logger object at 0x7d39d3822120>
<__main__.Logger object at 0x7d39d3822120>
```

In java, more intuitive:

```java
class Logger {
    private static Logger instance;
    private Logger() {}
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
}
public class Main {
    public static void main(String[] args) {
        System.out.println(Logger.getInstance());
        System.out.println(Logger.getInstance());
    }
}
```

```
Logger@8efb846
Logger@8efb846
```

### Builder Pattern

With complex objects, instead of complex constructor, "build" the object step-by-step:

```java
public class Car {
    private final String engine;
    private final int wheels;

    private Car(Builder builder) {
        this.engine = builder.engine;
        this.wheels = builder.wheels;
    }

    public static class Builder {
        private String engine = "default";
        private int wheels = 4;

        public Builder setEngine(String engine) {
            this.engine = engine;
            return this;
        }

        public Builder setWheels(int wheels) {
            this.wheels = wheels;
            return this;
        }

        public Car build() {
            return new Car(this);
        }
    }
}

// Usage:
Car car = new Car.Builder()
    .setEngine("V8")
    .setWheels(6)
    .build();
```

In python, just used named parameters (not builder, but conventional).

```python
class Car:
    def __init__(self, engine="default", wheels=4):
        self.engine = engine
        self.wheels = wheels

# Usage:
car = Car(engine="V8", wheels=6)
```

There is a more complex pattern with the director and more boilerplate , but in practice it is not frequently used outside of textbook examples.

### Factory Pattern

Used for hiding the complexities of object creation. Use a simpler interface and the complex part is behind the scenes.

No factory

```python
from abc import ABC, abstractmethod

class Car(ABC):
    @abstractmethod
    def go(self):
        pass

class Truck(Car):
    def __init__(self, engine):
        self.engine = engine
    def go(self):
        print(f"{self.engine} Truck goes slow")

class Sedan(Car):
    def __init__(self, engine):
        self.engine = engine
    def go(self):
        print(f"{self.engine} Sedan goes fast")

cartype = "Truck"
enginetype = "Electric"

car = None
if cartype == "Truck":
    car = Truck(enginetype)
else:
    car = Sedan(enginetype)
```

Factory:

```python
class CarFactory:
    @staticmethod
    def createCar(cartype, enginetype):
        if cartype == "Truck":
            return Truck(enginetype)
        elif cartype == "Sedan":
            return Sedan(enginetype)
        else:
            raise ValueError(f"Unknown car type: {cartype}")

car = CarFactory.createCar(cartype, enginetype)
```

This gives us a cleaner "interface"



## Structural

How objects relate to eachother.

### Facade

"Fancy way to encapsulate"

Hides all the nasty behaviour.

Maybe you have a bunch of complex subsystems that are required for a payment processor. Fraud detection, inventory, shipping calculator etc. Rather than exposing all this logic and having a bunch of if statements, just provide your single facade that is for making an order with the payment processor. It hides all the details required in the background.

[Facade](https://algomaster.io/learn/lld/facade)

### Adapter

Provide a standard interface, and then wrap other implementations such as legacy implementations by implementing the interface and holding a dependency to a concrete type that is the other implementation.

EG, Weather app expects celsius, third party API is in F.

We implement a standard interface and hide the 'adapting'

[Adapter](https://algomaster.io/learn/lld/adapter)

## Behavioral

Behavioral patterns are all about efficient communication and the delegation of responsibilities among objects. They help in defining not just how objects are composed, but also how they communicate within the application's architecture.

### Strategy

And there's multiple ways to do something, you can dynamically change strategies. Define an interface for a strategy, and then implement it for each strategy. This is in contrast with just a bunch of different if statements inside a single class based on the strategy in use without the pattern.

You can dynamically change the strategy just by setting the strategy.

[Strategy](https://algomaster.io/learn/lld/strategy)

### Observer

"Observe" events happening to other objects.

In this way you can subscribe to events that are happening to another object, and be notified when the event occurs.

[Observer](https://algomaster.io/learn/lld/observer)
