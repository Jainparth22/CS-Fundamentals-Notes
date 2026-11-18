# Structural Design Patterns

---

## 1. Adapter

**Intent:** Convert the interface of a class into another interface clients expect. Lets incompatible interfaces work together.

```python
# Third-party payment library has its own interface
class StripePayment:
    def charge_card(self, card_number: str, amount_cents: int) -> bool:
        print(f"Stripe: charging {amount_cents} cents to {card_number[-4:]}")
        return True

# Your app expects this interface
class PaymentProcessor:
    def pay(self, amount: float) -> bool: ...

# Adapter bridges the gap
class StripeAdapter(PaymentProcessor):
    def __init__(self, stripe: StripePayment, card_number: str):
        self._stripe = stripe
        self._card_number = card_number

    def pay(self, amount: float) -> bool:
        amount_cents = int(amount * 100)
        return self._stripe.charge_card(self._card_number, amount_cents)

# Usage: app only knows PaymentProcessor
processor: PaymentProcessor = StripeAdapter(StripePayment(), "4242424242424242")
processor.pay(29.99)  # Stripe: charging 2999 cents to 4242
```

---

## 2. Decorator

**Intent:** Attach additional responsibilities to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.

```python
from abc import ABC, abstractmethod

class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float: ...
    @abstractmethod
    def description(self) -> str: ...

class SimpleCoffee(Coffee):
    def cost(self) -> float: return 1.00
    def description(self) -> str: return "Coffee"

# Base decorator
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee

class MilkDecorator(CoffeeDecorator):
    def cost(self) -> float: return self._coffee.cost() + 0.25
    def description(self) -> str: return self._coffee.description() + ", Milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self) -> float: return self._coffee.cost() + 0.10
    def description(self) -> str: return self._coffee.description() + ", Sugar"

class VanillaDecorator(CoffeeDecorator):
    def cost(self) -> float: return self._coffee.cost() + 0.50
    def description(self) -> str: return self._coffee.description() + ", Vanilla"

# Dynamically build the order
coffee = SimpleCoffee()
coffee = MilkDecorator(coffee)
coffee = SugarDecorator(coffee)
coffee = VanillaDecorator(coffee)
print(f"{coffee.description()} = ${coffee.cost():.2f}")
# Coffee, Milk, Sugar, Vanilla = $1.85
```

**Python `@decorator` syntax is this pattern!** `functools.lru_cache`, `functools.wraps`, Flask's `@app.route` are all decorator pattern.

---

## 3. Facade

**Intent:** Provide a simplified interface to a complex subsystem.

```python
# Complex subsystem
class VideoDecoder:
    def decode(self, file: str): print(f"Decoding {file}")

class AudioDecoder:
    def decode(self, file: str): print(f"Decoding audio of {file}")

class VideoRenderer:
    def render(self, frames): print("Rendering frames")

class AudioRenderer:
    def render(self, audio): print("Rendering audio")

class SubtitleLoader:
    def load(self, file: str): print(f"Loading subtitles for {file}")

# Facade: simple API for complex subsystem
class VideoPlayer:
    def __init__(self):
        self._video_decoder = VideoDecoder()
        self._audio_decoder = AudioDecoder()
        self._video_renderer = VideoRenderer()
        self._audio_renderer = AudioRenderer()
        self._subtitle_loader = SubtitleLoader()

    def play(self, filename: str):
        """One simple method hides all the complexity."""
        self._video_decoder.decode(filename)
        self._audio_decoder.decode(filename)
        self._subtitle_loader.load(filename)
        self._video_renderer.render(None)
        self._audio_renderer.render(None)

# Client only needs to know one method
player = VideoPlayer()
player.play("movie.mp4")
```

---

## 4. Proxy

**Intent:** Provide a surrogate or placeholder for another object to control access to it.

### Virtual Proxy (Lazy Initialization)
```python
class RealDatabase:
    def __init__(self):
        print("Expensive DB connection initialized!")  # slow
        self._data = {"user:1": "Alice", "user:2": "Bob"}

    def query(self, key: str) -> str:
        return self._data.get(key, "not found")

class LazyDatabaseProxy:
    def __init__(self):
        self._real_db = None  # not created yet

    def query(self, key: str) -> str:
        if self._real_db is None:
            self._real_db = RealDatabase()  # create only when needed
        return self._real_db.query(key)

# DB not connected until actually needed
db = LazyDatabaseProxy()
print("Proxy created, no DB connection yet")
print(db.query("user:1"))  # NOW the DB connects
```

### Protection Proxy (Access Control)
```python
class SecureDatabase:
    def __init__(self, db: RealDatabase, role: str):
        self._db = db
        self._role = role

    def query(self, key: str) -> str:
        if self._role not in ("admin", "read"):
            raise PermissionError("Not authorized")
        return self._db.query(key)

    def delete(self, key: str):
        if self._role != "admin":
            raise PermissionError("Only admin can delete")
        # proceed with delete
```

---

## 5. Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies. Treat individual objects and compositions uniformly.

```python
from abc import ABC, abstractmethod

class FileSystemItem(ABC):
    def __init__(self, name: str):
        self.name = name

    @abstractmethod
    def size(self) -> int: ...

    @abstractmethod
    def display(self, indent: int = 0): ...

class File(FileSystemItem):       # Leaf
    def __init__(self, name: str, size_bytes: int):
        super().__init__(name)
        self._size = size_bytes

    def size(self) -> int: return self._size
    def display(self, indent=0): print(" " * indent + f"📄 {self.name} ({self._size}B)")

class Directory(FileSystemItem):  # Composite
    def __init__(self, name: str):
        super().__init__(name)
        self._children: list[FileSystemItem] = []

    def add(self, item: FileSystemItem): self._children.append(item)
    def size(self) -> int: return sum(c.size() for c in self._children)

    def display(self, indent=0):
        print(" " * indent + f"📁 {self.name}/ ({self.size()}B)")
        for child in self._children:
            child.display(indent + 2)

# Build tree
root = Directory("root")
src = Directory("src")
src.add(File("main.py", 2048))
src.add(File("utils.py", 512))
root.add(src)
root.add(File("README.md", 256))

root.display()
# 📁 root/ (2816B)
#   📁 src/ (2560B)
#     📄 main.py (2048B)
#     📄 utils.py (512B)
#   📄 README.md (256B)
```

---

## Structural Patterns Summary

| Pattern | Wraps? | Intent | Example |
|---------|--------|--------|---------|
| Adapter | Yes (converts interface) | Interface compatibility | Stripe adapter |
| Decorator | Yes (adds behavior) | Dynamic feature addition | Coffee toppings |
| Facade | No (simplifies) | Hide complexity | VideoPlayer |
| Proxy | Yes (controls access) | Lazy init / access control | Lazy DB |
| Composite | No (tree structure) | Uniform tree treatment | File system |
| Bridge | No (separates) | Abstraction from implementation | Shapes × Renderers |
| Flyweight | Yes (shares state) | Memory efficiency | Character rendering in text editor |
