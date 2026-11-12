# Session IX: Concepts Guide
## Advanced Object-Oriented Programming Techniques

In Session VIII, you learned how to create classes, objects, attributes, and instance methods. In this session, you will go further by building **more flexible class hierarchies** using inheritance, class methods, static methods, and polymorphism.

These tools let you design systems that are easier to extend and maintain. For example:
- a `RecurringTransaction` can extend a base `Transaction`
- a class method can build a transaction from a dictionary (an alternative constructor)
- a static method can validate input without needing a specific object
- different subclasses can respond to the same method call in different ways

---

## 1. Inheritance

**Inheritance** lets one class reuse and extend another class.

```python
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    pass

d = Dog()
print(d.speak())   # Some sound
```

`Dog` inherits the behavior of `Animal`.

### Adding New Behavior in the Subclass

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def bark(self):
        return f"{self.name} says woof!"
```

Now `Dog` has both:
- inherited attributes from `Animal`
- new methods defined in `Dog`

> Go to **Exercise 1 (Transaction Inheritance)**, where you will create a `RecurringTransaction` subclass (there, `Transaction` also stores a `description`).

---

## 2. Using `super()`

When a subclass needs to reuse the parent class constructor or method, use `super()`.

```python
class Transaction:
    def __init__(self, amount, category):
        self.amount = amount
        self.category = category

class RecurringTransaction(Transaction):
    def __init__(self, amount, category, frequency):
        super().__init__(amount, category)
        self.frequency = frequency
```

`super().__init__(...)` initializes the inherited attributes from the parent class.

Without `super()`, you would have to duplicate setup code.

### Extending Parent Behavior

The same two classes, now each with a `describe()` method: the subclass reuses the parent's text with `super()`.

```python
class Transaction:
    def __init__(self, amount, category):
        self.amount = amount
        self.category = category

    def describe(self):
        return f"{self.category}: ${self.amount:.2f}"

class RecurringTransaction(Transaction):
    def __init__(self, amount, category, frequency):
        super().__init__(amount, category)
        self.frequency = frequency

    def describe(self):
        base = super().describe()        # the parent's text
        return f"{base} ({self.frequency})"

rent = RecurringTransaction(900, "Rent", "monthly")
print(rent.describe())   # Rent: $900.00 (monthly)
```

> Go to **Exercise 1 (Transaction Inheritance)**, where you will override `describe()` with `super()` and add `annual_cost()`.

---

## 3. Method Overriding

A subclass can **override** a method from the parent class by defining a new version with the same name.

```python
class Animal:
    def speak(self):
        return "Some sound"

class Cat(Animal):
    def speak(self):
        return "Meow"
```

Now:

```python
a = Animal()
c = Cat()

print(a.speak())   # Some sound
print(c.speak())   # Meow
```

This is useful when subclasses share a common structure but need custom behavior.

> You will override formatting and scheduling behavior in the recurring transaction exercises.

---

## 4. Class Methods

A **class method** works with the class itself instead of a specific object. It uses `cls` instead of `self`.

```python
class Student:
    school = "Python University"

    @classmethod
    def get_school(cls):
        return cls.school
```

Call it on the class:

```python
print(Student.get_school())
```

### Why Use a Class Method?

Use a class method when:
- the logic belongs to the whole class, not one object
- you want an **alternative constructor**: another way to create objects, here from a dictionary
- you want to summarize or process a collection of objects

Example: an alternative constructor

```python
class Transaction:
    def __init__(self, amount, category):
        self.amount = amount
        self.category = category

    @classmethod
    def from_dict(cls, data):
        return cls(data["amount"], data["category"])

data = {"amount": 12.5, "category": "Food"}
t = Transaction.from_dict(data)
print(t.category, t.amount)   # Food 12.5
```

`cls` is the class the method was called on, so `cls(...)` creates a new object exactly like `Transaction(...)` would.

### Summaries: Class or Static Method?

A class method can also process a collection of objects. But look closely: `total_amount()` never uses `cls`, so it could just as well be a **static method**. You *need* a class method only when the code uses `cls`, as `from_dict()` does.

```python
# Inside class Transaction (after __init__ and from_dict):
    @classmethod
    def total_amount(cls, transactions):     # cls is never used...
        return sum(t.amount for t in transactions)

    @staticmethod
    def total_amount_static(transactions):   # ...so this works the same
        return sum(t.amount for t in transactions)
```

> Go to **Exercise 2 (Expense Summary with Class Methods)**, where `from_dict()` builds transactions with `cls(...)` and class methods summarize them.

---

## 5. Static Methods

A **static method** is a function stored inside a class that does not need `self` or `cls`.

```python
class Validator:
    @staticmethod
    def is_positive_number(value):
        if isinstance(value, bool):  # not a number here
            return False
        return isinstance(value, (int, float)) and value > 0

print(Validator.is_positive_number(10))    # True
print(Validator.is_positive_number(-3))    # False
print(Validator.is_positive_number(True))  # False
```

Call it on the class: `Validator.is_positive_number(10)`.

Watch the edge case: `bool` is a subclass of `int`, so without the first check `True` would count as the positive number 1.

### When to Use a Static Method

Use a static method when:
- the function is logically related to the class
- but it does not depend on a specific object
- and it does not need the class itself either

Example: a **basic** email check:

```python
class Contact:
    @staticmethod
    def is_valid_email(email):
        # basic check only: real email rules are much longer
        return "@" in email and "." in email

print(Contact.is_valid_email("alice@email.com"))   # True
print(Contact.is_valid_email("wrong-email"))       # False
print(Contact.is_valid_email("@."))                # True (!)
```

> Go to **Exercise 3 (Validation Utilities with Static Methods)**, where you will use static methods to keep validation logic organized.

---

## 6. Polymorphism

**Polymorphism** means different classes can respond to the same method name in their own way.

```python
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

animals = [Dog(), Cat()]
for animal in animals:
    print(animal.speak())
```

Output:
```python
Woof
Meow
```

The loop does not need to know which subclass each object is. It just calls `.speak()`.

### Why This Matters

Polymorphism makes your code more flexible:
- one loop can process many related object types
- new subclasses can be added later without changing the loop
- shared interfaces make designs cleaner

For this to work, every subclass must provide the shared method. The base class can insist on it.

### Methods Every Subclass Must Provide

```python
class Transaction:
    def __init__(self, amount, description):
        self.amount = amount
        self.description = description

    def monthly_cost(self):
        raise NotImplementedError("subclasses must override this")

class OneTimeTransaction(Transaction):
    def monthly_cost(self):
        return self.amount

class Forgotten(Transaction):
    pass

print(OneTimeTransaction(120, "Books").monthly_cost())
# 120
Forgotten(5, "Oops").monthly_cost()
# NotImplementedError: subclasses must override this
```

The base class has a **placeholder** method that each subclass overrides. Make the placeholder `raise NotImplementedError`: if a subclass forgets the override, you get a clear error instead of a silently wrong total (as a placeholder that returns 0 would give).

Stricter option: inherit from `abc.ABC` and mark the method `@abstractmethod`. Python then refuses to create `Forgotten()` at all (`TypeError: Can't instantiate abstract class …`).

> Go to **Exercise 5 (Polymorphic Budget Planner)**, where `Transaction.monthly_cost()` is the placeholder that each subclass overrides.

---

## 7. Designing an OOP Hierarchy

Suppose you are building a budget app.

### Base Class
```python
class Transaction:
    def __init__(self, amount, category):
        self.amount = amount
        self.category = category
```

### Subclass
```python
class RecurringTransaction(Transaction):
    def __init__(self, amount, category, frequency):
        super().__init__(amount, category)
        self.frequency = frequency
```

### Related Utility Methods

Methods added to the `Transaction` class:

```python
# Inside class Transaction (after __init__ and describe):
    @staticmethod
    def is_valid_category(category):
        return category.strip() != ""

    @classmethod
    def from_dict(cls, data):
        return cls(data["amount"], data["category"])
```

This design combines:
- inheritance
- `super()`
- static methods
- class methods

That is the core of this session.

---

## 8. Refactoring Earlier Work into OOP

In earlier sessions, your contact book may have looked like this:

```python
contacts = {
    "Alice": {"phone": "555-1234", "email": "alice@email.com"}
}
```

An OOP version is cleaner and easier to extend:

```python
class Contact:
    def __init__(self, name, phone, email):
        self.name = name
        self.phone = phone
        self.email = email

    @staticmethod
    def is_valid_email(email):       # basic check only
        return "@" in email and "." in email
```

```python
class ContactBook:
    def __init__(self):
        self.contacts = {}

    def add_contact(self, contact):
        self.contacts[contact.name.lower()] = contact
```

This design lets you add validation, printing, searching, and updating more naturally.

> Go to **Exercise 4 (Contact Book 2.0)**, where you will redesign the contact book in a more complete OOP style.

---

## 9. Choosing Between Instance, Class, and Static Methods

Pick the method type by what the code needs: one object, the class, or neither.

| Method Type | First Parameter | Works With | Use It When |
|------------|----------------|-----------|-------------|
| Instance method | `self` | One specific object | Behavior depends on object state |
| Class method | `cls` | The class itself | Behavior applies to the class as a whole |
| Static method | none | Neither object nor class | Utility logic related to the class |

### Example Side by Side

```python
class Example:
    count = 0

    def instance_method(self):
        return "Uses self"

    @classmethod
    def class_method(cls):
        return f"Count = {cls.count}"

    @staticmethod
    def static_method(x, y):
        return x + y
```

---

## Quick Reference Cheat Sheet

```python
class Parent:
    def greet(self):
        return "Hello"

class Child(Parent):
    def greet(self):
        return "Hi"

    @classmethod
    def make_default(cls):
        return cls()

    @staticmethod
    def is_valid(value):
        return value > 0
```

```text
INHERITANCE         class Child(Parent)
super()             Call parent constructor/method
OVERRIDING          Redefine inherited method
CLASS METHOD        @classmethod + cls
ALT CONSTRUCTOR     return cls(...)
STATIC METHOD       @staticmethod (no self/cls)
MUST OVERRIDE       raise NotImplementedError
POLYMORPHISM        Same method, different classes
HIERARCHY           Parent class + subclasses
```

---

By the end of this session, you should be able to design small but realistic class hierarchies, organize validation inside classes, and write code that works cleanly with multiple related object types.
