# Session X: Concepts Guide
## Lab — Building an Interactive Personal Budget Manager

This session is a **hands-on lab** where you combine everything from Sessions I–IX into a single, complete application. It is mostly **applied engineering**: the only new tools are small ones, each explained where it first appears: `__repr__`, `math.isfinite()`, `os.path.dirname()`, `os.makedirs()`, `os.replace()`, and running a multi-file project with `uv run python -m src.main`.

---

## 1. Project Architecture

Before writing any code, plan the structure:

**How to read this example:** This tree separates the application into responsibilities: models define transaction data, validators check input, file handling persists records, reports calculate summaries, and the entry point coordinates the workflow. Changing report formatting should not require changing JSON storage.

```
budget_manager/            # project root: run commands here
├── src/
│   ├── __init__.py        # marks src/ as a package
│   ├── models.py          # OOP: Transaction class, clean_records()
│   ├── file_handler.py    # File I/O: JSON save/load, CSV export
│   ├── validators.py      # Input validation and error handling
│   ├── reports.py         # Summary and reporting functions
│   └── main.py            # Entry point: menu loop
├── data/
│   ├── budget.json        # Saved transactions
│   └── export.csv         # Written by the CSV export
├── pyproject.toml
├── README.md
└── .gitignore
```

> Run the app from the project root with `uv run python -m src.main`. The paths `data/budget.json` and `data/export.csv` are relative, so Python resolves them from the folder you run the command in. Section 7 shows how `main.py` imports the other files; Session XIII covers modules and packages in depth.

### Why This Structure Matters

| Concern | Module | Sessions Applied |
|---------|--------|-----------------|
| Data modeling | `models.py` | Session VIII–IX (OOP) |
| Persistence | `file_handler.py` | Session V (File I/O) |
| Input safety | `validators.py` | Session II (Error Handling), VI (String Cleaning) |
| Reporting | `reports.py` | Session IV (Idioms), III (Data Structures) |
| User interface | `main.py` | Session II (Menus), VII (CLI) |

This is a classic software design principle, **separation of concerns**: each part of the system does one job, so the rest of the code stays simple.

When a user adds an expense, the validators turn text into a positive amount and a known category, the model creates a `Transaction`, the main loop appends it, the file handler saves it, and the reports summarize the same records. If a result is wrong, that flow tells you where to look: bad input points to the validators, a bad record to the model, and a bad total to the reports.

---

## 2. The Model Layer — OOP Design

Design your data as **classes**, not raw dictionaries.

**How to read this example:** This class is a blueprint for one transaction. The two category lists are shared class data. The initializer rejects an amount that is not a positive, finite number and a category that does not match the transaction type, then stores the fields. Every `Transaction` is checked here, so the rest of the app can trust it.

```python
import math
from datetime import datetime


class Transaction:
    """Represents a single income or expense entry."""

    INCOME_CATEGORIES = ["Salary", "Freelance", "Gift", "Other income"]
    EXPENSE_CATEGORIES = ["Food", "Transport", "Entertainment",
                          "Housing", "Utilities", "Other"]

    def __init__(self, amount: float, category: str, description: str,
                 is_income: bool = False) -> None:
        allowed = self.INCOME_CATEGORIES if is_income else self.EXPENSE_CATEGORIES
        if not math.isfinite(amount) or amount <= 0:
            raise ValueError(f"Amount must be a positive number, got {amount}.")
        if category not in allowed:
            raise ValueError(f"Unknown category: {category}.")
        self.amount = float(amount)
        self.category = category
        self.description = description
        self.is_income = is_income
        self.date = datetime.now().isoformat()

    def to_dict(self) -> dict:
        return {
            "amount": self.amount, "category": self.category,
            "description": self.description, "is_income": self.is_income,
            "date": self.date,
        }

    @classmethod
    def from_dict(cls, data: dict) -> Transaction:
        t = cls(data["amount"], data["category"], data["description"], data["is_income"])
        t.date = data["date"]
        return t

    def __repr__(self) -> str:
        kind = "INCOME" if self.is_income else "EXPENSE"
        return f"[{kind}] {self.category}: ${self.amount:.2f} - {self.description}"
```

`to_dict()` turns the object into a plain dictionary that JSON can store. `from_dict()` is a class method that rebuilds the object through `__init__`, so loaded data is validated too. In Python 3.14 the hint `-> Transaction` needs no quotes, even inside the class.

> This applies **Sessions VIII–IX**: classes, `__init__`, `@classmethod`, type hints.

### A Round Trip with to_dict() and from_dict()

A `Transaction` bundles values that belong together. Saving it with `to_dict()` and loading it back with `from_dict()` forms a **round trip**:

**How to read this example:** The example creates a transaction, turns it into a dictionary and rebuilds it. The last two lines show `__repr__` at work, both for `print()` and inside a list.

```python
transaction = Transaction(12.50, "Food", "Lunch")
saved_record = transaction.to_dict()
restored = Transaction.from_dict(saved_record)

print(restored.amount)       # 12.5
print(restored)              # [EXPENSE] Food: $12.50 - Lunch
print([restored])            # [[EXPENSE] Food: $12.50 - Lunch]
```

> **New: `__repr__`.** Session VIII used `__str__` for `print()`. `__repr__` returns the text Python shows when it displays an object for a developer, for example inside a list. If a class defines only `__repr__`, `print()` uses it too.

> Go to **Exercise 1 (Define the Transaction Model)**, where you will build this class, with its validation, and test the round trip.

---

## 3. The Validation Layer — Defensive Programming

`input()` always returns a string, so validation solves two problems: **can the text be converted?** (`"twenty"` cannot become a float) and **does the value make sense?** (`-20` is a number but not a valid amount). Wrap every `input()` call in a function that keeps asking until both answers are yes:

**How to read this example:** Invalid text cannot become a float, while zero, negative and non-finite amounts are rejected even though they are numbers.

```python
import math


def get_valid_amount(prompt: str) -> float:
    """Keep asking until the user enters a positive number."""
    while True:
        try:
            value = float(input(prompt))
            if not math.isfinite(value) or value <= 0:
                print("Error: Amount must be a positive number.")
                continue
            return value
        except ValueError:
            print("Error: Please enter a valid number.")
```

> **New: `math.isfinite()`.** `float()` also accepts the text `nan` (not a number) and `inf` (infinity). `math.isfinite()` is `False` for both. Without it, the app would show `$nan` and save `NaN` into the JSON file.

**How to read this example:** The category choice must be an integer inside the displayed range before it is used as an index. The description is stripped first, so a line of spaces counts as empty.

```python
def get_valid_category(categories: list[str]) -> str:
    """Display a numbered menu and return the selected category."""
    while True:
        print("\nCategories:")
        for i, cat in enumerate(categories, 1):
            print(f"  [{i}] {cat}")
        try:
            choice = int(input("Select a category: "))
            if 1 <= choice <= len(categories):
                return categories[choice - 1]
            print(f"Error: Choose between 1 and {len(categories)}.")
        except ValueError:
            print("Error: Enter a number.")


def get_valid_description() -> str:
    """Keep asking until the description is not empty."""
    while True:
        text = input("Description: ").strip()
        if text:
            return text
        print("Error: Description cannot be empty.")
```

Each loop lets the user fix a mistake without restarting the app. This applies **Session II** (try/except, while loops) and **Session VI** (cleaning input with `strip()`).

> Go to **Exercise 2 (Input Validation Functions)**, where you will write all three helpers and test them with bad input.

---

## 4. The Persistence Layer — JSON File I/O

Variables disappear when the program ends; writing the records to JSON keeps them between runs. Saving must never destroy the previous file, even if something fails halfway:

**How to read this example:** `json.dump()` writes to a temporary file. Only after the write has finished does `os.replace()` swap it into place, so an error during the dump leaves the old `budget.json` untouched. Opening the real file with `"w"` would empty it before writing.

```python
import json
import os


def save_transactions(transactions: list[dict],
                      filepath: str = "data/budget.json") -> None:
    """Write to a temporary file first, then swap it into place."""
    os.makedirs(os.path.dirname(filepath) or ".", exist_ok=True)
    temp_path = filepath + ".tmp"
    with open(temp_path, "w", encoding="utf-8") as f:
        json.dump(transactions, f, indent=2)
    os.replace(temp_path, filepath)   # old file intact until here
    print(f"Saved {len(transactions)} transactions to {filepath}.")
```

> **New tools from `os`.** `os.path.dirname("data/budget.json")` returns the folder part, `data` (an empty string for a bare file name, hence `or "."`). `os.makedirs(folder, exist_ok=True)` creates the folder if it is missing. `os.replace(src, dst)` renames `src` to `dst`, replacing `dst` in one step.

```python
def load_transactions(filepath: str = "data/budget.json") -> list[dict]:
    """Load saved records; set a damaged file aside instead of losing it."""
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)
    except FileNotFoundError:
        print("No saved data yet. Starting fresh.")
        return []
    except json.JSONDecodeError:
        data = None                     # not valid JSON: see below
    if not isinstance(data, list):      # bad JSON or wrong shape
        backup = filepath + ".bak"
        os.replace(filepath, backup)    # keep it for repair
        print(f"Warning: {filepath} is damaged; moved it to {backup}.")
        return []
    return data
```

**How to read this example:** Each `except` handles one situation. `FileNotFoundError` is a normal first run: there is no saved budget yet. A file that is not valid JSON, or valid JSON that is not a list, is damaged: it is moved to `budget.json.bak` with a warning, so a later Save & Quit cannot overwrite the user's data.

Bad individual records are handled in section 8.

> This applies **Session V** (File I/O, JSON, error handling).

> Go to **Exercise 3 (JSON Persistence)**, where you will write both functions and test them with a missing and a damaged file.

---

## 5. The Reporting Layer — Data Analysis

A list of transactions is raw information. A report answers questions such as “What is my balance?” and “Which expense category is largest?”

**How to read this example:** The first generator expression reads as “add the amount for each transaction that is income”; expenses use the opposite condition. The dictionary `by_category` acts as a set of running counters: each category is a key, and its accumulated amount is the value.

```python
def generate_report(transactions: list[dict]) -> None:
    if not transactions:
        print("No transactions to report.")
        return

    income = sum(t["amount"] for t in transactions if t["is_income"])
    expenses = sum(t["amount"] for t in transactions if not t["is_income"])
    balance = income - expenses

    by_category = {}
    for t in transactions:
        if not t["is_income"]:
            cat = t["category"]
            by_category[cat] = by_category.get(cat, 0) + t["amount"]

    print("\n" + "=" * 50)
    print("         BUDGET SUMMARY REPORT")
    print("=" * 50)
    print(f"  Total Income:   ${income:>10.2f}")
    print(f"  Total Expenses: ${expenses:>10.2f}")
    print(f"  Balance:        ${balance:>10.2f}")
    if by_category:                    # no heading for an income-only list
        print("-" * 50)
        print("  Expenses by Category:")
        for cat, total in sorted(by_category.items(), key=lambda x: x[1], reverse=True):
            pct = total / expenses * 100   # by_category is not empty, so expenses > 0
            print(f"    {cat:<20} ${total:>8.2f}  ({pct:.1f}%)")
    print("=" * 50)
```

**How to read this example:** The category block only appears when there is at least one expense, so an income-only report has no empty heading. The percentage needs no extra zero check: every amount is positive, so a non-empty `by_category` means `expenses > 0`. Sorting puts the largest category first.

> This applies **Session III** (dictionaries) and **Session IV** (generator expressions, lambda, sorted).

> Go to **Exercise 5 (Summary Report)**, where you will build this report and test it with an empty and an income-only list.

---

## 6. CSV Export

JSON is convenient for storing structured application data; CSV is convenient for spreadsheets and analysis tools:

**How to read this example:** The writer maps dictionary fields to columns, writes a header, and then writes records. `newline=""` lets the csv module control line endings on every platform, and the empty check avoids reporting an export when there were no rows.

```python
import csv
import os


def export_to_csv(transactions: list[dict], filepath: str = "data/export.csv") -> None:
    if not transactions:
        print("Nothing to export.")
        return
    fieldnames = ["date", "category", "description", "amount", "is_income"]
    os.makedirs(os.path.dirname(filepath) or ".", exist_ok=True)   # create data/ if missing
    with open(filepath, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(transactions)
    print(f"Exported {len(transactions)} transactions to {filepath}")
```

> Go to **Exercise 4 (CSV Export)**, where you will write this function and read the file back to check it.

---

## 7. The Main Loop

`main.py` is the only file that knows about all the others. It imports what it needs from each module:

```python
from src.file_handler import export_to_csv, load_transactions, save_transactions
from src.models import Transaction, clean_records
from src.reports import generate_report
from src.validators import get_valid_amount, get_valid_category, get_valid_description


def ask_transaction(is_income: bool) -> Transaction:
    """Ask for one income or expense, using the validation helpers."""
    if is_income:
        categories = Transaction.INCOME_CATEGORIES
    else:
        categories = Transaction.EXPENSE_CATEGORIES
    amount = get_valid_amount("Amount: $")
    category = get_valid_category(categories)
    description = get_valid_description()
    return Transaction(amount, category, description, is_income)
```

**How to read this example:** `ask_transaction()` picks the right category list, then delegates every question to a validation helper; the `Transaction` it returns is already valid.

> Run it from the project root with `uv run python -m src.main`. The `-m` option runs `main.py` as part of the `src` package, so `from src.models import …` works; `python src/main.py` would fail with `ModuleNotFoundError`.

The menu loop is the coordinator: it displays choices, reads one and calls the right operation. It does not parse JSON, validate input or compute totals itself, which keeps it far easier to maintain than one giant `while True` loop with everything mixed together.

**How to read this example:** The loop starts from the cleaned saved records (`clean_records()` is in section 8). Options 1 and 2 share one helper, and `unsaved` remembers that there are changes to save.

```python
def main() -> None:
    transactions = clean_records(load_transactions())
    unsaved = False
    while True:
        print("\n=== PERSONAL BUDGET MANAGER ===")
        print("[1] Add Income  [2] Add Expense  [3] View All")
        print("[4] Report  [5] Export CSV  [6] Save & Quit  [Q] Quit")
        choice = input("Select: ").strip().lower()
        if choice in ("1", "2"):
            transaction = ask_transaction(is_income=(choice == "1"))
            transactions.append(transaction)
            unsaved = True
            print(f"Added: {transaction}")
        elif choice == "3":
            if not transactions:
                print("No transactions yet.")
            for transaction in transactions:
                print(transaction)
        elif choice == "4":
            generate_report([t.to_dict() for t in transactions])
        elif choice == "5":
            export_to_csv([t.to_dict() for t in transactions])
        elif choice == "6":
            save_transactions([t.to_dict() for t in transactions])
            break
        elif choice == "q":
            if unsaved:
                answer = input("Quit without saving? (y/n): ").strip().lower()
                if answer != "y":
                    continue
            print("Goodbye!")
            break
        else:
            print("Invalid option: choose 1-6 or Q.")


if __name__ == "__main__":
    try:
        main()
    except (EOFError, KeyboardInterrupt):    # Ctrl-D or Ctrl-C at a prompt
        print("\nInput closed. Quitting without saving.")
```

**How to read this example:** The report, export and save functions work with dictionaries, so each call converts the objects with `[t.to_dict() for t in transactions]`. Q asks before discarding unsaved changes, and an unknown key gets a message.

The `if __name__ == "__main__":` guard runs the menu only when the file is executed, not when it is imported. Catching `EOFError` (Ctrl-D) and `KeyboardInterrupt` (Ctrl-C) ends the program with a message instead of a traceback.

> Go to **Exercise 6 (Main Application Loop)**, where you will wire all the pieces together into this menu.

---

## 8. Professional Habits for Real Budget Apps

A project like this becomes much stronger when you treat the user input and the data files as two different kinds of problems:

- user input is unpredictable and must be validated
- stored data is persistent and must be safe to load and re-save
- repeated calculations should live in functions, not inside the menu loop

### Loading Saved Data Defensively

A saved file can contain records that were edited by hand or written by an older version of the app. Add this function to `models.py`:

**How to read this example:** `clean_records()` converts each saved record with `from_dict()`, so the checks in `__init__` run on it. A record with a missing key, the wrong type or an invalid value is reported and skipped with `continue`, so one bad record cannot stop the app from starting.

```python
def clean_records(records: list[dict]) -> list[Transaction]:
    """Turn saved records into Transactions, skipping invalid ones."""
    transactions = []
    for number, record in enumerate(records, 1):
        try:
            transaction = Transaction.from_dict(record)
        except (KeyError, TypeError, ValueError) as error:
            print(f"Skipping record {number} ({type(error).__name__}: {error})")
            continue
        transactions.append(transaction)
    return transactions
```

Four records, three of them broken in different ways. Only the valid one survives, and every skipped record is reported:

```python
records = [
    {"amount": 12.5, "category": "Food", "description": "Lunch",
     "is_income": False, "date": "2026-10-01T12:30:00"},
    {"amount": -3, "category": "Food", "description": "Typo",
     "is_income": False, "date": "2026-10-02T09:00:00"},
    {"amount": 40, "category": "Pets", "description": "Vet",
     "is_income": False, "date": "2026-10-03T18:00:00"},
    {"amount": 8, "category": "Food", "description": "No date",
     "is_income": False},
]
transactions = clean_records(records)
print(transactions)
# Skipping record 2 (ValueError: Amount must be a positive number, got -3.)
# Skipping record 3 (ValueError: Unknown category: Pets.)
# Skipping record 4 (KeyError: 'date')
# [[EXPENSE] Food: $12.50 - Lunch]
```

### Why This Matters

When your application grows, small issues become expensive:
- a duplicated transaction can distort totals
- a missing category makes reporting confusing
- invalid numbers can crash the program in the middle of a save

The solution is always the same: validate early, keep calculations separate, and print clear feedback.

> Go to **Exercise 6 (Main Application Loop)** to practice the same idea in a realistic setting.

---

## 9. Common Mistakes to Watch For

- Forgetting to convert strings to numeric values before doing math
- Overwriting the only copy of the data file, so one failed save loses everything
- Duplicating the same calculation code in each menu branch
- Mixing business logic and UI code in one big function
- Catching broad exceptions without explaining what went wrong

A small validation layer saves you a lot of debugging time later.

---

## 10. Production Mindset

Before you finish a project, ask:

1. Can the user recover from a bad input?
2. Can the program start with no data?
3. Can it export data without losing information?
4. Does every function do one clear job?
5. Can a future developer read the code without confusion?

This is how professional Python projects stay maintainable over time.

---

## Where to Practice: the Exercises

- Exercise 1 (Define the Transaction Model)
- Exercise 2 (Input Validation Functions)
- Exercise 3 (JSON Persistence)
- Exercise 4 (CSV Export)
- Exercise 5 (Summary Report)
- Exercise 6 (Main Application Loop)

---

## Recap: the Design Checklist

```
✓  OOP: Transaction class with validation, to_dict, __repr__
✓  @classmethod: from_dict for deserialization
✓  File I/O: safe JSON save (temp file + os.replace) and load
✓  CSV: export_to_csv with csv.DictWriter
✓  Validation: a retry loop around every user input
✓  Loops: while True main menu, for loops in reports
✓  Data Structures: list of dicts, category totals in a dict
✓  Generator expressions: sum() over filtered records
✓  Functions: each concern in its own function/module
✓  String Formatting: f-strings with alignment for reports
✓  Type Hints: on every function signature
```
