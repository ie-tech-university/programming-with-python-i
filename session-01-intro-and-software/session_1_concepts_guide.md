# Session I: Concepts Guide
## Professional Python Foundations

Welcome to Python I. This guide covers everything you need to set up a **production-ready Python project** and start writing clean, professional code from day one. Read through the explanations and follow along on your own machine **before** attempting the exercises.

> **Why start with setup?**
> In the real world, writing Python code is only half the job. Professional developers spend significant time on **project structure**, **dependency management**, and **version control**. Getting these right from the start saves hours of debugging later and teaches habits that transfer to any software team.

---

## 1. Installing Your Toolchain

Before writing a single line of Python, you need four tools installed on your machine:

| Tool | What It Does | Download |
|------|-------------|----------|
| **Python** | The language interpreter itself (uv can also install it for you) | [python.org/downloads](https://www.python.org/downloads/) |
| **Visual Studio Code** | A powerful, free code editor | [code.visualstudio.com](https://code.visualstudio.com/) |
| **Git** | Version control system | [git-scm.com](https://git-scm.com/) |
| **uv** | Modern Python package & project manager | [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/) |

### Verifying Your Installation

After installing, open a terminal (PowerShell on Windows, Terminal on macOS/Linux) and run:

```bash
uv --version                # Should print uv 0.x.x
uv python install 3.14      # Let uv install Python 3.14
uv run python --version     # Should print Python 3.14.x
git --version               # Should print git version 2.x.x
code --version              # Should print a version number
```

If any of these fail, revisit the installation step before continuing.

> Why not `python --version`? The command is `python3` on macOS and many Linux systems and often `py` on Windows, so we let uv manage Python for us. On macOS, `code` only works after you run **Shell Command: Install 'code' command in PATH** from the VS Code Command Palette.

### Recommended VS Code Extensions

Once VS Code is installed, add these extensions (search in the Extensions panel):

- **Python** (by Microsoft) — run and debug Python; it also installs **Pylance** (autocomplete and type checking) and the **Python Debugger**
- **Jupyter** — open and run the `.ipynb` notebooks used for every exercise
- **Ruff** (optional) — linting and formatting
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
| Python version management | Requires `pyenv` separately | Built-in: `uv python install 3.14` |

### Essential `uv` Commands

| Command | What It Does |
|---------|-------------|
| `uv init <project-name>` | Initialize a new Python project with default structure |
| `uv venv` | Create a new virtual environment in the current project |
| `uv add <package>` | Add a package to the project's dependencies |
| `uv remove <package>` | Remove a package from the project's dependencies |
| `uv add --upgrade <package>` | Upgrade a package to its latest version |
| `uv add -r requirements.txt` | Add all packages from a requirements file to the project (unlike `uv pip install`, they are recorded in `pyproject.toml`) |
| `uv run script.py` | Run a Python script inside the project's environment |
| `uv sync` | Install all dependencies to match the `uv.lock` file |
| `uv tool install <tool>` | Install a CLI tool globally (e.g., `ruff`, `black`) |
| `uvx <tool> [args]` | Run a CLI tool without installing it permanently |

> Go to **Exercise 1 (Project Setup (Terminal Exercise))**, where you will create your first project using `uv`.

---

## 3. Creating a Python Project with `uv`

### Scaffolding a New Project

```bash
$ uv init --no-package ie-python-course
```

This single command creates a complete project structure (`--no-package` keeps it a simple project with a `main.py`; without it, recent uv versions create a `src/` package instead):

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
requires-python = ">=3.14"
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
Using CPython 3.14.4
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
- A **snapshot** of all tracked files at that moment (the full state, not just the edits)
- A reference to its **parent commit** (the one before it)
- A unique **hash** (e.g., `fb2d2ec5069fc6776c80b3ad6b7cbde3cade4e1a`)
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

### One-Time Git Setup

Git stamps every commit with your name and email. Set them **once per computer**, before your first commit (otherwise `git commit` fails with *Author identity unknown*):

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "you@example.com"
# Optional: new repositories start on a branch called main
$ git config --global init.defaultBranch main
```

#### Signing in to GitHub

GitHub does not accept your account password for `git push`. Pick one option:
- **GitHub CLI**: install `gh` and run `gh auth login` (choose HTTPS and log in with the browser); Git then uses that login
- **Git Credential Manager** (included with Git for Windows): the first `git push` opens a browser window to sign in
- **Personal access token**: create one on GitHub (Settings → Developer settings) and paste it when Git asks for a password

### The Three Git Stages

Your local repository has three "areas" managed by Git:

1. **Working Directory** — your actual files on disk
2. **Staging Area (Index)** — files you have marked to include in the next commit
3. **Repository** — the committed history (`HEAD` is a pointer to the commit you are currently on)

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
| `git switch <branch>` | Switch to a different branch (older form: `git checkout <branch>`) |
| `git switch -c <name>` | Create a new branch and switch to it |
| `git merge <branch>` | Merge another branch into the current one |
| `git remote add origin <url>` | Link your local repo to a GitHub repo |
| `git push -u origin main` | Push your commits to GitHub |
| `git pull origin main` | Pull the latest changes from GitHub |

### The `.gitignore` File

A `.gitignore` file tells Git which files to **never track**. This is critical for keeping your repository clean. `uv init` creates one for you; it is a starting point you can extend.

What `uv init` generates:

```gitignore
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info

# Virtual environments
.venv
```

Lines worth adding:

```gitignore
# OS files
.DS_Store
Thumbs.db

# IDE settings
.vscode/
.idea/

# Environment variables (secrets!)
.env
```

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

> Go to **Exercise 1 (Project Setup (Terminal Exercise))**, where you will check with `git status` that `uv init` also created a Git repository; **Exercise 6 (Modular Code & Git Workflow)** then has you make commits and push to GitHub.

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
| `NoneType` | `None` | Represents "no value" |

> Use `type()` to check the type of any value: `print(type(42))` prints `<class 'int'>` and `print(type(None))` prints `<class 'NoneType'>`.

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

> Go to **Exercise 2 (Python Warm-Up)**, where you will practice variables, `print()`, and `input()`.

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

A **docstring** is a string written as the first statement in the function body that describes what the function does. It is enclosed in triple quotes:

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

In `divide`, the `if` block runs only when its condition is true; the preview below and Session 2 cover `if` in detail.

### Common Type Hint Patterns

```python
# Basic types
def process(name: str, age: int, gpa: float, active: bool) -> str:
    ...

# Optional values (can be None)
def find_user(user_id: int) -> str | None:
    ...

# Lists and dicts (collections, covered in Sessions 2 and 3)
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
print(calculate_tip(85.50, 20.0))     # 17.1
```

> Go to **Exercise 3 (Multi-Function Calculator)**, where you will build a complete calculator using functions with type hints and docstrings.

### A Preview of Session 2

Some exercises need a few tools that Session 2 covers in depth. Here is the minimum you need today.

**Decisions with `if` / `elif` / `else`**

```python
average = 85.5

if average >= 90:
    grade = "A"
elif average >= 80:
    grade = "B"
else:
    grade = "C or lower"

print(grade)   # B
```

Python checks the conditions from top to bottom and runs only the first block whose condition is true.

**Repeating with a `for` loop**

```python
for n in range(1, 10, 4):
    print(n, n * n)
# 1 1
# 5 25
# 9 81
```

The indented lines run once per value. `range(start, stop, step)` stops **before** `stop`, so use `range(0, 101, 20)` to include 100.

> Go to **Exercise 4 (Temperature Converter with Type Hints)**, where you will write small functions that call each other and print a conversion table with a loop.

**Lists, tuples and built-in functions**

```python
scores = [90, 85, 92]           # list: values in order
print(len(scores), sum(scores), max(scores))   # 3 267 92

student = ("Alice", [90, 85])   # tuple: a fixed group
name, marks = student           # unpack it
print(name, marks)              # Alice [90, 85]

students = [("Alice", [90, 85]), ("Bob", [78, 81])]
for name, marks in students:
    print(name, sum(marks) / len(marks))
# Alice 87.5
# Bob 79.5
```

`len()`, `sum()` and `max()` count, add up and find the largest item of a list.

**Cleaning and formatting text**

```python
text = "  Hello World  "
print(text.strip().lower())   # hello world

amount = 1234.5
print(f"${amount:,.2f}")      # $1,234.50
```

`.strip()` removes surrounding spaces, `.lower()` converts to lowercase, and the format spec `:,.2f` adds thousands separators and two decimals.

> Go to **Exercise 5 (Student Grade Functions)**, where you will combine functions, `if`/`elif`, lists of tuples and built-ins into a grading utility.

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
├── main.py                  # Entry point: uv run main.py
├── src/
│   ├── __init__.py          # Makes src/ a package (can be empty)
│   ├── calculator.py        # Your module with functions
│   └── utils.py             # Helper/utility functions
├── tests/
│   └── test_calculator.py   # Tests (later in the course)
├── data/                    # Data files (CSV, JSON)
│   └── sample.csv
└── notebooks/               # Jupyter notebooks for exploration
    └── warmup.ipynb
```

`main.py` stays at the project root, and your modules live in `src/`, a package folder (a folder with an `__init__.py` file).

### Modules in a `src/` Package

Instead of putting all code in `main.py`, organize it into modules inside a package folder named `src/`:

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
print(f"10 + 5 = {result}")   # 10 + 5 = 15
```

> Run it from the project root with `uv run main.py`, so Python can find `src`. This simple course convention is not the official PyPA *src layout*, where `src/` holds an installable package such as `src/my_project/`.

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
$ uv init --no-package my-project
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
$ git branch -M main
$ git push -u origin main
```

On Windows PowerShell, `touch` does not exist: use `New-Item src/__init__.py` or create the empty file in VS Code. This is the foundation you will build on for the entire course.

> Go to **Exercise 6 (Modular Code & Git Workflow)**, where you will set up a complete project from scratch, write modular code, and push it to GitHub.

---

## Quick Reference Cheat Sheet

```
UV COMMANDS
  uv init <name>          Create a new project
  uv add <pkg>            Add a dependency
  uv remove <pkg>         Remove a dependency
  uv run script.py        Run inside the venv
  uv sync                 Install from lock file

GIT COMMANDS
  git status              See what changed
  git add .               Stage all changes
  git commit -m "msg"     Save a snapshot
  git push origin main    Upload to GitHub
  git pull origin main    Download from GitHub
  git log --oneline       View commit history

PYTHON BASICS
  variable = value        Assignment
  print(f"text {var}")    Formatted output
  input("prompt")         Read user input (→ str)
  int()  float()  str()   Type conversion

FUNCTIONS
  def name(param: type) -> return_type:
      """Docstring."""
      return result

PROJECT STRUCTURE
  pyproject.toml          Project config
  src/                    Your Python modules
  main.py                 Entry point
  .gitignore              Files Git should ignore
  README.md               Project documentation
```

---
