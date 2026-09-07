# Object-Oriented Programming (OOP) in Python — Complete Reference Guide

A practical, example-driven guide to OOP in Python: classes, objects, the four pillars (encapsulation, inheritance, polymorphism, abstraction), special methods, and design patterns commonly used in data and application code.

---

## Table of Contents

1. [Why OOP?](#1-why-oop)
2. [Classes and Objects](#2-classes-and-objects)
3. [Attributes and Methods](#3-attributes-and-methods)
4. [The `__init__` Constructor](#4-the-__init__-constructor)
5. [Encapsulation](#5-encapsulation)
6. [Inheritance](#6-inheritance)
7. [Polymorphism](#7-polymorphism)
8. [Abstraction](#8-abstraction)
9. [Special (Dunder) Methods](#9-special-dunder-methods)
10. [Class Methods, Static Methods, Properties](#10-class-methods-static-methods-properties)
11. [Dataclasses](#11-dataclasses)
12. [Composition vs Inheritance](#12-composition-vs-inheritance)
13. [Practical Example: A Small Data Pipeline](#13-practical-example-a-small-data-pipeline)

---

## 1. Why OOP?

Object-Oriented Programming organizes code around **objects** — bundles of data (attributes) and behavior (methods) — instead of loose functions and variables. It helps you:

- Model real-world entities (a `Customer`, a `Dataset`, a `Model`) directly in code.
- Bundle related data and logic together instead of scattering them.
- Reuse code through inheritance instead of copy-pasting.
- Swap implementations easily through polymorphism.
- Hide internal details behind clean interfaces (encapsulation).

The four pillars of OOP: **Encapsulation, Inheritance, Polymorphism, Abstraction.**

---

## 2. Classes and Objects

A **class** is a blueprint. An **object** (or instance) is a concrete thing built from that blueprint.

```python
class Dog:
    pass

my_dog = Dog()            # my_dog is an object (instance) of class Dog
print(type(my_dog))        # <class '__main__.Dog'>
```

```python
dog1 = Dog()
dog2 = Dog()
print(dog1 is dog2)        # False — each instance is a separate object in memory
```

---

## 3. Attributes and Methods

- **Attributes** = data stored on an object (variables).
- **Methods** = functions defined inside a class that operate on that data.

```python
class Dog:
    species = "Canis familiaris"       # class attribute (shared by all instances)

    def __init__(self, name, age):
        self.name = name                  # instance attribute (unique per object)
        self.age = age

    def bark(self):                          # instance method
        return f"{self.name} says Woof!"

d = Dog("Rex", 3)
print(d.name)          # Rex
print(d.species)         # Canis familiaris
print(d.bark())            # Rex says Woof!
```

**Class attributes** are shared across all instances unless overridden; **instance attributes** belong to a specific object.

```python
d2 = Dog("Luna", 2)
Dog.species = "Updated species"     # changes for all instances
print(d.species, d2.species)          # both reflect the change
```

---

## 4. The `__init__` Constructor

`__init__` runs automatically when an object is created. `self` refers to the instance itself and must be the first parameter of every instance method.

```python
class Employee:
    def __init__(self, name, salary, department="General"):
        self.name = name
        self.salary = salary
        self.department = department

    def raise_salary(self, percent):
        self.salary += self.salary * (percent / 100)

e = Employee("Alice", 5000)
e.raise_salary(10)
print(e.salary)          # 5500.0
```

---

## 5. Encapsulation

Encapsulation restricts direct access to an object's internal state, exposing controlled interfaces instead.

### 5.1 Naming Conventions

```python
class Account:
    def __init__(self, balance):
        self.balance = balance           # public — accessible from anywhere
        self._pin = "1234"                  # protected (convention only) — "internal use"
        self.__secret = "hidden"              # private (name-mangled) — hard to access outside

a = Account(1000)
print(a.balance)          # OK
print(a._pin)               # works, but signals "don't touch this externally"
# print(a.__secret)         # AttributeError
print(a._Account__secret)     # 'hidden' — name mangling reveals real attribute name
```

| Prefix | Convention | Meaning |
|--------|-----------|---------|
| `name` | Public | Free to access/modify |
| `_name` | Protected | Internal use, accessible but discouraged externally |
| `__name` | Private | Name-mangled, meant to stay inside the class |

### 5.2 Controlled Access with Properties

```python
class Account:
    def __init__(self, balance):
        self._balance = balance

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value

acc = Account(100)
acc.balance = 200        # calls the setter, runs validation
print(acc.balance)         # calls the getter -> 200
# acc.balance = -50        # raises ValueError
```

Properties let you validate or compute values while keeping the simple `object.attribute` syntax for users of the class.

---

## 6. Inheritance

Inheritance lets a class (child/subclass) reuse and extend the behavior of another class (parent/superclass).

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"{self.name} makes a sound"

class Dog(Animal):                    # Dog inherits from Animal
    def speak(self):                    # method overriding
        return f"{self.name} barks"

class Cat(Animal):
    def speak(self):
        return f"{self.name} meows"

d = Dog("Rex")
c = Cat("Whiskers")
print(d.speak())          # Rex barks
print(c.speak())            # Whiskers meows
```

### 6.1 Using `super()`

`super()` calls the parent class's implementation — useful for extending rather than fully replacing behavior.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)      # reuse parent's constructor
        self.team_size = team_size

    def describe(self):
        return f"{self.name} manages {self.team_size} people"

m = Manager("Alice", 8000, 5)
print(m.describe())          # Alice manages 5 people
print(m.salary)                # 8000 (inherited attribute)
```

### 6.2 Multiple Inheritance

```python
class Flyer:
    def fly(self):
        return "Flying"

class Swimmer:
    def swim(self):
        return "Swimming"

class Duck(Flyer, Swimmer):        # inherits from both
    pass

d = Duck()
print(d.fly(), d.swim())            # Flying Swimming
```

Python resolves conflicts using the **Method Resolution Order (MRO)**:

```python
print(Duck.__mro__)          # shows the lookup order for methods/attributes
```

### 6.3 Checking Relationships

```python
isinstance(d, Duck)         # True
isinstance(d, Flyer)          # True — Duck is-a Flyer
issubclass(Duck, Flyer)         # True
```

---

## 7. Polymorphism

Polymorphism means objects of different classes can be used interchangeably if they share a common interface — "many forms" of the same operation.

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

class Square:
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

shapes = [Circle(3), Square(4)]

for shape in shapes:
    print(shape.area())          # each object responds differently to the same call
```

This works because both classes implement `.area()` — Python doesn't care about the underlying type, only that the method exists (**duck typing**: "if it walks like a duck and quacks like a duck...").

### 7.1 Operator Overloading (a form of polymorphism)

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other):                # overload the + operator
        return Vector(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)          # Vector(4, 6)
```

---

## 8. Abstraction

Abstraction hides implementation details and exposes only the essential interface. Python implements this formally with the `abc` module.

```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount):
        """Every subclass must implement this."""
        pass

class CreditCard(PaymentMethod):
    def pay(self, amount):
        return f"Paid {amount} using Credit Card"

class PayPal(PaymentMethod):
    def pay(self, amount):
        return f"Paid {amount} using PayPal"

# p = PaymentMethod()          # TypeError: can't instantiate an abstract class
cc = CreditCard()
print(cc.pay(100))               # Paid 100 using Credit Card
```

Abstract base classes (ABCs) enforce that subclasses implement required methods — useful for defining consistent interfaces across a codebase (e.g., every data loader must implement `.load()`).

---

## 9. Special (Dunder) Methods

"Dunder" (double underscore) methods let your objects integrate with Python's built-in syntax (`print()`, `+`, `len()`, `==`, iteration, etc.).

```python
class Book:
    def __init__(self, title, pages):
        self.title = title
        self.pages = pages

    def __str__(self):                    # readable string, used by print()
        return f"'{self.title}' ({self.pages} pages)"

    def __repr__(self):                     # unambiguous representation, used in debugging/repr()
        return f"Book(title={self.title!r}, pages={self.pages})"

    def __len__(self):                        # enables len(book)
        return self.pages

    def __eq__(self, other):                    # enables ==
        return self.title == other.title

    def __lt__(self, other):                      # enables <, used by sorted()
        return self.pages < other.pages

b1 = Book("Python 101", 250)
b2 = Book("Advanced Python", 400)

print(b1)                    # 'Python 101' (250 pages)
print(len(b1))                 # 250
print(b1 == b2)                  # False
print(sorted([b2, b1]))            # sorted by pages using __lt__
```

### 9.1 Common Dunder Methods

| Method | Triggered by |
|--------|--------------|
| `__init__` | Object creation |
| `__str__` | `print(obj)`, `str(obj)` |
| `__repr__` | `repr(obj)`, debugger/console display |
| `__len__` | `len(obj)` |
| `__eq__`, `__lt__`, `__gt__` | `==`, `<`, `>` |
| `__add__`, `__sub__` | `+`, `-` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = value` |
| `__iter__`, `__next__` | `for x in obj` |
| `__call__` | `obj()` — makes an instance callable like a function |
| `__enter__`, `__exit__` | `with obj as x:` context manager |

### 9.2 Making an Object Iterable

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

for num in Countdown(3):
    print(num)          # 3, 2, 1
```

---

## 10. Class Methods, Static Methods, Properties

```python
class Circle:
    pi = 3.14159

    def __init__(self, radius):
        self.radius = radius

    def area(self):                           # instance method — needs an instance
        return self.pi * self.radius ** 2

    @classmethod
    def from_diameter(cls, diameter):            # classmethod — receives the class, not an instance
        return cls(diameter / 2)                    # alternate constructor

    @staticmethod
    def is_valid_radius(radius):                       # staticmethod — no access to self or cls
        return radius > 0

c1 = Circle(5)
c2 = Circle.from_diameter(10)         # alternate way to construct an object
print(Circle.is_valid_radius(-3))       # False — utility function grouped with the class
```

| Decorator | First parameter | Use case |
|-----------|-----------------|----------|
| (none) | `self` | Operates on a specific instance |
| `@classmethod` | `cls` | Operates on the class itself; alternate constructors |
| `@staticmethod` | none | Utility function logically related to the class |

---

## 11. Dataclasses

For classes that mostly hold data, `@dataclass` (Python 3.7+) auto-generates `__init__`, `__repr__`, and `__eq__`, cutting boilerplate.

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0
    tags: list = field(default_factory=list)

    def total_value(self):
        return self.price * self.quantity

p = Product("Laptop", 1200.0, 3)
print(p)                    # Product(name='Laptop', price=1200.0, quantity=3, tags=[])
print(p.total_value())        # 3600.0

p2 = Product("Laptop", 1200.0, 3)
print(p == p2)                  # True — dataclasses auto-generate __eq__
```

Use `@dataclass(frozen=True)` for immutable objects (raises an error if you try to modify an attribute after creation).

---

## 12. Composition vs Inheritance

Inheritance models **"is-a"** relationships; composition models **"has-a"** relationships. Composition is often more flexible and easier to maintain.

```python
# Inheritance ("is-a"): a Car IS-A Vehicle
class Vehicle:
    def move(self):
        return "Moving"

class Car(Vehicle):
    pass

# Composition ("has-a"): a Car HAS-A Engine
class Engine:
    def start(self):
        return "Engine starting"

class Car:
    def __init__(self):
        self.engine = Engine()          # Car "has an" Engine

    def start(self):
        return self.engine.start()

car = Car()
print(car.start())          # Engine starting
```

**Rule of thumb:** prefer composition when the relationship isn't a strict specialization — it avoids fragile, deep inheritance hierarchies and lets you swap components independently.

---

## 13. Practical Example: A Small Data Pipeline

Bringing the concepts together — abstraction (interface), inheritance (shared behavior), encapsulation (validation), and polymorphism (interchangeable loaders):

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field


class DataLoader(ABC):
    """Abstract interface every loader must follow."""

    @abstractmethod
    def load(self) -> list[dict]:
        pass


class CSVLoader(DataLoader):
    def __init__(self, filepath):
        self.filepath = filepath

    def load(self) -> list[dict]:
        import csv
        with open(self.filepath, "r") as f:
            return list(csv.DictReader(f))


class JSONLoader(DataLoader):
    def __init__(self, filepath):
        self.filepath = filepath

    def load(self) -> list[dict]:
        import json
        with open(self.filepath, "r") as f:
            return json.load(f)


@dataclass
class Dataset:
    records: list = field(default_factory=list)

    def __len__(self):
        return len(self.records)

    def __getitem__(self, index):
        return self.records[index]

    @classmethod
    def from_loader(cls, loader: DataLoader):
        return cls(records=loader.load())


# Usage — polymorphism in action: any DataLoader subclass works interchangeably
# dataset = Dataset.from_loader(CSVLoader("data.csv"))
# dataset = Dataset.from_loader(JSONLoader("data.json"))
# print(len(dataset), dataset[0])
```

This pattern — an abstract base class defining a contract, concrete subclasses implementing it, and a data container wrapping the results — is common in real-world data engineering code (loaders, transformers, exporters, model wrappers).

---

## Quick-Reference Cheat Sheet

```python
# Define a class
class MyClass:
    class_attr = "shared"

    def __init__(self, x):
        self.x = x                      # instance attribute

    def method(self):                     # instance method
        return self.x

    @classmethod
    def alt_constructor(cls, x):            # classmethod
        return cls(x)

    @staticmethod
    def helper():                             # staticmethod
        return "utility"

    @property
    def doubled(self):                           # computed attribute
        return self.x * 2

# Inheritance
class Child(MyClass):
    def __init__(self, x, y):
        super().__init__(x)
        self.y = y

# Abstract base class
from abc import ABC, abstractmethod
class Base(ABC):
    @abstractmethod
    def run(self): pass

# Dataclass
from dataclasses import dataclass
@dataclass
class Point:
    x: int
    y: int

# Dunder methods
class Wrapper:
    def __str__(self): return "readable"
    def __repr__(self): return "Wrapper()"
    def __eq__(self, other): return True
    def __len__(self): return 0
```

---

*This guide covers the essentials of Object-Oriented Programming in Python. Solid OOP fundamentals make it far easier to build maintainable data pipelines, model wrappers, and reusable libraries.*