# Design Patterns

## Creational

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

## Builder Pattern

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

There is a more complex pattern with the director and more boilerplate wedding practice it is not frequently used outside of textbook examples.

## Factory Pattern

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
