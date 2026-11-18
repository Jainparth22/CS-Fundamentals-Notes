# Behavioral Design Patterns

---

## 1. Observer

**Intent:** Define a one-to-many dependency so when one object changes state, all dependents are notified automatically.

```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, event: str, data: dict) -> None: ...

class Subject:
    def __init__(self):
        self._observers: List[Observer] = []

    def subscribe(self, obs: Observer): self._observers.append(obs)
    def unsubscribe(self, obs: Observer): self._observers.remove(obs)

    def _notify(self, event: str, data: dict):
        for obs in self._observers:
            obs.update(event, data)

class OrderService(Subject):
    def place_order(self, order: dict):
        # process...
        self._notify("ORDER_PLACED", order)

    def cancel_order(self, order_id: str):
        # cancel...
        self._notify("ORDER_CANCELLED", {"order_id": order_id})

class EmailService(Observer):
    def update(self, event: str, data: dict):
        if event == "ORDER_PLACED":
            print(f"Email: Order {data['id']} placed! Total: {data['total']}")

class SMSService(Observer):
    def update(self, event: str, data: dict):
        if event == "ORDER_PLACED":
            print(f"SMS: Your order is confirmed!")

class InventoryService(Observer):
    def update(self, event: str, data: dict):
        if event == "ORDER_PLACED":
            print(f"Inventory: Reserving items for order {data['id']}")

# Usage
order_svc = OrderService()
order_svc.subscribe(EmailService())
order_svc.subscribe(SMSService())
order_svc.subscribe(InventoryService())
order_svc.place_order({"id": "ORD-001", "total": 299.99})
```

---

## 2. Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable.

```python
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: list) -> list: ...

class BubbleSort(SortStrategy):
    def sort(self, data: list) -> list:
        arr = data[:]
        n = len(arr)
        for i in range(n):
            for j in range(0, n-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr

class QuickSort(SortStrategy):
    def sort(self, data: list) -> list:
        if len(data) <= 1: return data
        pivot = data[len(data) // 2]
        left  = [x for x in data if x < pivot]
        mid   = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + mid + self.sort(right)

class MergeSort(SortStrategy):
    def sort(self, data: list) -> list:
        if len(data) <= 1: return data
        m = len(data) // 2
        L, R = self.sort(data[:m]), self.sort(data[m:])
        result, i, j = [], 0, 0
        while i < len(L) and j < len(R):
            if L[i] <= R[j]: result.append(L[i]); i += 1
            else: result.append(R[j]); j += 1
        return result + L[i:] + R[j:]

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: SortStrategy):
        self._strategy = strategy

    def sort(self, data: list) -> list:
        return self._strategy.sort(data)

sorter = Sorter(QuickSort())
print(sorter.sort([3, 1, 4, 1, 5, 9, 2]))

sorter.set_strategy(MergeSort())  # swap strategy at runtime
print(sorter.sort([3, 1, 4, 1, 5, 9, 2]))
```

---

## 3. Command

**Intent:** Encapsulate a request as an object, allowing parameterization, queuing, and undo/redo.

```python
from abc import ABC, abstractmethod
from typing import List

class Command(ABC):
    @abstractmethod
    def execute(self) -> None: ...
    @abstractmethod
    def undo(self) -> None: ...

class TextEditor:
    def __init__(self): self.text = ""
    def insert(self, pos: int, s: str):
        self.text = self.text[:pos] + s + self.text[pos:]
    def delete(self, pos: int, length: int):
        self.text = self.text[:pos] + self.text[pos+length:]

class InsertCommand(Command):
    def __init__(self, editor: TextEditor, pos: int, text: str):
        self._editor, self._pos, self._text = editor, pos, text
    def execute(self): self._editor.insert(self._pos, self._text)
    def undo(self): self._editor.delete(self._pos, len(self._text))

class CommandHistory:
    def __init__(self): self._history: List[Command] = []

    def execute(self, cmd: Command):
        cmd.execute()
        self._history.append(cmd)

    def undo(self):
        if self._history:
            self._history.pop().undo()

editor = TextEditor()
history = CommandHistory()
history.execute(InsertCommand(editor, 0, "Hello"))
history.execute(InsertCommand(editor, 5, " World"))
print(editor.text)  # Hello World
history.undo()
print(editor.text)  # Hello
```

---

## 4. State

**Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

```python
from abc import ABC, abstractmethod

class TrafficLight:
    def __init__(self):
        self._state: LightState = RedState(self)

    def change_state(self, state: 'LightState'):
        self._state = state

    def next(self):
        self._state.next()

    def __str__(self):
        return str(self._state)

class LightState(ABC):
    def __init__(self, light: TrafficLight): self._light = light
    @abstractmethod
    def next(self): ...

class RedState(LightState):
    def next(self):
        print("Red → Green")
        self._light.change_state(GreenState(self._light))
    def __str__(self): return "🔴 RED"

class GreenState(LightState):
    def next(self):
        print("Green → Yellow")
        self._light.change_state(YellowState(self._light))
    def __str__(self): return "🟢 GREEN"

class YellowState(LightState):
    def next(self):
        print("Yellow → Red")
        self._light.change_state(RedState(self._light))
    def __str__(self): return "🟡 YELLOW"

light = TrafficLight()
print(light)  # 🔴 RED
light.next()  # Red → Green
print(light)  # 🟢 GREEN
light.next()  # Green → Yellow
```

---

## 5. Template Method

**Intent:** Define the skeleton of an algorithm in a base class, letting subclasses override specific steps without changing the structure.

```python
from abc import ABC, abstractmethod

class DataProcessor(ABC):
    # Template method — fixed algorithm
    def process(self, source: str) -> str:
        raw = self.read(source)
        parsed = self.parse(raw)
        validated = self.validate(parsed)
        return self.format(validated)

    @abstractmethod
    def read(self, source: str) -> str: ...

    @abstractmethod
    def parse(self, raw: str) -> dict: ...

    def validate(self, data: dict) -> dict:  # concrete with default
        assert data, "Data cannot be empty"
        return data

    @abstractmethod
    def format(self, data: dict) -> str: ...

class CSVProcessor(DataProcessor):
    def read(self, source: str) -> str:
        return "col1,col2\nval1,val2"  # simulate file read
    def parse(self, raw: str) -> dict:
        lines = raw.strip().split("\n")
        headers = lines[0].split(",")
        return dict(zip(headers, lines[1].split(",")))
    def format(self, data: dict) -> str:
        return str(data)

class JSONProcessor(DataProcessor):
    def read(self, source: str) -> str: return '{"key": "value"}'
    def parse(self, raw: str) -> dict:
        import json; return json.loads(raw)
    def format(self, data: dict) -> str:
        import json; return json.dumps(data, indent=2)
```

---

## Behavioral Patterns Summary

| Pattern | Purpose | Key Mechanism |
|---------|---------|--------------|
| Observer | Notify many on state change | Subscribe/publish list |
| Strategy | Swap algorithms at runtime | Inject strategy object |
| Command | Encapsulate requests | execute/undo methods |
| State | Behavior changes with state | Delegate to state object |
| Template Method | Algorithm skeleton | Abstract methods in base class |
| Chain of Responsibility | Chain request handlers | Each handler passes or handles |
| Mediator | Centralize communication | Hub object |
| Iterator | Uniform collection traversal | `__iter__`, `__next__` |
| Memento | Undo/snapshot | Capture state in memento object |
