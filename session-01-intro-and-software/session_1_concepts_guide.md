# Session I: Concepts Guide
## Professional Python Foundations and Functional Programming

Welcome to Python I. This guide covers everything you need to set up a **production-ready Python project** and start writing clean, professional code from day one. Read through the explanations and follow along on your own machine **before** attempting the exercises.

> **Why start with setup?**
> In the real world, writing Python code is only half the job. Professional developers spend significant time on **project structure**, **dependency management**, and **version control**. Getting these right from the start saves hours of debugging later and teaches habits that transfer to any software team.

---

## 1. Installing Your Toolchain

Before writing a single line of Python, you need four tools installed on your machine:

| Tool | What It Does | Download |
|------|-------------|----------|
| **Python** | The language interpreter itself | [python.org/downloads](https://www.python.org/downloads/) |
| **Visual Studio Code** | A powerful, free code editor | [code.visualstudio.com](https://code.visualstudio.com/) |
| **Git** | Version control system | [git-scm.com](https://git-scm.com/) |
| **uv** | Modern Python package & project manager | [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/) |

### Verifying Your Installation

After installing, open a terminal (Command Prompt on Windows, Terminal on macOS/Linux) and run:

```bash
python --version       # Should print Python 3.12+ or similar
git --version          # Should print git version 2.x.x
uv --version           # Should print uv 0.x.x
code --version         # Should print a version number
```

If any of these fail, revisit the installation step before continuing.

### Recommended VS Code Extensions

Once VS Code is installed, add these extensions (search in the Extensions panel):

- **Python** (by Microsoft) — syntax highlighting, linting, debugging
- **Pylance** — advanced type checking and autocomplete
- **GitLens** — visualize Git history inside the editor

---

## 2. What Is `uv`?

**uv** is a modern, ultra-fast Python package and project manager built by [Astral](https://astral.sh/) (written in Rust). It aims to replace several older tools — `pip`, `virtualenv`, `pip-tools`, `pipx`, and `pyenv` — with a single, unified workflow.

### Why `uv` Instead of `pip`?

| Feature | `pip` + `venv` (traditional) | `uv` (modern) |
|---------|------------------------------|----------------|
| Speed | Slow for large dependency trees | 10–100× faster (Rust-based resolver) |
| Virtual environments | Manual: `python -m venv .venv` | Automatic: created on first `uv run` |
| Project scaffolding | None — you set up everything yourself | `uv init` creates the full project |
| Lock files | Not built-in | `uv.lock` for reproducible environments |
| Python version management | Requires `pyenv` separately | Built-in: `uv python install 3.12` |

### Essential `uv` Commands

| Command | What It Does |
|---------|-------------|
| `uv init <project-name>` | Initialize a new Python project with default structure |
| `uv venv` | Create a new virtual environment in the current project |
| `uv add <package>` | Add a package to the project's dependencies |
| `uv remove <package>` | Remove a package from the project's dependencies |
| `uv add --upgrade <package>` | Upgrade a package to its latest version |
| `uv pip install -r requirements.txt` | Install all packages from a requirements file |
| `uv run script.py` | Run a Python script inside the project's environment |
| `uv sync` | Install all dependencies to match the `uv.lock` file |
| `uv tool install <tool>` | Install a CLI tool globally (e.g., `ruff`, `black`) |
| `uvx <tool> [args]` | Run a CLI tool without installing it permanently |

> Go to **Exercise 1 (Project Setup)** where you will create your first project using `uv`.

---

## 3. Creating a Python Project with `uv`

### Scaffolding a New Project

```bash
$ uv init ie-python-course
```

This single command creates a complete project structure:

```
ie-python-course/
├── .git/               # Git repository (already initialized!)
├── .gitignore          # Files Git should ignore
├── .python-version     # Pinned Python version
├── README.md           # Project documentation
├── main.py             # Entry-point script
└── pyproject.toml      # Project configuration
```

### Understanding `pyproject.toml`

This is the **central configuration file** for any modern Python project. It replaces the older `setup.py` and `requirements.txt` approach:

```toml
[project]
name = "ie-python-course"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []
```

When you add dependencies, they appear here automatically:

```bash
$ uv add requests
```

```toml
dependencies = [
    "requests>=2.32.3",
]
```

### Running Your Project

```bash
$ uv run main.py
Using CPython 3.13.2
Creating virtual environment at: .venv
Hello from ie-python-course!
```

The first time you run this, `uv` automatically:
1. Creates a virtual environment (`.venv/`)
2. Installs all dependencies
3. Runs your script inside that environment

### Managing Dependencies

```bash
# Add a package
$ uv add requests

# Upgrade a package
$ uv add --upgrade requests

# Remove a package
$ uv remove requests

# Sync environment with lock file (useful after pulling from Git)
$ uv sync
```

---

## 4. Version Control with Git & GitHub

### What Is Git?

**Git** is a distributed version control system. It tracks every change you make to your code, creating a complete history that you can navigate, compare, and revert at any time.

Why Git matters:
- **History**: see what changed, when, and who did it
- **Safety**: revert to any previous version if something breaks
- **Collaboration**: multiple people can work on the same project simultaneously
- **Distributed**: every developer has a full copy of the entire history locally

### What Is GitHub?

**GitHub** is the largest web-based hosting service for Git repositories. It adds collaboration features on top of Git:
- A web UI for browsing code and history
- Pull requests for code review
- Issue tracking and project management
- **Your professional portfolio** — employers look at your GitHub

### Core Git Concepts

#### Snapshot

A **snapshot** is how Git saves history. It records the state of all tracked files at a specific moment in time. You decide when to take a snapshot and which files to include.

#### Commit

A **commit** is the act of creating a snapshot. Each commit contains:
- The changes made since the last commit
- A reference to its **parent commit** (the one before it)
- A unique **hash** (e.g., `fb2d2ec5069fc6776c80b3ad6b7cbde3cade4e`)
- A human-readable **message** describing what changed

A project is essentially a chain of commits.

#### Repository

A **repository** (repo) is the collection of all files plus their entire commit history. It can be:
- **Local** — on your own machine
- **Remote** — on a server like GitHub

Key operations between local and remote:

| Operation | What It Does |
|-----------|-------------|
| `git clone` | Copy an entire remote repository to your machine |
| `git pull` | Download new commits from the remote |
| `git push` | Upload your local commits to the remote |

#### Branches

A **branch** is an independent line of development. The main branch is typically called `main` (or `master` in older repos). Developers create branches to work on features separately, then **merge** them back into `main` when ready.

### The Three Git Stages

Your local repository has three "areas" managed by Git:

1. **Working Directory** — your actual files on disk
2. **Staging Area (Index)** — files you have marked to include in the next commit
3. **Repository (HEAD)** — the committed history

The basic workflow:

```bash
# 1. Make changes to files in your Working Directory
# 2. Stage them (tell Git which changes to include)
$ git add main.py

# 3. Commit them (create the snapshot)
$ git commit -m "Add calculator functions"
```

### Essential Git Commands

| Command | What It Does |
|---------|-------------|
| `git init` | Initialize a new Git repository |
| `git status` | Show which files are modified, staged, or untracked |
| `git add <file>` | Stage a file for the next commit |
| `git add .` | Stage **all** changed files |
| `git commit -m "message"` | Create a commit with a descriptive message |
| `git log --oneline` | Show commit history (compact format) |
| `git branch` | List all local branches |
| `git branch <name>` | Create a new branch |
| `git checkout <branch>` | Switch to a different branch |
| `git merge <branch>` | Merge another branch into the current one |
| `git remote add origin <url>` | Link your local repo to a GitHub repo |
| `git push -u origin main` | Push your commits to GitHub |
| `git pull origin main` | Pull the latest changes from GitHub |

### The `.gitignore` File

A `.gitignore` file tells Git which files to **never track**. This is critical for keeping your repository clean:

```gitignore
# Virtual environments
.venv/

# Python cache files
__pycache__/
*.pyc

# OS files
.DS_Store
Thumbs.db

# IDE settings
.vscode/
.idea/

# Environment variables (secrets!)
.env
```

> `uv init` creates a `.gitignore` for you automatically, but you should understand what it contains and why.

### A Typical Git Workflow

```bash
# Start your day — pull latest changes
$ git pull origin main

# Write code...

# Check what changed
$ git status

# Stage and commit
$ git add .
$ git commit -m "Add expense tracking function"

# Push to GitHub
$ git push origin main
```

> Go to **Exercise 1 (Project Setup)** where you will initialize a repo, make commits, and push to GitHub.

---

## 5. Python Fundamentals: Variables, Types, and `print()`

Now that your environment is set up, let's write Python. This section is a rapid overview of the language basics.

### Variables and Assignment

In Python, you create a variable by assigning a value to a name. No type declaration is needed:

```python
name = "Alice"
age = 22
gpa = 3.85
is_enrolled = True
```

### Basic Data Types

| Type | Example | Description |
|------|---------|-------------|
| `str` | `"hello"` | Text |
| `int` | `42` | Whole numbers |
| `float` | `3.14` | Decimal numbers |
| `bool` | `True` / `False` | Logical values |
| `None` | `None` | Represents "no value" |

### The `print()` Function

`print()` outputs values to the terminal:

```python
name = "Alice"
age = 22

print("Hello!")
print(name)
print("Name:", name, "Age:", age)
```

### f-strings (Formatted String Literals)

The most Pythonic way to embed variables inside strings:

```python
name = "Alice"
score = 95.678

print(f"Student: {name}")
print(f"Score: {score:.2f}")       # 95.68 (2 decimal places)
print(f"{'Item':<15} {'Price':>8}") # Aligned columns
```

### Getting User Input

The `input()` function reads text from the terminal. It **always returns a string**:

```python
name = input("What is your name? ")
age_str = input("How old are you? ")
age = int(age_str)                    # Convert to integer

print(f"Hello, {name}! You are {age} years old.")
```

> Go to **Exercise 2 (Python Warm-Up)** where you will practice variables, `print()`, and `input()`.

---

## 6. Functions, Docstrings, and Type Hints

Functions are the **building blocks** of clean, professional Python code. In this course, we write functions from day one.

### Defining a Function

```python
def greet(name):
    """Print a personalized greeting."""
    print(f"Hello, {name}!")

greet("Alice")   # Hello, Alice!
greet("Bob")     # Hello, Bob!
```

### Return Values

A function can **return** a result to the caller:

```python
def add(a, b):
    """Return the sum of two numbers."""
    return a + b

result = add(3, 5)
print(result)   # 8
```

### Default Parameters

```python
def power(base, exponent=2):
    """Raise base to the given exponent (default: squared)."""
    return base ** exponent

print(power(5))       # 25
print(power(5, 3))    # 125
```

### Docstrings

A **docstring** is a string on the first line of a function that describes what it does. It is enclosed in triple quotes:

```python
def calculate_bmi(weight_kg, height_m):
    """Calculate Body Mass Index.

    Args:
        weight_kg: Patient's weight in kilograms.
        height_m: Patient's height in meters.

    Returns:
        The BMI as a float, rounded to 2 decimal places.
    """
    bmi = weight_kg / (height_m ** 2)
    return round(bmi, 2)
```

> **Rule of thumb:** Every function you write in this course should have a docstring. No exceptions.

### Type Hints

**Type hints** annotate what types a function expects and returns. Python does **not enforce** them at runtime, but they serve as documentation and enable powerful editor autocomplete:

```python
def add(a: int, b: int) -> int:
    """Return the sum of two integers."""
    return a + b

def greet(name: str) -> str:
    """Return a greeting string."""
    return f"Hello, {name}!"

def divide(a: float, b: float) -> float | None:
    """Divide a by b. Returns None if b is zero."""
    if b == 0:
        return None
    return a / b
```

### Common Type Hint Patterns

```python
# Basic types
def process(name: str, age: int, gpa: float, active: bool) -> str:
    ...

# Optional values (can be None)
def find_user(user_id: int) -> str | None:
    ...

# Lists and dicts
def average(scores: list[float]) -> float:
    ...

def count_words(text: str) -> dict[str, int]:
    ...
```

### Combining Everything: A Professional Function

```python
def calculate_tip(bill: float, tip_percent: float = 18.0) -> float:
    """Calculate the tip amount for a restaurant bill.

    Args:
        bill: The total bill amount in dollars.
        tip_percent: The tip percentage (default: 18%).

    Returns:
        The tip amount rounded to 2 decimal places.
    """
    tip = bill * (tip_percent / 100)
    return round(tip, 2)


# Usage
print(calculate_tip(85.50))           # 15.39 (default 18%)
print(calculate_tip(85.50, 20.0))     # 17.10
```

> Go to **Exercise 3 (Multi-Function Calculator)** where you will build a complete calculator using functions with type hints and docstrings.

---

## 7. Project Structure Best Practices

As your projects grow, organizing your files becomes critical. Here is the recommended structure for this course:

```
ie-python-course/
├── .git/
├── .gitignore
├── .python-version
├── README.md
├── pyproject.toml
├── src/
│   ├── __init__.py          # Makes src/ a Python package
│   ├── calculator.py        # Your module with functions
│   └── utils.py             # Helper/utility functions
├── tests/
│   └── test_calculator.py   # Tests (later in the course)
├── data/                    # Data files (CSV, JSON)
│   └── sample.csv
└── notebooks/               # Jupyter notebooks for exploration
    └── warmup.ipynb
```

### The `src/` Layout

Instead of putting all code in `main.py`, organize it into modules inside a `src/` folder:

```python
# src/calculator.py

def add(a: float, b: float) -> float:
    """Return the sum of two numbers."""
    return a + b

def subtract(a: float, b: float) -> float:
    """Return the difference of two numbers."""
    return a - b
```

```python
# main.py

from src.calculator import add, subtract

result = add(10, 5)
print(f"10 + 5 = {result}")
```

### The `README.md` File

Every project should have a README that explains:
- What the project does
- How to install and run it
- Any relevant usage examples

```markdown
# IE Python Course

My coursework for Python I at IE University.

## Setup

```bash
uv sync
```

## Run

```bash
uv run main.py
```
```

### The `if __name__ == "__main__"` Pattern

This pattern lets a file work both as an importable module **and** as a standalone script:

```python
# src/calculator.py

def add(a: float, b: float) -> float:
    """Return the sum of two numbers."""
    return a + b

if __name__ == "__main__":
    # This only runs if you execute this file directly
    print(add(3, 5))
```

---

## 8. Putting It All Together

Here is the complete workflow for starting a new Python project professionally:

```bash
# 1. Create the project
$ uv init my-project
$ cd my-project

# 2. Create your source folder
$ mkdir src
$ touch src/__init__.py

# 3. Write your code in src/
#    (use functions, type hints, and docstrings)

# 4. Run your code
$ uv run main.py

# 5. Add dependencies as needed
$ uv add requests

# 6. Track changes with Git
$ git add .
$ git commit -m "Initial project structure"

# 7. Connect to GitHub and push
$ git remote add origin https://github.com/yourusername/my-project.git
$ git push -u origin main
```

This is the foundation you will build on for the entire course.

> Go to **Exercise 4 (Full Project Walkthrough)** where you will set up a complete project from scratch, write modular code, and push it to GitHub.

---

## Quick Reference Cheat Sheet

```
┌──────────────────────────────────────────────────────┐
│  UV COMMANDS                                          │
│    uv init <name>          Create a new project       │
│    uv add <pkg>            Add a dependency            │
│    uv remove <pkg>         Remove a dependency         │
│    uv run script.py        Run inside the venv         │
│    uv sync                 Install from lock file      │
│                                                        │
│  GIT COMMANDS                                          │
│    git status              See what changed             │
│    git add .               Stage all changes            │
│    git commit -m "msg"     Save a snapshot              │
│    git push origin main    Upload to GitHub             │
│    git pull origin main    Download from GitHub         │
│    git log --oneline       View commit history          │
│                                                        │
│  PYTHON BASICS                                         │
│    variable = value        Assignment                   │
│    print(f"text {var}")    Formatted output             │
│    input("prompt")         Read user input (→ str)      │
│    int()  float()  str()   Type conversion              │
│                                                        │
│  FUNCTIONS                                             │
│    def name(param: type) -> return_type:                │
│        """Docstring."""                                 │
│        return result                                   │
│                                                        │
│  PROJECT STRUCTURE                                     │
│    pyproject.toml          Project config               │
│    src/                    Your Python modules          │
│    main.py                 Entry point                  │
│    .gitignore              Files Git should ignore      │
│    README.md               Project documentation        │
└──────────────────────────────────────────────────────┘
```

---
