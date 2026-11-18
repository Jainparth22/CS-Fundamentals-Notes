# OOP — The Four Pillars

---

## 1. Encapsulation

**Idea:** Bundle data and the methods that operate on it into one unit (class). Hide internal state; expose only what's necessary through a public interface.

```cpp
// BAD — no encapsulation
struct BankAccount {
    double balance;  // anyone can set balance = -1000000
};

// GOOD — encapsulated
class BankAccount {
    double balance;    // private
    string owner;
public:
    BankAccount(string o, double initial) : owner(o), balance(initial) {}

    bool deposit(double amount) {
        if (amount <= 0) return false;
        balance += amount;
        return true;
    }

    bool withdraw(double amount) {
        if (amount <= 0 || amount > balance) return false;
        balance -= amount;
        return true;
    }

    double getBalance() const { return balance; }  // read-only getter
};
```

```python
class BankAccount:
    def __init__(self, owner: str, initial: float):
        self._balance = initial   # convention: _ = "private"
        self.__owner = owner      # name-mangled: _BankAccount__owner

    def deposit(self, amount: float) -> bool:
        if amount <= 0:
            return False
        self._balance += amount
        return True

    @property
    def balance(self) -> float:    # getter as property
        return self._balance
```

**Benefits:** Invariants enforced (can't set negative balance), implementation can change without breaking callers.

---

## 2. Inheritance

**Idea:** A subclass inherits attributes and methods of its superclass. Enables code reuse and "is-a" relationships.

```cpp
class Animal {
protected:
    string name;
public:
    Animal(const string& n) : name(n) {}
    virtual void speak() { cout << name << " makes a sound\n"; }
    virtual ~Animal() {}  // ALWAYS make destructor virtual in base class
};

class Dog : public Animal {
public:
    Dog(const string& n) : Animal(n) {}
    void speak() override { cout << name << " says Woof!\n"; }
    void fetch() { cout << name << " fetches the ball\n"; }
};

class Cat : public Animal {
public:
    Cat(const string& n) : Animal(n) {}
    void speak() override { cout << name << " says Meow!\n"; }
};

// Polymorphic usage
vector<Animal*> animals = {new Dog("Rex"), new Cat("Whiskers")};
for (auto a : animals) a->speak();  // calls correct speak() at runtime
```

```python
class Animal:
    def __init__(self, name: str):
        self.name = name

    def speak(self) -> str:
        return f"{self.name} makes a sound"

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self) -> str:
        return f"{self.name} says Meow!"
```

**Multiple Inheritance (Python):** `class C(A, B)` — resolved by MRO (Method Resolution Order, C3 linearization).

---

## 3. Polymorphism

**Idea:** Same interface, different implementations. "One interface, many forms."

### Compile-time (Static) Polymorphism — Function Overloading, Templates
```cpp
// Overloading
int add(int a, int b)       { return a + b; }
double add(double a, double b) { return a + b; }

// Templates (compile-time)
template<typename T>
T add(T a, T b) { return a + b; }
```

### Runtime (Dynamic) Polymorphism — Virtual Functions
```cpp
Animal* a = new Dog("Rex");
a->speak();  // calls Dog::speak() despite pointer being Animal*
// Resolved at runtime via vtable (virtual function table)
```

**vtable:** Each class with virtual functions has a vtable — an array of function pointers. Each object has a vptr pointing to its class's vtable.

### Duck Typing (Python)
```python
# No inheritance needed — if it walks like a duck and quacks like a duck
class Duck:
    def speak(self): return "Quack"

class Person:
    def speak(self): return "Hello"

def make_speak(thing):  # works for ANY object with a speak() method
    print(thing.speak())

make_speak(Duck())   # Quack
make_speak(Person()) # Hello
```

---

## 4. Abstraction

**Idea:** Hide implementation complexity; expose only relevant interface. Pure virtual functions in C++, abstract base classes in Python.

```cpp
// Abstract class — cannot be instantiated directly
class Shape {
public:
    virtual double area() const = 0;      // pure virtual
    virtual double perimeter() const = 0; // pure virtual
    virtual void draw() const { cout << "Drawing shape\n"; } // concrete method
    virtual ~Shape() {}
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override { return 3.14159 * radius * radius; }
    double perimeter() const override { return 2 * 3.14159 * radius; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area() const override { return w * h; }
    double perimeter() const override { return 2 * (w + h); }
};
```

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self) -> str:
        return f"Area: {self.area():.2f}, Perimeter: {self.perimeter():.2f}"

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
    def area(self) -> float: return 3.14159 * self.radius ** 2
    def perimeter(self) -> float: return 2 * 3.14159 * self.radius
```

---

## 5. Pillars Summary

| Pillar | Key Idea | Mechanism |
|--------|----------|-----------|
| Encapsulation | Hide internal state | Access modifiers, getters/setters |
| Inheritance | Reuse and extend | Subclass, `extends`/`:` |
| Polymorphism | Same interface, many forms | Virtual functions, overloading, duck typing |
| Abstraction | Hide complexity | Abstract classes, interfaces |

---

## 6. Interview Questions

| Question | Answer |
|----------|--------|
| Inheritance vs Composition? | "Prefer composition over inheritance" — composition is more flexible; inheritance creates tight coupling |
| When to use abstract class vs interface? | Abstract class: some shared implementation; interface (pure virtual): pure contract with no shared state |
| What is the diamond problem? | Multiple inheritance: B and C inherit from A; D inherits from B and C — which A does D have? C++ uses virtual inheritance to solve it |
| Is Python's `__init__` a constructor? | It's an initializer; `__new__` is the actual constructor that allocates the object |
| What is `super()`? | Calls the parent class's method; in Python uses MRO for correct order in multiple inheritance |
