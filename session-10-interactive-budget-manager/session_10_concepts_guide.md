# Session X: Concepts Guide
## Lab — Building an Interactive Personal Budget Manager

This session is a **hands-on lab** where you combine everything from Sessions I–IX into a single, complete application. There is no new theory — only **applied engineering**.

---

## 1. Project Architecture

Before writing any code, plan the structure:

**How to read this example:** This tree separates the application into responsibilities: models define transaction data, validators check input, file handling persists records, reports calculate summaries, and the entry point coordinates the workflow. Changing report formatting should not require changing JSON storage.

```
budget_manager/
├── src/
│   ├── __init__.py
│   ├── models.py          # OOP: Transaction, Budget classes
│   ├── file_handler.py    # File I/O: JSON and CSV load/save
│   ├── validators.py      # Input validation and error handling
│   ├── reports.py          # Summary and reporting functions
│   └── main.py             # Entry point: menu loop
├── data/
│   └── budget.json         # Persistent data storage
├── pyproject.toml
├── README.md
└── .gitignore
```

### Why This Structure Matters

| Concern | Module | Sessions Applied |
|---------|--------|-----------------|
| Data modeling | `models.py` | Session VIII–IX (OOP) |
| Persistence | `file_handler.py` | Session V (File I/O) |
| Input safety | `validators.py` | Session II (Error Handling), VI (Regex) |
| Reporting | `reports.py` | Session IV (Idioms), III (Data Structures) |
| User interface | `main.py` | Session II (Menus), VII (CLI) |

---

## 2. The Model Layer — OOP Design

Design your data as **classes**, not raw dictionaries:

**How to read this example:** This class is a blueprint for one transaction. The category list is shared class data; the initializer assigns each instance fields; serialization methods convert objects to and from dictionaries; and the representation method makes the object readable during debugging.

```python
from datetime import datetime

class Transaction:
    """Represents a single income or expense entry."""

    VALID_CATEGORIES = ["Food", "Transport", "Entertainment", "Housing", "Utilities", "Other"]

    def __init__(self, amount: float, category: str, description: str, is_income: bool = False):
        self.amount = amount
        self.category = category
        self.description = description
        self.is_income = is_income
        self.date = datetime.now().isoformat()

    def to_dict(self) -> dict:
        return {
            "amount": self.amount, "category": self.category,
            "description": self.description, "is_income": self.is_income,
            "date": self.date
        }

    @classmethod
    def from_dict(cls, data: dict) -> "Transaction":
        t = cls(data["amount"], data["category"], data["description"], data["is_income"])
        t.date = data["date"]
        return t

    def __repr__(self) -> str:
        kind = "INCOME" if self.is_income else "EXPENSE"
        return f"[{kind}] {self.category}: ${self.amount:.2f} — {self.description}"
```

> This applies **Sessions VIII–IX**: classes, `__init__`, `@classmethod`, `__repr__`, type hints.

---

## 3. The Validation Layer — Defensive Programming

Never trust user input. Wrap every `input()` call in a validation function:

**How to read this example:** The helpers handle conversion and business rules separately: invalid text cannot become a float, and zero or negative amounts are rejected even though they are numeric. Category input must convert to an integer and fall within the displayed choices before indexing the list.

```python
def get_valid_amount(prompt: str) -> float:
    """Keep asking until the user enters a positive number."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("Error: Amount must be positive.")
                continue
            return value
        except ValueError:
            print("Error: Please enter a valid number.")

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
```

> This applies **Session II** (try/except, while loops) and **Session VI** (input cleaning).

---

## 4. The Persistence Layer — JSON File I/O

**How to read this example:** The example saves structured records as JSON and restores them on a later run. A missing file can be a normal first-run case, whereas malformed JSON indicates damaged data; the context manager safely closes the file.

```python
import json, os

DATA_FILE = "data/budget.json"

def save_transactions(transactions: list[dict]) -> None:
    os.makedirs(os.path.dirname(DATA_FILE), exist_ok=True)
    with open(DATA_FILE, "w") as f:
        json.dump(transactions, f, indent=2)
    print(f"Saved {len(transactions)} transactions.")

def load_transactions() -> list[dict]:
    try:
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    except FileNotFoundError:
        print("No existing data found. Starting fresh.")
        return []
    except json.JSONDecodeError:
        print("Warning: Data file is corrupted. Starting fresh.")
        return []
```

> This applies **Session V** (File I/O, JSON, error handling).

---

## 5. The Reporting Layer — Data Analysis

**How to read this example:** The generator expressions select income or expense rows before summing. A dictionary accumulates category totals, and the zero check protects percentage calculations when no expenses exist.

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
    print("-" * 50)
    print("  Expenses by Category:")
    for cat, total in sorted(by_category.items(), key=lambda x: x[1], reverse=True):
        pct = (total / expenses * 100) if expenses > 0 else 0
        print(f"    {cat:<20} ${total:>8.2f}  ({pct:.1f}%)")
    print("=" * 50)
```

> This applies **Session III** (dictionaries), **Session IV** (comprehensions, lambda, sorted).

---

## 6. The Main Loop

**How to read this example:** The menu dispatches user choices to operations implemented elsewhere. Keeping validation, persistence, and calculations outside the loop keeps the interface readable and testable.

```python
def main():
    raw_data = load_transactions()
    transactions = [Transaction.from_dict(d) for d in raw_data]

    while True:
        print("\n=== PERSONAL BUDGET MANAGER ===")
        print("[1] Add Income  [2] Add Expense  [3] View All")
        print("[4] Report      [5] Save & Quit  [Q] Quit")

        choice = input("Select: ").strip().lower()
        if choice == "1":
            # add income...
            pass
        elif choice == "5":
            save_transactions([t.to_dict() for t in transactions])
            break
        elif choice == "q":
            break

if __name__ == "__main__":
    main()
```

---

## 7. CSV Export

**How to read this example:** The writer maps dictionary fields to columns, writes a header, and then writes records. The empty check avoids implying that data was exported when no rows existed.

```python
import csv

def export_to_csv(transactions: list[dict], filename: str = "data/export.csv") -> None:
    if not transactions:
        print("Nothing to export.")
        return
    fieldnames = ["date", "category", "description", "amount", "is_income"]
    with open(filename, "w", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(transactions)
    print(f"Exported {len(transactions)} transactions to {filename}")
```

---

## 8. Design Checklist

**How to read this example:** This is a review checklist for the application rather than new executable logic. Each item points to a design choice that helps users recover from mistakes and helps developers locate behavior.

```
┌──────────────────────────────────────────────────────────────────┐
│  ✓  OOP: Transaction class with __init__, __repr__, to_dict     │
│  ✓  @classmethod: from_dict for deserialization                 │
│  ✓  File I/O: JSON save/load with error handling                │
│  ✓  CSV: Export function with csv.DictWriter                    │
│  ✓  Validation: try/except for every user input                 │
│  ✓  Loops: while True main menu, for loops in reports           │
│  ✓  Data Structures: list of dicts, category accumulation       │
│  ✓  Comprehensions: sum() with generator expressions            │
│  ✓  Functions: each concern in its own function/module          │
│  ✓  String Formatting: f-strings with alignment for reports     │
│  ✓  Type Hints: on all function signatures                      │
└──────────────────────────────────────────────────────────────────┘
```

---


---

## 9. Professional Habits for Real Budget Apps

A project like this becomes much stronger when you treat the user input and the data files as two different kinds of problems:

- user input is unpredictable and must be validated
- stored data is persistent and must be safe to load and re-save
- repeated calculations should live in functions, not inside the menu loop

### Defensive Programming Checklist

**How to read this example:** This is a review checklist for the application rather than new executable logic. Each item points to a design choice that helps users recover from mistakes and helps developers locate behavior.

```python
# Good habits for a budget app
for transaction in transactions:
    if transaction["amount"] <= 0:
        raise ValueError("Amounts should be positive.")
    if transaction["category"] not in valid_categories:
        print("Unknown category; skipping record.")
```

### Why This Matters

When your application grows, small issues become expensive:
- a duplicated transaction can distort totals
- a missing category makes reporting confusing
- invalid numbers can crash the program in the middle of a save

The solution is always the same: validate early, keep calculations separate, and print clear feedback.

> Go to **Exercise 6 (Budget Alert + Monthly Summary)** to practice the same idea in a realistic setting.

---

## 10. Common Mistakes to Watch For

- Forgetting to convert strings to numeric values before doing math
- Saving the same data twice without checking the output file
- Recomputing totals from scratch inside every menu option
- Mixing business logic and UI code in one big function
- Catching broad exceptions without explaining what went wrong

A small validation layer saves you a lot of debugging time later.

---

## 11. Production Mindset

Before you finish a project, ask:

1. Can the user recover from a bad input?
2. Can the program start with no data?
3. Can it export data without losing information?
4. Does every function do one clear job?
5. Can a future developer read the code without confusion?

This is how professional Python projects stay maintainable over time.


---

## A Closer Explanation of the Budget Manager Concepts

The earlier sections show the shape of the application. This section explains the decisions behind that shape: what each layer is responsible for, how information moves through the program, and what can go wrong if the responsibilities are mixed together.

### Architecture: Give Each Part One Job

The folders and modules in the project outline are not decoration. They help you answer the question, “Where should I look when this behavior needs to change?”

- If transaction data needs a new field, start in `models.py`.
- If a category selection accepts invalid input, look in `validators.py`.
- If saved data cannot be read, inspect `file_handler.py`.
- If a total is calculated incorrectly, inspect `reports.py`.
- If the menu does not call the right operation, inspect `main.py`.

For example, adding a new report should not require rewriting the JSON-loading code. The report receives transactions and calculates from them; it does not need to know how those transactions were originally stored. This separation makes each part easier to understand and test.

### The Model: Represent One Real Transaction

A `Transaction` object bundles values that belong together: amount, category, description, date, and whether the record represents income. Without a class, those values might be passed around as unrelated variables or copied into dictionaries with slightly different keys.

The `to_dict()` method converts the object into basic data that JSON can serialize. The `from_dict()` class method performs the reverse operation when the program starts again. Together they form a **round trip**:

**How to read this example:** The example creates a transaction, turns it into a dictionary, and reconstructs it. Comparing the restored amount demonstrates a serialization round trip, which is how an object can be saved as JSON-compatible data and restored on a later run.

```python
transaction = Transaction(12.50, "Food", "Lunch")
saved_record = transaction.to_dict()
restored = Transaction.from_dict(saved_record)

print(restored.amount)       # 12.5
print(restored.description)  # Lunch
```

The class also gives the program one consistent place to describe how a transaction should be displayed. `__repr__()` is useful while debugging because printing a transaction can show meaningful information instead of an unhelpful object address.

### Validation: Turn Untrusted Text into Trusted Values

`input()` always returns a string. Even when a person types `25.50`, the program receives `"25.50"`, not a number. Converting it with `float()` can raise `ValueError`; accepting zero or a negative amount may also violate the budget app's rules.

Validation handles these two different problems:

1. **Can the text be converted?** `"twenty"` cannot become a float.
2. **Does the converted value make sense?** `-20.0` is numeric but not a valid positive expense.

**How to read this example:** The text is first converted into a float, then checked against the rule that amounts must be positive. This distinction explains why nonnumeric text and a negative numeric amount are different errors and should receive useful feedback.

```python
text = "25.50"
amount = float(text)

if amount <= 0:
    raise ValueError("The amount must be greater than zero.")
```

A loop around this logic lets the user correct a mistake without restarting the application. A category validator uses a different kind of check: the choice must be an integer and must point to an item in the displayed list. Separating these checks into functions means the menu can ask for a valid amount without knowing how the validation works.

### Persistence: Keep Data Between Program Runs

Variables disappear when a Python process ends. Writing transactions to JSON gives the program a simple persistent store: Python dictionaries and lists are converted into a text representation, and later read back into Python values.

The `with open(...)` block manages the file resource and closes it even if an error occurs. The `indent=2` option makes the resulting JSON easier for a person to inspect. `os.makedirs(..., exist_ok=True)` handles a first run when the `data/` directory does not yet exist.

Two file errors have different meanings:

- `FileNotFoundError` can be a normal first-run situation: there is no saved budget yet.
- `JSONDecodeError` means the file exists but its contents are not valid JSON; that deserves a visible warning because saved data may need repair.

Treating both cases intentionally prevents a missing file from feeling like a crash, while still making damaged data visible.

### Reporting: Convert Records into Useful Answers

A list of transactions is raw information. A report answers questions such as “What was my balance?” and “Which expense category is largest?”

The generator expression below reads as “add the amount for each transaction that is income”:

**How to read this example:** The income and expense expressions filter records by the income flag before summing. A separate category accumulation answers where expenses occurred. This shows how generator expressions and dictionaries turn raw records into a financial summary.

```python
income = sum(t["amount"] for t in transactions if t["is_income"])
```

Expenses use the complementary condition. The balance is income minus expenses. To calculate totals by category, a dictionary acts as a set of running counters: each category is a key, and its accumulated amount is the value.

Percentages need a zero check. If there are no expenses, dividing a category total by `expenses` would raise `ZeroDivisionError`; the conditional expression selects `0` when there is nothing to divide by. Sorting the category totals from largest to smallest then puts the most important result first.

### The Main Loop: Coordinate, Do Not Contain Everything

The menu loop is the application's coordinator. It displays choices, reads a choice, and calls the relevant operation. It should not also implement JSON parsing, category validation, and report mathematics inline.

Think of each menu option as a small journey:

**How to read this example:** The arrows trace a user choice through validation and model creation into the current collection. Each step has a focused responsibility, leaving the menu to coordinate the workflow rather than mixing input, storage, and calculations.

```text
user chooses “Add Expense”
        ↓
ask validation helpers for amount and category
        ↓
create a Transaction object
        ↓
append it to the transaction collection
        ↓
confirm the result to the user
```

The `if __name__ == "__main__":` guard makes the file runnable as a program while allowing its classes and functions to be imported elsewhere without automatically starting the menu.

### CSV Export: Make Data Useful in Other Tools

JSON is convenient for storing structured application data. CSV is convenient for spreadsheet software and many analysis tools. `csv.DictWriter` maps dictionary keys to columns, writes one header row, and then writes one record per transaction.

The `newline=""` argument is recommended when opening CSV output so Python's CSV module can handle line endings consistently across platforms. Returning early for an empty collection avoids creating a misleading “successful export” with no records.

### Putting the Ideas Together

When a user adds an expense, each layer has a clear responsibility:

1. The validator turns typed text into a valid positive number and known category.
2. The model creates a consistent transaction record.
3. The application loop adds that record to the current collection.
4. The persistence layer saves it when requested.
5. The report layer summarizes the same records in a human-friendly way.

If a result is wrong, this flow helps you locate the problem: wrong input points toward validation, a malformed record toward the model, and an incorrect total toward reporting. That is the practical value of a layered design.


---

## Deep Dive: Building a Budget App Step by Step

The real idea behind this session is not just “make a menu.” It is to build a small system with a clear separation of responsibilities:

- the model stores business data (`Transaction` objects)
- the validator protects user input
- the file layer saves and loads data
- the report layer transforms data into decisions

This is a classic software design pattern: each part of the system does one thing, and the rest of the code stays simple.

**How to read this example:** This compact class revisits the transaction model with a lunch example. Attributes keep related values together, and the representation method formats the amount and description for a reader. Compare it with the fuller model to see the same concept in a smaller example.

```python
from datetime import datetime

class Transaction:
    def __init__(self, amount, category, description, is_income=False):
        self.amount = float(amount)
        self.category = category.strip()
        self.description = description.strip()
        self.is_income = is_income
        self.date = datetime.now().isoformat()

    def __repr__(self):
        tag = "INCOME" if self.is_income else "EXPENSE"
        return f"[{tag}] {self.category}: ${self.amount:.2f} - {self.description}"
```

Notice how the class concentrates the main rules in one place. Instead of repeating logic across the app, we keep the rules near the data itself. This is one of the most important habits in professional Python work.

A good budget app also creates a natural flow:

1. user enters data
2. validation checks the values
3. object creation stores the transaction
4. saving persists the update
5. reporting summarizes the dataset

That pipeline is much more maintainable than a single giant `while True` loop with everything mixed together.
