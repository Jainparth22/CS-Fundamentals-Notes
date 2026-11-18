# Design Patterns Overview — All 23 GoF Patterns

> The Gang of Four (Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides) catalogued 23 patterns in "Design Patterns: Elements of Reusable Object-Oriented Software" (1994).

---

## Pattern Categories

### Creational — Object Creation Mechanisms

| Pattern | Intent | Key Benefit |
|---------|--------|-------------|
| **Singleton** | Ensure only one instance exists | Shared resource (DB pool, logger) |
| **Factory Method** | Subclasses decide which class to instantiate | Decouple object creation from usage |
| **Abstract Factory** | Create families of related objects | Product line consistency |
| **Builder** | Construct complex objects step by step | Avoid telescoping constructors |
| **Prototype** | Clone existing objects | Expensive initialization avoided |

### Structural — Class/Object Composition

| Pattern | Intent | Key Benefit |
|---------|--------|-------------|
| **Adapter** | Convert one interface to another | Integrate incompatible interfaces |
| **Bridge** | Separate abstraction from implementation | Independent extension of both |
| **Composite** | Tree structures of objects | Uniform treatment of leaf and composite |
| **Decorator** | Add behavior dynamically | Open/Closed: extend without modifying |
| **Facade** | Simplified interface to a subsystem | Hide complexity |
| **Flyweight** | Share common state across many objects | Memory efficiency (thousands of objects) |
| **Proxy** | Control access to an object | Lazy init, access control, logging |

### Behavioral — Object Communication

| Pattern | Intent | Key Benefit |
|---------|--------|-------------|
| **Chain of Responsibility** | Pass request along a chain of handlers | Decouple sender from receivers |
| **Command** | Encapsulate a request as an object | Undo/redo, queuing, logging |
| **Interpreter** | Grammar for a language | DSLs, expression parsers |
| **Iterator** | Sequential access without exposing internals | Uniform collection traversal |
| **Mediator** | Centralize object communication | Reduces coupling (many-to-many → hub) |
| **Memento** | Capture and restore object state | Undo functionality |
| **Observer** | Notify dependents on state change | Event-driven systems, MVC |
| **State** | Change behavior based on internal state | Finite state machines |
| **Strategy** | Interchangeable algorithms | Switch algorithm at runtime |
| **Template Method** | Skeleton of an algorithm, steps in subclasses | Reuse with customization |
| **Visitor** | Add operations to objects without changing them | Double dispatch, type-based operations |

---

## Most Interview-Relevant (Ranked)

| Rank | Pattern | Why Important |
|------|---------|--------------|
| 1 | Observer | Used in every event system, React state, pub/sub |
| 2 | Singleton | Logging, DB connection pools — and its drawbacks |
| 3 | Factory Method | Almost every framework uses it |
| 4 | Strategy | Algorithm swapping, payment processors |
| 5 | Decorator | Python `@decorator`, middleware, Python's `functools.wraps` |
| 6 | Builder | Complex object construction, test builders |
| 7 | Adapter | Third-party API integration |
| 8 | Command | Undo/redo, job queues |
| 9 | Proxy | Lazy loading, access control, logging wrappers |
| 10 | Template Method | Frameworks (Django views, JUnit lifecycle) |

---

## Pattern Selection Guide

| Situation | Use Pattern |
|-----------|------------|
| Need exactly one instance of a class | Singleton |
| Creating objects without specifying exact class | Factory Method |
| Adding features to object without subclassing | Decorator |
| Object needs to notify multiple others on change | Observer |
| Want to switch algorithms at runtime | Strategy |
| Object's behavior depends on its state | State |
| Simplify a complex subsystem | Facade |
| Undo/redo functionality | Command + Memento |
| Walk through a collection uniformly | Iterator |
| Many small objects sharing common data | Flyweight |
