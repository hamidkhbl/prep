# 🐍 Python Functions, Classes & OOP — Complete Review
> From First Principles to Advanced Patterns

---

## TABLE OF CONTENTS

1. [Functions — Fundamentals](#1-functions--fundamentals)
2. [Function Arguments — All Forms](#2-function-arguments--all-forms)
3. [Scope & Closures](#3-scope--closures)
4. [Lambdas & Higher-Order Functions](#4-lambdas--higher-order-functions)
5. [Decorators](#5-decorators)
6. [Classes — Fundamentals](#6-classes--fundamentals)
7. [Instance vs Class vs Static](#7-instance-vs-class-vs-static)
8. [Properties & Encapsulation](#8-properties--encapsulation)
9. [Dunder (Magic) Methods](#9-dunder-magic-methods)
10. [Inheritance](#10-inheritance)
11. [Multiple Inheritance & MRO](#11-multiple-inheritance--mro)
12. [Abstract Classes & Interfaces](#12-abstract-classes--interfaces)
13. [Dataclasses](#13-dataclasses)
14. [OOP Design Principles (SOLID)](#14-oop-design-principles-solid)
15. [Common OOP Design Patterns](#15-common-oop-design-patterns)
16. [Interview Q&A](#16-interview-qa)

---

## 1. Functions — Fundamentals

### Defining and Calling

```python
# Basic definition
def greet(name: str) -> str:
    """Docstring: describes what the function does."""
    return f"Hello, {name}!"

greet("Alice")          # "Hello, Alice!"

# Functions are first-class objects — assign, pass, return
say_hi = greet
say_hi("Bob")           # "Hello, Bob!"

# Check metadata
greet.__name__          # "greet"
greet.__doc__           # "Docstring: ..."
greet.__annotations__   # {'name': <class 'str'>, 'return': <class 'str'>}
```

### Return Values

```python
def nothing():
    pass                    # implicitly returns None

def multi():
    return 1, "a", True     # returns a tuple

x, y, z = multi()           # unpack on the caller side

def early_exit(n):
    if n < 0:
        return              # returns None, exits immediately
    return n * 2
```

### Nested Functions

```python
def outer(x):
    def inner(y):           # inner is local to outer
        return x + y        # inner accesses x from enclosing scope
    return inner            # return the function object

add5 = outer(5)
add5(3)                     # 8
```

---

## 2. Function Arguments — All Forms

### Overview of All Argument Types

```python
def full_demo(
    pos_only,               # positional-only (before /)
    /,
    normal,                 # positional or keyword
    with_default = 10,      # has a default
    *args,                  # extra positional → tuple
    kw_only,                # keyword-only (after *)
    kw_default = "hi",      # keyword-only with default
    **kwargs                # extra keyword → dict
):
    print(pos_only, normal, with_default, args, kw_only, kw_default, kwargs)

full_demo(1, 2, 3, 4, 5, kw_only="required", extra=99)
# 1  2  3  (4, 5)  "required"  "hi"  {'extra': 99}
```

### Positional & Keyword Arguments

```python
def connect(host, port, timeout=30):
    ...

connect("localhost", 5432)              # positional
connect("localhost", port=5432)         # mixed
connect(timeout=60, host="db", port=3306)  # all keyword (any order)
```

### *args — Variable Positional

```python
def total(*args):
    return sum(args)

total(1, 2, 3)       # 6
total(*[1, 2, 3])    # 6  — unpack list into positional args

# args is a tuple inside the function
def show(*args):
    print(type(args), args)   # <class 'tuple'> (1, 2, 3)
```

### **kwargs — Variable Keyword

```python
def config(**kwargs):
    for key, val in kwargs.items():
        print(f"{key} = {val}")

config(debug=True, port=8080)

# kwargs is a dict inside the function
def merge_dicts(**kwargs):
    return dict(kwargs)

# Unpack dict into keyword arguments
settings = {"host": "localhost", "port": 5432}
connect(**settings)
```

### Mutable Default Argument — Classic Bug

```python
# ❌ WRONG — list is created ONCE at function definition
def append_to(item, lst=[]):
    lst.append(item)
    return lst

append_to(1)    # [1]
append_to(2)    # [1, 2]  ← unexpected! Same list reused

# ✅ CORRECT — use None as sentinel
def append_to(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

append_to(1)    # [1]
append_to(2)    # [2]  ← fresh list each time
```

### Argument Unpacking at Call Site

```python
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
add(*nums)              # unpack list → positional args = 6

opts = {"b": 2, "c": 3}
add(1, **opts)          # unpack dict → keyword args = 6

# Combine both
add(*[1], **{"b": 2, "c": 3})  # 6
```

### Forcing Keyword-Only Arguments

```python
def create_user(name, *, admin=False, active=True):
    # * in signature: everything after must be keyword
    ...

create_user("Alice")                    # OK
create_user("Alice", admin=True)        # OK
create_user("Alice", True)              # TypeError!
```

---

## 3. Scope & Closures

### LEGB Rule

```python
x = "global"               # G — Global

def outer():
    x = "enclosing"        # E — Enclosing

    def inner():
        x = "local"        # L — Local
        print(x)           # "local"

    inner()
    print(x)               # "enclosing"

outer()
print(x)                   # "global"

# Built-in (B): len, print, range, etc. — always available
```

### global and nonlocal

```python
counter = 0

def increment():
    global counter          # modify module-level variable
    counter += 1

def make_counter():
    count = 0
    def inc():
        nonlocal count      # modify enclosing (not global) variable
        count += 1
        return count
    return inc

c = make_counter()
c()   # 1
c()   # 2
```

### Closures

A closure is a function that **remembers** variables from its enclosing scope, even after that scope has finished executing.

```python
def multiplier(factor):
    def multiply(x):
        return x * factor   # factor is "closed over"
    return multiply

double = multiplier(2)
triple = multiplier(3)

double(5)   # 10
triple(5)   # 15

# Inspect closure
double.__closure__                    # tuple of cell objects
double.__closure__[0].cell_contents  # 2
```

### Late Binding Gotcha in Closures

```python
# ❌ All lambdas share the same variable i
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]    # [2, 2, 2] — all see the final value of i

# ✅ Bind at definition time using default argument
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]    # [0, 1, 2]
```

---

## 4. Lambdas & Higher-Order Functions

### Lambda

```python
# Syntax: lambda arguments: expression  (single expression only)
square  = lambda x: x ** 2
add     = lambda x, y: x + y

square(4)     # 16
add(2, 3)     # 5

# Common uses — inline, throwaway functions
nums = [3, 1, 4, 1, 5, 9]
sorted(nums, key=lambda x: -x)          # sort descending

people = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
sorted(people, key=lambda p: p["age"])  # sort by age

# Multi-key sort
sorted(people, key=lambda p: (p["age"], p["name"]))
```

### Higher-Order Functions

A function that takes a function as an argument or returns one.

```python
# map — apply function to every element (lazy)
list(map(lambda x: x**2, [1, 2, 3]))          # [1, 4, 9]
list(map(str, [1, 2, 3]))                      # ['1','2','3']

# filter — keep elements where function returns True (lazy)
list(filter(lambda x: x % 2 == 0, range(10))) # [0,2,4,6,8]
list(filter(None, [0,"","a",1,None]))           # ['a', 1]

# zip — pair elements from multiple iterables (lazy)
list(zip([1,2,3], ["a","b","c"]))              # [(1,'a'),(2,'b'),(3,'c')]

# Transpose a matrix using zip
matrix = [[1,2,3],[4,5,6],[7,8,9]]
list(zip(*matrix))   # [(1,4,7),(2,5,8),(3,6,9)]

# reduce — fold to a single value
from functools import reduce
reduce(lambda acc, x: acc + x, [1,2,3,4], 0)  # 10
reduce(lambda a,b: a if a > b else b, [3,1,4,1,5])  # 5 (max)
```

### functools Toolkit

```python
from functools import partial, lru_cache, wraps, cached_property

# partial — pre-fill some arguments
from functools import partial
power_of_2 = partial(pow, 2)    # pow(2, ?) — first arg fixed
power_of_2(10)                   # 1024

multiply = lambda x, y: x * y
double = partial(multiply, y=2)
double(5)                        # 10

# lru_cache — memoization
@lru_cache(maxsize=None)         # None = unlimited cache
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

fib(50)             # instant — cached
fib.cache_info()    # CacheInfo(hits=48, misses=51, ...)
fib.cache_clear()   # wipe the cache

# cached_property — compute once, store on instance (3.8+)
class Circle:
    def __init__(self, r): self.r = r

    @cached_property
    def area(self):
        print("computing...")
        return 3.14159 * self.r ** 2

c = Circle(5)
c.area   # "computing..." → 78.53...
c.area   # silent — cached on instance dict
```

---

## 5. Decorators

### What Is a Decorator?

A decorator is a callable that **wraps** another function to extend or modify its behavior without changing the original source code.

```python
@decorator
def func(): ...

# Exactly equivalent to:
def func(): ...
func = decorator(func)
```

### Basic Decorator

```python
import functools

def log_calls(func):
    @functools.wraps(func)   # preserve __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args} {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_calls
def add(a, b):
    return a + b

add(2, 3)
# Calling add with (2, 3) {}
# add returned 5
```

### Decorator with Arguments (Three Levels)

```python
def repeat(n=3):
    """Decorator factory — takes arguments, returns decorator."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(n=3)
def say(msg):
    print(msg)

say("hello")   # prints "hello" three times
```

### Timing Decorator

```python
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start  = time.perf_counter()
        result = func(*args, **kwargs)
        end    = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.5)
```

### Retry Decorator

```python
def retry(times=3, delay=1, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    print(f"Attempt {attempt} failed: {e}. Retrying...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(times=3, delay=2, exceptions=(ConnectionError,))
def fetch_data(url):
    ...
```

### Class-Based Decorator

```python
class Memoize:
    """Decorator as a class — keeps state between calls."""
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func  = func
        self.cache = {}

    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]

@Memoize
def expensive(n):
    return n ** 2
```

### Stacking Decorators

```python
@timer          # applied second (outer)
@log_calls      # applied first (inner)
def process():
    ...

# Equivalent to: process = timer(log_calls(process))
# Execution order: timer's wrapper → log_calls's wrapper → process
```

---

## 6. Classes — Fundamentals

### Anatomy of a Class

```python
class BankAccount:
    """A simple bank account."""

    interest_rate = 0.02            # class variable — shared by all instances

    def __init__(self, owner: str, balance: float = 0.0):
        """Constructor — called when object is created."""
        self.owner   = owner        # instance variable
        self.balance = balance      # instance variable
        self._id     = id(self)     # "private by convention" (single underscore)

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.balance += amount

    def withdraw(self, amount: float) -> float:
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        return amount

    def __str__(self) -> str:
        return f"BankAccount({self.owner}, ${self.balance:.2f})"

    def __repr__(self) -> str:
        return f"BankAccount(owner={self.owner!r}, balance={self.balance!r})"


# Instantiation
acc = BankAccount("Alice", 1000)
acc.deposit(500)
print(acc)           # BankAccount(Alice, $1500.00)
repr(acc)            # BankAccount(owner='Alice', balance=1500)
```

### Class vs Instance Variables

```python
class Dog:
    species = "Canis lupus"    # class variable — shared

    def __init__(self, name):
        self.name = name       # instance variable — unique per object

d1 = Dog("Rex")
d2 = Dog("Buddy")

Dog.species          # "Canis lupus"
d1.species           # "Canis lupus"  — found on class, not instance

Dog.species = "Canis familiaris"  # change on class → affects all instances
d1.species           # "Canis familiaris"

d1.species = "Wolf"  # creates an INSTANCE variable that shadows the class var
d2.species           # "Canis familiaris"  — class var unchanged
d1.__dict__          # {'name': 'Rex', 'species': 'Wolf'}
d2.__dict__          # {'name': 'Buddy'}
```

---

## 7. Instance vs Class vs Static

```python
class Temperature:
    _unit = "Celsius"               # class variable

    def __init__(self, degrees):
        self.degrees = degrees      # instance variable

    # Instance method — has access to self (the instance)
    def to_fahrenheit(self) -> float:
        return self.degrees * 9/5 + 32

    # Class method — has access to cls (the class), not the instance
    # Use for: factory methods, accessing/modifying class state
    @classmethod
    def set_unit(cls, unit: str) -> None:
        cls._unit = unit

    @classmethod
    def from_fahrenheit(cls, f: float) -> "Temperature":
        """Factory: create instance from Fahrenheit."""
        return cls((f - 32) * 5/9)

    # Static method — no access to self or cls
    # Use for: utility functions logically related to the class
    @staticmethod
    def is_valid(degrees: float) -> bool:
        return degrees >= -273.15   # absolute zero

    def __repr__(self):
        return f"Temperature({self.degrees}°{Temperature._unit})"


t1 = Temperature(100)
t1.to_fahrenheit()               # 212.0  — instance method

Temperature.set_unit("Kelvin")   # class method via class
t1.set_unit("Celsius")           # class method via instance (also works)

t2 = Temperature.from_fahrenheit(212)   # factory class method
Temperature.is_valid(-300)              # False — static method
```

### When to Use Each

| | `self` | `cls` | Neither |
|---|---|---|---|
| **Instance method** | ✅ | via `type(self)` | — |
| **@classmethod** | ✗ | ✅ | — |
| **@staticmethod** | ✗ | ✗ | ✅ |
| **Access instance data** | ✅ | ✗ | ✗ |
| **Access/modify class data** | via `self.__class__` | ✅ | ✗ |
| **Factory / alternate constructor** | ✗ | ✅ | ✗ |
| **Utility / pure function in class namespace** | ✗ | ✗ | ✅ |

---

## 8. Properties & Encapsulation

### Name Mangling

```python
class Person:
    def __init__(self, name, age):
        self.name   = name       # public
        self._age   = age        # "protected" — convention only
        self.__ssn  = "secret"   # name-mangled → _Person__ssn

p = Person("Alice", 30)
p.name             # "Alice"
p._age             # 30   — accessible but "please don't"
p.__ssn            # AttributeError
p._Person__ssn     # "secret" — still accessible if you know the mangled name
```

### @property — Controlled Attribute Access

```python
class Circle:
    def __init__(self, radius: float):
        self._radius = radius     # store on "private" attribute

    @property
    def radius(self) -> float:
        """Getter — accessed like an attribute."""
        return self._radius

    @radius.setter
    def radius(self, value: float) -> None:
        """Setter — validate before storing."""
        if value < 0:
            raise ValueError(f"Radius cannot be negative: {value}")
        self._radius = value

    @radius.deleter
    def radius(self) -> None:
        """Deleter — called on `del obj.radius`."""
        del self._radius

    @property
    def area(self) -> float:
        """Computed property — no setter needed."""
        return 3.14159 * self._radius ** 2

    @property
    def diameter(self) -> float:
        return self._radius * 2

    @diameter.setter
    def diameter(self, value: float) -> None:
        self.radius = value / 2    # reuse radius setter (with validation)


c = Circle(5)
c.radius              # 5        — calls getter
c.radius = 10         # calls setter
c.radius = -1         # ValueError!
c.area                # 314.159  — computed, read-only
c.diameter = 20       # calls diameter setter → sets radius to 10
```

---

## 9. Dunder (Magic) Methods

### String Representation

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __str__(self) -> str:
        """Human-readable — called by str() and print()."""
        return f"({self.x}, {self.y})"

    def __repr__(self) -> str:
        """Unambiguous — called by repr(), REPL, debuggers.
           Rule of thumb: eval(repr(obj)) should recreate obj."""
        return f"Vector({self.x!r}, {self.y!r})"
```

### Comparison Operators

```python
    def __eq__(self, other) -> bool:
        if not isinstance(other, Vector): return NotImplemented
        return self.x == other.x and self.y == other.y

    def __lt__(self, other) -> bool:
        return (self.x**2 + self.y**2) < (other.x**2 + other.y**2)

    # @functools.total_ordering fills in the rest from __eq__ and __lt__
    # (__le__, __gt__, __ge__)
```

### Arithmetic Operators

```python
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        """Called when scalar * vector (reversed operands)."""
        return self.__mul__(scalar)

    def __neg__(self):
        return Vector(-self.x, -self.y)

    def __abs__(self):
        return (self.x**2 + self.y**2) ** 0.5
```

### Container Protocol

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def __len__(self) -> int:
        return len(self._songs)

    def __getitem__(self, index):
        return self._songs[index]

    def __setitem__(self, index, value):
        self._songs[index] = value

    def __delitem__(self, index):
        del self._songs[index]

    def __contains__(self, item) -> bool:
        return item in self._songs

    def __iter__(self):
        return iter(self._songs)


p = Playlist()
len(p)             # 0
"Bohemian" in p    # False
for song in p: ... # iterates
```

### Context Manager Protocol

```python
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn          # value bound to `as` clause

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False              # False = don't suppress exceptions
                                  # True  = swallow exceptions

with DatabaseConnection() as conn:
    conn.query("SELECT 1")
```

### Callable Protocol

```python
class Adder:
    def __init__(self, n):
        self.n = n

    def __call__(self, x):
        return self.x + self.n

add5 = Adder(5)
add5(3)            # 8
callable(add5)     # True
```

### Complete Dunder Reference

| Method | Triggered by |
|---|---|
| `__init__` | `MyClass()` |
| `__del__` | garbage collection |
| `__str__` | `str(obj)`, `print(obj)` |
| `__repr__` | `repr(obj)`, REPL |
| `__len__` | `len(obj)` |
| `__bool__` | `bool(obj)`, `if obj:` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = val` |
| `__delitem__` | `del obj[key]` |
| `__contains__` | `x in obj` |
| `__iter__` | `for x in obj`, `iter(obj)` |
| `__next__` | `next(obj)` |
| `__eq__` | `obj == other` |
| `__ne__` | `obj != other` |
| `__lt__/le/gt/ge` | `< <= > >=` |
| `__hash__` | `hash(obj)`, dict key, set member |
| `__add__` | `obj + other` |
| `__sub__` | `obj - other` |
| `__mul__` | `obj * other` |
| `__truediv__` | `obj / other` |
| `__floordiv__` | `obj // other` |
| `__mod__` | `obj % other` |
| `__pow__` | `obj ** other` |
| `__neg__` | `-obj` |
| `__abs__` | `abs(obj)` |
| `__enter__` | `with obj:` |
| `__exit__` | end of `with` block |
| `__call__` | `obj(args)` |
| `__getattr__` | attribute not found normally |
| `__setattr__` | `obj.attr = val` |
| `__delattr__` | `del obj.attr` |
| `__class_getitem__` | `MyClass[int]` (generics) |

---

## 10. Inheritance

### Basic Inheritance

```python
class Animal:
    def __init__(self, name: str, sound: str):
        self.name  = name
        self.sound = sound

    def speak(self) -> str:
        return f"{self.name} says {self.sound}"

    def __str__(self):
        return self.name


class Dog(Animal):
    def __init__(self, name: str, breed: str):
        super().__init__(name, sound="Woof")  # call parent __init__
        self.breed = breed

    def fetch(self, item: str) -> str:
        return f"{self.name} fetches the {item}!"

    # Override parent method
    def speak(self) -> str:
        base = super().speak()          # call parent version
        return f"{base}! (tail wagging)"


class Cat(Animal):
    def __init__(self, name: str):
        super().__init__(name, sound="Meow")

    def speak(self) -> str:
        return f"{self.name} says {self.sound} (ignores you)"


# Polymorphism — same interface, different behavior
animals = [Dog("Rex","Labrador"), Cat("Whiskers"), Dog("Buddy","Poodle")]
for animal in animals:
    print(animal.speak())   # calls each class's own speak()
```

### isinstance and issubclass

```python
rex = Dog("Rex", "Lab")

isinstance(rex, Dog)     # True
isinstance(rex, Animal)  # True  — Dog IS-A Animal
isinstance(rex, Cat)     # False

issubclass(Dog, Animal)  # True
issubclass(Cat, Dog)     # False
issubclass(Dog, Dog)     # True  — a class is its own subclass
```

### super() Deep Dive

```python
class A:
    def method(self):
        print("A.method")

class B(A):
    def method(self):
        super().method()    # calls A.method
        print("B.method")

class C(A):
    def method(self):
        super().method()    # calls A.method
        print("C.method")

class D(B, C):
    def method(self):
        super().method()    # follows MRO: D→B→C→A
        print("D.method")

D().method()
# A.method
# C.method
# B.method
# D.method
```

---

## 11. Multiple Inheritance & MRO

### MRO — Method Resolution Order

Python uses the **C3 Linearization** algorithm to determine method lookup order.

```python
class A:
    def greet(self): return "A"

class B(A):
    def greet(self): return "B → " + super().greet()

class C(A):
    def greet(self): return "C → " + super().greet()

class D(B, C):
    def greet(self): return "D → " + super().greet()

D().greet()       # "D → B → C → A"
D.__mro__
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

### Mixin Pattern

Mixins provide reusable, narrowly focused behavior without forming deep hierarchies.

```python
class LogMixin:
    """Add logging to any class."""
    def log(self, message: str) -> None:
        print(f"[{self.__class__.__name__}] {message}")


class SerializeMixin:
    """Add JSON serialization to any class."""
    import json

    def to_json(self) -> str:
        import json
        return json.dumps(self.__dict__)

    @classmethod
    def from_json(cls, data: str):
        import json
        obj = cls.__new__(cls)
        obj.__dict__.update(json.loads(data))
        return obj


class User(LogMixin, SerializeMixin):
    def __init__(self, name, email):
        self.name  = name
        self.email = email

    def save(self):
        self.log(f"Saving user {self.name}")
        return self.to_json()


u = User("Alice", "alice@example.com")
u.save()
# [User] Saving user Alice
# '{"name": "Alice", "email": "alice@example.com"}'

User.from_json('{"name":"Bob","email":"bob@example.com"}')
```

---

## 12. Abstract Classes & Interfaces

### ABC — Abstract Base Class

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    """Interface: every Shape must implement area and perimeter."""

    @abstractmethod
    def area(self) -> float:
        """Must be overridden — no default implementation."""
        ...

    @abstractmethod
    def perimeter(self) -> float: ...

    # Concrete method — shared by all subclasses
    def describe(self) -> str:
        return (f"{self.__class__.__name__}: "
                f"area={self.area():.2f}, perimeter={self.perimeter():.2f}")

    @classmethod
    @abstractmethod
    def from_string(cls, s: str) -> "Shape":
        """Abstract class method — subclass must implement."""
        ...


class Rectangle(Shape):
    def __init__(self, w: float, h: float):
        self.w, self.h = w, h

    def area(self)      -> float: return self.w * self.h
    def perimeter(self) -> float: return 2 * (self.w + self.h)

    @classmethod
    def from_string(cls, s: str) -> "Rectangle":
        w, h = map(float, s.split("x"))
        return cls(w, h)


class Circle(Shape):
    import math
    def __init__(self, r: float): self.r = r
    def area(self)      -> float: return 3.14159 * self.r ** 2
    def perimeter(self) -> float: return 2 * 3.14159 * self.r

    @classmethod
    def from_string(cls, s: str) -> "Circle":
        return cls(float(s))


# Shape()      → TypeError: Can't instantiate abstract class
r = Rectangle(3, 4)
r.describe()   # "Rectangle: area=12.00, perimeter=14.00"
```

### Protocol — Structural Subtyping (Duck Typing + Type Hints)

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
    def resize(self, factor: float) -> None: ...


class Circle:
    def draw(self)              -> None: print("drawing circle")
    def resize(self, factor: float) -> None: self.r *= factor

# Circle doesn't inherit from Drawable — but it satisfies the protocol
def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())     # works — structural (duck) typing
isinstance(Circle(), Drawable)  # True — runtime_checkable
```

---

## 13. Dataclasses

```python
from dataclasses import dataclass, field, KW_ONLY
from typing import ClassVar

@dataclass
class Point:
    x: float
    y: float
    label: str = ""                       # field with default

@dataclass
class Inventory:
    items:    list[str] = field(default_factory=list)   # mutable default
    capacity: int       = field(default=100)
    _count:   int       = field(default=0, repr=False)  # excluded from repr

    # Class variable — NOT a field
    category: ClassVar[str] = "warehouse"

@dataclass(order=True, frozen=True)       # frozen = immutable (hashable)
class Version:
    major: int
    minor: int
    patch: int = 0

    def __str__(self):
        return f"{self.major}.{self.minor}.{self.patch}"


v1 = Version(1, 0)
v2 = Version(2, 0)
v1 < v2           # True  — order=True adds comparison operators
{v1, v2}          # set works — frozen=True makes it hashable

# Post-init hook
@dataclass
class Circle:
    radius: float

    def __post_init__(self):
        if self.radius < 0:
            raise ValueError("radius must be >= 0")
        self.area = 3.14159 * self.radius ** 2   # computed field
```

### dataclass vs namedtuple vs plain class

| Feature | `@dataclass` | `namedtuple` | Plain class |
|---|---|---|---|
| Mutable | ✅ (unless frozen) | ✗ | ✅ |
| Type hints | ✅ | ✅ (NamedTuple) | optional |
| Default values | ✅ | via `field()` | ✅ |
| `__repr__` auto | ✅ | ✅ | manual |
| `__eq__` auto | ✅ | ✅ | manual |
| Hashable | only if frozen | ✅ | if `__hash__` defined |
| Inheritance | ✅ | limited | ✅ |
| `__post_init__` | ✅ | ✗ | via `__init__` |

---

## 14. OOP Design Principles (SOLID)

### S — Single Responsibility Principle

> A class should have only one reason to change.

```python
# ❌ Bad — one class doing too much
class UserManager:
    def create_user(self, data): ...
    def send_welcome_email(self, user): ...   # email concern mixed in
    def save_to_database(self, user): ...     # DB concern mixed in

# ✅ Good — each class has one job
class UserService:
    def __init__(self, repo, mailer):
        self.repo   = repo
        self.mailer = mailer

    def create_user(self, data):
        user = User(**data)
        self.repo.save(user)
        self.mailer.send_welcome(user)
        return user

class UserRepository:
    def save(self, user): ...

class EmailService:
    def send_welcome(self, user): ...
```

### O — Open/Closed Principle

> Open for extension, closed for modification.

```python
# ❌ Bad — must modify existing code to add new shapes
def total_area(shapes):
    total = 0
    for s in shapes:
        if isinstance(s, Rectangle): total += s.w * s.h
        if isinstance(s, Circle):    total += 3.14 * s.r**2
    return total

# ✅ Good — add new shapes without touching total_area
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

def total_area(shapes: list[Shape]) -> float:
    return sum(s.area() for s in shapes)
```

### L — Liskov Substitution Principle

> Subclass instances must be usable wherever a base class is expected.

```python
# ❌ Bad — Square breaks the Rectangle contract
class Rectangle:
    def set_width(self, w):  self.w = w
    def set_height(self, h): self.h = h
    def area(self): return self.w * self.h

class Square(Rectangle):
    def set_width(self, w):  self.w = self.h = w   # breaks expectations!
    def set_height(self, h): self.w = self.h = h

# ✅ Good — make Square standalone, share behavior via interface
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Rectangle(Shape):
    def __init__(self, w, h): self.w, self.h = w, h
    def area(self): return self.w * self.h

class Square(Shape):
    def __init__(self, side): self.side = side
    def area(self): return self.side ** 2
```

### I — Interface Segregation Principle

> Clients should not be forced to depend on methods they don't use.

```python
# ❌ Bad — one fat interface
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...      # robots don't eat!
    @abstractmethod
    def sleep(self): ...    # robots don't sleep!

# ✅ Good — small, focused interfaces
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class HumanWorker(Workable, Eatable):
    def work(self): print("working")
    def eat(self):  print("eating")

class RobotWorker(Workable):
    def work(self): print("working tirelessly")
```

### D — Dependency Inversion Principle

> Depend on abstractions, not concretions.

```python
# ❌ Bad — high-level module depends on concrete low-level module
class ReportService:
    def __init__(self):
        self.db = MySQLDatabase()   # hardcoded dependency

# ✅ Good — depend on an abstraction
class Database(ABC):
    @abstractmethod
    def query(self, sql: str): ...

class ReportService:
    def __init__(self, db: Database):   # injected, not hardcoded
        self.db = db

class MySQLDatabase(Database):
    def query(self, sql): ...

class MockDatabase(Database):           # easy to test with!
    def query(self, sql): return []

service = ReportService(MockDatabase())
```

---

## 15. Common OOP Design Patterns

### Singleton — One Instance Only

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, value=None):
        if not hasattr(self, "_initialized"):
            self.value = value
            self._initialized = True


s1 = Singleton("first")
s2 = Singleton("second")
s1 is s2        # True — same object
s1.value        # "first" — init only ran once
```

### Factory Method — Delegate Object Creation

```python
class Animal(ABC):
    @abstractmethod
    def speak(self) -> str: ...

class Dog(Animal):
    def speak(self): return "Woof"

class Cat(Animal):
    def speak(self): return "Meow"

class AnimalFactory:
    _registry = {"dog": Dog, "cat": Cat}

    @classmethod
    def create(cls, kind: str) -> Animal:
        if kind not in cls._registry:
            raise ValueError(f"Unknown animal: {kind}")
        return cls._registry[kind]()

    @classmethod
    def register(cls, kind: str, animal_class):
        cls._registry[kind] = animal_class


animal = AnimalFactory.create("dog")
animal.speak()    # "Woof"
```

### Observer — Event / Pub-Sub

```python
from typing import Callable

class EventEmitter:
    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, *args, **kwargs) -> None:
        for cb in self._listeners.get(event, []):
            cb(*args, **kwargs)


class Store(EventEmitter):
    def __init__(self):
        super().__init__()
        self._data = {}

    def set(self, key, value):
        old = self._data.get(key)
        self._data[key] = value
        self.emit("change", key=key, old=old, new=value)


store = Store()
store.on("change", lambda key, old, new: print(f"{key}: {old} → {new}"))
store.set("name", "Alice")   # name: None → Alice
store.set("name", "Bob")     # name: Alice → Bob
```

### Strategy — Swappable Algorithms

```python
from typing import Protocol

class SortStrategy(Protocol):
    def sort(self, data: list) -> list: ...

class BubbleSort:
    def sort(self, data: list) -> list:
        d = data.copy()
        for i in range(len(d)):
            for j in range(len(d)-i-1):
                if d[j] > d[j+1]: d[j], d[j+1] = d[j+1], d[j]
        return d

class QuickSort:
    def sort(self, data: list) -> list:
        if len(data) <= 1: return data
        pivot = data[len(data)//2]
        left  = [x for x in data if x < pivot]
        mid   = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + mid + self.sort(right)

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: SortStrategy):
        self._strategy = strategy

    def sort(self, data: list) -> list:
        return self._strategy.sort(data)


sorter = Sorter(QuickSort())
sorter.sort([3,1,4,1,5])    # [1,1,3,4,5]
sorter.set_strategy(BubbleSort())
sorter.sort([3,1,4,1,5])    # [1,1,3,4,5]
```

### Context Manager Pattern (also a pattern)

```python
from contextlib import contextmanager

@contextmanager
def timer(label: str = ""):
    import time
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label} took {elapsed:.4f}s")

with timer("database query"):
    results = run_query()
```

---

## 16. Interview Q&A

**Q: What is the difference between a class variable and an instance variable?**
> A class variable is shared across all instances — changing it on the class changes it for every object. An instance variable is unique per object. If you assign to a class variable through an instance (`obj.class_var = x`), Python creates a new instance variable that shadows the class variable for that object only.

**Q: What does `super()` do and why is it preferred over calling the parent class directly?**
> `super()` returns a proxy that delegates method calls following the MRO. It's preferred over `Parent.method(self)` because it supports cooperative multiple inheritance correctly — each class in the MRO gets to run exactly once, in order. Calling the parent directly can cause it to run multiple times in a diamond inheritance scenario.

**Q: What is the difference between `__str__` and `__repr__`?**
> `__str__` is for human-readable output — it's what `print()` calls. `__repr__` is for an unambiguous developer representation, ideally one you could `eval()` to recreate the object. If only one is defined, `str()` falls back to `__repr__`. Always define `__repr__` at minimum.

**Q: Why must `__hash__` be defined when `__eq__` is defined?**
> Python sets `__hash__ = None` automatically when you define `__eq__`, making the object unhashable (can't be used as a dict key or set member). If your objects are meant to be equal by value AND still be hashable, you must define `__hash__` explicitly to be consistent with `__eq__`.

**Q: What is the difference between `@classmethod` and `@staticmethod`?**
> `@classmethod` receives the class (`cls`) as its first argument — useful for factory methods and accessing class-level state. `@staticmethod` receives nothing implicitly — it's a plain function that belongs in the class namespace for logical grouping, but doesn't need to access the class or instance.

**Q: What is a mixin?**
> A mixin is a class designed to add a specific set of methods to other classes through multiple inheritance. Mixins are narrow in scope (e.g., `LogMixin`, `SerializeMixin`) and are not meant to be instantiated on their own. They follow the Interface Segregation Principle by keeping concerns separate.

**Q: What is the MRO and how is it computed?**
> The Method Resolution Order defines the order Python searches for a method when it's called on an object. It's computed using C3 Linearization: roughly, depth-first, left-to-right, but a class never comes before its subclasses. You can inspect it with `ClassName.__mro__` or `ClassName.mro()`.

**Q: What makes a good `__eq__` implementation?**
> It should return `NotImplemented` (not `False`) when the comparison type is unsupported — this lets Python try the reflected operation on the other object. It should be reflexive, symmetric, and transitive. And if you define `__eq__`, also define `__hash__` if the object should remain hashable.

**Q: What is the difference between composition and inheritance?**
> Inheritance models an IS-A relationship (Dog IS-A Animal). Composition models a HAS-A relationship (Car HAS-A Engine). Prefer composition when you want to reuse behavior without committing to a type hierarchy — it's more flexible and avoids tight coupling. The rule of thumb: inherit only when the subclass truly is a specialized version of the parent.

**Q: What is a descriptor?**
> A descriptor is any object that defines `__get__`, `__set__`, or `__delete__`. `@property` is implemented as a descriptor. Descriptors enable reusable attribute validation logic across multiple attributes or classes, and are the mechanism behind `@property`, `@classmethod`, and `@staticmethod`.

```python
class Positive:
    """Descriptor: validates that a numeric attribute is positive."""
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None: return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self.name} must be positive, got {value}")
        obj.__dict__[self.name] = value

class Order:
    quantity = Positive()
    price    = Positive()

    def __init__(self, quantity, price):
        self.quantity = quantity
        self.price    = price

o = Order(5, 9.99)
o.quantity = -1     # ValueError: quantity must be positive
```

---

*Last updated: April 2026 | Python 3.10+*
