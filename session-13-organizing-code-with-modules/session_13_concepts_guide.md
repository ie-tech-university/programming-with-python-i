# Session XIII: Concepts Guide
## Organizing Code with Modules

This session covers how to split a program into modules and packages, how Python finds the code you import, how the `__name__` guard separates importing from running, which standard-library modules to reach for first, and how to add third-party packages to a project with uv.

---

## 1. What Is a Module?

Every `.py` file is a module: a reusable namespace of functions, classes, and constants.

### A Module Is a Reusable Namespace

Every `.py` file can be imported as a **module**. Its functions, classes, and constants become names inside that module, and other files reach them through the module prefix.

**How to read this example:** `main.py` imports `math_utils` and calls `math_utils.add()`. The prefix tells the reader where `add` comes from and prevents collisions when two modules define a function with the same name. Both files sit in the same folder, so `uv run python main.py` finds `math_utils.py`.

```python
# math_utils.py
PI = 3.14159


def add(a: float, b: float) -> float:
    return a + b


def circle_area(radius: float) -> float:
    return PI * radius ** 2
```

```python
# main.py
import math_utils

print(math_utils.add(5, 3))        # 8
print(math_utils.circle_area(2))   # 12.56636
```

Another file can now reuse the formula in `circle_area()` instead of copying it.

---

## 2. Import Styles

Choose how names enter your file, keep imports easy to scan, and know where Python looks for them.

### Import Styles Are Readability Choices

```python
import math_utils                  # full module:    math_utils.add(5, 3)
from math_utils import add, PI     # specific names: add(5, 3)
import math_utils as mu            # alias:          mu.add(5, 3)
```

`import math_utils` keeps the module name visible at each use. `from math_utils import add` gives a shorter call, but it is less obvious where `add` came from. An alias such as `import pandas as pd` is useful when it is a well-known convention.

Avoid wildcard imports such as `from module import *`: they bring an unknown set of names into the file and can silently overwrite your own. Import only what the file uses.

### Import Order (PEP 8)

Put imports at the top of the file, in three groups separated by a blank line:

1. **Standard library:** modules that ship with Python.
2. **Third-party:** packages installed with `uv add`.
3. **Local:** modules from your own project.

**How to read this example:** PEP 8 asks for one module per `import` line (`import json` and `import os`, not `import os, json`); `from module import a, b` is fine. Sorting each group alphabetically makes duplicates easy to spot.

```python
# 1. Standard library
import json
import os

# 2. Third-party
import requests

# 3. Local (your project)
from budget_manager.models import Transaction
```

### How Python Finds Modules

`import name` searches the folders listed in `sys.path`, in order, and uses the first match:

1. the folder of the script you run (`uv run python main.py`), or the current folder with `python -m` and in a notebook;
2. the standard library;
3. the packages installed in the project environment (`.venv`), including your own project when it was created with `uv init --package`.

```python
import sys

for folder in sys.path:
    print(folder)
```

> **Warning:** Because your folder comes first, a file named `random.py`, `json.py`, or `pandas.py` next to your script hides the real module: `import random` loads your file, and `random.randint` fails with an `AttributeError`. Give your modules names that do not clash.

---

## 3. The `__name__` Guard

Make a file usable both as a script and as an importable module.

### The Main Guard Separates Importing from Running

When Python imports a file, it runs every top-level statement in it, so a demo `print` or a menu loop at the top level would run on every import.

Python sets `__name__` to `"__main__"` in the file you run directly, and to the module name (`"math_utils"`) when the file is imported.

**How to read this example:** Run directly, the guard is true and `main()` prints `8`; imported, the guard is false, so `add` and `main` are defined but nothing runs. Keep only the start-up call inside the guard and the logic in functions that other files can import.

```python
# math_utils.py
def add(a: float, b: float) -> float:
    return a + b


def main() -> None:
    print(add(5, 3))


if __name__ == "__main__":
    main()
```

```bash
$ uv run python math_utils.py
8
$ uv run python -c "import math_utils"   # prints nothing
```

### Seeing the Guard Work from a Notebook

Every notebook cell runs in the `__main__` namespace, so a guard typed in a cell is always true. To watch it work, write a real file and then run or import it:

- `%%writefile greetings.py` (the first line of a cell) saves the rest of the cell to that file instead of running it.
- `!` runs a terminal command from a cell.
- `import greetings` loads the file as a module, so its guard is false.

```python
%%writefile greetings.py
def greet(name: str) -> str:
    return f"Hello, {name}!"


if __name__ == "__main__":
    print(greet("Ana"))
```

```python
!uv run python greetings.py
# Hello, Ana!
```

```python
import greetings                 # prints nothing
print(greetings.greet("Luis"))   # Hello, Luis!
```

> Python imports a module only once per session. After you edit the file, call `importlib.reload(greetings)` or restart the kernel.

> Go to **Exercise 1 (Create a Module)**, where you will write a `text_utils.py` module with `%%writefile`, run it as a script, and import it to see the guard in action.

---

## 4. Packages

A package groups related modules under one import name.

### A Package Organizes Related Modules

A **package** is a folder of modules imported under a shared name: `budget_manager.models` is the file `budget_manager/models.py`.

A regular (traditional) package contains an `__init__.py` file, which runs when the package is first imported and can define a small public interface. Keep it short: substantial behavior in `__init__.py` makes imports hard to follow.

> Since Python 3.3, a folder without `__init__.py` can also be imported as a *namespace package*, but for your own projects create regular packages.

```text
budget_manager/
    __init__.py
    models.py
    reports.py
    main.py
```

```python
from budget_manager.models import Transaction
from budget_manager import reports
```

**How to read this example:** Each module owns one responsibility; a package is worth creating when those modules belong together, not merely to have more files.

### Creating a Packaged Project with uv

Let uv create the package. `--package` puts the code under `src/` and installs the project into its own environment, so `import budget_manager` works from any folder of the project.

```bash
$ uv init --package budget-manager
$ cd budget-manager
$ uv run budget-manager
Hello from budget-manager!
```

The project name uses a hyphen; the import name uses an underscore, because a hyphen is not valid in a Python name.

```text
budget-manager/
    pyproject.toml
    README.md
    src/
        budget_manager/
            __init__.py   # def main(): ...
```

The `budget-manager` command comes from `[project.scripts]` in `pyproject.toml`. When you move the start-up code into `main.py`, point the command there:

```toml
[project.scripts]
budget-manager = "budget_manager.main:main"
```

### Running Code Inside a Package

From the project folder, run package code by its import name, never by file path:

```bash
$ uv run budget-manager                   # the [project.scripts] command
$ uv run python -m budget_manager.main    # run one module as a script
```

`python -m budget_manager.main` finds the module through `sys.path` and runs it with `__name__ == "__main__"` as part of its package, so its imports work.

> **Warning:** Running a file by path (`python src/budget_manager/main.py`) is fragile: Python puts `src/budget_manager/` first in `sys.path` instead of `src/`, so `import budget_manager` works only because uv installed the project, and relative imports fail with `ImportError: attempted relative import with no known parent package`.

### Absolute and Relative Imports

An **absolute import** spells out the full path from the package name. PEP 8 recommends it because it is readable and means the same thing in every file.

A **relative import** starts with a dot: `.` is the current package and `..` its parent. It only works in a module imported as part of a package (with `-m` or the project command), never in a file run by path.

**How to read this example:** Both lines import the same class. This course uses absolute imports; recognize relative ones when you read other projects.

```python
# src/budget_manager/reports.py

# absolute import (recommended by PEP 8)
from budget_manager.models import Transaction

# relative import: models.py in this package
from .models import Transaction
```

### A Practical Project Layout

```text
budget-manager/
    pyproject.toml    # dependencies + command
    README.md         # how to run the project
    src/
        budget_manager/
            __init__.py
            models.py     # Transaction, Budget
            storage.py    # JSON persistence
            reports.py    # calculations, formatting
            helpers.py    # small reusable utilities
            main.py       # command-line flow
```

**How to read this example:** Each module under `src/budget_manager/` owns one responsibility and `main.py` coordinates them; this is also the layout of the group project.

This layout is not mandatory for every small script. The goal is not to create many files; it is to make responsibilities easy to locate once the code has enough behavior to justify separation.

Start simply, then extract a module when a clear responsibility emerges.

> Go to **Exercise 2 (Package Structure)**, where you will build an `inventory` package with separate model, validator, and inventory modules and run it from a script.

---

## 5. Standard Library Highlights

Modules that ship with Python, so you do not have to reinvent them.

| Module | Use it for |
|--------|------------|
| `pathlib` | Paths: `/` to join, `.exists()`, `.mkdir()` |
| `json` | Python data ↔ JSON text |
| `csv` | Tabular text with correct quoting |
| `datetime` | Dates and times, not plain strings |
| `re` | Regular expressions |
| `collections` | `Counter`, `defaultdict` |
| `argparse` | Command-line options |
| `os` | Environment variables, OS services |

**How to read this example:** `Counter` tallies values and `most_common()` returns (value, count) pairs; `defaultdict(list)` creates the empty list the first time a key is used; `Path` joins paths with `/` and creates folders and files.

```python
from collections import Counter, defaultdict
from pathlib import Path

categories = ["Food", "Transport", "Food", "Rent", "Food"]
counts = Counter(categories)
print(counts.most_common(2))
# [('Food', 3), ('Transport', 1)]

groups = defaultdict(list)
groups["Food"].append("Lunch")
print(dict(groups))   # {'Food': ['Lunch']}

folder = Path("reports")
folder.mkdir(exist_ok=True)
report = folder / "summary.txt"
report.write_text("Food: 3", encoding="utf-8")
print([f.name for f in folder.iterdir()])
# ['summary.txt']
```

> Go to **Exercise 3 (Standard Library)**, where you will count words with `Counter`, group students with `defaultdict`, and create files with `Path`.

---

## 6. Installing Packages

Third-party packages belong to the **project environment**, not just to your computer: another developer needs the list of dependencies to recreate it.

**How to read this example:** `uv add` records each dependency in `pyproject.toml`, pins exact versions in `uv.lock`, and installs them into the project's `.venv`. `uv add -r requirements.txt` imports an existing requirements file the same way.

```bash
$ uv add requests pandas duckdb
$ uv add -r requirements.txt
$ uv sync
```

A teammate then runs `uv sync` to get the same environment. Install only the libraries the project actually uses.

> **Warning:** Avoid `uv pip install`: it installs a package without recording it, and the next `uv sync` removes it again.

---

## 7. Writing Code That Scales

A module is not just a place to put functions. It is a unit of responsibility.

A good rule is:
- one module handles one area of concern
- one function does one job well
- the main script coordinates the flow rather than containing all logic

### From One Script to a Small Project

A single file is fine for a quick demo, but once validation, summaries, and reports pile up, responsibilities become hard to find:

```python
# everything in one script
names = ["alice", "bob"]
for name in names:
    print(name.title())
```

**How to read this example:** Extracting a module per responsibility gives each one a name. The three files sit in one folder and run with `uv run python main.py`; Python finds `helpers` and `reports` because the script's folder comes first in `sys.path`.

```python
# helpers.py
def normalize_name(name: str) -> str:
    return name.strip().title()

# reports.py
from helpers import normalize_name

def summary(names: list[str]) -> str:
    cleaned = [normalize_name(n) for n in names]
    return ", ".join(cleaned)

# main.py
from reports import summary

if __name__ == "__main__":
    print(summary(["  alice ", "BOB"]))   # Alice, Bob
```

Each file now has a single responsibility, which makes the code easier to search, debug, test, and extend. Professional Python projects often follow this rule:

- `helpers.py` contains reusable utilities
- `storage.py` manages data access (loading and saving)
- `reports.py` turns data into summaries
- `main.py` controls the workflow
- `README.md` explains how to run the project

When the folder grows, move these modules into a package (Section 4).

> Go to **Exercise 4 (Refactor Monolithic Code (Capstone))**, where you will decide which module each part of a monolithic payroll script belongs in.

> Go to **Exercise 5 (Tiny Package Design)**, where you will split a score report into `helpers.py`, `reports.py`, and `main.py` and run it.

### Keep Dependencies One-Way

A **circular import** occurs when module A imports module B while module B imports module A. Python may reach one module before it has finished defining the names the other expects, and the import fails.

For example, if `models.py` imports a report function from `reports.py` and `reports.py` imports `Transaction` from `models.py`, their responsibilities are tangled. The simpler design is for the report module to depend on the model, while the model does not depend on the report.

Draw arrows from the module that uses a service to the module that provides it: a one-way dependency is easier to reason about.

---

## 8. Common Module Design Mistakes

- Naming a file after a module you need (`random.py`, `json.py`), which hides the real one
- Running package code by file path instead of `uv run python -m package.module`, so imports depend on where you happen to run from
- Having circular imports between files
- Putting all logic inside `if __name__ == "__main__":` instead of in importable functions
- Doing work at import time (reading files, calling APIs, printing) in a module's top level

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

## Quick Reference

**How to read this example:** This is a compact index of module and package terminology; the detailed sections explain how to apply each import and project-structure pattern.

```
MODULE      Any .py file
PACKAGE     Folder of modules (regular package: __init__.py)
IMPORT      import mod / from mod import name / import mod as m
ORDER       stdlib, third-party, local (blank line between)
GUARD       if __name__ == "__main__":
NOTEBOOK    %%writefile mod.py   then   !uv run python mod.py
PROJECT     uv init --package name   (code in src/name/)
RUN         uv run name / uv run python -m pkg.module
STDLIB      pathlib  json  csv  datetime  re  collections
INSTALL     uv add package / uv add -r requirements.txt
```
