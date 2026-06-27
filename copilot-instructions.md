# GitHub Copilot Instructions — Fluent Python Principles

These instructions guide Copilot toward idiomatic, Pythonic code. Apply them across all Python files in this
repository.

---

## 1. Data Model — Embrace Dunder Methods

- Implement special methods (`__repr__`, `__str__`, `__len__`, `__getitem__`,
  `__contains__`, etc.) to make custom classes feel like first-class Python
  objects.
- Prefer `__repr__` for unambiguous, developer-facing representations; use
  `__str__` for user-facing output.
- Use `__bool__` to define truthiness; avoid overriding `__len__` for boolean
  purposes.
- Implement `__eq__` together with `__hash__` (or explicitly set `__hash__ = None`
  for mutable types).

```python
# Good
class Vector:
    def __init__(self, x, y): self.x, self.y = x, y
    def __repr__(self): return f"Vector({self.x!r}, {self.y!r})"
    def __abs__(self): return (self.x**2 + self.y**2) ** 0.5
    def __bool__(self): return bool(abs(self))
    def __add__(self, other): return Vector(self.x + other.x, self.y + other.y)
```

---

## 2. Sequences — Use Built-in Protocols

- Prefer list comprehensions and generator expressions over explicit loops.
- Use slicing (`a[start:stop:step]`); implement `__getitem__` and `__len__` to
  make custom classes sliceable.
- Use `collections.namedtuple` or `typing.NamedTuple` for lightweight immutable
  records.
- Prefer `tuple` for heterogeneous, positional data; `list` for homogeneous,
  mutable sequences.
- Use `array.array` or `memoryview` for numeric data that requires memory
  efficiency.

```python
# Good — generator expression, not a list built just to be consumed
total = sum(x**2 for x in range(1000))

# Good — named tuple as a lightweight record
from typing import NamedTuple
class Coordinate(NamedTuple):
    lat: float
    lon: float
    label: str = ""
```

---

## 3. Dictionaries and Sets

- Use `dict` comprehensions: `{k: v for k, v in pairs}`.
- Prefer `collections.defaultdict` or `dict.setdefault` over manual
  key-existence checks.
- Use `collections.Counter` for tallying; `collections.OrderedDict` only when
  insertion-order contract must be explicit (standard `dict` preserves order
  in Python 3.7+).
- Use `collections.ChainMap` to layer multiple mappings.
- Exploit set operations (`|`, `&`, `-`, `^`) for membership logic.

```python
# Good
from collections import defaultdict
index = defaultdict(list)
for word, page in word_page_pairs:
    index[word].append(page)

# Good — set comprehension
seen = {normalize(w) for w in raw_words}
```

---

## 4. Text vs. Bytes

- Work exclusively with `str` internally; encode/decode at the boundaries
  (I/O, network, files).
- Always specify encoding explicitly: `open(path, encoding="utf-8")`.
- Use `bytes` / `bytearray` for binary data; never mix with `str`.
- Normalize Unicode strings with `unicodedata.normalize("NFC", s)` before
  comparison or storage.

```python
# Good
with open("data.txt", encoding="utf-8") as f:
    text = f.read()
```

---

## 5. First-Class Functions

- Treat functions as objects: pass them, store them, return them.
- Use `functools.partial` for partial application instead of writing thin
  wrapper lambdas.
- Use `functools.reduce` sparingly; prefer `sum`, `max`, `min`, `any`, `all`.
- Prefer `operator.itemgetter`, `operator.attrgetter`, and `operator.methodcaller`
  over equivalent lambdas for readability.

```python
import operator, functools

# Good
sorted_records = sorted(records, key=operator.attrgetter("date", "name"))

# Good — partial instead of lambda
triple = functools.partial(operator.mul, 3)
```

---

## 6. Design Patterns with First-Class Functions

- Replace Strategy, Command, and Template Method patterns with plain functions
  or `functools` where appropriate — Python's first-class functions make
  heavyweight class hierarchies unnecessary.
- Use closures to carry state instead of single-method classes when the
  interface is simple.

```python
# Good — closure replaces a one-method class
def make_averager():
    data = []
    def averager(value):
        data.append(value)
        return sum(data) / len(data)
    return averager
```

---

## 7. Decorators and Closures

- Use `@functools.wraps(func)` inside every decorator to preserve metadata.
- Keep decorators simple and single-purpose; chain them rather than building
  monolithic ones.
- Use `nonlocal` to rebind variables in enclosing scopes; avoid mutable default
  arguments as state.
- Prefer class-based decorators (with `__call__`) when the decorator needs
  configurable parameters or complex state.

```python
import functools

def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

---

## 8. Object References, Mutability, and Garbage Collection

- Distinguish between equality (`==`, `__eq__`) and identity (`is`, `id()`).
- Never use mutable objects as default argument values — use `None` as sentinel
  and create inside the function.
- Use `copy.copy` for shallow copies and `copy.deepcopy` for independent deep
  copies.
- Prefer immutable objects (tuples, frozensets, strings) for shared state.

```python
# Bad
def append_to(elem, to=[]):   # mutable default — shared across calls
    to.append(elem)
    return to

# Good
def append_to(elem, to=None):
    if to is None:
        to = []
    to.append(elem)
    return to
```

---

## 9. A Pythonic Object

- Implement `__slots__` on classes with many instances to reduce memory.
- Override `__repr__` to return a string that could reconstruct the object.
- Use `@classmethod` for alternative constructors; `@staticmethod` sparingly.
- Use `@property` for computed attributes; avoid getters/setters unless
  validation or side effects are truly needed.

```python
class Circle:
    __slots__ = ("_radius",)

    def __init__(self, radius): self._radius = float(radius)

    @property
    def radius(self): return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0: raise ValueError("Radius must be non-negative")
        self._radius = float(value)

    @classmethod
    def from_diameter(cls, d): return cls(d / 2)
```

---

## 10. Sequence Hacking, Hashing, and Slicing

- Delegate to sequences (list, tuple) inside custom sequence types rather than
  reimplementing iteration from scratch.
- Implement `__iter__` by returning `iter(self._data)` when wrapping a
  collection; define a separate `__reversed__` if reverse iteration is expected.
- Hash only immutable objects; combine component hashes with `functools.reduce`
  and `operator.xor` or use `hash(tuple(self))`.

---

## 11. Interfaces, Protocols, and ABCs

- Prefer duck typing and structural subtyping (protocols) over explicit
  inheritance from ABCs for flexibility.
- Register virtual subclasses with `ABC.register()` when you can't modify the
  third-party class.
- Use `collections.abc` ABCs (`Mapping`, `MutableMapping`, `Sequence`,
  `Iterable`, etc.) as type hints and for `isinstance` checks — not as base
  classes unless you need inherited mix-in methods.
- Use `typing.Protocol` for static structural typing without runtime coupling.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
```

---

## 12. Inheritance — Use It Right

- Favor composition over inheritance; reserve inheritance for genuine
  is-a relationships.
- Never inherit from multiple concrete classes; mix in from ABCs only.
- Always call `super()` without arguments in cooperative multiple inheritance.
- Understand MRO (`__mro__`); use `super()` to navigate it correctly.

```python
# Good — cooperative super() usage
class LoggedMixin:
    def save(self):
        print(f"Saving {self}")
        super().save()
```

---

## 13. Operator Overloading

- Implement `__add__` and return `NotImplemented` (not `NotImplementedError`)
  for unsupported types to allow Python to try the reflected method on the
  other operand.
- Always return a new instance from arithmetic operators — never mutate `self`.
- Implement augmented assignment (`__iadd__`) only when in-place mutation is
  semantically correct.

```python
def __add__(self, other):
    if not isinstance(other, type(self)):
        return NotImplemented
    return type(self)(self.x + other.x, self.y + other.y)
```

---

## 14. Iterables, Iterators, and Generators

- Prefer generator functions (`yield`) over building and returning lists when
  the caller may not need all values.
- Make container classes iterable via `__iter__`; make iterator classes via
  both `__iter__` (returning `self`) and `__next__`.
- Use `itertools` — `chain`, `islice`, `groupby`, `product`, `combinations`,
  `permutations` — before writing manual loops.
- Use `yield from` to delegate to sub-generators cleanly.

```python
import itertools

# Good — generator instead of list
def read_chunks(file, size=8192):
    while chunk := file.read(size):
        yield chunk

# Good — yield from
def flatten(items):
    for item in items:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item
```

---

## 15. Context Managers and `with` Blocks

- Implement `__enter__` / `__exit__` for resource management; use
  `contextlib.contextmanager` for generator-based context managers.
- Use `contextlib.suppress` to silence expected exceptions cleanly.
- Use `contextlib.ExitStack` to manage a dynamic number of context managers.

```python
from contextlib import contextmanager

@contextmanager
def managed_resource(name):
    resource = acquire(name)
    try:
        yield resource
    finally:
        release(resource)
```

---

## 16. Coroutines and `async`/`await`

- Use `async def` / `await` for I/O-bound concurrency; use `asyncio`.
- Avoid mixing blocking calls inside `async def`; offload to
  `asyncio.to_thread` or an executor.
- Use `async for` and `async with` when working with async iterables and
  context managers.
- Use `asyncio.gather` to run independent coroutines concurrently.

```python
import asyncio

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)
```

---

## 17. Type Hints (Modern Pythonic Style)

- Annotate function signatures; leave obvious local variables unannotated.
- Use `X | Y` union syntax (Python 3.10+) over `Union[X, Y]`.
- Use `collections.abc` types (`Iterable`, `Sequence`, `Mapping`) in
  signatures, not concrete types (`list`, `dict`), to accept broader inputs.
- Use `TypeVar` and `Generic` for reusable generic utilities.
- Run `mypy` or `pyright` in strict mode; treat type errors as bugs.

```python
from collections.abc import Iterable

def process(items: Iterable[int]) -> list[int]:
    return [x * 2 for x in items]
```

---

## General Pythonic Heuristics

- **Explicit over implicit** — name things clearly; avoid `*` imports.
- **Flat over nested** — early returns, guard clauses, and comprehensions
  reduce nesting.
- **Ask forgiveness, not permission** — prefer `try/except` over `if/else`
  for control flow around exceptions.
- **Don't repeat yourself** — extract repeated patterns into functions,
  decorators, or descriptors.
- **Standard library first** — exhaust `itertools`, `functools`,
  `collections`, `contextlib`, and `operator` before reaching for third-party
  libraries or hand-rolled solutions.
