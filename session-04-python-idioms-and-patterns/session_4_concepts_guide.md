# Session IV: Concepts Guide
## Python Idioms and Patterns

This guide covers the most common **Pythonic** idioms and patterns that separate beginner code from clean, professional Python. Read through the explanations and study the examples **before** attempting the exercises.

> **What does "Pythonic" mean?**
> Python has a design philosophy that favors **readability** and **simplicity**. Code that follows this philosophy — using the language's built-in features elegantly rather than reinventing the wheel — is called "Pythonic." You can read these principles by typing `import this` in any Python interpreter.

---

## 1. List Comprehensions — Compact, Readable Loops

A **list comprehension** creates a new list by transforming and/or filtering items from an existing iterable — all in a single, readable line.

### Basic Syntax

```python
new_list = [expression for item in iterable]
```

### From Loop to Comprehension

**Traditional loop:**

```python
squares = []
for n in range(1, 6):
    squares.append(n ** 2)
print(squares)   # [1, 4, 9, 16, 25]
```

**Pythonic comprehension:**

```python
squares = [n ** 2 for n in range(1, 6)]
print(squares)   # [1, 4, 9, 16, 25]
```

### Adding a Filter (`if` clause)

You can include a condition to filter which items are included:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Only even numbers
evens = [n for n in numbers if n % 2 == 0]
print(evens)   # [2, 4, 6, 8, 10]

# Only words longer than 4 characters
words = ["hello", "hi", "python", "go", "world"]
long_words = [w for w in words if len(w) > 4]
print(long_words)   # ['hello', 'python', 'world']
```

### Transform + Filter Combined

```python
# Squares of only the even numbers
even_squares = [n ** 2 for n in range(1, 11) if n % 2 == 0]
print(even_squares)   # [4, 16, 36, 64, 100]
```

### Nested Comprehensions (Flattening)

You can nest loops inside a comprehension to flatten a list of lists:

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
print(flat)   # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

> **When NOT to use comprehensions:** If the logic requires multiple `if/else` branches, variable assignments, or is longer than ~80 characters, use a regular `for` loop instead. Readability always wins.

> Go to **Exercise 1 (Loop Refactoring Workshop)** where you will convert traditional loops into clean comprehensions.

---

## 2. Dictionary and Set Comprehensions

The same comprehension syntax works for **dictionaries** and **sets**.

### Dictionary Comprehension

```python
# {key_expression: value_expression for item in iterable}

names = ["Alice", "Bob", "Charlie"]
name_lengths = {name: len(name) for name in names}
print(name_lengths)   # {'Alice': 5, 'Bob': 3, 'Charlie': 7}
```

### Filtering in a Dict Comprehension

```python
scores = {"Alice": 92, "Bob": 67, "Charlie": 85, "Diana": 45}

# Only students who passed (score >= 70)
passed = {name: score for name, score in scores.items() if score >= 70}
print(passed)   # {'Alice': 92, 'Charlie': 85}
```

### Inverting a Dictionary

A very common pattern — swapping keys and values:

```python
country_codes = {"ES": "Spain", "FR": "France", "DE": "Germany"}
code_lookup = {country: code for code, country in country_codes.items()}
print(code_lookup)   # {'Spain': 'ES', 'France': 'FR', 'Germany': 'DE'}
```

### Set Comprehension

```python
# Remove duplicates while transforming
words = ["Hello", "HELLO", "hello", "World", "WORLD"]
unique_lower = {w.lower() for w in words}
print(unique_lower)   # {'hello', 'world'}
```

> Go to **Exercise 2 (Data Transformer)** where you will use dict and set comprehensions to reshape data.

---

## 3. `zip()` — Combining Iterables in Parallel

`zip()` takes two (or more) iterables and pairs their elements together, position by position.

### Basic Usage

```python
names = ["Alice", "Bob", "Charlie"]
scores = [92, 78, 85]

for name, score in zip(names, scores):
    print(f"{name}: {score}")

# Output:
# Alice: 92
# Bob: 78
# Charlie: 85
```

### Creating a Dictionary from Two Lists

This is one of the most common `zip()` patterns:

```python
keys = ["name", "age", "city"]
values = ["Alice", 22, "Madrid"]

person = dict(zip(keys, values))
print(person)   # {'name': 'Alice', 'age': 22, 'city': 'Madrid'}
```

### Zipping More Than Two Lists

```python
names = ["Alice", "Bob", "Charlie"]
ages = [22, 25, 30]
cities = ["Madrid", "London", "Paris"]

for name, age, city in zip(names, ages, cities):
    print(f"{name}, age {age}, from {city}")
```

### What Happens with Unequal Lengths?

`zip()` stops at the **shortest** iterable:

```python
a = [1, 2, 3]
b = ["x", "y"]

print(list(zip(a, b)))   # [(1, 'x'), (2, 'y')] — the 3 is dropped
```

### Unzipping with `zip(*...)`

You can "unzip" a list of tuples back into separate lists:

```python
pairs = [("Alice", 92), ("Bob", 78), ("Charlie", 85)]
names, scores = zip(*pairs)

print(names)    # ('Alice', 'Bob', 'Charlie')
print(scores)   # (92, 78, 85)
```

> Go to **Exercise 3 (Parallel Data Merger)** where you will use `zip()` to combine and align multiple data sources.

---

## 4. `enumerate()` — Indexed Iteration

When you need both the **index** and the **value** while looping, use `enumerate()` instead of manually tracking a counter.

### Without `enumerate()` (Non-Pythonic)

```python
fruits = ["apple", "banana", "cherry"]
i = 0
for fruit in fruits:
    print(f"{i}: {fruit}")
    i += 1
```

### With `enumerate()` (Pythonic)

```python
fruits = ["apple", "banana", "cherry"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
```

### Custom Start Index

```python
# Start counting from 1 instead of 0
for rank, fruit in enumerate(fruits, start=1):
    print(f"#{rank}: {fruit}")

# Output:
# #1: apple
# #2: banana
# #3: cherry
```

### Finding the Index of a Maximum Value

```python
scores = [72, 95, 88, 61, 90]
max_score = max(scores)

for i, score in enumerate(scores):
    if score == max_score:
        print(f"Highest score is {score} at position {i}")
```

> You already used `enumerate()` in Session III. In this session, you'll combine it with comprehensions and other idioms.

---

## 5. `lambda` — Small Anonymous Functions

A **lambda** is a tiny, unnamed function defined in a single line. Lambdas are not meant to replace `def` — they are used for short, throwaway operations, typically passed as arguments to other functions.

### Syntax

```python
# Regular function
def double(x):
    return x * 2

# Equivalent lambda
double = lambda x: x * 2

print(double(5))   # 10
```

### Lambdas Are Most Useful as Arguments

You rarely assign a lambda to a variable. Instead, you pass it directly to functions like `sorted()`, `map()`, or `filter()`:

```python
students = [("Alice", 92), ("Bob", 78), ("Charlie", 85)]

# Sort by score (second element of each tuple)
by_score = sorted(students, key=lambda s: s[1])
print(by_score)   # [('Bob', 78), ('Charlie', 85), ('Alice', 92)]
```

### Multi-Argument Lambdas

```python
add = lambda x, y: x + y
print(add(3, 7))   # 10
```

### When NOT to Use Lambdas

If the logic is complex or needs multiple lines, use a regular `def` function:

```python
# Don't do this, too complex for a lambda
process = lambda x: x.strip().lower().replace(" ", "_") if isinstance(x, str) else str(x)

# Do this instead
def process(x):
    """Clean and normalize a value."""
    if isinstance(x, str):
        return x.strip().lower().replace(" ", "_")
    return str(x)
```

> Go to **Exercise 4 (Lambda Sorting Lab)** where you will use lambdas as sorting keys for complex data.

---

## 6. `map()` and `filter()` — Functional Transformations

`map()` and `filter()` apply a function to every item in an iterable. They come from the **functional programming** paradigm.

### `map(function, iterable)` — Transform Every Item

```python
numbers = [1, 2, 3, 4, 5]

# Using map with a lambda
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)   # [2, 4, 6, 8, 10]

# Equivalent list comprehension
doubled = [x * 2 for x in numbers]
```

### `map()` with a Named Function

```python
def celsius_to_fahrenheit(c):
    return round(c * 9/5 + 32, 1)

temps_c = [0, 20, 37, 100]
temps_f = list(map(celsius_to_fahrenheit, temps_c))
print(temps_f)   # [32.0, 68.0, 98.6, 212.0]
```

### `filter(function, iterable)` — Keep Only Matching Items

The function must return `True` or `False`. Only items where it returns `True` are kept:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Keep only even numbers
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)   # [2, 4, 6, 8, 10]

# Equivalent list comprehension
evens = [x for x in numbers if x % 2 == 0]
```

### Chaining `map()` and `filter()`

You can chain them together for a pipeline:

```python
# Get the squares of only the even numbers
numbers = range(1, 11)
result = list(map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, numbers)))
print(result)   # [4, 16, 36, 64, 100]

# Equivalent (and often more readable) comprehension
result = [x ** 2 for x in range(1, 11) if x % 2 == 0]
```

### `map()` vs. Comprehensions — When to Use Which?

| Use `map()` / `filter()` when... | Use a comprehension when... |
|-----------------------------------|-----------------------------|
| You already have a named function | You need an inline expression |
| Working in a functional style | You want maximum readability |
| Chaining with other functional tools | The logic includes `if/else` |

> Go to **Exercise 5 (Functional Data Pipeline)** where you will chain `map()` and `filter()` to clean and transform datasets.

---

## 7. `sorted()` with `key` — Custom Sorting

The built-in `sorted()` function returns a new sorted list. The `key` parameter lets you control **what** Python sorts by.

### Sorting Basics

```python
numbers = [5, 2, 8, 1, 9]
print(sorted(numbers))            # [1, 2, 5, 8, 9]
print(sorted(numbers, reverse=True))  # [9, 8, 5, 2, 1]
```

### Sorting by a Custom Key

```python
words = ["banana", "Apple", "cherry", "date"]

# Default: uppercase letters come before lowercase (ASCII order)
print(sorted(words))   # ['Apple', 'banana', 'cherry', 'date']

# Case-insensitive sort
print(sorted(words, key=str.lower))   # ['Apple', 'banana', 'cherry', 'date']

# Sort by word length
print(sorted(words, key=len))   # ['date', 'Apple', 'banana', 'cherry']
```

### Sorting Dictionaries in a List

This is where `key=lambda` truly shines:

```python
employees = [
    {"name": "Alice", "salary": 75000},
    {"name": "Bob", "salary": 62000},
    {"name": "Charlie", "salary": 90000},
]

# Sort by salary (ascending)
by_salary = sorted(employees, key=lambda e: e["salary"])
print(by_salary[0]["name"])   # "Bob"

# Sort by salary (descending)
top_earners = sorted(employees, key=lambda e: e["salary"], reverse=True)
print(top_earners[0]["name"])   # "Charlie"
```

### Sorting by Multiple Criteria

Return a **tuple** from the key function to sort by multiple fields:

```python
students = [
    {"name": "Alice", "grade": "A", "score": 92},
    {"name": "Bob", "grade": "A", "score": 88},
    {"name": "Charlie", "grade": "B", "score": 92},
]

# Sort by grade first, then by score (descending)
result = sorted(students, key=lambda s: (s["grade"], -s["score"]))
for s in result:
    print(f"{s['name']}: {s['grade']}, {s['score']}")
# Alice: A, 92
# Bob: A, 88
# Charlie: B, 92
```

> Go to **Exercise 4 (Lambda Sorting Lab)** where you will apply custom sorting to complex data structures.

---

## 8. Ternary Expressions — Inline Conditionals

A **ternary expression** lets you write a simple `if/else` in a single line:

### Syntax

```python
value = true_result if condition else false_result
```

### Examples

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)   # "adult"

# Inside an f-string
score = 85
print(f"Result: {'PASS' if score >= 70 else 'FAIL'}")   # "Result: PASS"

# Inside a list comprehension
numbers = [1, -2, 3, -4, 5]
labels = ["positive" if n > 0 else "negative" for n in numbers]
print(labels)   # ['positive', 'negative', 'positive', 'negative', 'positive']
```

### When NOT to Use Ternary Expressions

If the logic involves `elif` or complex conditions, use a regular `if/elif/else` block:

```python
# Don't nest ternaries, unreadable
label = "high" if score > 90 else "mid" if score > 70 else "low"

# Use a regular conditional
if score > 90:
    label = "high"
elif score > 70:
    label = "mid"
else:
    label = "low"
```

> Ternary expressions appear throughout the exercises — especially inside comprehensions and as arguments to functions.

---

## 9. Unpacking and the `*` Operator

Python lets you **unpack** iterables into individual variables. The `*` operator can capture "the rest."

### Basic Unpacking (Review)

```python
coordinates = (40.4168, -3.7038)
lat, lon = coordinates
print(lat)   # 40.4168
```

### Extended Unpacking with `*`

```python
first, *middle, last = [1, 2, 3, 4, 5]
print(first)    # 1
print(middle)   # [2, 3, 4]
print(last)     # 5

head, *tail = ["Alice", "Bob", "Charlie", "Diana"]
print(head)   # "Alice"
print(tail)   # ["Bob", "Charlie", "Diana"]
```

### Unpacking in Function Calls

```python
def greet(name, age, city):
    print(f"{name}, {age}, from {city}")

data = ["Alice", 22, "Madrid"]
greet(*data)   # Same as greet("Alice", 22, "Madrid")
```

### Swapping Variables (Pythonic)

```python
a, b = 1, 2
a, b = b, a    # Swap without a temp variable!
print(a, b)    # 2, 1
```

> Unpacking is used throughout the exercises — especially with `zip()`, `enumerate()`, and `.items()`.

---

## 10. Putting It All Together: Idiomatic Python Patterns

Here is a side-by-side comparison of **non-Pythonic** vs. **Pythonic** code. These are the transformations you will practice in the exercises:

| Task | Non-Pythonic | Pythonic |
|------|---------------|------------|
| Build a list | `for` loop + `.append()` | List comprehension |
| Combine two lists | Manual index tracking | `zip()` |
| Index + value | `range(len(lst))` | `enumerate()` |
| Simple transform | `def` + `for` + `.append()` | `map()` with lambda |
| Filter items | `for` + `if` + `.append()` | `filter()` or comprehension |
| Sort by attribute | Custom comparison | `sorted(key=lambda)` |
| Simple `if/else` assign | 4-line block | Ternary expression |
| Swap variables | Temp variable | `a, b = b, a` |
| Build a dict from lists | Loop + manual assignment | `dict(zip(keys, values))` |
| Check membership | Loop with flag | `x in set(...)` |

> Go to **Exercise 6 (Pythonic Refactoring Challenge)** — the capstone exercise where you will refactor an entire script from clunky, verbose code into clean, idiomatic Python.

---

## Quick Reference Cheat Sheet

```
┌───────────────────────────────────────────────────────────────┐
│  LIST COMPREHENSION    [expr for x in iterable if cond]       │
│  DICT COMPREHENSION    {k: v for k, v in iterable if cond}    │
│  SET COMPREHENSION     {expr for x in iterable}               │
│  ZIP                   for a, b in zip(list1, list2):          │
│  ENUMERATE             for i, val in enumerate(lst, start=0):  │
│  LAMBDA                lambda x: x * 2                        │
│  MAP                   list(map(func, iterable))               │
│  FILTER                list(filter(func, iterable))            │
│  SORTED + KEY          sorted(data, key=lambda x: x[1])       │
│  TERNARY               val = a if condition else b             │
│  UNPACKING             first, *rest = my_list                  │
│  SWAP                  a, b = b, a                             │
│  DICT FROM LISTS       dict(zip(keys, values))                 │
└───────────────────────────────────────────────────────────────┘
```

---
