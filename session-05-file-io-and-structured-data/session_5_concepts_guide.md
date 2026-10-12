# Session V: Concepts Guide
## File I/O and Structured Data

This guide covers how to **persist data** — saving it to files so it survives after your program ends, and loading it back when the program restarts. Read through the explanations and study the examples **before** attempting the exercises.

> **Why does this matter?**
> Every program you've written so far loses all its data the moment it stops running. Variables only exist in memory. File I/O is what turns a throwaway script into a **real application** — one that remembers things between sessions.

---

## 1. Reading and Writing Text Files

Python makes it easy to read from and write to plain `.txt` files using the built-in `open()` function.

### Opening a File

The `open()` function takes a filename and a **mode**:

| Mode | Meaning | Creates File? | Erases Existing? |
|------|---------|--------------|-----------------|
| `"r"` | Read only | No (raises error) | No |
| `"w"` | Write (overwrite) | Yes | **Yes** |
| `"a"` | Append (add to end) | Yes | No |
| `"r+"` | Read and write | No (raises error) | No |

### Writing to a Text File

```python
# Writing — creates the file if it doesn't exist
file = open("notes.txt", "w")
file.write("Line 1: Hello, world!\n")
file.write("Line 2: Python is great.\n")
file.close()
```

### Reading from a Text File

```python
# Reading the entire file at once
file = open("notes.txt", "r")
content = file.read()
print(content)
file.close()

# Reading line by line (more memory-efficient for large files)
file = open("notes.txt", "r")
for line in file:
    print(line.strip())    # .strip() removes the trailing \n
file.close()
```

### The `with` Statement (Always Use This)

The `with` statement **automatically closes** the file when the block ends — even if an error occurs. This is the Pythonic way to work with files:

```python
# Writing
with open("notes.txt", "w") as file:
    file.write("This file closes automatically.\n")

# Reading
with open("notes.txt", "r") as file:
    for line in file:
        print(line.strip())
```

> **Rule of thumb:** Always use `with open(...)`. Never use `file.close()` manually — it's error-prone and outdated.

### Reading Into a List of Lines

```python
with open("notes.txt", "r") as file:
    lines = file.readlines()    # Returns a list of strings

print(lines)
# ['Line 1: Hello, world!\n', 'Line 2: Python is great.\n']

# Clean version
clean_lines = [line.strip() for line in lines]
print(clean_lines)
# ['Line 1: Hello, world!', 'Line 2: Python is great.']
```

### Appending to a File

```python
with open("log.txt", "a") as file:
    file.write("New entry added.\n")
# Adds to the end without erasing existing content
```

> Go to **Exercise 1 (Daily Journal App)** where you will use text file reading, writing, and appending to build a persistent journal.

---

## 2. Handling File Errors Gracefully

Files live on disk — they can be missing, corrupted, or locked. Defensive programming is essential.

### `FileNotFoundError`

The most common file error: trying to read a file that doesn't exist.

```python
# This crashes if the file is missing
with open("data.txt", "r") as file:
    content = file.read()

# Handle it gracefully
try:
    with open("data.txt", "r") as file:
        content = file.read()
except FileNotFoundError:
    print("File not found. Starting with empty data.")
    content = ""
```

### Checking if a File Exists Before Opening

You can use the `os.path` module to check first:

```python
import os

if os.path.exists("data.txt"):
    with open("data.txt", "r") as file:
        content = file.read()
else:
    print("No data file found.")
```

### Combining Both Approaches

For maximum safety, use both — check first, but still wrap in `try/except`:

```python
import os

def load_data(filename):
    """Safely load text from a file."""
    if not os.path.exists(filename):
        print(f"'{filename}' not found. Starting fresh.")
        return ""
    
    try:
        with open(filename, "r") as file:
            return file.read()
    except Exception as e:
        print(f"Error reading file: {e}")
        return ""
```

> File error handling is used in **every exercise** this session. The pattern above will become second nature.

---

## 3. JSON — Structured Data Storage

**JSON** (JavaScript Object Notation) is the most popular format for storing structured data. It maps directly to Python dictionaries and lists.

### Why JSON?

| Feature | Text Files | JSON Files |
|---------|-----------|------------|
| Stores plain strings | ok | ok |
| Stores numbers, lists, dicts | not (must parse manually) | ok (automatic) |
| Human-readable | ok | ok |
| Used by APIs and databases | not | ok |

### JSON ↔ Python Type Mapping

| JSON | Python |
|------|--------|
| `{}` object | `dict` |
| `[]` array | `list` |
| `"string"` | `str` |
| `123` / `3.14` | `int` / `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

### Writing JSON to a File (`json.dump`)

```python
import json

tasks = [
    {"title": "Buy groceries", "done": False, "priority": "high"},
    {"title": "Read chapter 5", "done": True, "priority": "medium"},
]

with open("tasks.json", "w") as file:
    json.dump(tasks, file, indent=2)
```

This creates a file called `tasks.json` with nicely formatted content:

```json
[
  {
    "title": "Buy groceries",
    "done": false,
    "priority": "high"
  },
  {
    "title": "Read chapter 5",
    "done": true,
    "priority": "medium"
  }
]
```

### Reading JSON from a File (`json.load`)

```python
import json

with open("tasks.json", "r") as file:
    tasks = json.load(file)

print(tasks[0]["title"])   # "Buy groceries"
print(type(tasks))          # <class 'list'>
```

### The Save/Load Pattern

This is the core pattern for any file-based application:

```python
import json

FILENAME = "data.json"

def save_data(data):
    """Save data to a JSON file."""
    with open(FILENAME, "w") as file:
        json.dump(data, file, indent=2)
    print(f"Data saved to {FILENAME}.")

def load_data():
    """Load data from a JSON file, or return empty list if missing."""
    try:
        with open(FILENAME, "r") as file:
            return json.load(file)
    except FileNotFoundError:
        print("No existing data found. Starting fresh.")
        return []
    except json.JSONDecodeError:
        print("Error: File is corrupted. Starting fresh.")
        return []
```

> **`json.JSONDecodeError`** is raised when the file contains invalid JSON (e.g., someone manually edited it and broke the syntax). Always catch this alongside `FileNotFoundError`.

> Go to **Exercise 2 (To-Do List Manager)** where you will build a full CRUD application with JSON persistence.

---

## 4. CSV — Tabular Data

**CSV** (Comma-Separated Values) is the standard format for spreadsheet-like data. Each row is a line, and columns are separated by commas.

### What a CSV File Looks Like

```
name,age,city,score
Alice,22,Madrid,92
Bob,25,London,78
Charlie,30,Paris,85
```

### Reading a CSV with `csv.reader`

```python
import csv

with open("students.csv", "r") as file:
    reader = csv.reader(file)
    header = next(reader)      # Skip the header row
    
    for row in reader:
        name = row[0]
        age = int(row[1])
        score = int(row[3])
        print(f"{name} (age {age}): {score}")
```

### Reading a CSV as Dictionaries (`csv.DictReader`)

`DictReader` automatically uses the header row as dictionary keys — much cleaner:

```python
import csv

with open("students.csv", "r") as file:
    reader = csv.DictReader(file)
    
    for row in reader:
        print(f"{row['name']} from {row['city']}: {row['score']}")
```

Each `row` is a dictionary:
```python
{'name': 'Alice', 'age': '22', 'city': 'Madrid', 'score': '92'}
```

> **Important:** All CSV values are **strings**. You must convert numbers manually: `int(row['age'])`, `float(row['score'])`.

### Writing a CSV with `csv.writer`

```python
import csv

students = [
    ["Alice", 22, "Madrid", 92],
    ["Bob", 25, "London", 78],
    ["Charlie", 30, "Paris", 85],
]

with open("output.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["name", "age", "city", "score"])    # Header
    writer.writerows(students)                             # All data rows
```

> **Always pass `newline=""`** when opening a CSV for writing. Without it, you may get extra blank lines on Windows.

### Writing a CSV from Dictionaries (`csv.DictWriter`)

```python
import csv

students = [
    {"name": "Alice", "age": 22, "city": "Madrid", "score": 92},
    {"name": "Bob", "age": 25, "city": "London", "score": 78},
]

with open("output.csv", "w", newline="") as file:
    fieldnames = ["name", "age", "city", "score"]
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(students)
```

> Go to **Exercise 3 (CSV Expense Importer)** where you will read, validate, and process CSV data.

---

## 5. Validating File Content Before Loading

Never trust file data blindly. Files can be:
- **Missing** → `FileNotFoundError`
- **Empty** → your code may crash on empty lists
- **Corrupted** → invalid JSON, malformed CSV rows
- **Wrong structure** → missing keys, wrong column count

### Validating JSON Data

```python
import json

def load_and_validate_tasks(filename):
    """Load tasks from JSON and validate the structure."""
    try:
        with open(filename, "r") as file:
            data = json.load(file)
    except FileNotFoundError:
        print("File not found.")
        return []
    except json.JSONDecodeError:
        print("File contains invalid JSON.")
        return []
    
    # Validate: must be a list
    if not isinstance(data, list):
        print("Error: Expected a list of tasks.")
        return []
    
    # Validate each item
    valid_tasks = []
    for i, task in enumerate(data):
        if not isinstance(task, dict):
            print(f"  Skipping item {i}: not a dictionary.")
            continue
        if "title" not in task:
            print(f"  Skipping item {i}: missing 'title' key.")
            continue
        valid_tasks.append(task)
    
    print(f"Loaded {len(valid_tasks)} valid tasks.")
    return valid_tasks
```

### Validating CSV Rows

```python
import csv

def load_and_validate_csv(filename, expected_columns):
    """Load CSV and skip rows with wrong column count."""
    valid_rows = []
    skipped = 0
    
    try:
        with open(filename, "r") as file:
            reader = csv.reader(file)
            header = next(reader)
            
            if len(header) != expected_columns:
                print(f"Error: Expected {expected_columns} columns, got {len(header)}.")
                return []
            
            for i, row in enumerate(reader, start=2):
                if len(row) != expected_columns:
                    print(f"  Row {i}: wrong column count ({len(row)}). Skipped.")
                    skipped += 1
                    continue
                valid_rows.append(row)
    
    except FileNotFoundError:
        print(f"File '{filename}' not found.")
        return []
    
    print(f"Loaded {len(valid_rows)} rows. Skipped {skipped}.")
    return valid_rows
```

> Go to **Exercise 4 (File Content Validator)** where you will write a comprehensive validation tool for both JSON and CSV files.

---

## 6. Converting Between Formats

A very practical skill: reading data in one format and exporting it in another.

### JSON → CSV

```python
import json
import csv

# Load from JSON
with open("students.json", "r") as file:
    students = json.load(file)

# Write to CSV
with open("students.csv", "w", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=students[0].keys())
    writer.writeheader()
    writer.writerows(students)

print("Converted JSON → CSV.")
```

### CSV → JSON

```python
import csv
import json

# Load from CSV
with open("students.csv", "r") as file:
    reader = csv.DictReader(file)
    students = list(reader)

# Write to JSON
with open("students.json", "w") as file:
    json.dump(students, file, indent=2)

print("Converted CSV → JSON.")
```

> Go to **Exercise 5 (Format Converter & Exporter)** where you will build a tool that converts between JSON and CSV in both directions.

---

## 7. Designing File-Based Mini Databases

When you don't have a real database, a JSON file can act as a simple **file-based database**. The key is to follow a consistent pattern:

### The CRUD Pattern

**CRUD** stands for **Create, Read, Update, Delete** — the four basic operations of any data store.

```python
import json

DB_FILE = "contacts.json"

def load_db():
    """Read the entire database from disk."""
    try:
        with open(DB_FILE, "r") as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return {}

def save_db(data):
    """Write the entire database to disk."""
    with open(DB_FILE, "w") as f:
        json.dump(data, f, indent=2)

def create_contact(name, phone, email):
    """Add a new contact."""
    db = load_db()
    if name in db:
        print(f"'{name}' already exists.")
        return False
    db[name] = {"phone": phone, "email": email}
    save_db(db)
    print(f"Contact '{name}' created.")
    return True

def read_contact(name):
    """Look up a contact by name."""
    db = load_db()
    return db.get(name, None)

def update_contact(name, **fields):
    """Update specific fields of an existing contact."""
    db = load_db()
    if name not in db:
        print(f"'{name}' not found.")
        return False
    db[name].update(fields)
    save_db(db)
    print(f"Contact '{name}' updated.")
    return True

def delete_contact(name):
    """Remove a contact."""
    db = load_db()
    if name not in db:
        print(f"'{name}' not found.")
        return False
    del db[name]
    save_db(db)
    print(f"Contact '{name}' deleted.")
    return True
```

### Using the CRUD Functions

```python
create_contact("Alice", "555-1234", "alice@email.com")
create_contact("Bob", "555-5678", "bob@email.com")

print(read_contact("Alice"))
# {'phone': '555-1234', 'email': 'alice@email.com'}

update_contact("Alice", phone="555-9999")
delete_contact("Bob")
```

Each function call reads from and writes to `contacts.json`, so the data **persists** between program runs.

> Go to **Exercise 6 (Student Records Database — Capstone)** where you will build a complete file-based database with CRUD operations, CSV import/export, and data validation.

---

## Quick Reference Cheat Sheet

```
┌──────────────────────────────────────────────────────────┐
│  TEXT FILES                                               │
│    open("f.txt", "r"/"w"/"a")                            │
│    with open(...) as file:     ← always use this         │
│    file.read()  file.readlines()  file.write()           │
│                                                          │
│  JSON                                                    │
│    import json                                           │
│    json.dump(data, file, indent=2)   ← write to file    │
│    json.load(file)                   ← read from file   │
│    json.dumps(data)                  ← to string         │
│    json.loads(string)                ← from string       │
│                                                          │
│  CSV                                                     │
│    import csv                                            │
│    csv.reader(file) / csv.DictReader(file)   ← read     │
│    csv.writer(file) / csv.DictWriter(file, fieldnames)   │
│    writer.writerow([...])  writer.writerows([...])       │
│    writer.writeheader()                                  │
│    next(reader)              ← skip header row           │
│    newline="" in open()      ← always for writing CSV    │
│                                                          │
│  FILE ERRORS                                             │
│    FileNotFoundError         ← file doesn't exist        │
│    json.JSONDecodeError      ← invalid JSON              │
│    PermissionError           ← no read/write access      │
│                                                          │
│  FILE CHECKS                                             │
│    import os                                             │
│    os.path.exists("file.txt")    ← True / False         │
│    isinstance(data, list)        ← validate structure    │
│                                                          │
│  PATTERNS                                                │
│    Save/Load: json.dump + json.load with try/except     │
│    CRUD: load_db() → modify → save_db()                 │
│    Validate: check type, check keys, skip bad rows       │
│    Convert: JSON ↔ CSV using DictReader/DictWriter       │
└──────────────────────────────────────────────────────────┘
```

---
