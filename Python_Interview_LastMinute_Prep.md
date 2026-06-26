# Python Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, glance at the **Program** to see it in code, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Target this for a 2–4 hour final revision.

---

## Table of Contents

1. [Python Fundamentals](#1-python-fundamentals)
2. [Data Types & Data Structures](#2-data-types--data-structures)
3. [Mutability, Copy & Memory](#3-mutability-copy--memory)
4. [Strings](#4-strings)
5. [Functions, Args & Scope](#5-functions-args--scope)
6. [Comprehensions, Iterators & Generators](#6-comprehensions-iterators--generators)
7. [OOP in Python](#7-oop-in-python)
8. [Decorators & Closures](#8-decorators--closures)
9. [Exception Handling](#9-exception-handling)
10. [Modules, Packages & Files](#10-modules-packages--files)
11. [Advanced: GIL, Concurrency, Memory Management](#11-advanced-gil-concurrency-memory-management)
12. [Common Coding Programs (with solutions)](#12-common-coding-programs)
13. [Tricky Output / Gotcha Questions](#13-tricky-output--gotcha-questions)
14. [Rapid-Fire One-Liners](#14-rapid-fire-one-liners)
15. [Last-Minute Checklist](#15-last-minute-checklist)

---

## 1. Python Fundamentals

### Theory

**Python** is a high-level, interpreted, dynamically-typed, garbage-collected, general-purpose language that supports procedural, object-oriented, and functional programming.

Key properties an interviewer wants:

- **Interpreted** — code runs line-by-line via the interpreter; no separate compile step (CPython compiles to bytecode `.pyc`, then runs it on the Python Virtual Machine).
- **Dynamically typed** — type is checked at runtime; you don't declare types.
- **Strongly typed** — implicit unsafe conversions are not allowed (`"2" + 3` raises an error).
- **Everything is an object** — including functions, classes, and modules.

**Python 2 vs Python 3 (the classic question):** Python 3 uses `print()` as a function, `/` is true division (returns float), strings are Unicode by default, `range()` returns an iterator, and Python 2 is end-of-life. Always use Python 3.

**PEP 8** is the official style guide (4-space indentation, snake_case for variables/functions, PascalCase for classes, max line length 79).

**How Python runs:** Source `.py` → compiled to **bytecode** → executed by the **PVM (Python Virtual Machine)**. CPython is the reference implementation (written in C). Others: PyPy (JIT), Jython (JVM), IronPython (.NET).

### Program

```python
# Dynamic + strong typing
x = 10          # int
x = "hello"     # now a str — allowed (dynamic)
# print("2" + 3)  # TypeError — not allowed (strong)

print(type(x))   # <class 'str'>
print(7 / 2)     # 3.5  (true division)
print(7 // 2)    # 3    (floor division)
print(2 ** 3)    # 8    (power)
```

### Interview Q&A

**Q: Is Python compiled or interpreted?**
Both, technically. Source is first **compiled to bytecode**, which is then **interpreted** by the PVM. We commonly call it interpreted because there is no explicit compile step.

**Q: What is the difference between `is` and `==`?**
`==` compares **values**; `is` compares **identity** (whether two names point to the same object in memory). `a == b` can be `True` while `a is b` is `False`.

**Q: What are Python's key features?**
Easy/readable syntax, dynamically typed, interpreted, portable, huge standard library ("batteries included"), supports multiple paradigms, automatic memory management, and a large ecosystem.

---

## 2. Data Types & Data Structures

### Theory

**Built-in types:**

| Category | Types | Mutable? | Ordered? |
|---|---|---|---|
| Numeric | `int`, `float`, `complex`, `bool` | No | — |
| Sequence | `list`, `tuple`, `range` | list: yes, tuple: no | Yes |
| Text | `str` | No | Yes |
| Set | `set`, `frozenset` | set: yes | No |
| Mapping | `dict` | Yes | Yes (insertion order, 3.7+) |
| Binary | `bytes`, `bytearray` | bytearray: yes | Yes |

**The four core collections — know these cold:**

- **List `[]`** — ordered, mutable, allows duplicates, indexable.
- **Tuple `()`** — ordered, **immutable**, allows duplicates; faster, hashable (usable as dict key).
- **Set `{}`** — unordered, mutable, **no duplicates**, fast membership tests (O(1)).
- **Dict `{k:v}`** — key→value pairs, keys unique & hashable, insertion-ordered (3.7+).

### Program

```python
lst   = [1, 2, 2, 3]          # list  – duplicates ok
tup   = (1, 2, 3)             # tuple – immutable
st    = {1, 2, 2, 3}          # set   – {1,2,3}
dct   = {"a": 1, "b": 2}      # dict

# Common operations
lst.append(4); lst.pop()      # add / remove
print(2 in st)                # True  (O(1) lookup)
print(dct.get("z", "default"))# safe access -> 'default'
print(sorted(lst))            # [1, 2, 2, 3]

# Dict iteration
for k, v in dct.items():
    print(k, v)
```

### Interview Q&A

**Q: List vs Tuple — when to use which?**
Use a **list** when data changes (mutable). Use a **tuple** for fixed data — it's immutable, slightly faster, memory-efficient, and **hashable** so it can be a dict key or set element. Tuples signal "this shouldn't change."

**Q: How does a `set` give O(1) lookups?**
It's backed by a **hash table**; membership uses the element's hash, not a linear scan.

**Q: Can a list be a dictionary key?**
No — keys must be **hashable** (immutable). Lists are mutable, so they're unhashable. Use a tuple instead.

**Q: How do you remove duplicates from a list?**
`list(set(my_list))` (loses order) or `list(dict.fromkeys(my_list))` (preserves order).

---

## 3. Mutability, Copy & Memory

### Theory

- **Mutable:** `list`, `dict`, `set`, `bytearray` — can change in place.
- **Immutable:** `int`, `float`, `str`, `tuple`, `frozenset`, `bytes` — a "change" creates a new object.

**Shallow vs Deep copy** (very common question):

- **Assignment (`=`)** — just a new reference to the **same** object.
- **Shallow copy** (`copy.copy()`, `list[:]`, `dict.copy()`) — new outer object, but nested objects are **shared**.
- **Deep copy** (`copy.deepcopy()`) — recursively copies everything; fully independent.

### Program

```python
import copy

a = [[1, 2], [3, 4]]

b = a                       # same object
c = copy.copy(a)            # shallow: inner lists shared
d = copy.deepcopy(a)        # deep: fully independent

a[0][0] = 99
print(b)   # [[99, 2], [3, 4]]  – b is a
print(c)   # [[99, 2], [3, 4]]  – inner list shared!
print(d)   # [[1, 2], [3, 4]]   – untouched
```

### Interview Q&A

**Q: Difference between shallow and deep copy?**
A **shallow copy** duplicates the top-level container but keeps references to nested objects, so mutating a nested object affects both copies. A **deep copy** recursively clones nested objects, producing a fully independent structure.

**Q: Are arguments passed by value or by reference in Python?**
Neither exactly — Python uses **"pass by object reference"** (call by assignment). The reference is passed by value. Mutating a mutable argument in place affects the caller; reassigning the name inside the function does not.

**Q: Why are strings immutable?**
For safety, hashability (usable as dict keys), and performance (interning / caching of common strings).

---

## 4. Strings

### Theory

Strings are **immutable** sequences of Unicode characters. Slicing syntax: `s[start:stop:step]`. f-strings (`f"{var}"`, Python 3.6+) are the preferred formatting.

### Program

```python
s = "Python"

print(s[0], s[-1])      # P n
print(s[1:4])           # yth
print(s[::-1])          # nohtyP   (reverse)
print(s.upper())        # PYTHON
print("a,b,c".split(",")) # ['a', 'b', 'c']
print("-".join(["a","b"])) # a-b
print(f"len = {len(s)}")  # len = 6
print("  hi  ".strip())   # 'hi'
print(s.replace("Py","My")) # Mython
```

### Interview Q&A

**Q: How do you reverse a string?**
`s[::-1]` — slicing with step `-1`.

**Q: Check if a string is a palindrome.**
`s == s[::-1]`.

**Q: Difference between `split()` and `join()`?**
`split()` turns a string into a list; `join()` turns an iterable of strings into a single string. They're inverses.

**Q: What's the difference between `find()` and `index()`?**
Both return the position of a substring, but `find()` returns `-1` if not found while `index()` raises `ValueError`.

---

## 5. Functions, Args & Scope

### Theory

**Argument types:**

- **Positional** and **keyword** arguments.
- `*args` — collects extra positional args into a **tuple**.
- `**kwargs` — collects extra keyword args into a **dict**.
- **Default arguments** — `def f(x=10)`.

**`lambda`** — a single-expression anonymous function: `lambda x: x*2`.

**Scope — the LEGB rule:** name lookup goes **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in. Use `global` and `nonlocal` to rebind names in outer scopes.

### Program

```python
def demo(a, b=2, *args, **kwargs):
    print(a, b, args, kwargs)

demo(1)                       # 1 2 () {}
demo(1, 3, 4, 5, x=9)         # 1 3 (4, 5) {'x': 9}

double = lambda n: n * 2
print(double(5))              # 10

# map / filter / reduce
from functools import reduce
nums = [1, 2, 3, 4]
print(list(map(lambda x: x**2, nums)))      # [1, 4, 9, 16]
print(list(filter(lambda x: x % 2 == 0, nums)))  # [2, 4]
print(reduce(lambda a, b: a + b, nums))     # 10
```

### Interview Q&A

**Q: Difference between `*args` and `**kwargs`?**
`*args` captures a variable number of **positional** arguments as a tuple; `**kwargs` captures a variable number of **keyword** arguments as a dict.

**Q: What is the LEGB rule?**
The order Python resolves names: Local, Enclosing, Global, Built-in.

**Q: What's the mutable default argument trap?**
`def f(x=[])` — the list is created **once** at definition and shared across calls, so it "remembers" previous values. Fix with `def f(x=None): x = x or []`.

**Q: Lambda vs def?**
`lambda` is a one-line anonymous expression returning a value implicitly; `def` is a named function that can hold multiple statements. Use lambda for short throwaway functions (e.g., `key=` in `sorted`).

---

## 6. Comprehensions, Iterators & Generators

### Theory

**Comprehensions** — concise construction of collections:

- List: `[x*x for x in range(5)]`
- Dict: `{x: x*x for x in range(5)}`
- Set: `{x % 3 for x in range(5)}`

**Iterable vs Iterator:**

- **Iterable** — has `__iter__()` (list, str, dict). Can be looped over.
- **Iterator** — has `__next__()` and `__iter__()`; produces values lazily and remembers its state. `iter()` gets one; `next()` advances it.

**Generator** — a function using **`yield`** instead of `return`. It produces values **lazily**, one at a time, pausing/resuming state. Memory-efficient for large/infinite sequences. A **generator expression** is `(x*x for x in range(5))` with parentheses.

### Program

```python
# Comprehension
squares = [x*x for x in range(5)]      # [0,1,4,9,16]

# Generator function
def count_up(n):
    i = 0
    while i < n:
        yield i        # pauses here, resumes on next()
        i += 1

g = count_up(3)
print(next(g))  # 0
print(next(g))  # 1
print(list(g))  # [2]   (state continued)

# Generator expression – memory efficient
big_sum = sum(x*x for x in range(1_000_000))
```

### Interview Q&A

**Q: Difference between a list and a generator?**
A list stores all elements in memory at once; a generator **yields one value at a time lazily**, using far less memory — ideal for large or streaming data. A generator can only be iterated once.

**Q: `yield` vs `return`?**
`return` exits the function and sends back one value. `yield` **pauses** the function, returns a value, and resumes from the same point on the next call — turning the function into a generator.

**Q: Iterable vs iterator?**
An **iterable** can be looped over (implements `__iter__`). An **iterator** is the object that does the actual iteration (implements `__next__`), tracking position. `iter(iterable)` returns an iterator.

**Q: Why use generators?**
Lazy evaluation → low memory, can represent infinite sequences, faster startup. Trade-off: single-pass, no indexing.

---

## 7. OOP in Python

### Theory

**Four pillars:**

1. **Encapsulation** — bundling data + methods; hiding internals (`_protected`, `__private` name-mangled).
2. **Inheritance** — a class derives attributes/methods from a parent. Python supports **multiple inheritance** (resolved via **MRO** / C3 linearization).
3. **Polymorphism** — same interface, different behavior (method overriding, duck typing).
4. **Abstraction** — hide complexity behind a simple interface (`abc.ABC`, `@abstractmethod`).

**Key concepts:**

- `__init__` — constructor (initializer); `self` — reference to the instance.
- **Class vs instance variables** — class vars are shared; instance vars are per-object.
- **`@classmethod`** (gets `cls`), **`@staticmethod`** (no implicit first arg), **instance method** (gets `self`).
- **`@property`** — getter/setter as attribute access.
- **Dunder methods** — `__str__`, `__repr__`, `__len__`, `__eq__`, `__add__`, etc.
- **`super()`** — call the parent class's method.

### Program

```python
class Animal:
    species = "generic"            # class variable (shared)

    def __init__(self, name):
        self.name = name           # instance variable

    def speak(self):               # to be overridden
        return "..."

    def __str__(self):
        return f"Animal({self.name})"

class Dog(Animal):
    def speak(self):               # polymorphism via overriding
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

for a in (Dog("Rex"), Cat("Tom")):
    print(a.name, a.speak())       # Rex Woof / Tom Meow

print(Dog.__mro__)   # MRO: Dog -> Animal -> object
```

```python
class Temperature:
    def __init__(self, c):
        self._c = c

    @property
    def fahrenheit(self):          # access like an attribute
        return self._c * 9/5 + 32

    @staticmethod
    def info():
        return "Temperature class"

    @classmethod
    def from_f(cls, f):            # alternative constructor
        return cls((f - 32) * 5/9)

t = Temperature(100)
print(t.fahrenheit)     # 212.0
print(Temperature.from_f(212)._c)  # 100.0
```

### Interview Q&A

**Q: What is `self`?**
A reference to the **current instance**, passed automatically as the first argument to instance methods, giving access to that object's attributes and methods.

**Q: `@classmethod` vs `@staticmethod` vs instance method?**
Instance method takes `self` (works on object data). **Class method** takes `cls` (works on the class, e.g. alternative constructors). **Static method** takes neither — it's a utility function grouped under the class.

**Q: What is MRO?**
**Method Resolution Order** — the order Python searches base classes for a method in (multiple) inheritance, computed by the **C3 linearization** algorithm. View it with `Class.__mro__` or `Class.mro()`.

**Q: How is data hiding done?**
By convention: `_single` underscore = protected (hint only); `__double` = name-mangled to `_ClassName__attr`, making external access harder (not truly private).

**Q: `__str__` vs `__repr__`?**
`__str__` is the human-readable, "informal" string (used by `print`). `__repr__` is the unambiguous, developer-facing representation (used in the REPL / debugging), ideally valid Python to recreate the object.

**Q: Does Python support method overloading?**
Not in the traditional sense — a later definition replaces the earlier. You emulate it with default args, `*args`, or `functools.singledispatch`. **Overriding** is fully supported.

---

## 8. Decorators & Closures

### Theory

**Closure** — a nested function that **captures** variables from its enclosing scope and remembers them after the outer function returns.

**Decorator** — a callable that takes a function and returns a modified function, used to add behavior (logging, timing, auth) without changing the original. Applied with `@decorator` syntax. Use `functools.wraps` to preserve the wrapped function's metadata.

### Program

```python
import functools, time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.4f}s")
        return result
    return wrapper

@timer
def slow_add(a, b):
    time.sleep(0.1)
    return a + b

print(slow_add(2, 3))   # prints timing, then 5

# Closure example
def multiplier(factor):
    def inner(x):
        return x * factor      # captures 'factor'
    return inner

double = multiplier(2)
print(double(10))       # 20
```

### Interview Q&A

**Q: What is a decorator?**
A function that **wraps another function** to extend or modify its behavior without altering its source code. Common for logging, timing, caching, and access control.

**Q: What is a closure?**
A nested function that retains access to variables from its enclosing scope even after that outer function has finished executing.

**Q: Why use `functools.wraps`?**
To copy the original function's `__name__`, `__doc__`, etc. onto the wrapper, so introspection and debugging still show the real function.

**Q: Name built-in decorators.**
`@staticmethod`, `@classmethod`, `@property`, `@functools.lru_cache`, `@functools.wraps`.

---

## 9. Exception Handling

### Theory

Structure: `try` / `except` / `else` / `finally`.

- **`try`** — code that may fail.
- **`except`** — handles a specific exception (catch specific types, not bare `except`).
- **`else`** — runs only if no exception occurred.
- **`finally`** — always runs (cleanup, closing resources).
- **`raise`** — throw an exception manually.
- Custom exceptions subclass `Exception`.

### Program

```python
def divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        return "Cannot divide by zero"
    except TypeError as e:
        return f"Type error: {e}"
    else:
        return result          # runs if no error
    finally:
        print("done")          # always runs

print(divide(10, 2))   # done \n 5.0
print(divide(10, 0))   # done \n Cannot divide by zero

# Custom exception
class InsufficientFunds(Exception):
    pass

def withdraw(balance, amt):
    if amt > balance:
        raise InsufficientFunds("Not enough balance")
    return balance - amt
```

### Interview Q&A

**Q: Difference between `except:` and `except Exception:`?**
Bare `except:` catches **everything**, including `SystemExit`/`KeyboardInterrupt` — bad practice. `except Exception:` catches normal errors only. Always catch the **most specific** exception you can.

**Q: When does `finally` run?**
**Always** — whether or not an exception occurred, even if there's a `return` in the `try`/`except`. Used for cleanup.

**Q: `else` clause purpose?**
Runs only when the `try` block raises **no** exception — separates "risky" code from "runs on success" code.

**Q: How do you create a custom exception?**
Subclass `Exception` (or a more specific built-in) and `raise` it.

---

## 10. Modules, Packages & Files

### Theory

- **Module** — a single `.py` file of reusable code. Import with `import`, `from ... import ...`.
- **Package** — a directory of modules (historically with `__init__.py`).
- **`if __name__ == "__main__":`** — code runs only when the file is executed directly, not when imported.
- **File handling** — `open(path, mode)`; modes: `r`, `w`, `a`, `r+`, `b` (binary). Use **`with`** (context manager) to auto-close.
- **Virtual environments** — `venv` isolates project dependencies; `pip` installs packages.

### Program

```python
# Reading & writing files safely with context manager
with open("data.txt", "w") as f:
    f.write("hello\nworld")

with open("data.txt", "r") as f:
    for line in f:
        print(line.strip())     # hello / world
# file auto-closed here

# The __main__ guard
def main():
    print("Run directly")

if __name__ == "__main__":
    main()
```

### Interview Q&A

**Q: What does `if __name__ == "__main__":` do?**
When a file runs directly, its `__name__` is `"__main__"`, so the block executes. When the file is **imported**, `__name__` is the module name, so the block is skipped — letting a file act as both a script and an importable module.

**Q: Why use `with open(...)`?**
It's a **context manager** that guarantees the file is **closed** automatically, even if an exception occurs — no manual `close()` needed.

**Q: Difference between module and package?**
A module is a single `.py` file; a package is a folder containing multiple modules (and historically an `__init__.py`).

**Q: `import x` vs `from x import y`?**
`import x` brings in the whole module (`x.y`). `from x import y` brings the specific name `y` into the current namespace directly.

---

## 11. Advanced: GIL, Concurrency, Memory Management

### Theory

**GIL (Global Interpreter Lock)** — a mutex in CPython that allows **only one thread to execute Python bytecode at a time**. Consequence: threads don't give true parallelism for **CPU-bound** work, but are fine for **I/O-bound** work (the GIL is released during I/O).

**Concurrency choices:**

- **`threading`** — best for **I/O-bound** tasks (network, disk). Limited by GIL for CPU.
- **`multiprocessing`** — separate processes, separate interpreters → **true parallelism** for **CPU-bound** tasks.
- **`asyncio`** — single-threaded cooperative concurrency via `async`/`await`; great for many concurrent I/O operations.

**Memory management:**

- Python manages memory automatically with a **private heap**.
- **Reference counting** is the primary mechanism — an object is freed when its refcount hits 0.
- A **cyclic garbage collector** (`gc` module) handles reference cycles.
- **Interning** caches small ints (−5…256) and some strings.

### Program

```python
# I/O-bound -> threads help
import threading

def task(n):
    print(f"task {n}")

threads = [threading.Thread(target=task, args=(i,)) for i in range(3)]
[t.start() for t in threads]
[t.join() for t in threads]

# CPU-bound -> use processes for real parallelism
from multiprocessing import Pool

def square(x):
    return x * x

if __name__ == "__main__":
    with Pool(4) as p:
        print(p.map(square, [1, 2, 3, 4]))   # [1, 4, 9, 16]
```

### Interview Q&A

**Q: What is the GIL and why does it exist?**
The **Global Interpreter Lock** ensures only one thread executes Python bytecode at a time in CPython. It exists to make memory management (reference counting) thread-safe and simple. Downside: no true multithreaded CPU parallelism.

**Q: How do you achieve parallelism despite the GIL?**
Use **`multiprocessing`** (separate processes each with their own GIL), native extensions that release the GIL, or `asyncio`/threads for I/O-bound concurrency.

**Q: Threading vs multiprocessing — when?**
**Threading / asyncio** for **I/O-bound** (waiting on network/disk). **Multiprocessing** for **CPU-bound** (heavy computation) to use multiple cores.

**Q: How does Python manage memory?**
Automatic, using a **private heap**, **reference counting** as the main scheme, plus a **generational garbage collector** for cyclic references. Developers don't free memory manually.

**Q: What is garbage collection in Python?**
The automatic reclamation of memory from objects no longer referenced — driven by reference counting, with a cycle-detecting GC for circular references.

---

## 12. Common Coding Programs

These are the programs interviewers most often ask you to write on the spot.

### 1. Reverse a string

```python
def reverse(s):
    return s[::-1]
print(reverse("hello"))      # olleh
```

### 2. Check palindrome

```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
print(is_palindrome("Race car"))   # True
```

### 3. Fibonacci series

```python
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        print(a, end=" ")
        a, b = b, a + b
fib(7)                        # 0 1 1 2 3 5 8
```

### 4. Factorial (iterative & recursive)

```python
def fact(n):
    return 1 if n <= 1 else n * fact(n - 1)
print(fact(5))                # 120
```

### 5. Check prime

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True
print(is_prime(29))           # True
```

### 6. Count vowels / character frequency

```python
from collections import Counter
s = "interview"
print(Counter(s))             # {'i':2,'n':1,...}
print(sum(1 for c in s if c in "aeiou"))   # vowel count = 4
```

### 7. Find duplicates in a list

```python
from collections import Counter
nums = [1, 2, 2, 3, 3, 3, 4]
dupes = [k for k, v in Counter(nums).items() if v > 1]
print(dupes)                  # [2, 3]
```

### 8. Swap two numbers without a temp variable

```python
a, b = 5, 10
a, b = b, a
print(a, b)                   # 10 5
```

### 9. Find the largest / second largest

```python
nums = [10, 5, 8, 20, 15]
print(max(nums))              # 20
print(sorted(set(nums))[-2])  # 15 (second largest)
```

### 10. Anagram check

```python
def is_anagram(a, b):
    return sorted(a) == sorted(b)
print(is_anagram("listen", "silent"))   # True
```

### 11. FizzBuzz

```python
for i in range(1, 16):
    if i % 15 == 0: print("FizzBuzz")
    elif i % 3 == 0: print("Fizz")
    elif i % 5 == 0: print("Buzz")
    else: print(i)
```

### 12. Flatten a nested list

```python
nested = [[1, 2], [3, 4], [5]]
flat = [x for sub in nested for x in sub]
print(flat)                   # [1, 2, 3, 4, 5]
```

### 13. Merge two dictionaries

```python
d1 = {"a": 1}; d2 = {"b": 2}
print({**d1, **d2})           # {'a': 1, 'b': 2}
# Python 3.9+: d1 | d2
```

---

## 13. Tricky Output / Gotcha Questions

These catch people out — know the *why*.

```python
# 1) Mutable default argument
def add(x, lst=[]):
    lst.append(x)
    return lst
print(add(1))   # [1]
print(add(2))   # [1, 2]  <-- same list reused!
```
*The default list is created once at function definition and shared.*

```python
# 2) is vs ==
a = [1, 2, 3]; b = [1, 2, 3]
print(a == b)   # True  (same values)
print(a is b)   # False (different objects)
```

```python
# 3) Integer interning
x = 256; y = 256
print(x is y)   # True  (cached small int)
x = 257; y = 257
print(x is y)   # False (outside -5..256 cache; may vary)
```

```python
# 4) Late binding in closures
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])   # [2, 2, 2]  (i is looked up at call time)
# Fix: lambda i=i: i
```

```python
# 5) Truthiness
print(bool(0), bool(""), bool([]), bool(None))  # False False False False
print(bool("False"))   # True (non-empty string)
```

```python
# 6) Tuple with one element
print(type((5)))    # <class 'int'>   -- NOT a tuple!
print(type((5,)))   # <class 'tuple'> -- comma makes the tuple
```

```python
# 7) Chained comparison
print(1 < 2 < 3)    # True  (means 1<2 and 2<3)
```

---

## 14. Rapid-Fire One-Liners

- **`range()` is** a lazy iterator (Python 3), not a list.
- **`zip()`** pairs elements from multiple iterables.
- **`enumerate()`** yields `(index, value)` pairs.
- **`*` unpacks** iterables; **`**` unpacks** dicts.
- **`pass`** = do nothing (placeholder); **`continue`** = skip to next loop iteration; **`break`** = exit loop.
- **`None`** is the singleton "no value"; compare with `is None`.
- **`pickle`** serializes Python objects; **`json`** for cross-language text.
- **List slicing** `[start:stop:step]`; negative indices count from the end.
- **`del`** removes a name/element; **`pop()`** removes & returns.
- **`global`** rebinds a module-level name; **`nonlocal`** rebinds an enclosing-function name.
- **Shallow copy:** `obj.copy()` / `obj[:]`; **deep copy:** `copy.deepcopy()`.
- **`@lru_cache`** memoizes function results.
- **`__init__`** initializes; **`__new__`** actually creates the instance.
- **`dict.get(k, default)`** avoids `KeyError`.
- **`set`** ops: `|` union, `&` intersection, `-` difference, `^` symmetric difference.
- **`map`/`filter`/`reduce`** = functional toolkit; comprehensions are usually more Pythonic.
- **`a if cond else b`** = ternary expression.
- **`assert`** checks a condition (debugging); disabled with `-O`.
- **`type()` vs `isinstance()`** — prefer `isinstance()` (respects inheritance).
- **`*args` → tuple**, **`**kwargs` → dict**.

---

## 15. Last-Minute Checklist

Run through these the hour before your interview:

- [ ] Explain interpreted vs compiled, dynamic vs strong typing.
- [ ] List vs tuple vs set vs dict — properties + when to use.
- [ ] Mutable vs immutable; shallow vs deep copy.
- [ ] `*args` / `**kwargs`, default-arg trap, LEGB scope.
- [ ] Generators & `yield` vs `return`; iterable vs iterator.
- [ ] OOP: 4 pillars, `self`, classmethod/staticmethod, MRO, dunder methods.
- [ ] Decorators & closures (be able to write a `@timer`).
- [ ] try/except/else/finally; custom exceptions.
- [ ] `if __name__ == "__main__"`, `with open(...)`.
- [ ] GIL, threading vs multiprocessing vs asyncio, garbage collection.
- [ ] Be ready to whiteboard: reverse string, palindrome, Fibonacci, prime, anagram, duplicates, FizzBuzz.
- [ ] Know the gotchas: mutable defaults, `is` vs `==`, late-binding closures.

**Interview tips:** Think aloud, state your assumptions, start with a brute-force solution then optimize, mention time/space complexity, and test your code with an edge case (empty input, single element, negatives).

---

*Good luck, Tejas — you've got this. Read it once top-to-bottom, then re-read the bold lines and Section 13.*
