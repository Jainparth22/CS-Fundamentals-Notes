# SOLID Principles

> Five design principles that make OOP code maintainable, extensible, and testable. Coined by Robert C. Martin ("Uncle Bob").

---

## 1. S — Single Responsibility Principle (SRP)

> **A class should have only one reason to change.**

```python
# BAD — Order class handles business logic, persistence, AND email
class Order:
    def calculate_total(self): ...
    def save_to_db(self): ...        # persistence concern mixed in
    def send_confirmation_email(self): ...  # email concern mixed in

# GOOD — Each class has one job
class Order:
    def calculate_total(self): ...

class OrderRepository:
    def save(self, order: Order): ...

class OrderNotifier:
    def send_confirmation(self, order: Order): ...
```

**How to identify violation:** "This class handles X AND Y." If you can say AND, it likely violates SRP.

---

## 2. O — Open/Closed Principle (OCP)

> **Software entities should be open for extension but closed for modification.**

Add new behavior by adding new code, not modifying existing code.

```python
# BAD — adding a new discount type requires modifying this function
def apply_discount(order, discount_type):
    if discount_type == "percentage":
        return order.total * 0.9
    elif discount_type == "flat":
        return order.total - 50
    # Every new type = edit this function = risky

# GOOD — extend by adding new classes, never modify existing
from abc import ABC, abstractmethod

class Discount(ABC):
    @abstractmethod
    def apply(self, total: float) -> float: ...

class PercentageDiscount(Discount):
    def __init__(self, pct: float): self.pct = pct
    def apply(self, total: float) -> float: return total * (1 - self.pct)

class FlatDiscount(Discount):
    def __init__(self, amount: float): self.amount = amount
    def apply(self, total: float) -> float: return total - self.amount

class BuyOneGetOne(Discount):   # NEW type — no existing code changed
    def apply(self, total: float) -> float: return total / 2

def apply_discount(order, discount: Discount):  # closed for modification
    return discount.apply(order.total)
```

---

## 3. L — Liskov Substitution Principle (LSP)

> **Subtypes must be substitutable for their base types without altering correctness.**

If S is a subtype of T, you must be able to use S wherever T is expected.

```python
# BAD — Square "is-a" Rectangle but violates LSP
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):   self.width = self.height = w  # changes height too!
    def set_height(self, h):  self.width = self.height = h

def test_rectangle(r: Rectangle):
    r.set_width(5)
    r.set_height(4)
    assert r.area() == 20  # fails for Square! 4*4=16, not 5*4=20

# GOOD — Don't inherit Square from Rectangle; use a Shape base class
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

**LSP checks:**
- Preconditions must not be strengthened in subtype
- Postconditions must not be weakened in subtype
- Invariants must be preserved

---

## 4. I — Interface Segregation Principle (ISP)

> **Clients should not be forced to depend on methods they don't use.**

Split fat interfaces into smaller, specific ones.

```python
# BAD — one huge interface forces all implementors to implement all methods
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...    # robots don't eat!
    @abstractmethod
    def sleep(self): ...  # robots don't sleep!

class RobotWorker(Worker):
    def work(self): print("beep boop")
    def eat(self): raise NotImplementedError   # forced to implement meaningless method
    def sleep(self): raise NotImplementedError

# GOOD — segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class HumanWorker(Workable, Eatable):
    def work(self): print("coding")
    def eat(self): print("eating lunch")

class RobotWorker(Workable):  # only implements what it needs
    def work(self): print("beep boop")
```

---

## 5. D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

```python
# BAD — high-level OrderService depends on low-level MySQLDatabase directly
class MySQLDatabase:
    def save(self, data): ...  # MySQL-specific

class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # hard dependency — can't swap to Postgres!
    def place_order(self, order):
        self.db.save(order)

# GOOD — both depend on the abstraction (Database interface)
class Database(ABC):
    @abstractmethod
    def save(self, data): ...

class MySQLDatabase(Database):
    def save(self, data): ...  # MySQL implementation

class PostgreSQLDatabase(Database):
    def save(self, data): ...  # Postgres implementation

class OrderService:
    def __init__(self, db: Database):   # inject the dependency
        self.db = db                     # depends on abstraction, not concrete class
    def place_order(self, order):
        self.db.save(order)

# Usage:
svc = OrderService(db=MySQLDatabase())    # or PostgreSQLDatabase(), or MockDatabase()
```

**Dependency Injection (DI):** Passing dependencies in through the constructor (constructor injection) or method parameters. Enables testing with mocks.

---

## 6. SOLID Summary

| Principle | Violation Sign | Fix |
|-----------|---------------|-----|
| SRP | "This class does X AND Y" | Split into two classes |
| OCP | "Adding feature X requires editing class Y" | Use strategy/template pattern |
| LSP | Subclass throws `NotImplementedError` or breaks caller | Redesign hierarchy or use composition |
| ISP | Class forced to implement irrelevant methods | Split interfaces |
| DIP | `new ConcreteClass()` inside high-level class | Inject dependency through constructor |

---

## 7. Quick Interview Answers

| Q | A |
|---|---|
| Which principle does `import` of a concrete class violate? | DIP (depend on abstraction, not concrete) |
| Which principle does "God class" violate? | SRP |
| Why is LSP important? | Allows polymorphic code to work correctly with any subtype |
| DI vs IoC? | IoC = Inversion of Control (broader concept); DI is one implementation of IoC |
