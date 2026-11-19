# Python OOP: Interview Notes (including the diamond problem and MRO)

## 1. Classes and objects
```python
class Account:
    bank = "ABC"                      # class attribute (shared)
    def __init__(self, owner, balance=0):
        self.owner = owner            # instance attribute
        self._balance = balance       # "protected" by convention
        self.__pin = 1234             # name-mangled to _Account__pin
    def deposit(self, amt):
        if amt <= 0: raise ValueError("amount must be positive")
        self._balance += amt
    @property
    def balance(self): return self._balance
    @balance.setter
    def balance(self, v):
        if v < 0: raise ValueError
        self._balance = v
    @classmethod
    def from_string(cls, s): o, b = s.split(","); return cls(o, int(b))
    @staticmethod
    def is_valid(amount): return amount > 0
    def __repr__(self): return f"Account({self.owner!r}, {self._balance})"
```
- **Instance attribute lookup:** instance `__dict__` -> class -> base classes (MRO). Class attributes are shared; assigning on an instance shadows the class attribute.
- **Encapsulation in Python is by convention:** `_x` (internal), `__x` (name mangling to avoid subclass clashes, not true privacy), properties for controlled access.

## 2. The four pillars in Python
| Pillar | Python way |
|---|---|
| Encapsulation | `_name`, `@property`, modules |
| Inheritance | `class B(A)`, `super()`, multiple inheritance |
| Polymorphism | duck typing, method overriding, operator overloading |
| Abstraction | `abc.ABC` and `@abstractmethod`, protocols |

**Duck typing:** "if it walks like a duck and quacks like a duck". Any object with the needed method works; no common base class required.

## 3. Inheritance and `super()`
```python
class Animal:
    def __init__(self, name): self.name = name
    def speak(self): raise NotImplementedError

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)        # call the next class in the MRO
        self.breed = breed
    def speak(self): return "Woof"
```
- `super()` does not mean "the parent"; it means "the next class in the MRO". This is what makes cooperative multiple inheritance work.
- **Overriding** replaces a method; **overloading** by signature does not exist in Python (use default args, `*args`, or `functools.singledispatch`).

## 4. Multiple inheritance and the diamond problem
```
        A
       / \
      B   C
       \ /
        D
```
**The problem:** if `D` inherits from both `B` and `C`, which inherits from `A`, then a method defined in `A` (and overridden in `B` and/or `C`) is ambiguous. In languages without a defined rule, `A`'s data may be duplicated or the wrong version called (C++ needs `virtual` inheritance).

**Python's solution: the MRO (Method Resolution Order) computed by C3 linearization.** It guarantees:
1. A class appears before its parents.
2. The order of base classes in the class statement is preserved.
3. Every class appears exactly once.
```python
class A:
    def hello(self): print("A")
class B(A):
    def hello(self): print("B"); super().hello()
class C(A):
    def hello(self): print("C"); super().hello()
class D(B, C):
    def hello(self): print("D"); super().hello()

D().hello()          # D, B, C, A   (A runs once)
print(D.__mro__)     # (D, B, C, A, object)
print(D.mro())
```
Because `super()` follows the MRO, `A.hello` is called exactly once even though both `B` and `C` call `super().hello()`.

### C3 linearization (how to compute it)
`L[D] = D + merge(L[B], L[C], [B, C])`
- `L[B] = [B, A, object]`, `L[C] = [C, A, object]`
- merge takes the first head that does not appear in the tail of any other list:
  - take B (not in other tails) -> list `[A, object]`, `[C, A, object]`, `[C]`
  - A is in the tail of `[C, A, object]`, so skip; take C
  - then A, then object
- Result: `[D, B, C, A, object]`.

**When C3 fails:** inconsistent orderings raise `TypeError: Cannot create a consistent method resolution order`. Example: `class X(A, B)` and `class Y(B, A)`, then `class Z(X, Y)`.

### Cooperative multiple inheritance rules
- Always use `super()` in every class in the hierarchy, and pass `**kwargs` along so `__init__` chains work.
- Do not call `A.method(self)` directly in a diamond; that bypasses the MRO and can call `A` twice.
- Prefer **composition** or **mixins** (small classes that add behaviour and do not define state) over deep multiple inheritance.

```python
class LoggingMixin:
    def save(self, *a, **kw):
        print("saving"); return super().save(*a, **kw)
class Model:
    def save(self): print("db write")
class User(LoggingMixin, Model): ...
User().save()       # saving, db write
```

**Interview answer in one breath:** "The diamond problem is ambiguity when a class inherits the same base through two paths. Python resolves it deterministically with C3 linearization, which gives a single MRO; `super()` walks that MRO so each class is visited once. You can inspect it with `Class.__mro__`."

## 5. Dunder (magic) methods
| Method | Purpose |
|---|---|
| `__init__`, `__new__` | construct and initialise |
| `__repr__`, `__str__` | developer vs user string |
| `__eq__`, `__lt__`, `__hash__` | comparison, hashing (define `__hash__` when you define `__eq__`) |
| `__len__`, `__getitem__`, `__iter__`, `__contains__` | container behaviour |
| `__call__` | make instances callable |
| `__enter__`, `__exit__` | context manager |
| `__add__`, `__mul__`, `__radd__` | operator overloading |
| `__getattr__`, `__setattr__`, `__getattribute__` | attribute access hooks |
| `__del__` | finaliser (do not rely on it) |

`@functools.total_ordering` fills in comparison methods from `__eq__` and one of `__lt__`, etc.

## 6. Abstract classes and protocols
```python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): ...
class Circle(Shape):
    def __init__(self, r): self.r = r
    def area(self): return 3.14159 * self.r ** 2
Shape()        # TypeError: can't instantiate abstract class
```
`typing.Protocol` gives structural typing (static duck typing): any class with the right methods satisfies it.

## 7. Dataclasses, slots, enums
```python
from dataclasses import dataclass, field
@dataclass(frozen=True, order=True)       # immutable and hashable, comparison methods
class Point:
    x: int
    y: int = 0
    tags: list = field(default_factory=list)   # not a mutable default
```
- `__slots__ = ('x', 'y')` saves memory and blocks arbitrary attributes.
- `enum.Enum` for named constants; `IntEnum`, `Flag`, `auto()`.
- `namedtuple` and `NamedTuple`: immutable records.

## 8. Class vs instance vs static: when to use
- **Instance method:** needs `self` state.
- **Class method:** alternative constructors, factories, class-level config.
- **Static method:** utility that belongs logically to the class but uses neither.

## 9. Metaclasses (brief)
A class is an instance of a metaclass (`type` by default). `type(name, bases, dict)` creates classes dynamically. Used by ORMs (Django models, SQLAlchemy declarative), `ABCMeta`, and enums. Prefer class decorators or `__init_subclass__` unless you really need one.

## 10. Descriptors
Objects defining `__get__`, `__set__`, `__delete__`. `property`, methods, `classmethod` and `staticmethod` are descriptors. Data descriptors (with `__set__`) take priority over instance `__dict__`.

## 11. Composition over inheritance
Use inheritance for "is-a" with a stable contract; use composition ("has-a") to combine behaviours without tight coupling. See `OOP/02_solid_principles.md` for SOLID.

## 12. Singleton, factory in Python
- A module is already a singleton.
- `__new__`-based singleton:
```python
class Singleton:
    _inst = None
    def __new__(cls, *a, **kw):
        if cls._inst is None: cls._inst = super().__new__(cls)
        return cls._inst
```
- Factory: a function or classmethod returning instances of different classes by key (a dict of classes works).

## Quick interview Q&A
- **What is the MRO?** The order Python searches classes for an attribute; `Class.__mro__`.
- **What is `super()`?** A proxy to the next class in the MRO, not necessarily the direct parent.
- **Does Python support method overloading?** Not by signature; the last definition wins. Use defaults, `*args` or `singledispatch`.
- **Difference between `__str__` and `__repr__`?** `__str__` is for end users; `__repr__` is unambiguous for developers (ideally `eval`-able).
- **What is name mangling?** `__x` inside a class becomes `_ClassName__x` to avoid accidental overriding in subclasses.
- **Why define `__hash__` with `__eq__`?** Defining `__eq__` alone sets `__hash__` to None, making instances unhashable.
- **Abstract class vs interface?** Python has no interface keyword; use ABCs or Protocols.
- **What is a mixin?** A small class meant to be combined through multiple inheritance to add behaviour.
- **Is Python pass-by-value or by reference?** Pass by object reference (assignment): the function receives a reference to the same object; rebinding a name inside does not affect the caller, mutating the object does.
