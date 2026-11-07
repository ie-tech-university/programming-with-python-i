# Session IX: Concepts Guide
## Advanced Object-Oriented Programming Techniques

In Session VIII, you learned how to create classes, objects, attributes, and instance methods. In this session, you will go further by building **more flexible class hierarchies** using inheritance, class methods, static methods, and polymorphism.

These tools let you design systems that are easier to extend and maintain. For example:
- a `RecurringTransaction` can extend a base `Transaction`
- a class method can summarize many transactions at once
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

> Go to **Exercise 1 (Transaction Inheritance)** where you will create a `RecurringTransaction` subclass.

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

```python
class Transaction:
    def describe(self):
        return f"{self.category}: ${self.amount:.2f}"

class RecurringTransaction(Transaction):
    def describe(self):
        base = super().describe()
        return f"{base} every {self.frequency}"
```

> This pattern is central to **Exercise 1 (Transaction Inheritance)**.

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
- you want an alternative constructor
- you want to summarize or process a collection of objects

Example:

```python
class Transaction:
    def __init__(self, amount, category):
        self.amount = amount
        self.category = category

    @classmethod
    def total_amount(cls, transactions):
        return sum(t.amount for t in transactions)
```

> Go to **Exercise 2 (Expense Summary with Class Methods)** where the class will summarize many transaction objects.

---

## 5. Static Methods

A **static method** is a function stored inside a class that does not need `self` or `cls`.

```python
class Validator:
    @staticmethod
    def is_positive_number(value):
        return isinstance(value, (int, float)) and value > 0
```

Call it like this:

```python
print(Validator.is_positive_number(10))   # True
```

### When to Use a Static Method

Use a static method when:
- the function is logically related to the class
- but it does not depend on a specific object
- and it does not need the class itself either

Example for input validation:

```python
class Contact:
    @staticmethod
    def is_valid_email(email):
        return "@" in email and "." in email
```

> Go to **Exercise 3 (Validation Utilities)** where you will use static methods to keep validation logic organized.

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

> You will use this in **Exercise 5 (Polymorphic Budget Planner)**.

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
```python
class Transaction:
    @staticmethod
    def is_valid_category(category):
        return category.strip() != ""

    @classmethod
    def summarize_by_category(cls, transactions):
        totals = {}
        for t in transactions:
            totals[t.category] = totals.get(t.category, 0) + t.amount
        return totals
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
    def is_valid_email(email):
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

> Go to **Exercise 4 (Contact Book 2.0)** where you will redesign the contact book in a more complete OOP style.

---

## 9. Choosing Between Instance, Class, and Static Methods

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

```
┌──────────────────────────────────────────────────────┐
│ INHERITANCE         class Child(Parent)             │
│ super()             Call parent constructor/method  │
│ OVERRIDING          Redefine inherited method       │
│ CLASS METHOD        @classmethod + cls              │
│ STATIC METHOD       @staticmethod                   │
│ POLYMORPHISM        Same method, different classes  │
│ HIERARCHY           Parent class + subclasses       │
└──────────────────────────────────────────────────────┘
```

---

By the end of this session, you should be able to design small but realistic class hierarchies, organize validation inside classes, and write code that works cleanly with multiple related object types.
