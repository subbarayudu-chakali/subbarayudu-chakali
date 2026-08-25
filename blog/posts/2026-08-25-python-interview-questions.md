# Python Interview Questions & Answers

A practical, interview-ready reference for **Python** — from language fundamentals and
data structures through functions, OOP, iterators, decorators, exceptions, memory
internals, concurrency, packaging, the standard library, and testing/typing. Grouped by
theme, numbered continuously from 1 to 100, with answers concise enough to say aloud and
code snippets where they make the point clearer.

---

## Fundamentals

**1. Is Python interpreted or compiled?**

Both, in a sense. CPython **compiles** your source to **bytecode** (`.pyc`) and then an
interpreter (a virtual machine) executes that bytecode. There's no separate ahead-of-time
machine-code step like C, so it's usually called an interpreted language — but the
compile-to-bytecode stage is real and happens automatically.

**2. What does "dynamically typed" mean in Python?**

Types are attached to **objects**, not to variable names, and are checked at **runtime**.
A name can be rebound to a value of a different type at any time. Contrast with static
typing (Java, C++) where types are fixed and checked at compile time.

```python
x = 5        # int
x = "hello"  # now a str — perfectly legal
```

**3. Is Python strongly or weakly typed?**

**Strongly** typed. It does not silently coerce unrelated types — `"3" + 5` raises a
`TypeError` rather than guessing. Dynamic typing (when types are checked) is orthogonal
to strong typing (how strictly types are enforced).

**4. What is PEP 8?**

The official **style guide** for Python code — naming conventions (`snake_case` for
functions/variables, `CapWords` for classes), 4-space indentation, 79-char line guidance,
import ordering, and whitespace rules. Tools like `flake8`, `black`, and `ruff` help
enforce it.

**5. What is a PEP?**

A **Python Enhancement Proposal** — a design document proposing a new feature, process,
or convention (e.g. PEP 8 for style, PEP 20 for the Zen of Python, PEP 484 for type
hints). It's how the language evolves.

**6. What is the difference between mutable and immutable objects?**

**Mutable** objects can be changed in place after creation (`list`, `dict`, `set`,
`bytearray`). **Immutable** objects cannot (`int`, `float`, `str`, `tuple`, `frozenset`,
`bytes`); "modifying" one creates a new object. Immutability makes objects hashable and
safe to share.

**7. What's the difference between `is` and `==`?**

`==` compares **values** (calls `__eq__`); `is` compares **identity** (whether two names
point to the exact same object in memory). Use `is` only for singletons like `None`,
`True`, `False`.

```python
a = [1, 2]; b = [1, 2]
a == b   # True  (same value)
a is b   # False (different objects)
```

**8. How do variables and references work in Python?**

Variables are **names bound to objects**, not boxes holding values. Assignment binds a
name to an object; multiple names can reference the same object. This is why mutating a
list through one name is visible through another name pointing at it.

**9. Is Python pass-by-value or pass-by-reference?**

Neither exactly — it's **pass-by-object-reference** (call by sharing). The function
receives a reference to the same object. Mutating a mutable argument in place affects the
caller; rebinding the parameter name does not.

**10. What does the Zen of Python emphasize?**

Readability and simplicity — "Explicit is better than implicit," "Simple is better than
complex," "Readability counts," "There should be one obvious way to do it." Run
`import this` to see it.

**11. What is truthiness in Python?**

Any object can be evaluated in a boolean context. **Falsy** values include `None`, `False`,
`0`, `0.0`, `""`, `[]`, `{}`, `set()`, and `range(0)`; almost everything else is truthy.
Custom classes control this via `__bool__` (or `__len__`).

**12. What is the difference between `/` and `//`?**

`/` is **true division** and always returns a `float` (`7 / 2 == 3.5`). `//` is **floor
division**, returning the largest integer not greater than the result (`7 // 2 == 3`,
`-7 // 2 == -4`).

---

## Data types & structures

**13. What's the difference between a list, tuple, set, and dict?**

- **list** — ordered, mutable, allows duplicates, indexed by position.
- **tuple** — ordered, **immutable**, allows duplicates; hashable if its contents are.
- **set** — unordered, mutable, **unique** elements, fast membership tests.
- **dict** — key-value mapping, keys unique and hashable, insertion-ordered (3.7+).

**14. When would you use a tuple over a list?**

When the collection is **fixed** and shouldn't change (coordinates, records, function
returns), when you need a **hashable** key for a dict/set, or as a lightweight,
memory-efficient, and slightly faster fixed sequence.

**15. How is a Python dict implemented internally?**

As a **hash table**. Keys are hashed to find a slot; collisions are resolved by open
addressing. Since 3.6/3.7 dicts preserve **insertion order** and use a compact
two-array layout (an index array plus a dense entries array) that saves memory. Average
lookup/insert is O(1).

**16. Why must dict keys be hashable?**

The dict computes `hash(key)` to locate the bucket. Hashable objects have a stable
`__hash__` and support `__eq__`. Mutable containers like lists are unhashable because
their hash could change, breaking lookups.

**17. What is a list comprehension?**

A concise expression that builds a list from an iterable, optionally filtered.

```python
squares = [n * n for n in range(10) if n % 2 == 0]
```

They're more readable and often faster than an equivalent `for` loop with `.append()`.

**18. What are dict and set comprehensions?**

Same syntax, different braces.

```python
sq = {n: n * n for n in range(5)}     # dict comprehension
evens = {n for n in range(10) if n % 2 == 0}  # set comprehension
```

**19. What is a generator expression and how does it differ from a list comprehension?**

A generator expression uses parentheses and produces values **lazily**, one at a time,
without building the whole list in memory.

```python
total = sum(n * n for n in range(1_000_000))  # no giant list created
```

Use it for large or streaming data; use a list comprehension when you need the full list.

**20. How does slicing work?**

`seq[start:stop:step]` returns a new sub-sequence; `start` is inclusive, `stop` exclusive.
Negative indices count from the end, negative `step` reverses.

```python
s = "abcdef"
s[1:4]    # 'bcd'
s[::-1]   # 'fedcba' (reverse)
s[::2]    # 'ace'
```

**21. Why are Python strings immutable, and what are the implications?**

Immutability makes strings hashable (usable as dict keys), thread-safe to share, and
allows interning/caching. The implication: any "modification" creates a new string, so
building a big string with `+=` in a loop is O(n²) — use `"".join(list_of_parts)` instead.

**22. Name some common string methods.**

`.strip()/.lstrip()/.rstrip()`, `.split()/.rsplit()`, `.join()`, `.replace()`,
`.lower()/.upper()/.title()`, `.startswith()/.endswith()`, `.find()/.index()`,
`.format()`, and predicates like `.isdigit()/.isalpha()`.

**23. What are f-strings and why prefer them?**

Formatted string literals (PEP 498, 3.6+) that embed expressions inline. They're concise,
fast, and readable, and support format specs and (3.8+) self-documenting `=`.

```python
name, n = "Ada", 3
f"{name} has {n} items, doubled = {n * 2}, {n=}"
```

**24. What's the difference between `append()` and `extend()`?**

`append(x)` adds `x` as a **single element**; `extend(iterable)` adds **each element** of
the iterable.

```python
a = [1, 2]; a.append([3, 4])  # [1, 2, [3, 4]]
b = [1, 2]; b.extend([3, 4])  # [1, 2, 3, 4]
```

**25. How do you copy a list, and what's a shallow vs deep copy?**

`list.copy()`, `list[:]`, or `copy.copy()` make a **shallow** copy — nested objects are
shared. `copy.deepcopy()` recursively copies everything, so nested mutables are
independent.

**26. What does the `in` operator do, and its complexity?**

Membership test. For `list`/`tuple` it's **O(n)** (linear scan); for `set`/`dict` it's
**O(1)** average (hash lookup). Prefer sets/dicts for frequent membership checks.

**27. What's the difference between `sort()` and `sorted()`?**

`list.sort()` sorts **in place** and returns `None`; `sorted(iterable)` returns a **new
sorted list** and works on any iterable. Both accept `key=` and `reverse=`.

**28. What is tuple unpacking?**

Assigning the elements of a sequence to multiple names at once, including starred capture.

```python
a, b, *rest = [1, 2, 3, 4]   # a=1, b=2, rest=[3, 4]
x, y = y, x                  # swap without a temp
```

---

## Functions

**29. What are positional and keyword arguments?**

**Positional** args are matched by order; **keyword** args are matched by name
(`func(x=1)`). Keyword args improve readability and let you skip earlier optional
parameters.

**30. What are `*args` and `**kwargs`?**

`*args` collects extra **positional** arguments into a tuple; `**kwargs` collects extra
**keyword** arguments into a dict. They let functions accept a variable number of
arguments.

```python
def f(*args, **kwargs):
    print(args, kwargs)
f(1, 2, a=3)   # (1, 2) {'a': 3}
```

**31. What is the mutable default argument pitfall?**

Default values are evaluated **once** at function definition, so a mutable default is
shared across calls.

```python
def bad(x, items=[]):      # BUG: same list reused
    items.append(x); return items

def good(x, items=None):   # fix
    if items is None: items = []
    items.append(x); return items
```

**32. What does "functions are first-class objects" mean?**

Functions can be assigned to variables, passed as arguments, returned from other
functions, and stored in data structures — just like any other object. This enables
callbacks, higher-order functions, and decorators.

**33. What is a closure?**

A nested function that **captures and remembers** variables from its enclosing scope even
after that scope has finished executing.

```python
def multiplier(factor):
    def multiply(n):
        return n * factor   # captures `factor`
    return multiply
double = multiplier(2); double(5)  # 10
```

**34. What is a lambda, and when should you use one?**

An anonymous, single-expression function. Use it for small throwaway callables, typically
as a `key=` or with `map`/`filter`/`sorted`. For anything non-trivial, use `def` for
readability.

```python
sorted(words, key=lambda w: len(w))
```

**35. Explain the LEGB scope rule.**

Name lookup order: **Local → Enclosing → Global → Built-in**. Python searches the current
function first, then any enclosing functions, then the module (global) namespace, then
built-ins.

**36. What do `global` and `nonlocal` do?**

`global` lets a function rebind a **module-level** name. `nonlocal` lets a nested function
rebind a name in the **nearest enclosing** function (not global). Without them, assignment
creates a new local name.

**37. What's the difference between arguments and parameters?**

**Parameters** are the names in the function definition; **arguments** are the actual
values passed at the call site.

**38. What are positional-only and keyword-only parameters?**

Using `/` marks preceding params **positional-only**; using `*` marks following params
**keyword-only**.

```python
def f(a, b, /, c, *, d):
    ...   # a,b positional-only; d keyword-only
```

**39. What does a function return if there's no `return` statement?**

`None`. Every function returns something; absent an explicit `return`, the implicit result
is `None`.

**40. What is a higher-order function?**

A function that takes another function as an argument and/or returns a function. Examples:
`map`, `filter`, `sorted` (via `key`), and decorators.

---

## OOP

**41. What is `self` and why is it explicit?**

`self` is the reference to the **current instance**, passed automatically as the first
argument to instance methods. Python makes it explicit for clarity — "explicit is better
than implicit" — so you always see which object a method operates on.

**42. What does `__init__` do? Is it a constructor?**

`__init__` is the **initializer**: it sets up instance attributes on an already-created
object. The actual object creation happens in `__new__`. In everyday terms `__init__`
plays the constructor role.

**43. How does inheritance work, and what is MRO?**

A subclass inherits attributes/methods from its base class(es). With multiple inheritance,
the **Method Resolution Order (MRO)** — computed by the **C3 linearization** algorithm —
determines the order Python searches classes. Inspect it via `ClassName.__mro__`.

**44. What does `super()` do?**

Returns a proxy that delegates method calls to the **next class in the MRO**, letting you
extend rather than replace parent behavior and cooperate correctly in multiple
inheritance.

```python
class B(A):
    def __init__(self, x):
        super().__init__(x)
        ...
```

**45. What are dunder (magic) methods?**

Double-underscore methods that hook into language syntax: `__init__`, `__str__`,
`__repr__`, `__len__`, `__eq__`, `__add__`, `__getitem__`, `__iter__`, etc. They let your
objects behave like built-in types.

**46. Difference between `__str__` and `__repr__`?**

`__str__` is the **human-readable** form (used by `print`/`str`); `__repr__` is the
**unambiguous, developer** form (used in the REPL and by `repr`), ideally reproducing the
object. If you define only one, define `__repr__`.

**47. Difference between `@classmethod` and `@staticmethod`?**

A `@classmethod` receives the class as `cls` (great for alternative constructors/factories).
A `@staticmethod` receives neither `self` nor `cls` — it's a plain function namespaced on
the class.

```python
class Pizza:
    @classmethod
    def margherita(cls): return cls(["tomato", "cheese"])
    @staticmethod
    def oven_temp(): return 220
```

**48. What is a property?**

The `@property` decorator turns a method into a **managed attribute**, letting you add
getter/setter/deleter logic while keeping attribute-style access. It's the Pythonic
alternative to Java-style getX/setX.

```python
class Circle:
    def __init__(self, r): self._r = r
    @property
    def area(self): return 3.14159 * self._r ** 2
```

**49. What is a dataclass?**

A class decorated with `@dataclass` (PEP 557, 3.7+) that auto-generates `__init__`,
`__repr__`, `__eq__`, and more from annotated fields — great for data-holding classes with
much less boilerplate.

```python
from dataclasses import dataclass
@dataclass
class Point:
    x: int
    y: int = 0
```

**50. What is `__slots__` and why use it?**

`__slots__` declares a fixed set of instance attributes, preventing creation of a
per-instance `__dict__`. This **saves memory** (valuable for many small objects) and
speeds attribute access, at the cost of dynamic attribute assignment.

**51. What is duck typing?**

"If it walks like a duck and quacks like a duck, it's a duck." Python cares about whether
an object **supports the needed behavior** (methods/attributes), not its declared type.
You write to interfaces implicitly.

**52. What are abstract base classes (ABCs)?**

Classes (via the `abc` module) that define an interface with `@abstractmethod`s that
subclasses **must implement**. They can't be instantiated directly and formalize
"is-a" contracts.

```python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): ...
```

**53. What's the difference between class attributes and instance attributes?**

**Class** attributes are shared by all instances (defined in the class body); **instance**
attributes are per-object (usually set in `__init__` via `self`). Beware: a mutable class
attribute is shared state across instances.

**54. What are public, protected, and private conventions?**

Python has no true access control. Convention: a leading underscore `_x` means
"internal/protected"; a double leading underscore `__x` triggers **name mangling** to
`_ClassName__x` to avoid subclass clashes. Nothing is truly private.

**55. How do you implement operator overloading?**

Define the relevant dunder methods: `__add__` for `+`, `__eq__` for `==`, `__lt__` for
`<`, `__getitem__` for indexing, etc. Python dispatches operators to these methods.

**56. What is composition and why favor it over inheritance?**

Composition builds objects by **containing** other objects ("has-a") rather than inheriting
("is-a"). It's more flexible, avoids deep fragile hierarchies, and reduces coupling —
hence "favor composition over inheritance."

---

## Iterators & generators

**57. What is the iterator protocol?**

An **iterable** implements `__iter__()` returning an iterator; an **iterator** implements
`__next__()` returning the next item and raising `StopIteration` when exhausted. `for`
loops use this protocol under the hood.

**58. Difference between an iterable and an iterator?**

An **iterable** can produce an iterator (via `iter()`) and can be looped over repeatedly
(e.g. a list). An **iterator** is single-use and stateful — it produces values via
`next()` and is exhausted once consumed.

**59. What is a generator?**

A function that uses `yield` to produce a **lazy sequence** of values, suspending and
resuming its state between calls. It returns an iterator automatically and computes values
on demand.

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1
```

**60. What does `yield` do?**

It **pauses** the function, returns a value to the caller, and preserves local state so
execution resumes right after the `yield` on the next `next()` call. This is what makes
generators lazy and memory-efficient.

**61. What is lazy evaluation and why does it matter?**

Values are computed **only when needed** rather than all up front. This lets you process
huge or infinite streams with constant memory and can short-circuit work you never
actually consume.

**62. What is `yield from`?**

Delegates iteration to a sub-iterator/sub-generator, yielding all its values (and
propagating sends/returns). It flattens nested generator plumbing.

```python
def chain(a, b):
    yield from a
    yield from b
```

**63. What is the `itertools` module good for?**

Building fast, memory-efficient iterator pipelines: `count`, `cycle`, `repeat` (infinite);
`chain`, `islice`, `groupby`, `tee`; and combinatorics like `product`, `permutations`,
`combinations`, plus `accumulate`.

**64. Can you iterate a generator twice?**

No — generators are **exhausted** after one pass. Re-create the generator, or materialize
its output into a list if you need to iterate multiple times.

---

## Decorators & context managers

**65. What is a decorator and how does it work?**

A callable that **takes a function and returns a new function**, typically wrapping it to
add behavior (logging, timing, caching, auth) without modifying the original. `@deco` is
sugar for `func = deco(func)`.

```python
def timer(fn):
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        print(time.perf_counter() - start)
        return result
    return wrapper
```

**66. Why use `functools.wraps` in a decorator?**

Without it, the wrapper replaces the original function's metadata (`__name__`, `__doc__`,
signature). `@functools.wraps(fn)` copies that metadata onto the wrapper, preserving
introspection and debuggability.

**67. How do you write a decorator that takes arguments?**

Add another layer: a decorator factory that returns the actual decorator.

```python
def repeat(n):
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*a, **k):
            for _ in range(n): result = fn(*a, **k)
            return result
        return wrapper
    return deco
```

**68. What is a context manager?**

An object that defines runtime setup/teardown for a `with` block via `__enter__` and
`__exit__`. It guarantees cleanup (closing files, releasing locks) even if an exception
occurs.

**69. How does the `with` statement work?**

It calls `__enter__()` (binding its return to the `as` name), runs the body, then always
calls `__exit__(exc_type, exc_val, tb)` — even on exception. Returning `True` from
`__exit__` suppresses the exception.

```python
with open("f.txt") as f:
    data = f.read()   # file closed automatically after the block
```

**70. What is `contextlib.contextmanager`?**

A decorator to build a context manager from a generator: code before `yield` is setup,
code after (typically in a `finally`) is teardown.

```python
from contextlib import contextmanager
@contextmanager
def tag(name):
    print(f"<{name}>")
    yield
    print(f"</{name}>")
```

**71. Give real uses for decorators.**

Logging/timing, caching (`functools.lru_cache`), access control/authentication, retry
logic, input validation, registering plugins/routes (e.g. Flask's `@app.route`), and
converting methods (`@property`, `@classmethod`, `@staticmethod`).

---

## Exceptions

**72. Explain `try`/`except`/`else`/`finally`.**

`try` wraps risky code; `except` handles specific exceptions; `else` runs only if **no**
exception was raised; `finally` **always** runs (cleanup), whether or not an exception
occurred.

```python
try:
    x = risky()
except ValueError as e:
    handle(e)
else:
    use(x)          # only if no exception
finally:
    cleanup()       # always
```

**73. How do you create a custom exception?**

Subclass `Exception` (or a more specific built-in). Add attributes if you need to carry
context.

```python
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        super().__init__(f"Need {amount}, have {balance}")
        self.balance, self.amount = balance, amount
```

**74. What is EAFP vs LBYL?**

**EAFP** ("Easier to Ask Forgiveness than Permission") — try the operation and catch the
exception; the Pythonic default. **LBYL** ("Look Before You Leap") — check conditions
first. EAFP avoids race conditions and redundant checks.

```python
# EAFP
try: value = d["key"]
except KeyError: value = default
```

**75. What's the difference between `raise` and `raise from`?**

`raise` re-raises or raises an exception. `raise NewError from original` sets
`__cause__`, **explicitly chaining** exceptions so the traceback shows the root cause —
clearer than implicit chaining.

**76. Why avoid a bare `except:`?**

It catches **everything**, including `KeyboardInterrupt` and `SystemExit`, hiding bugs and
making the program hard to stop or debug. Catch specific exceptions (or `except Exception`
at worst).

**77. What's the difference between `Exception` and `BaseException`?**

`BaseException` is the root of all exceptions; `Exception` is its subclass for "ordinary"
errors. System-exiting exceptions (`SystemExit`, `KeyboardInterrupt`, `GeneratorExit`)
derive from `BaseException` but **not** `Exception`, so `except Exception` won't swallow
them.

**78. What does `finally` do if the `try` block returns?**

`finally` still runs before the function actually returns. If `finally` itself returns a
value, it **overrides** the earlier return — a common gotcha to avoid.

---

## Memory & internals

**79. What is CPython?**

The **reference implementation** of Python, written in C. It's what you get from
python.org. Others include PyPy (JIT-compiled), Jython (JVM), and IronPython (.NET).

**80. How does Python manage memory?**

Primarily via **reference counting**: each object tracks how many references point to it,
and is freed when the count hits zero. A supplementary **cyclic garbage collector**
handles reference cycles that counting alone can't reclaim.

**81. What is reference counting and its main limitation?**

Every object keeps a count of references to it; reaching zero triggers immediate
deallocation. Its limitation is **reference cycles** (A references B, B references A) —
the counts never reach zero, so the cyclic GC is needed to detect and collect them.

**82. How does Python's garbage collector work?**

The `gc` module runs a **generational**, cycle-detecting collector. Objects are grouped
into three generations; younger generations are collected more frequently (most objects
die young). It detects unreachable cycles and frees them. You can tune or disable it via
`gc`.

**83. What is the GIL?**

The **Global Interpreter Lock** — a mutex in CPython that lets only **one thread execute
Python bytecode at a time**. It simplifies memory management (protects reference counts)
but prevents true parallelism of CPU-bound Python threads.

**84. What are the practical implications of the GIL?**

CPU-bound multithreading gets **no speedup** (often slower) in CPython — use
`multiprocessing` or native extensions that release the GIL. I/O-bound threads still help,
because the GIL is released during blocking I/O. Note: recent CPython offers an
experimental free-threaded (no-GIL) build.

**85. What is interning?**

CPython **caches** and reuses certain immutable objects — small integers (−5 to 256) and
some short strings — so identical literals share one object. That's why `a is b` can be
`True` for small ints but not for large ones.

**86. Why is `"".join(list)` preferred over `+=` in a loop for strings?**

Strings are immutable, so `s += part` in a loop creates a new string each iteration —
O(n²) total. `"".join(parts)` allocates once and is O(n).

---

## Concurrency

**87. Threading vs multiprocessing vs asyncio — when to use each?**

- **threading** — best for **I/O-bound** concurrency; light, shared memory, but limited by
  the GIL for CPU work.
- **multiprocessing** — best for **CPU-bound** work; separate processes sidestep the GIL
  for true parallelism, at the cost of IPC overhead.
- **asyncio** — best for **high-concurrency I/O** (thousands of connections) using a single
  thread and cooperative scheduling.

**88. Why doesn't threading speed up CPU-bound code in CPython?**

Because the **GIL** allows only one thread to run Python bytecode at a time. CPU-bound
threads contend for the GIL and run effectively serially (plus locking overhead). Use
processes instead.

**89. What is `asyncio` and how does `async`/`await` work?**

`asyncio` is a single-threaded **cooperative concurrency** framework built on an event
loop. `async def` defines a coroutine; `await` suspends it at I/O points, yielding control
to the event loop to run other coroutines until the awaited result is ready.

```python
async def fetch(session, url):
    async with session.get(url) as resp:
        return await resp.text()
```

**90. What is the event loop?**

The core of asyncio — it schedules and runs coroutines/tasks, resuming each one when its
awaited operation completes. It runs on a single thread, interleaving many I/O-bound tasks
without OS-thread overhead.

**91. What's the difference between concurrency and parallelism?**

**Concurrency** is dealing with many tasks by interleaving progress (can be single-core).
**Parallelism** is executing multiple tasks **simultaneously** on multiple cores. asyncio
and threading give concurrency; multiprocessing gives parallelism.

**92. What is a race condition, and how do you prevent it?**

When the outcome depends on the timing of concurrent access to shared state. Prevent it
with synchronization primitives (`threading.Lock`, `RLock`, `Semaphore`), immutable/local
data, or thread-safe structures like `queue.Queue`.

---

## Modules, packaging & environments

**93. What's the difference between a module and a package?**

A **module** is a single `.py` file. A **package** is a directory of modules (historically
containing `__init__.py`), enabling dotted imports like `import mypkg.sub.mod`.

**94. What does `if __name__ == "__main__":` do?**

`__name__` is `"__main__"` when a file is **run directly**, and the module's name when
**imported**. The guard lets code run as a script yet be safely importable without
executing that block.

```python
if __name__ == "__main__":
    main()
```

**95. What's the difference between `import x` and `from x import y`?**

`import x` binds the module name, accessed as `x.y`. `from x import y` binds `y` directly
into your namespace. Avoid `from x import *` — it pollutes the namespace and obscures
origins.

**96. What are `venv`/virtual environments and why use them?**

An isolated per-project Python environment with its own installed packages, so projects
don't clash over dependency versions.

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**97. What is `requirements.txt` vs `pyproject.toml`?**

`requirements.txt` pins a flat list of dependencies (`pip install -r`). `pyproject.toml`
(PEP 518/621) is the modern, standardized project config declaring metadata, dependencies,
and build system — used by pip, build, poetry, hatch, etc.

**98. What is a wheel?**

A **built distribution** format (`.whl`) — a pre-built, zipped package that installs fast
without a compile/build step, unlike an sdist (source distribution) which must be built on
install.

---

## Standard library, testing & typing

**99. What are useful types in `collections`?**

`defaultdict` (auto-default values), `Counter` (counting/tallies), `deque` (O(1) appends
and pops at both ends), `namedtuple` (lightweight immutable records), `OrderedDict`, and
`ChainMap`.

```python
from collections import Counter
Counter("mississippi").most_common(1)  # [('i', 4)]
```

**100. What are type hints and how are they checked?**

Optional annotations (PEP 484) declaring expected types. They don't affect runtime
behavior — Python doesn't enforce them — but a **static checker** like `mypy` or
`pyright` verifies them, and editors use them for autocomplete. Testing frameworks
(`pytest`, `unittest`) plus mocking (`unittest.mock`) round out the quality toolkit.

```python
def greet(name: str, times: int = 1) -> str:
    return f"Hi {name}! " * times
```

---

## Quick-fire round

- **Swap two variables?** `a, b = b, a`.
- **Merge two dicts (3.9+)?** `d1 | d2`.
- **Reverse a string?** `s[::-1]`.
- **Remove duplicates, keep order?** `list(dict.fromkeys(seq))`.
- **Count items fast?** `collections.Counter`.
- **Flatten one level?** `itertools.chain.from_iterable(nested)`.
- **Ternary syntax?** `x if cond else y`.
- **Check type?** `isinstance(obj, T)`, not `type(obj) == T`.
- **Read a file safely?** `with open(path) as f:`.
- **Memoize a pure function?** `@functools.lru_cache`.
- **Get env var with default?** `os.environ.get("KEY", default)`.
- **Pretty-print JSON?** `json.dumps(obj, indent=2)`.

---

These questions cover most Python interviews end to end — from dynamic typing and data
structures through decorators, the GIL, concurrency trade-offs, packaging, and typing. The
best follow-up prep is to write real code: build a small package with a virtual
environment and `pyproject.toml`, add type hints and pytest tests, profile a hot loop, and
try the same workload with threads, processes, and asyncio to feel the GIL first-hand.
Doing it once makes every answer above concrete.
