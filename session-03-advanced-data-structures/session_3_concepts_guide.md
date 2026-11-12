# Session III: Concepts Guide
## Advanced Data Structures

This guide covers the Python data structures you will use throughout the Session III exercises. Read through the explanations and study the examples **before** attempting the exercises.

---

## 1. Lists — Ordered, Mutable Collections

A **list** is the most common data structure in Python. It stores items in order, and you can add, remove, or change items at any time.

### Creating and Accessing

```python
fruits = ["apple", "banana", "cherry"]
print(fruits[0])       # apple
print(fruits[-1])      # cherry (last item)
print(fruits[1:3])     # ['banana', 'cherry'] (slicing)
```

### Common List Methods

| Method | What It Does | Example |
|--------|-------------|---------|
| `.append(x)` | Add `x` to the end | `fruits.append("date")` |
| `.insert(i, x)` | Insert `x` at index `i` | `fruits.insert(0, "avocado")` |
| `.remove(x)` | Remove the first occurrence of `x` | `fruits.remove("banana")` |
| `.pop(i)` | Remove and return item at index `i` | `fruits.pop(0)` |
| `.sort()` | Sort the list in place | `fruits.sort()` |
| `.extend(lst)` | Append all items from another list | `fruits.extend(["fig", "grape"])` |

All of these are **methods**: you call them on a list with a dot (`fruits.append(…)`). `len(fruits)` is different: it is a **built-in function** that takes the list as an argument. The next section shows more of them.

### Built-in Functions for Lists

```python
scores = [85, 92, 78]
print(len(scores), sum(scores))    # 3 255
print(min(scores), max(scores))    # 78 92
print(sum(scores) / len(scores))   # 85.0

print(sorted(scores))   # [78, 85, 92]  (a new list)
print(scores)           # [85, 92, 78]  (unchanged)

scores.sort()           # sorts the list itself, returns None
print(scores)           # [78, 85, 92]
```

`len()`, `sum()`, `min()`, `max()` and `sorted()` are built-in functions: you pass the list in the parentheses.

`sorted(lst)` returns a **new** sorted list and leaves the original alone. The method `lst.sort()` sorts the list **itself** and returns `None`, so never write `lst = lst.sort()`.

### Iterating Over a List

`enumerate()` pairs each item with a counter; `start=1` makes it count from 1 instead of 0:

```python
scores = [85, 92, 78]

for score in scores:
    print(score)

# With a position number
for i, score in enumerate(scores, start=1):
    print(f"Student {i}: {score}")
# Student 1: 85
# Student 2: 92
# Student 3: 78
```

### List Comprehensions (Compact Loops)

A list comprehension creates a new list by transforming each item in one line:

```python
numbers = [1, 2, 3, 4, 5]
squares = [n ** 2 for n in numbers]        # [1, 4, 9, 16, 25]
evens = [n for n in numbers if n % 2 == 0]  # [2, 4]
```

> Go to **Exercise 1 (Student Grade Tracker)**, where you will use lists to store scores and compute averages.

---

## 2. Tuples — Ordered, Immutable Records

A **tuple** is like a list, but it **cannot be changed** after creation. Use tuples for fixed data — things that should not be modified.

### Creating and Accessing

```python
coordinates = (40.4168, -3.7038)    # Madrid
print(coordinates[0])                # 40.4168
print(coordinates[1])                # -3.7038

# Tuples cannot be modified
coordinates[0] = 41.0               # TypeError!
```

### Tuple Unpacking

You can assign each element of a tuple to its own variable in one step:

```python
student = ("Alice", 22, "A+")
name, age, grade = student          # Unpacking

print(name)    # Alice
print(age)     # 22
print(grade)   # A+
```

Functions often **return a tuple** to send back several results at once (`return items, total`), and the caller unpacks it the same way.

> Go to **Exercise 4 (Inventory Deduplicator)**, where your function returns a tuple `(clean_list, duplicate_skus_set)` that the caller unpacks into two variables.

### When to Use Tuples vs. Lists

| Use a **List** when... | Use a **Tuple** when... |
|------------------------|------------------------|
| Items will change (add, remove, sort) | Data is fixed and should not change |
| A collection of similar items | A record with different fields |
| `["apple", "banana", "cherry"]` | `("Alice", 22, "A+")` |

### Tuples as Dictionary Keys

Because tuples are immutable, they can be used as dictionary keys (lists cannot):

```python
locations = {
    (40.4168, -3.7038): "Madrid",
    (48.8566, 2.3522): "Paris",
}
print(locations[(40.4168, -3.7038)])   # Madrid
```

---

## 3. Sets — Unordered, Unique Items

A **set** stores items with **no duplicates** and **no defined order**. Sets are ideal for membership testing and removing duplicates.

### Creating a Set

```python
colors = {"red", "green", "blue"}
from_list = set([1, 2, 2, 3, 3, 3])
print(from_list)   # {1, 2, 3} (duplicates removed)

empty_set = set()   # an empty set
empty_dict = {}     # careful: {} is an empty dict!
print(type(empty_set), type(empty_dict))
# <class 'set'> <class 'dict'>
```

> **Careful:** `{}` creates an empty **dictionary**, not an empty set. Use `set()` when you need an empty set, for example to start a list of attendees.

### Common Set Operations

| Operation | Syntax | Result |
|-----------|--------|--------|
| Add an item | `s.add("yellow")` | Adds to set |
| Remove an item | `s.discard("red")` | Removes (no error if missing) |
| Union (combine) | `a \| b` or `a.union(b)` | All items from both sets |
| Intersection | `a & b` or `a.intersection(b)` | Items in both sets |
| Difference | `a - b` or `a.difference(b)` | Items in `a` but not `b` |
| Membership test | `"red" in s` | `True` or `False` |

### Practical Example: Removing Duplicates

```python
emails = ["a@x.com", "b@x.com", "a@x.com", "c@x.com"]
unique_emails = sorted(set(emails))   # a set has no order
print(unique_emails)   # ['a@x.com', 'b@x.com', 'c@x.com']
```

### Practical Example: Finding Common Items

```python
python_students = {"Alice", "Bob", "Charlie"}
sql_students = {"Bob", "Diana", "Charlie"}

both = python_students & sql_students
print(sorted(both))    # ['Bob', 'Charlie'] (sorted: sets have no order)
```

> Go to **Exercise 4 (Inventory Deduplicator)** and **Exercise 5 (Movie Collection Analyzer)**, where you will use sets for deduplication and category tracking.

---

## 4. Dictionaries — Key-Value Storage

Dictionaries store data as **key-value pairs**. They are the most powerful and flexible built-in data structure.

### Creating and Accessing

```python
student = {
    "name": "Alice",
    "age": 22,
    "grade": "A"
}

print(student["name"])             # Alice
print(student.get("email", "N/A"))  # N/A (safe access with default)
```

### Adding, Updating, and Deleting

```python
student["email"] = "alice@uni.edu"   # Add a new key
student["grade"] = "A+"               # Update existing key
del student["age"]                     # Delete a key
```

### Looping Through a Dictionary

```python
prices = {"Zone 1": 2.50, "Zone 2": 4.00, "Zone 3": 6.50}

# Keys only (same as looping over prices.keys())
for zone in prices:
    print(zone)

# Keys and values together
for zone, price in prices.items():
    print(f"{zone}: ${price:.2f}")
```

### Safe Access with `.get()`

Using `dict[key]` raises a `KeyError` if the key doesn't exist. Using `.get(key, default)` returns a safe default instead:

```python
student = {"name": "Alice"}

# Dangerous — will crash if key is missing
# print(student["email"])  # KeyError

# Safe
print(student.get("email", "Not provided"))  # Not provided
```

### The Accumulation Pattern

Counting or summing values into a dictionary is extremely common:

```python
words = ["hello", "world", "hello", "python", "hello"]

counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

print(counts)   # {'hello': 3, 'world': 1, 'python': 1}
```

> Dictionaries are used in **every exercise** this session. Master `.get()`, `.items()`, and the accumulation pattern.

---

## 5. Nested Data Structures

Real-world data is rarely flat. Python lets you nest structures inside each other — lists inside dicts, dicts inside lists, and so on.

### List of Dictionaries (Most Common Pattern)

This is the standard way to represent a collection of records, like a database table:

```python
students = [
    {"name": "Alice", "grade": "A", "score": 95},
    {"name": "Bob", "grade": "B", "score": 82},
    {"name": "Charlie", "grade": "A", "score": 91},
]

# Access a specific student's score
print(students[0]["score"])   # 95

# Loop and filter
for s in students:
    if s["grade"] == "A":
        print(f"{s['name']} got an A!")
```

### Dictionary of Lists

Useful when one key maps to multiple values:

```python
courses = {
    "Python I": ["Alice", "Bob", "Charlie"],
    "SQL Basics": ["Bob", "Diana"],
}

# Add a student to a course
courses["Python I"].append("Eve")

# Check enrollment
if "Alice" in courses["Python I"]:
    print("Alice is enrolled in Python I")
```

### Dictionary of Dictionaries (Nested Dicts)

Useful for hierarchical or structured data, similar to JSON:

```python
contacts = {
    "Alice": {
        "phone": "555-1234",
        "email": "alice@email.com",
        "city": "Madrid"
    },
    "Bob": {
        "phone": "555-5678",
        "email": "bob@email.com",
        "city": "London"
    }
}

# Access nested data
print(contacts["Alice"]["email"])   # alice@email.com

# Loop through all contacts
for name, info in contacts.items():
    print(f"{name}: {info['phone']}")
```

### Navigating Deeply Nested Data (JSON-like)

Real API responses are often deeply nested. Navigate step by step:

```python
data = {
    "company": "TechCorp",
    "departments": [
        {"name": "Engineering", "employees": ["Ava", "Liam"]},
        {"name": "HR", "employees": ["Mia", "Noah"]}
    ]
}

# Step by step
departments = data["departments"]           # List of dicts
engineering = departments[0]                 # First dict
employees = engineering["employees"]         # List of strings
print(employees[0])                          # Ava

# Or all at once
print(data["departments"][0]["employees"][0])  # Ava
```

> Go to **Exercise 2 (Contact Book with Nested Dictionaries)** for nested dictionaries, **Exercise 3 (Company Data Parser (JSON-like Dict))** for JSON-like navigation, and **Exercise 5 (Movie Collection Analyzer)** for lists of dictionaries.

---

## 6. Choosing the Right Data Structure

Match the structure to the job: order, uniqueness, lookups or records.

| Need | Best Structure | Why |
|------|---------------|-----|
| Ordered collection you'll modify | **List** | Mutable, indexed |
| Fixed record (won't change) | **Tuple** | Immutable, lightweight |
| Unique items / deduplication | **Set** | Automatic uniqueness |
| Key-value lookup | **Dictionary** | Fast lookup by key (constant time on average) |
| Collection of records | **List of Dicts** | Each dict is a row |
| Hierarchical / nested data | **Nested Dicts** | Mirrors JSON structure |

### Decision Flowchart

```
Do you need key-value pairs?
├── YES → Dictionary
│   └── Are values themselves structured? → Nested Dict / Dict of Lists
└── NO
    ├── Must items be unique? → Set
    ├── Will data change? → List
    └── Is data fixed? → Tuple
```

> Go to **Exercise 6 (Event Planner (Capstone))**, the capstone exercise, where you will choose and combine the right structures for each part of the problem.

---

## 7. Common Patterns Reference

Reusable recipes for grouping, lookups, filtering and sorting your data.

### Pattern 1: Grouping Items by Category

```python
transactions = [
    {"category": "Food", "amount": 25},
    {"category": "Transport", "amount": 10},
    {"category": "Food", "amount": 30},
]

by_category = {}
for t in transactions:
    cat = t["category"]
    if cat not in by_category:
        by_category[cat] = []
    by_category[cat].append(t["amount"])

print(by_category)   # {'Food': [25, 30], 'Transport': [10]}
```

The same grouping with `.setdefault()`:

```python
by_category = {}
for t in transactions:
    by_category.setdefault(t["category"], []).append(t["amount"])

print(by_category)   # {'Food': [25, 30], 'Transport': [10]}
```

`d.setdefault(key, [])` returns the list stored under `key`; if the key is missing, it first stores the empty list. One call replaces the `if … not in …` check.

### Pattern 2: Building a Lookup Table

```python
students = [
    {"id": "S001", "name": "Alice"},
    {"id": "S002", "name": "Bob"},
]

lookup = {}
for s in students:
    lookup[s["id"]] = s["name"]

print(lookup["S001"])   # Alice
```

```python
# The same table in one line: a dict comprehension
lookup = {s["id"]: s["name"] for s in students}
print(lookup)   # {'S001': 'Alice', 'S002': 'Bob'}
```

A **dict comprehension** works like a list comprehension with `key: value` before `for`. Map each id to a whole record with `{s["id"]: s for s in students}`.

### Choosing What to Compare: `key=` and `lambda`

```python
words = ["kiwi", "banana", "fig"]
print(max(words, key=len))      # banana
print(sorted(words, key=len))   # ['fig', 'kiwi', 'banana']

products = [
    {"name": "Mouse", "price": 25},
    {"name": "Laptop", "price": 999},
]
cheapest = min(products, key=lambda p: p["price"])
print(cheapest["name"])         # Mouse

averages = {"Alice": 89.0, "Bob": 78.0}
print(max(averages, key=averages.get))   # Alice
```

`max()`, `min()` and `sorted()` accept a `key=` function: Python calls it on every item and compares the results instead of the items.

`lambda p: p["price"]` is a tiny unnamed function that returns the price of `p` (Session 4 covers `lambda` in depth). For a dictionary, `max(d, key=d.get)` returns the key with the largest value.

> Go to **Exercise 5 (Movie Collection Analyzer)**, where you will use `key=` to rank movies by rating, and reuse `max(..., key=...)` to find the top student in Exercise 1 and the largest department in Exercise 3.

### Pattern 3: Filtering a List of Dicts

```python
products = [
    {"name": "Laptop", "price": 999},
    {"name": "Mouse", "price": 25},
    {"name": "Monitor", "price": 300},
]

expensive = [p for p in products if p["price"] > 100]
print([p["name"] for p in expensive])   # ['Laptop', 'Monitor']
```

### Pattern 4: Sorting a List of Dicts

```python
# Sort by price (ascending)
sorted_products = sorted(products, key=lambda p: p["price"])

# Sort by price (descending)
sorted_desc = sorted(products, key=lambda p: p["price"], reverse=True)
print([p["name"] for p in sorted_desc])   # ['Laptop', 'Monitor', 'Mouse']
```

---

## Quick Reference Cheat Sheet

```
LISTS         [x, y, z]   ordered, mutable
  .append()  .remove()  .pop()  .sort()  .extend()
  len(lst)  sum(lst)  max(lst)  sorted(lst)
  lst[i]    lst[a:b]    enumerate(lst, start=1)

TUPLES        (x, y, z)   ordered, immutable
  name, age = ("Alice", 22)   # unpacking

SETS          {x, y, z}   unordered, unique
  set()  empty set   ({} is an empty dict!)
  .add()  .discard()  |  &  -  in
  set(list) → remove duplicates

DICTS         {k: v}      key-value pairs
  d[key]  d.get(key, default)  d.items()  d.keys()
  del d[key]   key in d

NESTED        [{"k": v}, ...]   list of dicts
              {"k": {"k2": v}}  dict of dicts
              {"k": [v1, v2]}   dict of lists

PATTERNS
  Accumulate:  d[k] = d.get(k, 0) + 1
  Group:       d.setdefault(k, []).append(v)
  Filter:      [x for x in lst if condition]
  Sort dicts:  sorted(lst, key=lambda x: x["field"])
  Top item:    max(lst, key=lambda x: x["field"])
  Lookup:      {x["id"]: x["name"] for x in lst}
```

---
