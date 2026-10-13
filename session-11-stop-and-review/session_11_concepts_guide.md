# Session XI: Concepts Guide
## Python I Foundations Review

This session is a **comprehensive review** of every concept from Sessions I–X. Use it as a quick-reference cheat sheet before the second half of the course.

---

## 1. Project Setup (Session I)

**How to read this example:** These commands initialize a project, record dependencies, run the application inside its environment, and save a Git snapshot. Recording dependencies makes the setup repeatable on another machine.

```bash
$ uv init my-project && cd my-project
$ uv add requests pandas
$ uv run main.py
$ git init && git add . && git commit -m "Initial commit"
```

---

## 2. Data Types & Conversion (Session II)

**How to read this example:** The assignments demonstrate built-in types; `int()` and `float()` explicitly convert text to numbers. Conversion can fail for text that is not numeric, so user input needs exception handling.

```python
x = 42        # int
y = 3.14      # float
s = "hello"   # str
b = True      # bool
n = None      # NoneType

age = int("25")       # str -> int
price = float("19.99") # str -> float
```

---

## 3. Control Flow (Session II)

**How to read this example:** The conditional checks thresholds in order; the `for` loop visits a known collection, and the `while` loop repeats until state changes. `break` exits and `continue` skips the current iteration.

```python
if score >= 90:    grade = "A"
elif score >= 80:  grade = "B"
else:              grade = "C"

for item in collection:
    process(item)

while condition:
    if done: break
    if skip: continue
```

---

## 4. Error Handling (Session II)

**How to read this example:** `try` encloses risky work, specific `except` clauses handle known failures, `else` runs only on success, and `finally` always runs for cleanup.

```python
try:
    result = risky_operation()
except ValueError:
    print("Bad value")
except (TypeError, KeyError) as e:
    print(f"Error: {e}")
else:
    print("Success!")
finally:
    print("Always runs")
```

---

## 5. Functions (Sessions I-II)

**How to read this example:** The function names a reusable calculation, type hints communicate the expected interface, and a default argument lets callers omit the usual tax rate. `return` passes the computed result back to its caller.

```python
def calculate_total(price: float, tax_rate: float = 0.21) -> float:
    """Calculate price + tax."""
    return round(price * (1 + tax_rate), 2)
```

---

## 6. Data Structures (Session III)

| Structure | Mutable | Ordered | Duplicates | Use Case |
|-----------|---------|---------|------------|----------|
| `list`    | Yes     | Yes     | Yes        | General  |
| `tuple`   | No      | Yes     | Yes        | Fixed    |
| `set`     | Yes     | No      | No         | Unique   |
| `dict`    | Yes     | Yes*    | Keys: No   | Lookup   |

---

## 7. Pythonic Idioms (Session IV)

**How to read this example:** The comprehension transforms and filters values, `zip()` pairs matching items, `enumerate()` supplies indexes, and the lambda defines the key used to sort records.

```python
squares = [x**2 for x in range(10) if x % 2 == 0]
paired = dict(zip(names, scores))
for i, item in enumerate(collection, 1): ...
students.sort(key=lambda s: s["grade"], reverse=True)
```

---

## 8. File I/O (Session V)

**How to read this example:** Context managers handle file lifetime; JSON represents nested data, while `DictReader` maps CSV columns to keys. The mode determines whether a file is read or overwritten.

```python
import json, csv

with open("data.json", "w") as f: json.dump(data, f, indent=2)
with open("data.json", "r") as f: data = json.load(f)

with open("data.csv", "r") as f:
    rows = list(csv.DictReader(f))
```

---

## 9. Strings & Regex (Session VI)

**How to read this example:** String methods normalize whitespace and case; regular expressions find or replace text matching a pattern. A regex match checks shape, not whether a value is real or deliverable.

```python
import re
clean = raw.strip().lower()
emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', text)
cleaned = re.sub(r'[^a-zA-Z0-9\s]', '', text)
```

---

## 10. CLI & Automation (Session VII)

**How to read this example:** The parser declares a required positional filename and an optional verbose switch. `parse_args()` turns command-line text into attributes the program can use.

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("filename")
parser.add_argument("-v", "--verbose", action="store_true")
args = parser.parse_args()
```

---

## 11. OOP (Sessions VIII-IX)

**How to read this example:** The initializer stores object state, methods use it, and class/static methods are for operations that do not need an instance. Inheritance lets `Dog` reuse initialization with `super()`.

```python
class Animal:
    def __init__(self, name: str, sound: str):
        self.name = name
        self._sound = sound
    def speak(self) -> str:
        return f"{self.name} says {self._sound}!"
    @classmethod
    def from_dict(cls, d): return cls(d["name"], d["sound"])
    @staticmethod
    def is_valid(name): return bool(name) and name[0].isupper()

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name, "Woof")
        self.breed = breed
```

---

## Quick Reference

**How to read this example:** This block is an index of constructs reviewed in the session; use it to locate the full explanation and examples rather than as a program to run.

```
SETUP       uv init / uv add / git add+commit
TYPES       int  float  str  bool  None
FLOW        if/elif/else  for/while  break/continue
ERRORS      try/except/else/finally
FUNCTIONS   def name(p: type) -> rtype: return
STRUCTURES  list  tuple  set  dict  nested
IDIOMS      comprehensions  zip  enumerate  lambda
FILE I/O    json.dump/load  csv.DictReader/Writer
STRINGS     strip  lower  split  join  re.match
CLI         argparse  ArgumentParser
OOP         class  __init__  inheritance  super()
```

---


---

## 9. Review Habits That Make You a Better Developer

The goal of a review session is not only to check what works, but also to understand why it works and how to improve it. Good code is usually not the largest code; it is the code that is easier to read, test, and extend.

### A Simple Review Framework

**How to read this example:** This example applies the concept of A Simple Review Framework. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
# Ask five questions before finishing a feature:
# 1. Is the input validated?
# 2. Are edge cases handled?
# 3. Is the code readable with clear names?
# 4. Can the logic be reused?
# 5. Is the output easy to understand?
```

### Common Review Findings

- Functions that do too much at once
- Repeated logic copied in multiple places
- Missing error handling for bad input
- Variables with unclear names like `x` or `temp`
- Hard-to-read loops with no comments or structure

Refactoring is not about perfection; it is about clarity and maintainability.

> Go to **Exercise 6 (Review & Debugging Mini Lab)** to practice a compact code-review workflow.

---

## 10. Debugging Strategy

When code misbehaves, do not start by guessing. Follow a quick process:

1. Read the error message carefully
2. Check the exact input values
3. Use a print statement or small function to inspect state
4. Isolate the failing step
5. Fix the root cause instead of the symptom

This habit is the difference between a beginner and a reliable problem-solver.

---

## 11. A Professional Mindset

By this point, you have learned the main building blocks:
- variables and types
- loops and conditionals
- functions and modules
- dictionaries, files, and errors
- OOP and real-world project design

The next step is not writing longer code. It is writing clearer code that a teammate can understand and maintain in a week.


---

## Deep Dive: Reviewing Code Like a Developer

A review session is not only about checking if the project runs. It is about asking “Is this code clear, safe, and easy to change?”

A weak version of a function often looks like this:

**How to read this example:** The first version mixes prompting, conversion, validation, and output; a bare exception hides unexpected failures and `pass` silently discards them. The revised function improves this by giving the input rule one reusable place.

```python
# Messy code
value = input("value: ")
if value:
    try:
        a = float(value)
        if a > 0:
            print(a)
    except:
        pass
```

This is hard to read because it mixes multiple tasks: asking for input, converting, validating, and printing. The improved version separates concerns:

**How to read this example:** The first version mixes prompting, conversion, validation, and output; a bare exception hides unexpected failures and `pass` silently discards them. The revised function improves this by giving the input rule one reusable place.

```python
def get_valid_positive_number(prompt: str) -> float:
    while True:
        try:
            value = float(input(prompt))
            if value > 0:
                return value
            print("Please enter a number greater than zero.")
        except ValueError:
            print("That is not a valid number.")
```

The difference is not just cosmetic. It makes the program easier to test, easier to debug, and easier for another developer to extend. That is what “good code” really means.

When reviewing a project, look for:

- repeated logic that should become a function
- unclear names like `x` or `tmp`
- no handling for invalid input
- big functions doing too much at once
- code that works only for happy-path examples

Great developers are not only coders — they are also reviewers and refactors.


---

## Understanding the Review Checklist in Practice

The checklist above is most useful when each question is connected to a concrete example. A program can run for one test input and still be fragile; reviewing means looking for the assumptions hidden inside the code.

### Validate Inputs and Think About Edge Cases

Suppose a function calculates an average. It should work for ordinary values such as `[8, 9, 10]`, but a reviewer should also ask what happens for an empty list, a text value, or a negative value that the domain does not allow.

**How to read this example:** The empty-list check defines an edge case before division. For nonempty scores, sum divided by count calculates the mean. Raising a specific error explains why a score is required instead of exposing an unrelated division failure.

```python
def average(scores: list[float]) -> float:
    if not scores:
        raise ValueError("At least one score is required.")
    return sum(scores) / len(scores)
```

The empty-list check is not unnecessary complexity: it makes the function's expectation explicit and avoids division by zero. Edge cases are not unusual accidents; they are part of the input space a function must define.

### Readable Names and Focused Functions

Names such as `total_expenses` and `category` carry meaning. Names such as `x`, `temp`, and `thing` force the reader to reconstruct that meaning from surrounding code. A good name acts like a small explanation attached to the value.

Similarly, a function that reads input, validates it, updates a file, calculates a report, and prints a menu is difficult to test because there is no single behavior to check. Splitting it into small functions lets each one have a clear contract:

**How to read this example:** This function isolates the balance formula and names the inputs by meaning. Income 3000 and expenses 1250 produce 1750. The result can be tested or reused without invoking a menu or printing a report.

```python
def calculate_balance(income: float, expenses: float) -> float:
    return income - expenses
```

Now the calculation can be checked independently of the menu or file system.

### Repeated Logic Is a Maintenance Risk

If the same validation code is copied into three menu options, a later rule change has to be applied in all three places. Missing one copy creates inconsistent behavior. A reusable function gives the rule one home:

**How to read this example:** This helper turns the positive-value rule into a reusable Boolean operation. Since it does not prompt or print, a menu, web form, or test can choose what to do when the value is invalid.

```python
def is_positive_amount(amount: float) -> bool:
    return amount > 0
```

Avoid extracting every two-line sequence automatically; abstraction is helpful when it gives repeated or meaningful behavior a clear name.

### Error Handling Should Preserve Useful Information

The broad `except:` in the messy example catches nearly every kind of failure, including failures the code does not know how to recover from. The `pass` then hides the error, making the program appear to have succeeded.

Catch the specific expected exception instead:

**How to read this example:** Conversion is inside `try` because user text may not represent a number. Catching `ValueError` gives a clearer message and chains the original cause; the later comparison handles numeric values that still violate the positive-amount rule.

```python
try:
    amount = float(user_text)
except ValueError:
    print("Enter an amount such as 12.50.")
else:
    print(f"Accepted amount: {amount:.2f}")
```

This handles the predictable conversion problem without concealing unrelated bugs. When reviewing error handling, ask whether the program informs the user, preserves the cause, and has a sensible next step.

### Debugging: Follow the Value, Not Your First Guess

If a balance is wrong, inspect the calculation's inputs before changing the formula. A useful debugging path is:

1. Confirm which records are included.
2. Print or inspect the income and expense totals separately.
3. Check whether each record has the expected type and `is_income` value.
4. Reproduce the problem with the smallest example that still fails.
5. Correct the faulty assumption, then rerun both the small example and normal cases.

This is more reliable than changing several lines at once. A minimal failing example also makes a good regression test: once fixed, the same input should continue to produce the correct result.

### Comments: Explain Reasons, Not Obvious Syntax

A comment such as `# add one to i` repeats what `i += 1` already says. A useful comment explains a constraint or decision that is not obvious from the code:

**How to read this example:** The comment explains why the original date is preserved, not merely what the assignment does. Comments are most useful when they record a domain decision that a maintainer could otherwise accidentally undo.

```python
# Keep the original date string so exports preserve the recorded timestamp.
record["date"] = saved_date
```

When reviewing comments, check that they still agree with the code. Incorrect comments are worse than no comment because they mislead future readers.

### A Small Before-and-After Review

**How to read this example:** This example applies the concept of A Small Before-and-After Review. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
# Before: unclear purpose, broad exception, silent failure
def f(v):
    try:
        return float(v)
    except:
        pass
```

**How to read this example:** This example applies the concept of A Small Before-and-After Review. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
# After: clear purpose and an explicit failure contract
def parse_amount(text: str) -> float:
    """Convert user-entered text to a number or explain why it is invalid."""
    try:
        amount = float(text)
    except ValueError as exc:
        raise ValueError("Amount must be numeric, for example 12.50.") from exc

    if amount <= 0:
        raise ValueError("Amount must be greater than zero.")
    return amount
```

The second function is longer, but its behavior is easier to reason about: it tells us what it accepts, rejects invalid values, and preserves useful error context. A review is successful when it improves those qualities without adding complexity for its own sake.
