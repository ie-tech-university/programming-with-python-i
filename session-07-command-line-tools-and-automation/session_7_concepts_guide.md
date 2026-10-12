# Session VII: Concepts Guide
## Command Line Tools and Automation

This guide covers the Python concepts you will apply throughout the Session VII exercises. Read through the explanations and study the examples **before** attempting the exercises.

> **Why does this matter?**
> A lot of real Python work is not flashy web apps — it is automation. Python is widely used to back up files, process data from the terminal, run scheduled jobs, and connect small scripts into useful workflows. Learning command line tools helps students move from “writing code in a notebook” to “building programs people can actually run.”

---

## 1. Running Python Scripts from the Terminal

A Python script can be executed directly from the command line:

```bash
python app.py
```

You can also pass extra values after the script name. These are called **command-line arguments**:

```bash
python app.py Alice 25
```

Inside Python, those values are available through `sys.argv`.

```python
import sys

print(sys.argv)
# Example output:
# ['app.py', 'Alice', '25']
```

- `sys.argv[0]` is the script name
- `sys.argv[1:]` are the user-provided arguments

### Basic Example

```python
import sys

if len(sys.argv) < 2:
    print("Usage: python app.py <name>")
else:
    name = sys.argv[1]
    print(f"Hello, {name}!")
```

> Go to **Exercise 1 (CLI Greeter and Calculator)** where you will parse a simple argument list manually.

---

## 2. Why `argparse` Is Better Than Manual Parsing

For very small scripts, `sys.argv` is enough. But real tools need:
- optional flags like `--category`
- automatic help messages
- type conversion
- input validation

Python's built-in `argparse` module handles all of this.

### Basic `argparse` Pattern

```python
import argparse

parser = argparse.ArgumentParser(description="Simple calculator")
parser.add_argument("x", type=float)
parser.add_argument("y", type=float)
args = parser.parse_args()

print(args.x + args.y)
```

If the user runs this incorrectly, `argparse` prints a helpful error automatically.

### Optional Arguments

```python
import argparse

parser = argparse.ArgumentParser(description="Expense filter")
parser.add_argument("--category", help="Filter by category")
parser.add_argument("--min", type=float, default=0, help="Minimum amount")
args = parser.parse_args()
```

### Parsing a Custom List

Inside notebooks or tests, you can parse a list directly:

```python
args = parser.parse_args(["--category", "Food", "--min", "20"])
print(args.category)   # Food
print(args.min)        # 20.0
```

> Go to **Exercise 2 (Expense Lister with `argparse`)** and **Exercise 6 (Automation Suite)** where you will build real parsers.

---

## 3. Organizing CLI Programs with Functions and `main()`

As scripts grow, everything should not live in global scope. Break logic into small functions:

```python
def load_data(filename):
    return []

def print_report(data):
    print("Report goes here")

def main():
    data = load_data("expenses.json")
    print_report(data)

if __name__ == "__main__":
    main()
```

### Why use `if __name__ == "__main__":`?

This line means:
- run the script normally when executed directly
- but do **not** run the main program automatically if the file is imported somewhere else

This is essential for reusable modules and testable code.

> You will use this structure repeatedly in the later exercises.

---

## 4. Working with Files and Paths

Automation scripts often read, copy, move, or rename files. Python's `pathlib` module is the modern way to handle paths.

```python
from pathlib import Path

file_path = Path("data/expenses.json")

print(file_path.name)        # expenses.json
print(file_path.suffix)      # .json
print(file_path.exists())    # True or False
```

### Creating Folders Safely

```python
from pathlib import Path

backup_dir = Path("backups")
backup_dir.mkdir(exist_ok=True)
```

### Joining Paths

```python
from pathlib import Path

backup_file = Path("backups") / "expenses_backup.json"
print(backup_file)
# backups/expenses_backup.json
```

### Copying Files

Use `shutil.copy2()` to copy a file while preserving metadata:

```python
import shutil
from pathlib import Path

source = Path("expenses.json")
destination = Path("backups") / "expenses_backup.json"

shutil.copy2(source, destination)
```

> Go to **Exercise 3 (Budget Backup Script)** where you will validate a source file and copy it into a timestamped backup folder.

---

## 5. Reading Structured Data for Automation

Command-line tools often work with **JSON** or **CSV** files.

### JSON Example

```python
import json

with open("expenses.json", "r") as file:
    expenses = json.load(file)

print(expenses[0]["category"])
```

### CSV Example

```python
import csv

with open("expenses.csv", "r", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row["category"], row["amount"])
```

### Validate Before Trusting

Always check that the data you loaded has the expected structure:

```python
if not isinstance(expenses, list):
    print("Error: JSON must contain a list.")
```

```python
required = {"date", "category", "amount"}
if not required.issubset(row.keys()):
    print("Skipping malformed row")
```

> Go to **Exercise 4 (CSV Expense Summary CLI)** and **Exercise 6 (Automation Suite)** where you will load and validate structured data from files.

---

## 6. Timestamps for Backups and Logs

A very common automation pattern is adding a timestamp to file names.

```python
from datetime import datetime

timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
print(timestamp)
# 20261005_224700
```

### Example Backup Name

```python
from pathlib import Path
from datetime import datetime

filename = Path("expenses.json")
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
backup_name = f"{filename.stem}_{timestamp}{filename.suffix}"

print(backup_name)
# expenses_20261005_224700.json
```

This prevents overwriting old backups.

> You will use this pattern in **Exercise 3** and **Exercise 6**.

---

## 7. Automating Repeated Tasks with Schedulers

One of the biggest benefits of command-line tools is that they can run **without you manually opening Python every time**.

### Cron (Linux/macOS)

A cron schedule has 5 fields:

```text
minute hour day_of_month month day_of_week
```

Example:

```text
30 18 * * * python /path/to/backup.py
```

This means: run every day at 18:30.

### Task Scheduler (Windows)

Windows Task Scheduler can run a command like:

```text
python C:\scripts\backup.py
```

at a specific time every day.

### Important Idea

Your Python script does not create the scheduler itself in these exercises. Instead, you will **generate the command or schedule string correctly**, which is a very realistic beginner automation task.

> Go to **Exercise 5 (Scheduling Helper)** where you will generate cron expressions and Windows task commands.

---

## 8. Designing Good Command-Line Interfaces

A clean command-line tool should:
- have a clear purpose
- validate input
- print helpful error messages
- avoid crashing
- separate data logic from display logic

### Example of a Good CLI Design

```python
import argparse

def build_parser():
    parser = argparse.ArgumentParser(description="Expense tool")
    parser.add_argument("filename")
    parser.add_argument("--category")
    return parser

def main(argv=None):
    parser = build_parser()
    args = parser.parse_args(argv)
    print(args)

if __name__ == "__main__":
    main()
```

### A Useful Pattern: Subcommands

Many real tools use commands like:

```bash
python tool.py list
python tool.py backup
python tool.py summary
```

This can be built with `argparse` subparsers.

```python
subparsers = parser.add_subparsers(dest="command")
subparsers.add_parser("list")
subparsers.add_parser("backup")
subparsers.add_parser("summary")
```

> Go to **Exercise 6 (Automation Suite)** where you will combine multiple commands into one reusable CLI tool.

---

## 9. Defensive Programming for Automation Scripts

Automation scripts should fail **gracefully**, not silently and not with confusing crashes.

### Common Defensive Checks

```python
from pathlib import Path

file_path = Path("data.json")

if not file_path.exists():
    print("Error: File not found.")
```

```python
try:
    amount = float("hello")
except ValueError:
    print("Amount must be numeric.")
```

```python
if args.limit < 0:
    print("Limit must be non-negative.")
```

### Why This Matters

Automation often runs on scheduled tasks or shared systems. If the script crashes, the user may not even be present to see the error. Clear validation makes your tools reliable.

> Every exercise in this session uses defensive programming in some way.

---

## Quick Reference Cheat Sheet

```text
┌──────────────────────────────────────────────────────────────┐
│ RUN SCRIPT            python app.py                         │
│ RAW ARGUMENTS         sys.argv                              │
│ ARGPARSE              parser.add_argument(...)              │
│ OPTIONAL FLAG         --category Food                       │
│ PATHS                 Path("folder") / "file.txt"           │
│ CREATE FOLDER         path.mkdir(exist_ok=True)             │
│ COPY FILE             shutil.copy2(src, dst)                │
│ TIMESTAMP             datetime.now().strftime(...)          │
│ JSON LOAD             json.load(file)                       │
│ CSV LOAD              csv.DictReader(file)                  │
│ MAIN GUARD            if __name__ == "__main__":           │
│ CRON FORMAT           minute hour day month weekday         │
└──────────────────────────────────────────────────────────────┘
```

---
