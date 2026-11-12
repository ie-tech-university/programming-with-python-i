# Session VII: Concepts Guide
## Command Line Tools and Automation

This guide covers the Python concepts you will apply throughout the Session VII exercises. Read through the explanations and study the examples **before** attempting the exercises.

> **Why does this matter?**
> A lot of real Python work is not flashy web apps — it is automation. Python is widely used to back up files, process data from the terminal, run scheduled jobs, and connect small scripts into useful workflows. Learning command-line tools helps you move from “writing code in a notebook” to “building programs people can actually run.”

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
    sys.exit("Usage: python app.py <name>")

name = sys.argv[1]
print(f"Hello, {name}!")
```

`sys.exit("message")` prints the message to the error stream (stderr) and ends the script with exit code 1, so whoever ran it knows it failed.

> Go to **Exercise 1 (CLI Greeter and Calculator)**, where you will parse a simple argument list manually.

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
parser.add_argument("--min", type=float, default=0.0, help="Minimum amount")
args = parser.parse_args()
```

Options start with `--` and may be left out: `--category` is then `None` and `--min` takes its default. Write the default as `0.0`, not `0`: argparse converts what the user types with `type=float`, but uses the default exactly as you wrote it.

### Parsing a Custom List

Inside notebooks or tests, you can parse a list directly:

```python
args = parser.parse_args(["--category", "Food", "--min", "20"])
print(args.category)   # Food
print(args.min)        # 20.0

args = parser.parse_args([])          # nothing given: defaults
print(args.category, args.min)  # None 0.0
```

> **In Jupyter, always pass a list.** A bare `parse_args()` reads `sys.argv`, which in a notebook holds the kernel's own arguments (`-f …/kernel-….json`), so argparse stops with `SystemExit: 2`.

> Go to **Exercise 2 (Expense Lister with `argparse`)** and **Exercise 6 (Automation Suite)**, where you will build real parsers.

---

## 3. Organizing CLI Programs with Functions and `main()`

As scripts grow, not everything should live in global scope. Break logic into small functions:

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

Use `shutil.copy2()` to copy a file and its timestamps:

```python
import shutil
from pathlib import Path

source = Path("expenses.json")
destination = Path("backups") / "expenses_backup.json"

shutil.copy2(source, destination)
```

> Go to **Exercise 3 (Budget Backup Script)**, where you will validate a source file and copy it into the `backups` folder under a timestamped file name.

---

## 5. Reading Structured Data for Automation

Command-line tools often work with **JSON** or **CSV** files.

### JSON Example

```python
import json

with open("expenses.json", "r", encoding="utf-8") as file:
    expenses = json.load(file)

print(expenses[0]["category"])
```

### CSV Example

```python
import csv

with open("expenses.csv", "r", newline="", encoding="utf-8") as file:
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

`csv.DictReader` always gives every header key (an empty cell is `""`, a short row gives `None`), so check the values, not the keys:

```python
import csv

valid = []
with open("expenses.csv", newline="", encoding="utf-8") as file:
    for row in csv.DictReader(file):
        if not row.get("category") or not row.get("amount"):
            continue          # empty cell ("") or short row (None)
        try:
            row["amount"] = float(row["amount"])
        except ValueError:
            continue          # "forty" is not a number
        valid.append(row)
```

> Go to **Exercise 4 (CSV Expense Summary CLI)** and **Exercise 6 (Automation Suite)**, where you will load and validate structured data from files.

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

On Linux and macOS, **cron** runs commands on a schedule (edit your list with `crontab -e`). A cron line has 5 schedule fields, followed by the command to run:

```text
minute hour day_of_month month day_of_week  command
```

Example:

```text
30 18 * * * cd /path/to/project && /path/to/project/.venv/bin/python backup.py >> backup.log 2>&1
```

This means: every day at 18:30, go to the project folder, run `backup.py` with the project's own Python, and append its output and errors to `backup.log`.

| Field | Value | Allowed values |
|---|---|---|
| minute | `30` | 0–59 |
| hour | `18` | 0–23 |
| day of month | `*` (every) | 1–31 |
| month | `*` (every) | 1–12 |
| day of week | `*` (every) | 0–7 (0 or 7 = Sunday) |

Why the long command? cron runs your line with almost nothing set up:
- **A minimal PATH:** plain `python` may not exist (macOS and Ubuntu ship `python3`), and cron knows nothing about your uv virtual environment. Use the absolute path to the project's `.venv/bin/python` (or to uv: `/home/you/.local/bin/uv run backup.py`).
- **It starts in your home folder,** so relative paths such as `expenses.json` break. `cd` into the project first, or use absolute paths.
- **Nobody sees the output:** `>> backup.log 2>&1` appends both output and errors to a log file.

### Task Scheduler (Windows)

Windows Task Scheduler runs a command at the time you choose. Create the task in the Task Scheduler app, or from a terminal with `schtasks`:

```text
schtasks /Create /SC DAILY /TN BudgetBackup /TR "C:\budget\.venv\Scripts\python.exe C:\budget\backup.py" /ST 18:30
```

`/SC DAILY` every day · `/TN` task name · `/TR` the command to run · `/ST` start time

- Use the full path to the project's `.venv\Scripts\python.exe`: plain `python` may not be found, or may be a different Python without your packages.
- A task does not start in your project folder. In the Task Scheduler app, set **Start in (optional)** to the project folder so relative paths like `expenses.json` work; the basic command has no option for it, so with `schtasks` use absolute paths in the script.

### Important Idea

In these exercises your Python code does not create the task: it **generates the cron line or schtasks command as a string**, a realistic beginner automation task. Exercise 5 uses plain `python` to keep the strings short; on a real machine, use the full interpreter path.

> Go to **Exercise 5 (Scheduling Helper)**, where you will generate cron expressions and Windows task commands.

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

Many real tools have one script with several commands (`tool.py expenses.json list`, `… backup`, `… summary`). `argparse` builds them with **subparsers**; options that belong to one command go on that command's own parser:

```python
import argparse

parser = argparse.ArgumentParser(description="Expense tool", suggest_on_error=True)
parser.add_argument("filename", help="JSON expense file")
subparsers = parser.add_subparsers(dest="command", required=True)
subparsers.add_parser("list", help="print all expenses")
subparsers.add_parser("backup", help="make a timestamped copy")
summary = subparsers.add_parser("summary", help="totals by category")
summary.add_argument("--category")
summary.add_argument("--min", type=float, default=0.0)

args = parser.parse_args()
print(args)
```

- `required=True` makes argparse stop with an error when no command is given (without it, `args.command` is silently `None`).
- `suggest_on_error=True` (new in Python 3.14) adds a “maybe you meant …?” hint to typos.

### Anatomy of a Command Line

```text
python tool.py expenses.json summary --category Food --min 20
```

- `sys.argv` → `['tool.py', 'expenses.json', 'summary', '--category', 'Food', '--min', '20']` (all strings)
- `argparse` turns it into `Namespace(filename='expenses.json', command='summary', category='Food', min=20.0)`
- positional argument · subcommand · options of the `summary` subcommand, with type conversion, defaults and an automatic `--help`

The order follows the parser: `filename` belongs to the main parser, so it comes first; `--category` and `--min` belong to the `summary` parser, so they come after `summary`.

```text
$ python tool.py expenses.json summary --category Food --min 20
Namespace(filename='expenses.json', command='summary', category='Food', min=20.0)
$ python tool.py summary expenses.json
usage: tool.py [-h] filename {list,backup,summary} ...
tool.py: error: argument command: invalid choice: 'expenses.json' (choose from list, backup, summary)
```

> Go to **Exercise 6 (Automation Suite)**, where you will build this parser (file name first, then the command) and route each command to its own function.

### Help and Errors for Free

The `help=` texts become the `--help` page (real Python 3.14 output):

```text
$ python tool.py --help
usage: tool.py [-h] filename {list,backup,summary} ...

Expense tool

positional arguments:
  filename              JSON expense file
  {list,backup,summary}
    list                print all expenses
    backup              make a timestamped copy
    summary             totals by category

options:
  -h, --help            show this help message and exit
```

A missing or mistyped command stops the script with a usage line, a clear message and exit code 2:

```text
$ python tool.py expenses.json
usage: tool.py [-h] filename {list,backup,summary} ...
tool.py: error: the following arguments are required: command
$ python tool.py expenses.json lst
usage: tool.py [-h] filename {list,backup,summary} ...
tool.py: error: argument command: invalid choice: 'lst', maybe you meant 'list'? (choose from list, backup, summary)
```

---

## 9. Defensive Programming for Automation Scripts

Automation scripts should fail **gracefully**, not silently and not with confusing crashes.

### Common Defensive Checks

When a check fails, stop with a clear message on stderr and a non-zero exit code:

```python
import argparse
import sys
from pathlib import Path

parser = argparse.ArgumentParser(description="Expense filter")
parser.add_argument("filename")
parser.add_argument("--min", type=float, default=0.0)
args = parser.parse_args()

if args.min < 0:
    parser.error("--min must be non-negative")    # usage + message, exit code 2

file_path = Path(args.filename)
if not file_path.exists():
    sys.exit(f"Error: {file_path} not found.")    # message to stderr, exit code 1
```

`type=float` already rejects `--min abc` the same way: usage line, error message, exit code 2.

### Why This Matters

Automation often runs on scheduled tasks or shared systems. If the script fails, nobody may be there to read the message, so the **exit code** is how the scheduler, the shell and other scripts find out:
- Exit code `0` means success; anything else means failure.
- Task Scheduler shows it as the *Last Run Result*; in a shell, `python backup.py && echo done` only continues when it is 0.
- A script that prints “Error” and then ends normally returns 0, so every tool thinks it worked.
- Errors go to stderr (`sys.exit` does it; otherwise use `print(…, file=sys.stderr)`), so `2>&1` can send them to a log.

```text
$ python check.py data.json --min -5
usage: check.py [-h] [--min MIN] filename
check.py: error: --min must be non-negative
$ echo $?
2
$ python check.py data.json
Error: data.json not found.
$ echo $?
1
```

> Every exercise in this session uses defensive programming in some way.

---

## Quick Reference Cheat Sheet

```text
RUN SCRIPT            python app.py
RAW ARGUMENTS         sys.argv
STOP WITH ERROR       sys.exit("Error: ...")   (exit code 1)
ARGPARSE              parser.add_argument(...)
OPTIONAL FLAG         --category Food
SUBCOMMANDS           parser.add_subparsers(dest="command", required=True)
PARSER ERROR          parser.error("...")      (exit code 2)
PATHS                 Path("folder") / "file.txt"
CREATE FOLDER         path.mkdir(exist_ok=True)
COPY FILE             shutil.copy2(src, dst)
TIMESTAMP             datetime.now().strftime("%Y%m%d_%H%M%S")
JSON LOAD             json.load(file)
CSV LOAD              csv.DictReader(file)
MAIN GUARD            if __name__ == "__main__":
CRON FORMAT           minute hour day month weekday command
```

---
