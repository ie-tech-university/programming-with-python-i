# Session XIII: Concepts Guide
## Organizing Code with Modules

Structure code into modules, packages, use __init__.py, standard library.

---

## 1. What Is a Module?

A **module** is any `.py` file containing functions, classes, or variables.

**How to read this example:** The first file defines a reusable function and constant; the second imports that file and accesses names through its namespace. The prefix clarifies the origin of `add()` and prevents collisions with similarly named functions.

```python
# math_utils.py
def add(a, b): return a + b
PI = 3.14159
```

**How to read this example:** A module defines names that another file can import. Keeping the module prefix visible makes a function's origin clear and prevents accidental name collisions.

```python
# main.py
import math_utils
print(math_utils.add(5, 3))
```

---

## 2. Import Styles

**How to read this example:** These forms trade explicit module qualification for shorter names. Group imports by source and avoid wildcard imports because they hide which names enter the namespace.

```python
import math_utils                    # Full module
from math_utils import add, PI      # Specific items
import math_utils as mu             # Alias
```

### Import Order (PEP 8)

**How to read this example:** These forms trade explicit module qualification for shorter names. Group imports by source and avoid wildcard imports because they hide which names enter the namespace.

```python
# 1. Standard library
import os, json
# 2. Third-party
import requests
# 3. Local
from src.models import Transaction
```

---

## 3. The `__name__` Guard

**How to read this example:** The function can be imported without running the demonstration; the `__name__` check runs it only when the file is executed directly.

```python
# math_utils.py
def add(a, b): return a + b

if __name__ == "__main__":
    print(add(5, 3))  # Only runs when executed directly
```

---

## 4. Packages

A **package** is a folder with `__init__.py`:

**How to read this example:** The tree groups related modules beneath one package path. Imports can identify where a class or function comes from; packages are useful when responsibilities are related, not merely to increase the number of files.

```
src/
├── __init__.py
├── models.py
├── validators.py
└── reports.py
```

**How to read this example:** The directory groups related modules under one package path. Imports reflect that structure; each file should have a distinct responsibility rather than splitting code arbitrarily.

```python
from src.models import Transaction
from src import reports
```

---

## 5. Standard Library Highlights

| Module | Purpose |
|--------|---------|
| `os` | File system |
| `json` | JSON parsing |
| `csv` | CSV files |
| `re` | Regex |
| `datetime` | Dates/times |
| `pathlib` | Modern paths |
| `collections` | Counter, defaultdict |
| `argparse` | CLI parsing |

**How to read this example:** `Counter` counts repetitions, `defaultdict(list)` creates groups conveniently, and `Path` represents filesystem locations. Each tool replaces repetitive bookkeeping.

```python
from pathlib import Path
from collections import Counter, defaultdict

counts = Counter(["a","b","a","c","a"])
print(counts.most_common(2))  # [("a",3),("b",1)]

groups = defaultdict(list)
groups["A"].append("Alice")
```

---

## 6. Installing Packages

**How to read this example:** The directory groups related modules under one package path. Imports reflect that structure; each file should have a distinct responsibility rather than splitting code arbitrarily.

```bash
$ uv add requests pandas duckdb
$ uv pip install -r requirements.txt
```

---

## Quick Reference

**How to read this example:** This is a compact index of module and package terminology; the detailed sections explain how to apply each import and project-structure pattern.

```
MODULE      Any .py file
PACKAGE     Folder with __init__.py
IMPORT      import mod / from mod import func
GUARD       if __name__ == "__main__":
STDLIB      os  json  csv  re  pathlib  collections
INSTALL     uv add package
```

---


---

## 7. Writing Code That Scales

A module is not just a place to put functions. It is a unit of responsibility.

A good rule is:
- one module handles one area of concern
- one function does one job well
- the main script coordinates the flow rather than containing all logic

### Better Structure

**How to read this example:** The helper module defines reusable behavior and the main module imports it to coordinate the workflow. This is a simple one-way dependency between files.

```python
# helpers.py

def normalize_name(name: str) -> str:
    return name.strip().title()

# main.py
from helpers import normalize_name

print(normalize_name("  jane doe  "))
```

This pattern is easier to test and easier to maintain.

> Go to **Exercise 6 (Tiny Package Design)** to build a small group of related modules instead of everything in one file.

---

## 8. Common Module Design Mistakes

- Importing modules from the wrong path
- Having circular imports between files
- Putting all logic inside `if __name__ == "__main__":` blocks
- Saving data in a module without a clear reason
- Using absolute imports when a project is intended to be package-like

A clean project is easier to extend and much easier to debug.

---

## 9. Professional Project Hygiene

Before a project is considered complete, ask:
- Are modules predictable and reusable?
- Are import names clear and small?
- Are the entry points easy to run?
- Can a teammate add a new feature without editing ten files?

Good structure is part of the quality of the solution.


---

## Deep Dive: From One Script to a Small Project

A script works fine at first, but the moment you add multiple responsibilities, it becomes much harder to understand. That is exactly why modules exist.

Imagine all of this in one file:

**How to read this example:** The single-file loop is clear while it handles one task. As validation and reporting are added, responsibilities become harder to locate; extracting a helper gives a transformation a reusable name.

```python
# everything in one script
names = ["alice", "bob"]
for name in names:
    print(name.title())
```

This is fine for a quick demo, but it becomes messy once you add validation, summaries, and report generation. A more maintainable version is:

**How to read this example:** The single-file loop is clear while it handles one task. As validation and reporting are added, responsibilities become harder to locate; extracting a helper gives a transformation a reusable name.

```python
# helpers.py

def normalize_name(name: str) -> str:
    return name.strip().title()

# reports.py
from helpers import normalize_name

def summary(names):
    cleaned = [normalize_name(name) for name in names]
    return ", ".join(cleaned)
```

This separation is important because each file has a single responsibility. That makes the code easier to search, debug, test, and expand later.

Professional Python projects often follow this rule:

- `helpers.py` contains reusable utilities
- `data.py` manages data access
- `main.py` controls the workflow
- `README.md` explains how to run the project

That structure turns a tiny demonstration into a maintainable app.


---

## Understanding Modules, Imports, and Packages in More Detail

As a Python program grows, its main challenge often changes from “How do I write this function?” to “Where should this function live, and how should another part of the program use it?” Modules and packages provide an answer.

### A Module Is a Reusable Namespace

Every `.py` file can be imported as a module. Its functions, classes, and constants become names inside that module:

**How to read this example:** The `geometry` prefix makes the function origin explicit, and the function uses the module constant to calculate circle area. Another file can reuse the formula without copying it, while the namespace avoids ambiguous global names.

```python
# geometry.py
PI = 3.14159

def circle_area(radius: float) -> float:
    return PI * radius ** 2
```

**How to read this example:** A module defines names that another file can import. Keeping the module prefix visible makes a function's origin clear and prevents accidental name collisions.

```python
# main.py
import geometry

area = geometry.circle_area(2)
print(area)
```

The `geometry.` prefix tells the reader where `circle_area` came from. This namespace also helps avoid collisions when two modules contain functions with the same name. For a small project, `from geometry import circle_area` can be convenient; for a large project, explicit module prefixes often make code provenance clearer.

### Import Styles Are Readability Choices

`import math_utils` keeps the module name visible at each use. `from math_utils import add` makes the imported function shorter to call, but it can be less obvious where `add` came from. An alias such as `import pandas as pd` is useful when the alias is conventional and consistently understood.

Avoid wildcard imports such as `from module import *`: they place an unknown set of names into the current namespace and can overwrite names unexpectedly. Import only what the file uses, and group imports into standard-library, third-party, and project-local sections so dependencies are easy to identify.

### The Main Guard Separates Importing from Running

When Python imports a file, it executes the file's top-level statements. This is why placing a menu loop or test print directly at the top level can unexpectedly run it during an import.

**How to read this example:** The guard runs `main()` only when this file is executed directly. When imported, Python defines the function without starting the application, so the module can serve both as a script and as a reusable source of functions.

```python
def main() -> None:
    print("Run the application here.")

if __name__ == "__main__":
    main()
```

When the file is executed directly, Python sets `__name__` to `"__main__"` and calls `main()`. When another file imports it, `__name__` is the module's name, so the application does not start automatically. The functions remain available for reuse.

### A Package Organizes Related Modules

A package is a directory that groups modules under a shared name. For example, `budget_manager/models.py` and `budget_manager/reports.py` express that both modules belong to the `budget_manager` package.

**How to read this example:** The package tree groups models, storage, reports, and the entry point under one import path. A one-way dependency lets higher-level application code use lower-level models and utilities without making those reusable modules import the application back.

```text
budget_manager/
    __init__.py
    models.py
    reports.py
    main.py
```

Then project code can use imports such as `from budget_manager.models import Transaction`. The `__init__.py` file marks the directory as a package in the traditional package layout and can be used to define a small public interface. Keep it simple: putting substantial application behavior in `__init__.py` can make imports harder to understand.

### Standard Library Modules Save You from Reinventing Tools

The standard library ships with Python, so it is often the first place to look for common tasks:

- `pathlib.Path` represents filesystem paths and offers readable operations such as `.exists()` and `.mkdir()`.
- `json` converts between Python data and JSON text.
- `csv` reads and writes tabular text with correct quoting behavior.
- `datetime` represents dates and times rather than leaving them as unstructured strings.
- `collections.Counter` counts repeated values; `defaultdict(list)` creates a list automatically when a new key is first used.
- `argparse` converts command-line options into validated program arguments.

For example, a `Counter` can summarize transaction categories without a manual dictionary initialization loop:

**How to read this example:** `Counter` tallies repeated values and returns item-count pairs from `most_common()`. This is useful for word or category frequencies without manually updating a dictionary; understand the returned pair structure when displaying or unpacking it.

```python
from collections import Counter

categories = ["Food", "Transport", "Food", "Food"]
counts = Counter(categories)
print(counts.most_common(1))  # [('Food', 3)]
```

Choosing a standard-library tool makes intent clear and reduces custom code that would otherwise need to be debugged.

### Third-Party Packages Belong to the Project Environment

`uv add requests pandas duckdb` records dependencies for the project and makes them available in its environment. This is different from merely having a package installed somewhere on the computer: another developer needs a manifest or lock file so the project can be recreated.

Install only the libraries the project actually uses. A reproducible environment, clear imports, and a documented run command make it much more likely that the application works on another machine.

### Avoid Circular Imports by Choosing Dependencies Deliberately

A circular import occurs when module A imports module B while module B imports module A. Python may encounter one module before it has finished defining the names the other module expects.

For example, if `models.py` imports a report function from `reports.py` and `reports.py` imports `Transaction` from `models.py`, their responsibilities may be tangled. Often the simpler design is for the report module to depend on the model, while the model does not depend on the report. Draw arrows from the module that uses a service to the module that provides it; a one-way dependency is easier to reason about.

### A Practical Project Layout

**How to read this example:** This example applies the concept of A Practical Project Layout. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```text
src/
    budget_manager/
        __init__.py
        models.py       # Transaction and Budget data types
        storage.py      # JSON or database persistence
        reports.py      # calculations and report formatting
        main.py         # command-line flow
```

This layout is not mandatory for every small script. The goal is not to create many files; it is to make responsibilities easy to locate once the code has enough behavior to justify separation. Start simply, then extract a module when a clear responsibility emerges.
