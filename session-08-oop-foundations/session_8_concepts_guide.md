# Session VIII: Concepts Guide
## Object-Oriented Programming Foundations

This guide introduces the core ideas of **object-oriented programming (OOP)** in Python. Until now, most of your programs have been **procedural**: a sequence of functions, loops, and variables. In this session, you will begin modeling data and behavior together using **classes** and **objects**.

OOP is useful when your program works with **real-world entities** that have both:
- **data** (attributes)
- **behavior** (methods)

Examples:
- A `Contact` has a name, phone, and email.
- A `Student` has a name and scores, and can calculate an average.
- A `Transaction` has an amount and category, and can format itself as a report line.

---

## 1. Classes and Objects

A **class** is a blueprint. An **object** is one specific instance created from that blueprint.

```python
class Dog:
    pass

my_dog = Dog()
print(type(my_dog))   # <class '__main__.Dog'>
```

The class defines what an object *can have* and *can do*. Each object is its own separate instance.

```python
class Dog:
    def __init__(self, name):
        self.name = name

dog1 = Dog("Milo")
dog2 = Dog("Luna")

print(dog1.name)   # Milo
print(dog2.name)   # Luna
```

> Go to **Exercise 1 (Student Grade Tracker OOP)** where you will create `Student` objects with attributes and methods.

---

## 2. The `__init__()` Method and Attributes

The `__init__()` method runs automatically when you create a new object. It is used to initialize the object's attributes.

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

alice = Student("Alice", [90, 85, 92])
print(alice.name)      # Alice
print(alice.scores)    # [90, 85, 92]
```

### What is `self`?

`self` refers to the specific object calling the method.

```python
class Student:
    def __init__(self, name):
        self.name = name

s1 = Student("Alice")
s2 = Student("Bob")

print(s1.name)   # Alice
print(s2.name)   # Bob
```

Every object keeps its own data. `self.name` for `s1` is different from `self.name` for `s2`.

### Adding Default Values

You can give attributes default values:

```python
class Task:
    def __init__(self, title, done=False):
        self.title = title
        self.done = done

t = Task("Read chapter 8")
print(t.done)   # False
```

> You will use `__init__()` in **every exercise** this session.

---

## 3. Instance Methods

An **instance method** is a function defined inside a class. It works with a specific object and usually reads or modifies that object's attributes.

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        return sum(self.scores) / len(self.scores)

alice = Student("Alice", [90, 85, 92])
print(alice.average())   # 89.0
```

### Methods Can Modify State

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

account = BankAccount("Alice")
account.deposit(100)
print(account.balance)   # 100
```

### Methods Can Call Other Methods

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        return sum(self.scores) / len(self.scores)

    def is_passing(self):
        return self.average() >= 60
```

> Go to **Exercise 3 (Budget Ledger OOP)** where methods will update and summarize stored transactions.

---

## 4. Encapsulation

**Encapsulation** means keeping related data and behavior together inside the same class.

Instead of storing contact information in loose dictionaries everywhere:

```python
contact = {"name": "Alice", "phone": "555-1234", "email": "alice@email.com"}
```

you can package it into a class:

```python
class Contact:
    def __init__(self, name, phone, email):
        self.name = name
        self.phone = phone
        self.email = email

    def display(self):
        return f"{self.name} | {self.phone} | {self.email}"
```

This makes your code easier to organize, reuse, and extend.

### “Private” Attributes by Convention

Python does not enforce strict private fields like some other languages, but a leading underscore means “internal use”:

```python
class Counter:
    def __init__(self):
        self._count = 0
```

This is a convention, not a hard rule.

> In **Exercise 2 (Contact Book OOP)**, you will encapsulate contact operations inside a `ContactBook` class.

---

## 5. Collections of Objects

A program often manages **many objects at once**.

```python
class Student:
    def __init__(self, name):
        self.name = name

students = [
    Student("Alice"),
    Student("Bob"),
    Student("Charlie")
]

for student in students:
    print(student.name)
```

You can store objects in:
- lists
- dictionaries
- sets (if hashable)
- nested structures

### Dictionary of Objects

```python
contacts = {
    "alice": Contact("Alice", "555-1234", "alice@email.com"),
    "bob": Contact("Bob", "555-5678", "bob@email.com"),
}
```

This is a powerful pattern: use a dictionary for fast lookup, but store full objects as the values.

> Go to **Exercise 2 (Contact Book OOP)** where a dictionary will store `Contact` objects.

---

## 6. Refactoring Procedural Code into OOP

Here is a procedural approach:

```python
students = [
    {"name": "Alice", "scores": [90, 85, 92]},
    {"name": "Bob", "scores": [78, 81, 75]},
]

for student in students:
    avg = sum(student["scores"]) / len(student["scores"])
    print(student["name"], avg)
```

Here is the object-oriented version:

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        return sum(self.scores) / len(self.scores)

students = [
    Student("Alice", [90, 85, 92]),
    Student("Bob", [78, 81, 75]),
]

for student in students:
    print(student.name, student.average())
```

### Why this is better
- The logic for a student stays inside the `Student` class.
- The main program becomes cleaner.
- You can reuse the class in other programs.

> This refactoring idea is central to **Exercise 4 (Procedural to OOP Refactor)**.

---

## 7. `__str__()` for Friendly Printing

When you print an object directly, Python normally shows something like:

```python
<__main__.Student object at 0x...>
```

You can improve that by defining `__str__()`:

```python
class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def __str__(self):
        return f"Student(name={self.name}, scores={self.scores})"

alice = Student("Alice", [90, 85, 92])
print(alice)
```

Output:
```python
Student(name=Alice, scores=[90, 85, 92])
```

This is very useful for debugging and reports.

> You are encouraged to add `__str__()` methods in the later exercises for cleaner output.

---

## 8. When to Use OOP

Use OOP when:
- your program has **entities** with both data and behavior
- you need multiple similar objects
- you want cleaner organization than loose dictionaries and functions

Examples from this course:
- `Contact` and `ContactBook`
- `Student` and `GradeBook`
- `Transaction` and `BudgetLedger`

Avoid forcing OOP into tiny scripts where a simple function would be clearer.

---

## Common OOP Patterns in This Session

### Pattern 1: Simple Data + Behavior

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height
```

### Pattern 2: Manager Class Holding Many Objects

```python
class ContactBook:
    def __init__(self):
        self.contacts = {}

    def add_contact(self, contact):
        self.contacts[contact.name.lower()] = contact
```

### Pattern 3: Object Composition

A class can contain a collection of other objects:

```python
class GradeBook:
    def __init__(self):
        self.students = []

    def add_student(self, student):
        self.students.append(student)
```

---

## Quick Reference Cheat Sheet

```python
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"

p = Person("Alice")
print(p.name)       # attribute access
print(p.greet())    # method call
```

```
┌──────────────────────────────────────────────────────┐
│ CLASS              Blueprint for objects             │
│ OBJECT             Instance of a class               │
│ __init__()         Initializes object attributes     │
│ self               Refers to the current object      │
│ ATTRIBUTE          Variable stored in an object      │
│ METHOD             Function inside a class           │
│ ENCAPSULATION      Bundle data + behavior together   │
│ __str__()          Friendly string representation    │
│ COMPOSITION        Object stores other objects       │
└──────────────────────────────────────────────────────┘
```

---

By the end of this session, you should be able to take a procedural program and redesign it using **classes**, **objects**, and **methods** so that the code is more realistic, reusable, and easier to extend.
