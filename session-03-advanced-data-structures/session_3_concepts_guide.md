# Session III: Concepts Guide
## Advanced Data Structures

This guide covers the Python data structures you will use throughout the Session III exercises. Read through the explanations and study the examples **before** attempting the exercises.

---

## 1. Lists — Ordered, Mutable Collections

A **list** is the most common data structure in Python. It stores items in order, and you can add, remove, or change items at any time.

### Creating and Accessing

```python
fruits = ["apple", "banana", "cherry"]
print(fruits[0])       # "apple"
print(fruits[-1])      # "cherry" (last item)
print(fruits[1:3])     # ["banana", "cherry"] (slicing)
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
| `len(lst)` | Number of items | `len(fruits)` |

### Iterating Over a List

```python
scores = [85, 92, 78, 95, 88]

for score in scores:
    print(score)

# With index
for i, score in enumerate(scores):
    print(f"Student {i+1}: {score}")
```

### List Comprehensions (Compact Loops)

A list comprehension creates a new list by transforming each item in one line:

```python
numbers = [1, 2, 3, 4, 5]
squares = [n ** 2 for n in numbers]        # [1, 4, 9, 16, 25]
evens = [n for n in numbers if n % 2 == 0]  # [2, 4]
```

> Go to **Exercise 1 (Student Grade Tracker)** where you will use lists to store scores and compute averages.

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

print(name)    # "Alice"
print(age)     # 22
print(grade)   # "A+"
```

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
print(locations[(40.4168, -3.7038)])   # "Madrid"
```

> Go to **Exercise 4 (Inventory Deduplicator)** where you will use tuples to represent immutable product records.

---

## 3. Sets — Unordered, Unique Items

A **set** stores items with **no duplicates** and **no defined order**. Sets are ideal for membership testing and removing duplicates.

### Creating a Set

```python
colors = {"red", "green", "blue"}
from_list = set([1, 2, 2, 3, 3, 3])   # {1, 2, 3} — duplicates removed
```

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
unique_emails = list(set(emails))
print(unique_emails)   # ['a@x.com', 'b@x.com', 'c@x.com']
```

### Practical Example: Finding Common Items

```python
python_students = {"Alice", "Bob", "Charlie"}
sql_students = {"Bob", "Diana", "Charlie"}

both = python_students & sql_students
print(both)    # {'Bob', 'Charlie'}
```

> Go to **Exercise 4 (Inventory Deduplicator)** and **Exercise 5 (Movie Collection Analyzer)** where you will use sets for deduplication and category tracking.

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

print(student["name"])             # "Alice"
print(student.get("email", "N/A"))  # "N/A" (safe access with default)
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

# Keys only
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
print(student.get("email", "Not provided"))  # "Not provided"
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
print(contacts["Alice"]["email"])   # "alice@email.com"

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
print(employees[0])                          # "Ava"

# Or all at once
print(data["departments"][0]["employees"][0])  # "Ava"
```

> Go to **Exercise 2 (Contact Book)** for nested dictionaries, **Exercise 3 (Company Data Parser)** for JSON-like navigation, and **Exercise 5 (Movie Collection Analyzer)** for lists of dictionaries.

---

## 6. Choosing the Right Data Structure

| Need | Best Structure | Why |
|------|---------------|-----|
| Ordered collection you'll modify | **List** | Mutable, indexed |
| Fixed record (won't change) | **Tuple** | Immutable, lightweight |
| Unique items / deduplication | **Set** | Automatic uniqueness |
| Key-value lookup | **Dictionary** | O(1) access by key |
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

> Go to **Exercise 6 (Event Planner)**, the capstone exercise that requires you to choose and combine the right structures for each part of the problem.

---

## 7. Common Patterns Reference

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

### Pattern 2: Building a Lookup Table

```python
students = [
    {"id": "S001", "name": "Alice"},
    {"id": "S002", "name": "Bob"},
]

lookup = {}
for s in students:
    lookup[s["id"]] = s["name"]

print(lookup["S001"])   # "Alice"
```

### Pattern 3: Filtering a List of Dicts

```python
products = [
    {"name": "Laptop", "price": 999},
    {"name": "Mouse", "price": 25},
    {"name": "Monitor", "price": 300},
]

expensive = [p for p in products if p["price"] > 100]
print(expensive)   # [{'name': 'Laptop', ...}, {'name': 'Monitor', ...}]
```

### Pattern 4: Sorting a List of Dicts

```python
# Sort by price (ascending)
sorted_products = sorted(products, key=lambda p: p["price"])

# Sort by price (descending)
sorted_desc = sorted(products, key=lambda p: p["price"], reverse=True)
```

---

## Quick Reference Cheat Sheet

```
┌───────────────────────────────────────────────────────────┐
│  LISTS         [x, y, z]   ordered, mutable               │
│    .append()  .remove()  .pop()  .sort()  .extend()       │
│    len(lst)   lst[i]     lst[a:b]                         │
│                                                           │
│  TUPLES        (x, y, z)   ordered, immutable             │
│    name, age = ("Alice", 22)   # unpacking                │
│                                                           │
│  SETS          {x, y, z}   unordered, unique              │
│    .add()  .discard()  |  &  -  in                        │
│    set(list) → remove duplicates                          │
│                                                           │
│  DICTS         {k: v}      key-value pairs                │
│    d[key]  d.get(key, default)  d.items()  d.keys()       │
│    del d[key]   key in d                                  │
│                                                           │
│  NESTED        [{"k": v}, ...]   list of dicts            │
│                {"k": {"k2": v}}  dict of dicts             │
│                {"k": [v1, v2]}   dict of lists             │
│                                                           │
│  PATTERNS                                                 │
│    Accumulate:  d[k] = d.get(k, 0) + 1                   │
│    Group:       d.setdefault(k, []).append(v)             │
│    Filter:      [x for x in lst if condition]             │
│    Sort dicts:  sorted(lst, key=lambda x: x["field"])     │
│    Lookup:      {item["id"]: item for item in lst}        │
└───────────────────────────────────────────────────────────┘
```

---
