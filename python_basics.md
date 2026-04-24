# 🐍 Python Basics Interview Cheatsheet
> Core Language Fundamentals — Interview Ready

---

## TABLE OF CONTENTS
1. [Data Types & Variables](#1-data-types--variables)
2. [Strings](#2-strings)
3. [Lists](#3-lists)
4. [Tuples](#4-tuples)
5. [Dictionaries](#5-dictionaries)
6. [Sets](#6-sets)
7. [Control Flow](#7-control-flow)
8. [Functions](#8-functions)
9. [Comprehensions](#9-comprehensions)
10. [OOP](#10-oop)
11. [Iterators & Generators](#11-iterators--generators)
12. [Decorators](#12-decorators)
13. [Error Handling](#13-error-handling)
14. [Modules & Packages](#14-modules--packages)
15. [File I/O](#15-file-io)
16. [Functional Tools](#16-functional-tools)
17. [Memory & Scope](#17-memory--scope)
18. [Concurrency Basics](#18-concurrency-basics)
19. [Common Built-ins](#19-common-built-ins)
20. [Interview Gotchas & Q&A](#20-interview-gotchas--qa)

---

## 1. Data Types & Variables

### Type System
```python
# Immutable types
int, float, complex, bool, str, bytes, tuple, frozenset

# Mutable types
list, dict, set, bytearray

# Type checking
type(x)                    # exact type
isinstance(x, int)         # True if int or subclass
isinstance(x, (int, float))  # check multiple types at once
```

### Numbers
```python
a = 10 // 3      # 3       floor division
b = 10 % 3       # 1       modulo
c = 2 ** 10      # 1024    exponentiation
d = abs(-5)      # 5
e = round(3.567, 2)  # 3.57

# Integer has unlimited precision in Python 3
big = 10 ** 100   # no overflow!

# Float precision gotcha
0.1 + 0.2 == 0.3  # False! Use math.isclose()
import math
math.isclose(0.1 + 0.2, 0.3)   # True

# int / float conversion
int(3.9)    # 3  (truncates, does NOT round)
float(5)    # 5.0
int("42")   # 42
int("ff", 16)  # 255 — base conversion
```

### Booleans
```python
bool(0)        # False
bool("")       # False
bool([])       # False
bool(None)     # False
bool(0.0)      # False
bool({})       # False

bool(1)        # True
bool("a")      # True
bool([0])      # True  ← list with one element

# bool is a subclass of int
True + True    # 2
True * 5       # 5
```

### None
```python
x = None
x is None      # correct way to check
x == None      # works but not recommended (PEP 8)
```

### Type Conversion
```python
str(42)        # "42"
int("42")      # 42
float("3.14")  # 3.14
list("abc")    # ['a','b','c']
tuple([1,2])   # (1, 2)
set([1,1,2])   # {1, 2}
list({1,2,3})  # [1, 2, 3] — order not guaranteed
```

---

## 2. Strings

### Creation & Basics
```python
s1 = 'single'
s2 = "double"
s3 = """multi
line"""
raw = r"C:\Users\name"    # raw string — backslashes literal
b   = b"bytes"            # bytes literal

# Strings are immutable — every operation returns a NEW string
s = "hello"
s[0] = "H"   # TypeError!
```

### Indexing & Slicing
```python
s = "Python"
s[0]      # 'P'
s[-1]     # 'n'
s[1:4]    # 'yth'
s[::-1]   # 'nohtyP'   reverse
s[::2]    # 'Pto'      every 2nd character
```

### Common Methods
```python
s = "  Hello World  "
s.strip()             # "Hello World"      removes whitespace
s.lstrip()            # "Hello World  "
s.rstrip()            # "  Hello World"
s.lower()             # "  hello world  "
s.upper()             # "  HELLO WORLD  "
s.title()             # "  Hello World  "
s.replace("l","L")    # "  HeLLo WorLd  "
s.split()             # ['Hello', 'World']  splits on whitespace
"a,b,c".split(",")    # ['a','b','c']
",".join(["a","b"])   # "a,b"
s.find("World")       # 8   (-1 if not found)
s.index("World")      # 8   (raises ValueError if not found)
s.count("l")          # 3
s.startswith("  H")   # True
s.endswith("  ")      # True
s.isdigit()           # False
s.isalpha()           # False
"123".isdigit()       # True
s.center(20, "*")     # "**  Hello World  **"
```

### String Formatting
```python
name, score = "Alice", 98.5

# f-string (preferred, 3.6+)
f"Name: {name}, Score: {score:.1f}"

# format()
"Name: {}, Score: {:.1f}".format(name, score)
"Name: {n}, Score: {s}".format(n=name, s=score)

# % (legacy)
"Name: %s, Score: %.1f" % (name, score)

# f-string extras
f"{score!r}"           # repr
f"{1000000:,}"         # "1,000,000"
f"{'left':<10}"        # "left      "  left-align
f"{'right':>10}"       # "     right"  right-align
f"{42:05d}"            # "00042"       zero-pad
f"{255:#x}"            # "0xff"        hex with prefix
```

### Useful String Patterns
```python
# Check substring
"error" in log_line

# Split and unpack
first, *rest = "a,b,c,d".split(",")   # first='a', rest=['b','c','d']

# Multi-line string as template
template = (
    "Line one\n"
    "Line two\n"
    "Line three"
)

# Escape characters
"\n"   # newline
"\t"   # tab
"\\"   # backslash
"\'"   # single quote
```

---

## 3. Lists

### Creation
```python
a = [1, 2, 3]
b = list(range(5))       # [0, 1, 2, 3, 4]
c = [0] * 5              # [0, 0, 0, 0, 0]
d = [[0]*3 for _ in range(3)]   # 3x3 matrix — CORRECT way
e = [[0]*3] * 3          # WRONG — shares inner lists!
```

### Core Operations
```python
a = [3, 1, 4, 1, 5]
a.append(9)              # [3,1,4,1,5,9]    O(1)
a.insert(0, 0)           # [0,3,1,4,1,5,9]  O(n)
a.extend([7, 8])         # appends multiple  O(k)
a.pop()                  # removes last, returns it  O(1)
a.pop(0)                 # removes index 0            O(n)
a.remove(1)              # removes FIRST occurrence of value
a.index(4)               # returns index of value
a.count(1)               # frequency
a.reverse()              # in-place
a.sort()                 # in-place sort
a.sort(key=lambda x: -x) # sort descending
sorted(a)                # returns NEW sorted list
a.copy()                 # shallow copy
a.clear()                # empties the list
len(a)
```

### Slicing
```python
a = [0, 1, 2, 3, 4, 5]
a[2:5]       # [2, 3, 4]
a[:3]        # [0, 1, 2]
a[3:]        # [3, 4, 5]
a[::2]       # [0, 2, 4]
a[::-1]      # [5, 4, 3, 2, 1, 0]
a[1:5:2]     # [1, 3]

# Slice assignment
a[1:3] = [10, 20]    # replace elements
a[::2] = [0,0,0]     # replace every 2nd
```

### Common Patterns
```python
# Flatten nested list
nested = [[1,2],[3,4],[5]]
flat = [x for sub in nested for x in sub]   # [1,2,3,4,5]

# Remove duplicates while preserving order
seen = set()
unique = [x for x in items if not (x in seen or seen.add(x))]

# Zip two lists
pairs = list(zip([1,2,3], ["a","b","c"]))   # [(1,'a'),(2,'b'),(3,'c')]
keys, values = zip(*pairs)                   # unzip

# Enumerate
for i, val in enumerate(items, start=1):
    print(f"{i}: {val}")
```

---

## 4. Tuples

```python
t = (1, 2, 3)
t = 1, 2, 3        # parentheses optional
single = (1,)      # single element — comma required!
single = 1,        # also valid
empty  = ()

# Immutable — no append/remove/sort
t[0]               # indexing OK
t[1:3]             # slicing OK
len(t)             # 3
t.count(1)         # frequency
t.index(2)         # index of value

# Unpacking
a, b, c = (1, 2, 3)
first, *rest = (1, 2, 3, 4)   # first=1, rest=[2,3,4]
a, b = b, a                    # swap without temp variable

# Named tuple
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
p.x, p.y         # 3, 4
p._asdict()      # OrderedDict

# When to use tuple vs list?
# Tuple: heterogeneous data, fixed structure, dict keys, return multiple values
# List:  homogeneous data, dynamic collection
```

---

## 5. Dictionaries

### Creation
```python
d = {"a": 1, "b": 2}
d = dict(a=1, b=2)
d = dict([("a",1),("b",2)])   # from list of pairs
d = dict.fromkeys(["x","y"], 0)  # {"x":0, "y":0}
```

### Core Operations
```python
d["key"]              # KeyError if missing
d.get("key")          # None if missing
d.get("key", 0)       # default if missing

d["key"] = val        # set/update
d.update({"c":3})     # merge in another dict
d.update(c=3)         # keyword form
{**d1, **d2}          # merge (3.5+), d2 wins on conflict

del d["key"]          # raises KeyError if missing
d.pop("key")          # removes and returns, raises if missing
d.pop("key", None)    # safe pop with default

"key" in d            # membership test (checks keys)
len(d)

d.keys()              # dict_keys view
d.values()            # dict_values view
d.items()             # dict_items view  → use in for loops
d.clear()

# Iteration
for k in d: ...
for k, v in d.items(): ...
```

### defaultdict & Counter
```python
from collections import defaultdict, Counter

freq = defaultdict(int)
for word in words: freq[word] += 1

freq = Counter(words)
freq.most_common(3)      # [(word, count), ...]
freq["x"]                # 0 if not present (no KeyError)
```

### Nested Dict Patterns
```python
# Safe nested access
user.get("address", {}).get("city", "Unknown")

# setdefault — insert if key missing
d.setdefault("list_key", []).append(item)

# Merge (3.9+)
merged = d1 | d2          # new dict
d1 |= d2                  # in-place
```

---

## 6. Sets

```python
s = {1, 2, 3}
s = set([1, 1, 2, 3])    # {1, 2, 3}
empty = set()             # NOT {} — that's an empty dict!

s.add(4)
s.remove(4)               # KeyError if missing
s.discard(4)              # safe remove
s.pop()                   # removes arbitrary element

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
a | b                     # union         {1,2,3,4,5,6}
a & b                     # intersection  {3,4}
a - b                     # difference    {1,2}
a ^ b                     # symmetric diff {1,2,5,6}

a.issubset(b)             # a <= b
a.issuperset(b)           # a >= b
a.isdisjoint(b)           # no common elements

# frozenset — immutable, hashable (can be dict key or set element)
fs = frozenset([1, 2, 3])

# Common use: fast membership test  O(1) vs list O(n)
valid_statuses = {"active", "pending", "suspended"}
if user["status"] in valid_statuses: ...

# De-duplicate preserving order
unique = list(dict.fromkeys(items))    # Python 3.7+
```

---

## 7. Control Flow

### Conditionals
```python
# Ternary
x = "yes" if condition else "no"

# Chained comparisons
0 < x < 10    # Pythonic — avoids 0 < x and x < 10

# match-case (3.10+)
match command:
    case "quit":
        quit()
    case "go" | "move":
        move()
    case {"action": action, "target": target}:
        do(action, target)
    case _:
        print("Unknown")
```

### Loops
```python
# for / else  — else runs if loop completed without break
for item in items:
    if condition(item):
        break
else:
    print("Not found")

# while / else
while queue:
    process(queue.pop())
else:
    print("Queue empty")

# Loop controls
break       # exit loop immediately
continue    # skip to next iteration
pass        # no-op placeholder

# range
range(5)         # 0..4
range(1, 6)      # 1..5
range(0, 10, 2)  # 0,2,4,6,8
range(5, 0, -1)  # 5,4,3,2,1
```

---

## 8. Functions

### Signature Variants
```python
def f(pos1, pos2, /, normal, *, kw_only):
    """
    /  → everything before is positional-only (3.8+)
    *  → everything after is keyword-only
    """

def func(a, b=10, *args, **kwargs):
    pass

# Call examples
func(1)                  # a=1, b=10
func(1, 2, 3, 4, x=5)   # args=(3,4), kwargs={'x':5}
```

### Argument Passing
```python
# Python passes object references — "pass by assignment"
# Immutable: rebinding inside function has no effect outside
# Mutable: mutation IS visible outside (same object)

def bad_append(lst=[]):    # mutable default — BUG
    lst.append(1)
    return lst

def good_append(lst=None):  # correct
    lst = lst if lst is not None else []
    lst.append(1)
    return lst
```

### Lambda
```python
square = lambda x: x**2
add    = lambda x, y: x + y

# Common uses
sorted(people, key=lambda p: p["age"])
list(map(lambda x: x*2, nums))
list(filter(lambda x: x > 0, nums))
```

### Closures
```python
def make_counter(start=0):
    count = start
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

c = make_counter()
c()  # 1
c()  # 2

# nonlocal — modifies enclosing (non-global) variable
# global   — modifies module-level variable
```

### Type Hints (3.5+)
```python
from typing import Optional, Union, List, Dict, Tuple, Any

def greet(name: str, times: int = 1) -> str:
    return name * times

def process(data: List[Dict[str, Any]]) -> Optional[str]:
    ...

# Python 3.10+ — use | for union
def f(x: int | str | None) -> list[int]: ...
```

---

## 9. Comprehensions

```python
# List comprehension
squares   = [x**2 for x in range(10)]
evens     = [x for x in range(20) if x % 2 == 0]
flat      = [x for row in matrix for x in row]      # nested

# Dict comprehension
inverted  = {v: k for k, v in d.items()}
filtered  = {k: v for k, v in d.items() if v > 0}

# Set comprehension
unique_lengths = {len(word) for word in words}

# Generator expression — lazy, no brackets
total = sum(x**2 for x in range(1000))   # no list built in memory
any_fail = any(r == "FAIL" for r in results)
all_pass = all(r == "PASS" for r in results)

# Walrus in comprehension (3.8+)
results = [y for x in data if (y := process(x)) is not None]
```

---

## 10. OOP

### Class Anatomy
```python
class Animal:
    species = "Unknown"     # class variable — shared by all instances

    def __init__(self, name: str, age: int):
        self.name = name    # instance variable
        self.age  = age

    def speak(self) -> str:
        raise NotImplementedError

    def __str__(self) -> str:
        return f"{self.name} ({self.age})"

    def __repr__(self) -> str:
        return f"Animal(name={self.name!r}, age={self.age!r})"

    def __eq__(self, other) -> bool:
        if not isinstance(other, Animal): return NotImplemented
        return self.name == other.name and self.age == other.age

    def __hash__(self):
        return hash((self.name, self.age))

    def __lt__(self, other):
        return self.age < other.age
```

### Class / Static / Property
```python
class Circle:
    PI = 3.14159

    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, val):
        if val < 0: raise ValueError("Radius must be >= 0")
        self._radius = val

    @property
    def area(self):
        return self.PI * self._radius ** 2

    @classmethod
    def unit(cls):                   # factory method
        return cls(radius=1)

    @staticmethod
    def is_valid_radius(r):          # utility — no self/cls
        return r >= 0
```

### Inheritance & MRO
```python
class Dog(Animal):
    def speak(self):
        return "Woof"

    def __init__(self, name, age, breed):
        super().__init__(name, age)   # always call super().__init__
        self.breed = breed

# Multiple inheritance — MRO (C3 linearization)
class A: ...
class B(A): ...
class C(A): ...
class D(B, C): ...   # MRO: D → B → C → A

D.__mro__            # inspect MRO
```

### Abstract Classes
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self):               # concrete method — inherited
        return f"Area={self.area():.2f}"

class Rectangle(Shape):
    def __init__(self, w, h): self.w, self.h = w, h
    def area(self):      return self.w * self.h
    def perimeter(self): return 2 * (self.w + self.h)
```

### Dunder Methods Reference
| Method | Triggered by |
|---|---|
| `__init__` | `obj = Class()` |
| `__str__` | `str(obj)`, `print(obj)` |
| `__repr__` | `repr(obj)`, debugger |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = val` |
| `__contains__` | `x in obj` |
| `__iter__` | `for x in obj` |
| `__next__` | `next(obj)` |
| `__eq__` | `obj == other` |
| `__lt__` | `obj < other` |
| `__add__` | `obj + other` |
| `__enter__` / `__exit__` | `with obj:` |
| `__call__` | `obj()` |

---

## 11. Iterators & Generators

### Iterator Protocol
```python
# An iterator must implement __iter__ and __next__
class CountDown:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return self

    def __next__(self):
        if self.n <= 0:
            raise StopIteration
        self.n -= 1
        return self.n + 1

for x in CountDown(3): print(x)   # 3, 2, 1
```

### Generators
```python
# Generator function — uses yield
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

gen = fibonacci()
[next(gen) for _ in range(8)]   # [0,1,1,2,3,5,8,13]

# Generator expression (lazy)
squares = (x**2 for x in range(100))
next(squares)   # 0
next(squares)   # 1

# yield from — delegate to sub-generator
def chain(*iterables):
    for it in iterables:
        yield from it

list(chain([1,2], [3,4]))   # [1,2,3,4]

# send() — coroutine-style
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None: break
        total += value

acc = accumulator()
next(acc)         # prime the generator (advances to first yield)
acc.send(10)      # 10
acc.send(5)       # 15
```

### itertools
```python
import itertools as it

it.count(1)               # 1, 2, 3, ... (infinite)
it.cycle([1,2,3])         # 1,2,3,1,2,3,... (infinite)
it.repeat(x, n)           # x repeated n times
it.chain([1,2],[3,4])     # [1,2,3,4]
it.islice(gen, 5)         # first 5 items from generator
it.product("AB","12")     # cartesian product
it.permutations("abc",2)
it.combinations("abc",2)
it.groupby(sorted_data, key=lambda x: x["type"])
it.accumulate([1,2,3,4])  # [1,3,6,10]
it.takewhile(lambda x: x<5, count())
it.dropwhile(lambda x: x<5, count())
```

---

## 12. Decorators

### Basic Decorator
```python
import functools

def decorator(func):
    @functools.wraps(func)   # preserves __name__, __doc__
    def wrapper(*args, **kwargs):
        print(f"Before {func.__name__}")
        result = func(*args, **kwargs)
        print(f"After {func.__name__}")
        return result
    return wrapper

@decorator
def hello(): print("Hello!")
# equivalent to: hello = decorator(hello)
```

### Decorator with Arguments
```python
def repeat(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name): print(f"Hello {name}")
# greet("Alice") prints 3 times
```

### Class-based Decorator
```python
class Memoize:
    def __init__(self, func):
        self.func  = func
        self.cache = {}
        functools.update_wrapper(self, func)

    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]

@Memoize
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
```

### Context Managers as Decorators
```python
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)

with managed_resource() as r:
    use(r)
```

---

## 13. Error Handling

### Exception Hierarchy
```python
BaseException
 ├── SystemExit
 ├── KeyboardInterrupt
 └── Exception
      ├── ValueError
      ├── TypeError
      ├── KeyError
      ├── IndexError
      ├── AttributeError
      ├── NameError
      ├── FileNotFoundError  (OSError subclass)
      ├── ZeroDivisionError
      ├── StopIteration
      └── RuntimeError
```

### try / except / else / finally
```python
try:
    result = risky_operation()
except (ValueError, TypeError) as e:   # catch multiple
    print(f"Bad input: {e}")
except Exception as e:
    raise RuntimeError("Unexpected") from e   # chain exceptions
else:
    # runs ONLY if no exception was raised
    process(result)
finally:
    # ALWAYS runs — cleanup goes here
    cleanup()
```

### Custom Exceptions
```python
class AppError(Exception):
    """Base class for application errors."""

class ValidationError(AppError):
    def __init__(self, field, message):
        self.field = field
        super().__init__(f"{field}: {message}")

class NotFoundError(AppError):
    pass

# Usage
try:
    raise ValidationError("email", "invalid format")
except ValidationError as e:
    print(e.field)    # "email"
```

### Exception Chaining
```python
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("Config parse failed") from e
# Traceback will show both exceptions

# Suppress context (use None)
raise RuntimeError("Clean error") from None
```

---

## 14. Modules & Packages

### Import Styles
```python
import os
import os.path
from os import path, getcwd
from os.path import join, exists
import numpy as np          # alias
from . import module        # relative import
from ..utils import helper  # relative up one level
```

### `__name__` Guard
```python
def main():
    print("Running as script")

if __name__ == "__main__":
    main()
# Won't run when imported as a module
```

### `__all__` — Public API
```python
# mymodule.py
__all__ = ["PublicClass", "public_function"]  # controls `from module import *`
```

### Useful Standard Library Modules
| Module | Use |
|---|---|
| `os` | OS interaction, env vars, paths |
| `sys` | Interpreter, argv, exit |
| `pathlib` | Modern path handling |
| `json` | JSON encode/decode |
| `re` | Regular expressions |
| `datetime` | Dates and times |
| `collections` | defaultdict, Counter, deque, namedtuple |
| `itertools` | Lazy iterators |
| `functools` | wraps, lru_cache, partial, reduce |
| `contextlib` | contextmanager, suppress |
| `copy` | shallow/deep copy |
| `math` | Math functions |
| `random` | Random numbers |
| `hashlib` | Hashing (md5, sha256) |
| `threading` | Threads |
| `subprocess` | Shell commands |
| `logging` | Logging |
| `unittest` | Testing |
| `dataclasses` | @dataclass |
| `typing` | Type hints |

---

## 15. File I/O

### Reading & Writing
```python
# Always use context manager (with) — auto-closes file
with open("file.txt") as f:
    content = f.read()       # entire file as string
    lines   = f.readlines()  # list of lines with \n
    for line in f: ...       # memory-efficient line-by-line

with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

with open("out.txt", "w") as f:   # 'w' overwrites
    f.write("line one\n")
    f.writelines(["a\n","b\n"])

with open("log.txt", "a") as f:   # 'a' appends
    f.write("new entry\n")

# Modes: r, w, a, x (create new, fail if exists), b (binary), + (read+write)
```

### JSON & CSV
```python
import json, csv

# JSON
with open("data.json") as f:
    data = json.load(f)             # file → Python object
obj = json.loads('{"key": 1}')      # string → Python object

with open("data.json","w") as f:
    json.dump(data, f, indent=2)    # Python object → file
s = json.dumps(data, indent=2)      # Python object → string

# CSV
with open("data.csv", newline="") as f:
    reader = csv.DictReader(f)
    rows = list(reader)              # list of dicts

with open("out.csv","w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name","score"])
    writer.writeheader()
    writer.writerows([{"name":"Alice","score":95}])
```

### pathlib
```python
from pathlib import Path

p = Path("data/reports")
p.mkdir(parents=True, exist_ok=True)

p = Path("file.txt")
p.exists()
p.is_file()
p.is_dir()
p.suffix        # ".txt"
p.stem          # "file"
p.name          # "file.txt"
p.parent        # Path(".")
p.read_text()   # shortcut for open+read
p.write_text("content")

# Glob
list(Path(".").glob("**/*.py"))     # all .py files recursively
list(Path(".").glob("*.json"))
```

---

## 16. Functional Tools

### map / filter / reduce
```python
from functools import reduce

nums = [1, 2, 3, 4, 5]
doubled  = list(map(lambda x: x*2, nums))         # [2,4,6,8,10]
evens    = list(filter(lambda x: x%2==0, nums))   # [2,4]
total    = reduce(lambda acc, x: acc+x, nums, 0)  # 15
```

### functools
```python
from functools import lru_cache, partial, wraps, cached_property

# Memoization
@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

fib.cache_info()    # hits, misses, maxsize, currsize
fib.cache_clear()

# partial — pre-fill arguments
double = partial(lambda x, factor: x * factor, factor=2)
double(5)   # 10

# cached_property (3.8+) — computed once, then cached as attribute
class Circle:
    def __init__(self, r): self.r = r

    @cached_property
    def area(self):
        return 3.14 * self.r ** 2
```

---

## 17. Memory & Scope

### LEGB Rule
```python
# Python variable lookup order:
# Local → Enclosing → Global → Built-in

x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)   # "local"
    inner()
    print(x)       # "enclosing"

outer()
print(x)           # "global"

# Modify outer scope
def counter():
    count = 0
    def inc():
        nonlocal count   # without this, count += 1 raises UnboundLocalError
        count += 1
        return count
    return inc
```

### Mutability & Copying
```python
import copy

original = [[1, 2], [3, 4]]

ref      = original           # same object
shallow  = original.copy()    # or list(original) or original[:]
deep     = copy.deepcopy(original)

original[0].append(99)
ref[0]      # [1, 2, 99]    ← same object
shallow[0]  # [1, 2, 99]    ← inner list is SHARED
deep[0]     # [1, 2]        ← fully independent
```

### Memory — id, is, interning
```python
a = [1, 2]
b = [1, 2]
id(a) == id(b)   # False — different objects

# Small int caching: -5 to 256
x = 256; y = 256
x is y   # True  (cached)
x = 257; y = 257
x is y   # False (not cached — implementation detail, don't rely on this)

# String interning
a = "hello"
b = "hello"
a is b   # True (interned at compile time for literals)
```

### Dataclasses (3.7+)
```python
from dataclasses import dataclass, field

@dataclass(order=True, frozen=True)
class Point:
    x: float
    y: float
    label: str = ""
    tags: list = field(default_factory=list)   # mutable default!

p = Point(1.0, 2.0)
p.x          # 1.0
repr(p)      # "Point(x=1.0, y=2.0, label='', tags=[])"
```

---

## 18. Concurrency Basics

### Threading
```python
import threading

def task(name):
    print(f"Task {name} running")

t = threading.Thread(target=task, args=("A",), daemon=True)
t.start()
t.join()    # wait for completion

# Thread-safe counter
lock = threading.Lock()
count = 0

def increment():
    global count
    with lock:
        count += 1
```

### ThreadPoolExecutor
```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch(url):
    return requests.get(url).status_code

urls = ["http://example.com/1", "http://example.com/2"]
with ThreadPoolExecutor(max_workers=5) as ex:
    futures = {ex.submit(fetch, url): url for url in urls}
    for future in as_completed(futures):
        print(futures[future], future.result())
```

### GIL (Global Interpreter Lock)
```python
# GIL allows only ONE thread to execute Python bytecode at a time
# Threading is good for: I/O-bound tasks (network, disk)
# Threading is NOT good for: CPU-bound tasks (use multiprocessing instead)

from multiprocessing import Pool

def square(n): return n**2

with Pool(4) as p:
    results = p.map(square, range(100))
```

### asyncio Basics
```python
import asyncio

async def fetch_data(id):
    await asyncio.sleep(1)    # simulates async I/O
    return {"id": id, "data": "..."}

async def main():
    # Concurrent — both run in ~1s, not ~2s
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
    )
    return results

asyncio.run(main())
```

---

## 19. Common Built-ins

```python
# Math
abs(-5)         # 5
round(3.5)      # 4  (banker's rounding)
round(2.5)      # 2  ← Python rounds to even!
pow(2, 10)      # 1024
pow(2, 10, 100) # 24  (2^10 % 100, very fast for large numbers)
divmod(7, 3)    # (2, 1)  quotient and remainder
sum([1,2,3])    # 6
max(1,2,3)
min([4,5,6])
max(items, key=lambda x: x["score"])

# Sequences
len([1,2,3])    # 3
sorted([3,1,2]) # [1,2,3]
reversed([1,2,3])          # iterator
list(reversed([1,2,3]))    # [3,2,1]
enumerate(["a","b"])       # (0,'a'),(1,'b')
zip([1,2],[3,4])           # (1,3),(2,4)
zip(*[[1,2],[3,4]])        # unzip: (1,3),(2,4)

# Introspection
dir(obj)            # list attributes/methods
vars(obj)           # __dict__
hasattr(obj,"attr")
getattr(obj,"attr","default")
setattr(obj,"attr",val)
callable(obj)       # True if obj() works
id(obj)             # memory address
hash(obj)           # hash (only for immutable)

# Functional
map(func, iterable)
filter(func, iterable)
all(iterable)       # True if all truthy
any(iterable)       # True if any truthy

# I/O
input("prompt")     # reads from stdin (always returns str)
print(*args, sep=" ", end="\n", file=sys.stdout)

# Type construction
list(), dict(), set(), tuple(), int(), str(), float(), bool()
object()
```

---

## 20. Interview Gotchas & Q&A

### Classic Gotchas
```python
# 1. Mutable default argument
def append(item, lst=[]):    # BUG: lst persists between calls
    lst.append(item)
    return lst
append(1)   # [1]
append(2)   # [1, 2]  ← unexpected!

# Fix:
def append(item, lst=None):
    lst = [] if lst is None else lst
    lst.append(item)
    return lst

# 2. Late binding in closures
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2]  ← all capture the same i

# Fix: bind at definition time
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]   # [0, 1, 2]

# 3. is vs ==
a = [1,2]; b = [1,2]
a == b   # True   (value)
a is b   # False  (identity)

# 4. Integer division
5 / 2    # 2.5    (float division in Python 3)
5 // 2   # 2      (floor division)
-7 // 2  # -4     (floors toward negative infinity!)
-7 % 2   # 1      (not -1)

# 5. Chained assignment
a = b = []   # a and b point to SAME list
a.append(1)
b            # [1] ← same object!

# 6. Copy vs deepcopy
import copy
a = [[1,2],[3,4]]
b = a.copy()        # shallow
a[0].append(99)
b[0]                # [1, 2, 99] ← inner list shared!

# 7. String immutability performance
# BAD O(n²):
s = ""
for c in chars: s += c

# GOOD O(n):
s = "".join(chars)

# 8. Exception variable scope
try:
    raise ValueError("x")
except ValueError as e:
    pass
# e is deleted after the except block! (Python 3)
print(e)   # NameError
```

### Common Interview Q&A

**Q: What is the difference between `__str__` and `__repr__`?**
> `__str__` is for human-readable output (`print`). `__repr__` is for unambiguous developer representation (REPL, debugging). If only `__repr__` is defined, `str()` falls back to it. The rule of thumb: `repr` should ideally be code you could `eval()` to recreate the object.

**Q: What is a generator and when would you use one?**
> A generator is a function that uses `yield` to return values lazily, one at a time, without loading all values into memory. Use when: processing large files line-by-line, infinite sequences, pipelines where you don't need all results at once.

**Q: Explain the GIL.**
> The Global Interpreter Lock is a mutex in CPython that allows only one thread to execute Python bytecode at a time. This makes multi-threading safe for I/O-bound work but ineffective for CPU-bound parallelism. Use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` for CPU-bound tasks.

**Q: What's the difference between a shallow and deep copy?**
> A shallow copy creates a new container but shares references to the same inner objects. A deep copy recursively duplicates everything. Use `copy.copy()` vs `copy.deepcopy()`.

**Q: What does `@staticmethod` vs `@classmethod` do?**
> `@staticmethod` defines a method that doesn't receive the class or instance — it's essentially a plain function namespaced in the class. `@classmethod` receives the class as its first argument (`cls`), enabling factory methods and subclass-aware behavior.

**Q: What is a decorator?**
> A decorator is a callable that takes a function, wraps it in another function that can add behavior before/after, and returns the wrapper. Always use `@functools.wraps(func)` to preserve the original function's metadata.

**Q: How does Python's memory management work?**
> Python uses reference counting for most objects — when an object's ref count drops to zero it's freed. A cyclic garbage collector handles reference cycles. `sys.getrefcount(obj)` shows the current count.

**Q: What is `*args` and `**kwargs`?**
> `*args` collects extra positional arguments as a tuple. `**kwargs` collects extra keyword arguments as a dict. Used to write functions that accept variable numbers of arguments.

**Q: Difference between `list` and `tuple`?**
> Lists are mutable, tuples are immutable. Tuples are faster, hashable (if their contents are), and signal intent — use tuples for fixed structures (coordinates, records) and lists for dynamic collections.

**Q: What are Python's falsy values?**
> `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`. Any empty container or zero-valued number is falsy.

**Q: How do you handle circular imports?**
> Restructure code to extract shared code into a third module, use lazy imports (import inside functions), or use `TYPE_CHECKING` guard for type hints.

```python
from __future__ import annotations
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from module import HeavyClass
```

### Big-O Quick Reference
| Operation | list | dict | set |
|---|---|---|---|
| Access by index | O(1) | — | — |
| Search | O(n) | O(1) avg | O(1) avg |
| Insert at end | O(1) amortized | O(1) avg | O(1) avg |
| Insert at start | O(n) | — | — |
| Delete | O(n) | O(1) avg | O(1) avg |
| Membership test | O(n) | O(1) avg | O(1) avg |

---

*Last updated: April 2026 | Pure Python — no external libraries required*
