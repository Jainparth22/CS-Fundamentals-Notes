# Creational Design Patterns

---

## 1. Singleton

**Intent:** Ensure a class has only one instance and provide a global access point.

```cpp
// Thread-safe Singleton (C++11 — Meyers' Singleton)
class Logger {
public:
    static Logger& getInstance() {
        static Logger instance;  // created once, destroyed on program exit
        return instance;         // thread-safe by C++11 standard
    }
    void log(const string& msg) { cout << "[LOG] " << msg << "\n"; }
private:
    Logger() {}                           // private constructor
    Logger(const Logger&) = delete;       // no copy
    Logger& operator=(const Logger&) = delete; // no assign
};

// Usage
Logger::getInstance().log("Application started");
```

```python
# Thread-safe Singleton using metaclass
import threading

class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._instances:
                cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = None  # init once

    def connect(self, url: str):
        if not self.connection:
            self.connection = url  # simulate connection
        return self.connection
```

**Drawbacks:** Global state makes testing hard. Violates SRP (manages its own lifecycle). Consider Dependency Injection instead.

---

## 2. Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

```python
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

class EmailNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SMSNotification(Notification):
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

class PushNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Push: {message}")

# Factory Method
class NotificationFactory:
    @staticmethod
    def create(channel: str) -> Notification:
        factories = {
            "email": EmailNotification,
            "sms": SMSNotification,
            "push": PushNotification,
        }
        cls = factories.get(channel)
        if not cls:
            raise ValueError(f"Unknown channel: {channel}")
        return cls()

# Usage: caller doesn't know the concrete class
notif = NotificationFactory.create("email")
notif.send("Your order has shipped!")
```

---

## 3. Abstract Factory

**Intent:** Create families of related objects without specifying concrete classes.

```python
# UI toolkit example: Light theme vs Dark theme

class Button(ABC):
    @abstractmethod
    def render(self) -> str: ...

class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str: ...

class LightButton(Button):
    def render(self) -> str: return "[ Light Button ]"

class DarkButton(Button):
    def render(self) -> str: return "[ Dark Button ]"

class LightCheckbox(Checkbox):
    def render(self) -> str: return "☐ Light Checkbox"

class DarkCheckbox(Checkbox):
    def render(self) -> str: return "■ Dark Checkbox"

class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: ...
    @abstractmethod
    def create_checkbox(self) -> Checkbox: ...

class LightThemeFactory(GUIFactory):
    def create_button(self) -> Button: return LightButton()
    def create_checkbox(self) -> Checkbox: return LightCheckbox()

class DarkThemeFactory(GUIFactory):
    def create_button(self) -> Button: return DarkButton()
    def create_checkbox(self) -> Checkbox: return DarkCheckbox()

def render_ui(factory: GUIFactory):
    btn = factory.create_button()
    chk = factory.create_checkbox()
    print(btn.render(), chk.render())

render_ui(DarkThemeFactory())   # Dark theme throughout — consistent
```

---

## 4. Builder

**Intent:** Construct complex objects step by step. Same construction process can create different representations.

```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class QueryBuilder:
    _table: str = ""
    _conditions: list = field(default_factory=list)
    _columns: list = field(default_factory=list)
    _limit: Optional[int] = None
    _order_by: Optional[str] = None

    def from_table(self, table: str) -> 'QueryBuilder':
        self._table = table
        return self  # method chaining

    def select(self, *cols: str) -> 'QueryBuilder':
        self._columns.extend(cols)
        return self

    def where(self, condition: str) -> 'QueryBuilder':
        self._conditions.append(condition)
        return self

    def order_by(self, col: str) -> 'QueryBuilder':
        self._order_by = col
        return self

    def limit(self, n: int) -> 'QueryBuilder':
        self._limit = n
        return self

    def build(self) -> str:
        cols = ", ".join(self._columns) if self._columns else "*"
        sql = f"SELECT {cols} FROM {self._table}"
        if self._conditions:
            sql += " WHERE " + " AND ".join(self._conditions)
        if self._order_by:
            sql += f" ORDER BY {self._order_by}"
        if self._limit is not None:
            sql += f" LIMIT {self._limit}"
        return sql

# Usage
sql = (QueryBuilder()
    .from_table("orders")
    .select("id", "total", "status")
    .where("status = 'PAID'")
    .where("total > 100")
    .order_by("created_at DESC")
    .limit(50)
    .build())
print(sql)
# SELECT id, total, status FROM orders WHERE status = 'PAID' AND total > 100 ORDER BY created_at DESC LIMIT 50
```

**When to use:** Object has many optional parameters (avoid 10-parameter constructors), or construction requires multiple steps in a specific order.

---

## 5. Prototype

**Intent:** Create new objects by cloning an existing object (the prototype).

```python
import copy

class ShapePrototype:
    def clone(self):
        return copy.deepcopy(self)

class Circle(ShapePrototype):
    def __init__(self, radius: float, color: str):
        self.radius = radius
        self.color = color

    def __repr__(self):
        return f"Circle(r={self.radius}, color={self.color})"

# Usage
original = Circle(5.0, "red")
clone = original.clone()
clone.color = "blue"  # modify clone, original unchanged

print(original)  # Circle(r=5.0, color=red)
print(clone)     # Circle(r=5.0, color=blue)
```

**When to use:** Object initialization is expensive (DB query, file load, complex computation) and you need many similar objects.

---

## Creational Patterns Summary

| Pattern | Creates | Key Method | When |
|---------|---------|-----------|------|
| Singleton | One instance | `getInstance()` | Shared resource |
| Factory Method | Objects via subclass | `create()` | Decouple type from creation |
| Abstract Factory | Families of objects | `createA()`, `createB()` | Consistent product families |
| Builder | Complex objects step-by-step | `build()` | Many optional params |
| Prototype | Copies of objects | `clone()` | Expensive initialization |
