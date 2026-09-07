# Python for Data — Complete Reference Guide

A practical, example-driven guide to the core Python concepts every data practitioner needs: variables, data structures, strings, control flow, functions, error handling, file I/O, and modules.

---

## Table of Contents

1. [Variables and Data Types](#1-variables-and-data-types)
2. [Lists, Dictionaries, Sets, Tuples](#2-lists-dictionaries-sets-tuples)
3. [String Operations](#3-string-operations)
4. [Loops](#4-loops)
5. [Conditional Logic (if / elif / else)](#5-conditional-logic-if--elif--else)
6. [Functions (Reusability and Clarity)](#6-functions-reusability-and-clarity)
7. [Error Handling](#7-error-handling)
8. [File Handling](#8-file-handling)
9. [Modules](#9-modules)

---

## 1. Variables and Data Types

Variables are named references to values stored in memory. Python is **dynamically typed** — you don't declare a type, it's inferred at assignment — and **strongly typed** — types don't silently coerce in surprising ways.

### 1.1 Declaring Variables

```python
name = "Alice"          # str
age = 29                # int
height = 1.72            # float
is_active = True         # bool
salary = None             # NoneType
```

### 1.2 Core Data Types

| Type | Example | Description |
|------|---------|-------------|
| `int` | `42`, `-7` | Whole numbers, arbitrary precision |
| `float` | `3.14`, `-0.001` | Decimal / floating-point numbers |
| `complex` | `2 + 3j` | Complex numbers |
| `str` | `"data"` | Immutable text sequences |
| `bool` | `True`, `False` | Boolean values (subclass of int) |
| `NoneType` | `None` | Represents "no value" |

### 1.3 Checking and Converting Types

```python
x = "123"
print(type(x))          # <class 'str'>

y = int(x)               # convert str -> int
z = float(y)              # convert int -> float
s = str(z)                 # convert float -> str
b = bool(0)                  # False (0, "", None, [], {} are falsy)
```

### 1.4 Naming Rules & Conventions

- Must start with a letter or underscore, not a digit.
- Case-sensitive: `Data` and `data` are different variables.
- Use `snake_case` for variables and functions (PEP 8).
- Avoid reserved keywords (`class`, `for`, `if`, etc.).

```python
total_sales = 1000      # good
TotalSales = 1000        # avoid (reads like a class name)
2nd_value = 5              # invalid — starts with a digit
```

### 1.5 Multiple Assignment

```python
a, b, c = 1, 2, 3
x = y = z = 0            # all three point to 0
a, b = b, a                # swap values without a temp variable
```

### 1.6 Type Hints (Optional but Useful for Data Code)

```python
def average(values: list[float]) -> float:
    return sum(values) / len(values)

count: int = 0
ratio: float = 0.5
```

Type hints don't enforce types at runtime, but they improve readability and let tools like `mypy` catch bugs early — valuable when data pipelines grow complex.

---

## 2. Lists, Dictionaries, Sets, Tuples

Python's built-in collections are the backbone of everyday data manipulation, long before you reach for pandas or NumPy.

### 2.1 Lists — Ordered, Mutable

```python
fruits = ["apple", "banana", "cherry"]

fruits.append("date")             # add to end
fruits.insert(1, "avocado")        # insert at index
fruits.remove("banana")             # remove by value
last = fruits.pop()                   # remove & return last item
fruits[0] = "kiwi"                     # update by index

print(fruits[0])              # first element
print(fruits[-1])              # last element
print(fruits[1:3])              # slicing

squares = [x**2 for x in range(10)]     # list comprehension
evens = [x for x in range(20) if x % 2 == 0]
```

**Common operations:**

```python
len(fruits)                # length
sorted(fruits)               # new sorted list
fruits.sort()                  # sort in place
fruits.reverse()                # reverse in place
"kiwi" in fruits                  # membership test
```

### 2.2 Tuples — Ordered, Immutable

Tuples behave like lists but cannot be modified after creation. Use them for fixed collections (e.g., coordinates, database rows, function return values).

```python
point = (10, 20)
person = ("Alice", 29, "Engineer")

name, age, role = person       # unpacking
single = (5,)                    # trailing comma required for 1-item tuple

# Tuples are hashable (if contents are), so they can be dict keys or set members
locations = {(0, 0): "origin", (1, 1): "diagonal"}
```

### 2.3 Dictionaries — Key-Value Pairs

```python
student = {
    "name": "Alice",
    "age": 29,
    "grades": [88, 92, 79]
}

student["age"] = 30                  # update value
student["email"] = "a@x.com"         # add new key
del student["email"]                   # remove key

# Safe access
age = student.get("age", 0)               # default if key missing

for key, value in student.items():
    print(key, value)

# Dict comprehension
squares = {x: x**2 for x in range(5)}
```

**Useful methods:** `.keys()`, `.values()`, `.items()`, `.update()`, `.pop(key)`, `.setdefault(key, default)`

### 2.4 Sets — Unordered, Unique Elements

Sets automatically remove duplicates and support fast membership checks and mathematical set operations.

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a.add(10)
a.discard(1)

print(a | b)      # union
print(a & b)      # intersection
print(a - b)      # difference
print(a ^ b)      # symmetric difference

unique_values = set([1, 2, 2, 3, 3, 3])   # deduplicate a list
```

### 2.5 Choosing the Right Structure

| Need | Use |
|------|-----|
| Ordered, allow duplicates, mutable | `list` |
| Ordered, allow duplicates, immutable | `tuple` |
| Fast lookup by unique key | `dict` |
| Unique items, fast membership test | `set` |

---

## 3. String Operations

Strings are immutable sequences of characters — every "modification" produces a new string.

### 3.1 Basics

```python
s = "Data Science"

s.lower()                # "data science"
s.upper()                 # "DATA SCIENCE"
s.strip()                  # remove leading/trailing whitespace
s.replace("Data", "AI")     # "AI Science"
s.split(" ")                  # ["Data", "Science"]
"-".join(["a", "b", "c"])      # "a-b-c"
len(s)                            # 12
s[0:4]                              # slicing: "Data"
s[::-1]                              # reverse: "ecneicS ataD"
```

### 3.2 Formatting

```python
name, score = "Alice", 92.5

# f-strings (preferred, Python 3.6+)
print(f"{name} scored {score:.1f}%")

# .format()
print("{} scored {:.1f}%".format(name, score))

# % formatting (legacy)
print("%s scored %.1f%%" % (name, score))
```

### 3.3 Searching & Checking

```python
"Science" in s              # True
s.startswith("Data")          # True
s.endswith("Science")           # True
s.find("Sci")                     # index of substring, -1 if not found
s.count("a")                        # count occurrences
```

### 3.4 Validation Helpers

```python
"123".isdigit()          # True
"abc".isalpha()           # True
"abc123".isalnum()          # True
"   ".isspace()               # True
```

### 3.5 Multi-line & Raw Strings

```python
paragraph = """This spans
multiple lines."""

path = r"C:\Users\data\file.csv"    # raw string, ignores escape chars
```

---

## 4. Loops

Loops let you iterate over data — rows, files, records — without repeating code.

### 4.1 `for` Loops

```python
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

for i in range(5):              # 0 to 4
    print(i)

for i, fruit in enumerate(["apple", "banana"]):
    print(i, fruit)              # index + value

for key, value in {"a": 1, "b": 2}.items():
    print(key, value)

for a, b in zip([1, 2, 3], ["x", "y", "z"]):
    print(a, b)                    # pair elements from two lists
```

### 4.2 `while` Loops

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

### 4.3 Loop Control

```python
for i in range(10):
    if i == 3:
        continue        # skip this iteration
    if i == 7:
        break             # exit loop entirely
    print(i)
else:
    print("Loop completed without break")   # runs only if no break occurred
```

### 4.4 Comprehensions (Compact Loops)

```python
squares = [x**2 for x in range(10)]
evens_only = [x for x in range(20) if x % 2 == 0]
flat = [x for row in [[1,2],[3,4]] for x in row]        # flatten nested list
word_lengths = {w: len(w) for w in ["data", "science"]}
```

Comprehensions are often faster and more readable than manual loops for simple transformations — but avoid deeply nested comprehensions that hurt readability.

---

## 5. Conditional Logic (if / elif / else)

### 5.1 Basic Syntax

```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

print(grade)
```

### 5.2 Comparison & Logical Operators

```python
==   !=   >   <   >=   <=
and  or   not
```

```python
age = 25
has_license = True

if age >= 18 and has_license:
    print("Can drive")

if not has_license:
    print("Cannot drive")
```

### 5.3 Truthy / Falsy Values

Falsy: `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `None`, `False`
Everything else is truthy.

```python
data = []
if not data:
    print("No data available")
```

### 5.4 Ternary (Conditional) Expression

```python
status = "adult" if age >= 18 else "minor"
```

### 5.5 `match` Statement (Python 3.10+)

```python
def describe(value):
    match value:
        case 0:
            return "zero"
        case int() | float():
            return "number"
        case str():
            return "text"
        case _:
            return "unknown"
```

---

## 6. Functions (Reusability and Clarity)

Functions package logic into reusable, testable units — essential for clean data pipelines.

### 6.1 Defining Functions

```python
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
```

### 6.2 Default & Keyword Arguments

```python
def describe(name, age=30, city="Unknown"):
    return f"{name}, {age}, from {city}"

describe("Alice")                          # uses defaults
describe("Bob", city="Jakarta")               # keyword argument
describe(name="Cara", age=25, city="Bali")      # all keyword
```

### 6.3 `*args` and `**kwargs`

```python
def total(*numbers):                 # variable positional args
    return sum(numbers)

total(1, 2, 3, 4)                     # 10

def build_profile(**info):            # variable keyword args
    return info

build_profile(name="Alice", age=29)   # {'name': 'Alice', 'age': 29}
```

### 6.4 Return Values

```python
def stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

low, high, avg = stats([4, 8, 15, 16, 23, 42])
```

### 6.5 Lambda (Anonymous) Functions

```python
square = lambda x: x ** 2
add = lambda x, y: x + y

data = [5, 3, 8, 1]
sorted_data = sorted(data, key=lambda x: -x)     # sort descending
```

### 6.6 Docstrings & Type Hints

```python
def normalize(values: list[float]) -> list[float]:
    """
    Scale a list of numbers to the 0-1 range.

    Args:
        values: list of numeric values.

    Returns:
        A new list scaled between 0 and 1.
    """
    lo, hi = min(values), max(values)
    return [(v - lo) / (hi - lo) for v in values]
```

### 6.7 Scope

```python
x = 10                   # global scope

def modify():
    x = 20                  # local scope, doesn't affect global x
    return x

def modify_global():
    global x
    x = 20                   # explicitly modifies global x
```

---

## 7. Error Handling

Robust data code anticipates failure: missing files, malformed rows, division by zero, network errors.

### 7.1 try / except

```python
try:
    value = int("abc")
except ValueError:
    print("Conversion failed — not a valid number")
```

### 7.2 Multiple Exceptions

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except (TypeError, ValueError) as e:
    print(f"Invalid input: {e}")
```

### 7.3 else and finally

```python
try:
    data = [1, 2, 3]
    value = data[5]
except IndexError:
    print("Index out of range")
else:
    print("No errors occurred")            # runs only if try succeeds
finally:
    print("This always runs")                 # cleanup code (closing files, etc.)
```

### 7.4 Raising Custom Errors

```python
def load_ratio(numerator, denominator):
    if denominator == 0:
        raise ValueError("Denominator cannot be zero")
    return numerator / denominator

try:
    load_ratio(5, 0)
except ValueError as e:
    print(f"Error: {e}")
```

### 7.5 Custom Exception Classes

```python
class DataValidationError(Exception):
    """Raised when incoming data fails validation."""
    pass

def validate_age(age):
    if age < 0:
        raise DataValidationError(f"Invalid age: {age}")
    return age
```

### 7.6 Common Built-in Exceptions

| Exception | When it happens |
|-----------|-----------------|
| `ValueError` | Right type, invalid value (e.g. `int("abc")`) |
| `TypeError` | Wrong type used for an operation |
| `KeyError` | Missing dictionary key |
| `IndexError` | List index out of range |
| `FileNotFoundError` | File doesn't exist |
| `ZeroDivisionError` | Division by zero |
| `AttributeError` | Method/attribute doesn't exist on object |

---

## 8. File Handling

Reading and writing files is fundamental to any data workflow — logs, CSVs, JSON configs, reports.

### 8.1 Opening Files (context manager — always preferred)

```python
with open("data.txt", "r") as f:
    content = f.read()             # read entire file
```

Using `with` ensures the file is automatically closed, even if an error occurs.

### 8.2 Read Modes

```python
with open("data.txt", "r") as f:
    line = f.readline()          # read one line
    lines = f.readlines()          # read all lines as a list

with open("data.txt", "r") as f:
    for line in f:                     # memory-efficient line-by-line iteration
        print(line.strip())
```

### 8.3 Writing Files

```python
with open("output.txt", "w") as f:      # 'w' overwrites existing content
    f.write("First line\n")
    f.write("Second line\n")

with open("output.txt", "a") as f:        # 'a' appends without overwriting
    f.write("Appended line\n")
```

### 8.4 File Modes Reference

| Mode | Meaning |
|------|---------|
| `"r"` | Read (default, errors if file doesn't exist) |
| `"w"` | Write (creates or overwrites) |
| `"a"` | Append (creates if missing) |
| `"x"` | Create (errors if file already exists) |
| `"rb"` / `"wb"` | Binary read/write (images, pickled objects) |

### 8.5 Working with CSV Files

```python
import csv

# Reading
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])

# Writing
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 29})
```

### 8.6 Working with JSON Files

```python
import json

# Reading
with open("config.json", "r") as f:
    config = json.load(f)

# Writing
data = {"name": "Alice", "active": True}
with open("output.json", "w") as f:
    json.dump(data, f, indent=2)

# String <-> object conversion
json_string = json.dumps(data)
parsed = json.loads(json_string)
```

### 8.7 Checking File Existence

```python
import os

if os.path.exists("data.txt"):
    print("File found")

os.makedirs("output_folder", exist_ok=True)     # create directory safely
```

---

## 9. Modules

Modules let you organize code into reusable files and leverage Python's massive ecosystem.

### 9.1 Importing Modules

```python
import math
print(math.sqrt(16))            # 4.0

import math as m                  # alias
print(m.pi)

from math import sqrt, pi           # import specific names
print(sqrt(25))

from math import *                    # import everything (avoid — pollutes namespace)
```

### 9.2 Standard Library Highlights for Data Work

```python
import math          # mathematical functions
import random          # random number generation, sampling
import statistics        # mean, median, stdev
import datetime            # dates and times
import os                    # file/directory operations
import sys                     # interpreter-level operations
import re                        # regular expressions
import itertools                   # advanced iteration tools
import collections                   # Counter, defaultdict, OrderedDict
```

```python
import statistics
data = [4, 8, 15, 16, 23, 42]
statistics.mean(data)          # average
statistics.median(data)          # median
statistics.stdev(data)             # standard deviation

from collections import Counter
Counter(["a", "b", "a", "c", "b", "a"])    # {'a': 3, 'b': 2, 'c': 1}
```

### 9.3 Third-Party Data Libraries (installed via pip)

```python
# pip install pandas numpy matplotlib scikit-learn requests

import pandas as pd            # dataframes, tabular data manipulation
import numpy as np               # numerical arrays, linear algebra
import matplotlib.pyplot as plt    # data visualization
from sklearn.model_selection import train_test_split   # ML utilities
import requests                       # HTTP requests / API calls
```

### 9.4 Writing Your Own Module

**`utils.py`**
```python
def clean_text(text):
    return text.strip().lower()

def average(numbers):
    return sum(numbers) / len(numbers)
```

**`main.py`**
```python
import utils

print(utils.clean_text("  Hello World  "))
print(utils.average([1, 2, 3, 4]))
```

### 9.5 Packages

A package is a directory containing an `__init__.py` file (may be empty) plus one or more modules, allowing imports like:

```python
from mypackage.submodule import my_function
```

### 9.6 Managing Dependencies

```bash
pip install pandas                  # install a package
pip freeze > requirements.txt         # export current environment
pip install -r requirements.txt         # install from a requirements file
```

---

## Quick-Reference Cheat Sheet

```python
# Variables
x = 10; y = "text"; z = 3.14; flag = True

# Collections
lst = [1, 2, 3]
tup = (1, 2, 3)
dct = {"a": 1}
st = {1, 2, 3}

# Strings
f"{x} items"
"a,b,c".split(",")

# Loop
for i in range(5): ...
while x > 0: ...

# Conditional
if x > 0: ...
elif x == 0: ...
else: ...

# Function
def f(a, b=1, *args, **kwargs): return a + b

# Error handling
try: ...
except Exception as e: ...
finally: ...

# Files
with open("f.txt") as f: data = f.read()

# Modules
import os
from math import sqrt
```

---

*This guide covers Python fundamentals essential for data work. For deeper dives, explore pandas for tabular data, NumPy for numerical computing, and matplotlib/seaborn for visualization.*