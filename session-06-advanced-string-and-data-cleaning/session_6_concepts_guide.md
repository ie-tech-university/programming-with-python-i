# Session VI: Concepts Guide
## Advanced String Handling and Data Cleaning

This guide covers how to **manipulate, validate, and clean text data** — one of the most common and critical tasks in real-world programming. Read through the explanations and study the examples **before** attempting the exercises.

> **Why does this matter?**
> In the real world, data is *never* clean. CSV files have extra spaces, users type emails in ALL CAPS, phone numbers come in a dozen different formats, and text from the web contains invisible characters. The ability to wrangle messy strings into consistent, usable data is what separates a script that works on your laptop from one that works in production.

---

## 1. Advanced String Methods

You already know `.strip()`, `.lower()`, and `.upper()` from Session II. Python strings have many more powerful methods for transforming and inspecting text.

### Replacing and Removing Substrings

```python
text = "Hello,   World!   Welcome to   Python."

# Replace substrings
clean = text.replace("   ", " ")
print(clean)   # "Hello, World! Welcome to Python."

# Chain multiple replacements
phone = "(555) 123-4567"
digits_only = phone.replace("(", "").replace(")", "").replace("-", "").replace(" ", "")
print(digits_only)   # "5551234567"
```

### Splitting and Joining Strings

`split()` breaks a string into a list; `join()` does the reverse:

```python
# Splitting
csv_line = "Alice, 25, Madrid, alice@email.com"
fields = csv_line.split(",")
print(fields)   # ['Alice', ' 25', ' Madrid', ' alice@email.com']

# Strip each field after splitting
clean_fields = [f.strip() for f in csv_line.split(",")]
print(clean_fields)   # ['Alice', '25', 'Madrid', 'alice@email.com']

# Joining
words = ["Python", "is", "awesome"]
sentence = " ".join(words)
print(sentence)   # "Python is awesome"

# Join with a different separator
csv_output = ",".join(clean_fields)
print(csv_output)   # "Alice,25,Madrid,alice@email.com"
```

### The Split-Clean-Join Pattern

This is one of the most useful patterns for normalizing messy whitespace:

```python
messy = "   Hello    World    from     Python   "

# Split on any whitespace (default), then rejoin with single spaces
clean = " ".join(messy.split())
print(clean)   # "Hello World from Python"
```

> **Why this works:** `split()` with no arguments splits on *any* whitespace (spaces, tabs, newlines) and automatically ignores leading/trailing whitespace. It returns only the non-empty parts.

### Checking String Content

| Method | What It Tests | Example |
|--------|--------------|---------|
| `.startswith(s)` | Begins with `s` | `"hello".startswith("he")` → `True` |
| `.endswith(s)` | Ends with `s` | `"file.csv".endswith(".csv")` → `True` |
| `.isdigit()` | All characters are digits | `"123".isdigit()` → `True` |
| `.isalpha()` | All characters are letters | `"hello".isalpha()` → `True` |
| `.isalnum()` | All characters are letters or digits | `"abc123".isalnum()` → `True` |
| `.isspace()` | All characters are whitespace | `"  \t".isspace()` → `True` |

### Finding and Counting Substrings

```python
text = "the quick brown fox jumps over the lazy dog"

print(text.count("the"))        # 2
print(text.find("fox"))         # 16 (index where "fox" starts)
print(text.find("cat"))         # -1 (not found)
print("fox" in text)            # True (preferred for simple checks)
```

### Padding and Alignment

Useful for generating formatted reports:

```python
name = "Alice"
print(name.ljust(15))       # "Alice          "
print(name.rjust(15))       # "          Alice"
print(name.center(15))      # "     Alice     "
print(name.zfill(10))       # "00000Alice"

# Practical: formatting a table
for item, price in [("Coffee", 3.50), ("Sandwich", 7.25)]:
    print(f"{item:<15} ${price:>6.2f}")
# Coffee           $  3.50
# Sandwich         $  7.25
```

> Go to **Exercise 1 (Text Normalizer)** where you will use these methods to clean messy user input.

---

## 2. Regular Expressions (`re` Module)

Regular expressions (regex) are a powerful **pattern-matching language** built into Python via the `re` module. They let you search for, validate, and extract complex text patterns that would require dozens of lines of `if` statements to handle manually.

### Importing and Basic Usage

```python
import re

text = "Call me at 555-1234 or 555-5678"

# Find ALL matches of a pattern
matches = re.findall(r"\d{3}-\d{4}", text)
print(matches)   # ['555-1234', '555-5678']
```

### Essential Regex Syntax

| Pattern | Meaning | Example Match |
|---------|---------|--------------|
| `\d` | Any digit (0-9) | `"5"` |
| `\D` | Any non-digit | `"a"`, `" "` |
| `\w` | Any word character (letter, digit, `_`) | `"a"`, `"5"`, `"_"` |
| `\W` | Any non-word character | `" "`, `"!"` |
| `\s` | Any whitespace (space, tab, newline) | `" "`, `"\t"` |
| `\S` | Any non-whitespace | `"a"`, `"5"` |
| `.` | Any character except newline | `"a"`, `"5"`, `" "` |

### Quantifiers — How Many?

| Quantifier | Meaning | Example |
|-----------|---------|---------|
| `{3}` | Exactly 3 | `\d{3}` matches `"123"` |
| `{2,4}` | Between 2 and 4 | `\d{2,4}` matches `"12"`, `"1234"` |
| `*` | Zero or more | `\d*` matches `""`, `"123"` |
| `+` | One or more | `\d+` matches `"1"`, `"123"` |
| `?` | Zero or one (optional) | `\d?` matches `""`, `"5"` |

### Anchors — Where in the String?

| Anchor | Meaning |
|--------|---------|
| `^` | Start of string |
| `$` | End of string |

```python
import re

# Must be ALL digits from start to end
print(bool(re.match(r"^\d+$", "12345")))    # True
print(bool(re.match(r"^\d+$", "123abc")))   # False
```

### The Key `re` Functions

| Function | What It Does | Returns |
|----------|-------------|---------|
| `re.match(pattern, string)` | Match at the **start** of string | Match object or `None` |
| `re.search(pattern, string)` | Find **first** match anywhere | Match object or `None` |
| `re.findall(pattern, string)` | Find **all** matches | List of strings |
| `re.sub(pattern, replacement, string)` | **Replace** all matches | New string |
| `re.split(pattern, string)` | **Split** string by pattern | List of strings |

### `re.match()` vs `re.search()`

```python
import re

text = "Order #12345 confirmed"

# match() — only checks the BEGINNING of the string
print(re.match(r"\d+", text))       # None (string starts with "Order")

# search() — finds the first match ANYWHERE
print(re.search(r"\d+", text))      # <re.Match object; span=(7, 12), match='12345'>
print(re.search(r"\d+", text).group())   # "12345"
```

### `re.sub()` — Find and Replace with Patterns

```python
import re

# Remove all non-digit characters from a phone number
phone = "(555) 123-4567"
clean = re.sub(r"\D", "", phone)
print(clean)   # "5551234567"

# Replace multiple spaces with a single space
messy = "Hello    World     from   Python"
clean = re.sub(r"\s+", " ", messy).strip()
print(clean)   # "Hello World from Python"
```

### Groups — Extracting Parts of a Match

Parentheses `()` create **groups** that let you extract specific parts:

```python
import re

email = "alice.smith@university.edu"
match = re.match(r"^(\w[\w.]+)@([\w.]+)$", email)

if match:
    username = match.group(1)   # "alice.smith"
    domain = match.group(2)     # "university.edu"
    print(f"User: {username}, Domain: {domain}")
```

### Always Use Raw Strings (`r"..."`)

Regex patterns should always use **raw strings** (prefix `r`) to prevent Python from interpreting backslashes:

```python
# Without raw string — Python interprets \d as an escape sequence
pattern = "\d+"

# With raw string — backslash is passed literally to the regex engine
pattern = r"\d+"
```

> Go to **Exercise 2 (Regex Validator Suite)** where you will write regex patterns to validate emails, phone numbers, and dates.

---

## 3. Data Cleaning Patterns

Data cleaning is the process of transforming raw, messy data into a consistent, usable format. Here are the most common patterns you'll use.

### Pattern 1: Normalize Whitespace and Case

```python
def normalize_text(text):
    """Clean and standardize a text string."""
    if not isinstance(text, str):
        return ""
    # Remove leading/trailing whitespace
    text = text.strip()
    # Collapse multiple spaces into one
    text = " ".join(text.split())
    # Standardize case
    text = text.lower()
    return text

print(normalize_text("   HELLO    World  "))   # "hello world"
print(normalize_text(None))                     # ""
```

### Pattern 2: Clean a Row of Data

When processing CSV rows, every field may need different cleaning:

```python
def clean_row(row):
    """Clean a dictionary row from a CSV file."""
    cleaned = {}
    for key, value in row.items():
        # Clean the key itself
        clean_key = key.strip().lower().replace(" ", "_")
        # Clean the value
        clean_value = value.strip() if isinstance(value, str) else value
        cleaned[clean_key] = clean_value
    return cleaned

raw = {"  Name ": "  Alice  ", " Age": "25 ", "City ": " Madrid "}
print(clean_row(raw))
# {'name': 'Alice', 'age': '25', 'city': 'Madrid'}
```

### Pattern 3: Validate and Convert Types

```python
def safe_float(value, default=0.0):
    """Convert a value to float safely."""
    if value is None:
        return default
    try:
        cleaned = str(value).strip().replace(",", "").replace("$", "").replace("€", "")
        return float(cleaned)
    except (ValueError, TypeError):
        return default

print(safe_float("$1,234.56"))   # 1234.56
print(safe_float("N/A"))          # 0.0
print(safe_float(None))           # 0.0
```

### Pattern 4: Standardize Categories

Real data has the same category spelled multiple ways:

```python
CATEGORY_MAP = {
    "food": "Food & Dining",
    "dining": "Food & Dining",
    "restaurant": "Food & Dining",
    "restaurants": "Food & Dining",
    "transport": "Transportation",
    "transportation": "Transportation",
    "uber": "Transportation",
    "taxi": "Transportation",
}

def standardize_category(raw_category):
    """Map a raw category string to a standard category."""
    key = raw_category.strip().lower()
    return CATEGORY_MAP.get(key, "Other")

print(standardize_category("  FOOD  "))        # "Food & Dining"
print(standardize_category("restaurant"))       # "Food & Dining"
print(standardize_category("uber"))             # "Transportation"
print(standardize_category("mystery"))          # "Other"
```

> Go to **Exercise 3 (CSV Data Cleaner)** where you will combine all these patterns to clean a messy real-world CSV file.

---

## 4. Phone Number Parsing

Phone numbers are notoriously inconsistent. Users type them in dozens of formats. The goal is to parse any reasonable input into a single consistent format.

### Common Formats You'll Encounter

```
(555) 123-4567
555-123-4567
555.123.4567
5551234567
+1 555 123 4567
+1-555-123-4567
1-555-123-4567
```

### Strategy: Strip Everything, Then Reformat

```python
import re

def parse_phone(raw_phone):
    """Parse a phone number into (XXX) XXX-XXXX format."""
    # Step 1: Remove everything that isn't a digit
    digits = re.sub(r"\D", "", raw_phone)
    
    # Step 2: Handle country code (if 11 digits starting with 1)
    if len(digits) == 11 and digits.startswith("1"):
        digits = digits[1:]    # Remove leading country code
    
    # Step 3: Validate length
    if len(digits) != 10:
        return None    # Invalid phone number
    
    # Step 4: Reformat
    area = digits[:3]
    prefix = digits[3:6]
    line = digits[6:]
    return f"({area}) {prefix}-{line}"

# Test with various formats
tests = ["(555) 123-4567", "555.123.4567", "+1-555-123-4567", "5551234567"]
for t in tests:
    print(f"{t:>20}  →  {parse_phone(t)}")
```

Output:
```
   (555) 123-4567  →  (555) 123-4567
     555.123.4567  →  (555) 123-4567
  +1-555-123-4567  →  (555) 123-4567
       5551234567  →  (555) 123-4567
```

> Go to **Exercise 4 (Phone Number Parser)** where you will build a robust phone number cleaning function and process a batch of messy contact data.

---

## 5. Email Validation

Email validation is a classic problem that demonstrates why regex is so useful — and why it has limits.

### Basic Email Structure

A valid email address has the form: `local@domain.tld`

- **Local part:** Letters, digits, dots, underscores, hyphens
- **`@` symbol:** Exactly one
- **Domain:** Letters, digits, hyphens, separated by dots
- **TLD (Top-Level Domain):** At least 2 characters (e.g., `.com`, `.edu`, `.io`)

### A Practical Email Regex

```python
import re

def is_valid_email(email):
    """Validate an email address using regex."""
    pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    return bool(re.match(pattern, email.strip()))

# Test cases
tests = [
    "alice@example.com",       # Valid
    "bob.smith@uni.edu",       # Valid
    "user+tag@domain.co.uk",   # Valid
    "missing-at-sign.com",     # No @
    "@no-local.com",           # Empty local part
    "user@.com",               # Empty domain
    "user@domain",             # No TLD
    "user @domain.com",        # Space in address
]

for email in tests:
    status = "✅" if is_valid_email(email) else "❌"
    print(f"  {status} {email}")
```

### Breaking Down the Pattern

```
^[a-zA-Z0-9._%+-]+  →  Local part: one or more valid characters
@                     →  Literal @ symbol
[a-zA-Z0-9.-]+       →  Domain: one or more valid characters
\.                    →  Literal dot (escaped)
[a-zA-Z]{2,}$        →  TLD: at least 2 letters, then end of string
```

### Pre-Cleaning Before Validation

Always clean the email before validating:

```python
def clean_and_validate_email(raw_email):
    """Clean and validate an email address."""
    if not isinstance(raw_email, str):
        return None
    
    # Remove whitespace and convert to lowercase
    email = raw_email.strip().lower()
    
    # Remove common accidental characters
    email = email.replace(" ", "").replace(",", ".")
    
    # Validate
    pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    if re.match(pattern, email):
        return email
    return None
```

> Go to **Exercise 2 (Regex Validator Suite)** where you will build validators for emails, phone numbers, and dates.

---

## 6. String Encoding and Special Characters

When working with data from the web, files, or APIs, you'll encounter encoding issues. Understanding the basics will save you hours of debugging.

### What is Encoding?

Text is stored as **numbers** in memory. An **encoding** maps characters to numbers:

| Encoding | What It Covers | Notes |
|----------|---------------|-------|
| **ASCII** | English letters, digits, basic symbols (128 chars) | The oldest standard |
| **Latin-1** (ISO-8859-1) | Western European characters (256 chars) | Extends ASCII |
| **UTF-8** | Every character in every language | The modern standard — **always use this** |

### Reading Files with Encoding

```python
# Default encoding (usually UTF-8 on modern systems)
with open("data.txt", "r") as file:
    content = file.read()

# Explicitly specify encoding (safest)
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

# Handle files from older systems
with open("legacy_data.txt", "r", encoding="latin-1") as file:
    content = file.read()
```

### Handling Encoding Errors

```python
# Option 1: Skip characters that can't be decoded
with open("messy.txt", "r", encoding="utf-8", errors="ignore") as file:
    content = file.read()

# Option 2: Replace bad characters with a placeholder
with open("messy.txt", "r", encoding="utf-8", errors="replace") as file:
    content = file.read()    # Bad chars become "�"
```

### Removing Non-Printable Characters

```python
import re

def remove_non_printable(text):
    """Remove non-printable and control characters."""
    # Keep only printable ASCII + common whitespace
    return re.sub(r"[^\x20-\x7E\n\t]", "", text)

messy = "Hello\x00World\x07 with\x1b control chars"
print(remove_non_printable(messy))   # "HelloWorld with control chars"
```

### Normalizing Unicode Characters

Some characters that look identical are actually different Unicode code points:

```python
# These look the same but are different characters!
s1 = "café"          # 'é' as a single character (U+00E9)
s2 = "cafe\u0301"    # 'e' + combining accent (U+0301)

print(s1 == s2)       # False!
print(len(s1), len(s2))  # 4, 5

# Fix with unicodedata normalization
import unicodedata
s1_norm = unicodedata.normalize("NFC", s1)
s2_norm = unicodedata.normalize("NFC", s2)
print(s1_norm == s2_norm)   # True
```

> Go to **Exercise 5 (Encoding Fixer)** where you will clean files with encoding problems and special characters.

---

## 7. Building Reusable Cleaning Modules

As you write more cleaning code, you'll notice the same functions appearing in every project. The professional approach is to package them into a **reusable module**.

### What is a Module?

A module is simply a `.py` file containing functions that you can import from other files:

```python
# cleaners.py — your reusable cleaning module

import re

def clean_text(text):
    """Normalize whitespace and case."""
    if not isinstance(text, str):
        return ""
    return " ".join(text.strip().split()).lower()

def clean_phone(phone):
    """Parse phone to (XXX) XXX-XXXX format."""
    digits = re.sub(r"\D", "", phone)
    if len(digits) == 11 and digits.startswith("1"):
        digits = digits[1:]
    if len(digits) != 10:
        return None
    return f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"

def clean_email(email):
    """Normalize and validate an email."""
    if not isinstance(email, str):
        return None
    email = email.strip().lower()
    pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    return email if re.match(pattern, email) else None

def safe_float(value, default=0.0):
    """Convert to float, stripping currency symbols."""
    try:
        cleaned = str(value).strip().replace(",", "").replace("$", "").replace("€", "")
        return float(cleaned)
    except (ValueError, TypeError):
        return default
```

### Using Your Module

```python
# main.py — your application
from cleaners import clean_text, clean_phone, clean_email, safe_float

# Now you can use these functions anywhere
print(clean_text("   HELLO   WORLD   "))    # "hello world"
print(clean_phone("(555) 123-4567"))         # "(555) 123-4567"
print(clean_email("ALICE@EXAMPLE.COM"))      # "alice@example.com"
print(safe_float("$1,234.56"))               # 1234.56
```

### The `if __name__ == "__main__"` Guard

When you import a module, Python runs **all the code** in that file. The `__name__` guard prevents test code from running on import:

```python
# cleaners.py
import re

def clean_text(text):
    """Normalize whitespace and case."""
    if not isinstance(text, str):
        return ""
    return " ".join(text.strip().split()).lower()

# This only runs when you execute cleaners.py directly,
# NOT when another file does "from cleaners import ..."
if __name__ == "__main__":
    # Test code
    print(clean_text("  HELLO   WORLD  "))
    print("All tests passed!")
```

### Module Design Principles

| Principle | Why | Example |
|-----------|-----|---------|
| **One function, one job** | Easier to test and reuse | `clean_phone()` only cleans phones |
| **Consistent return types** | Callers know what to expect | Return `None` for invalid, cleaned string for valid |
| **No `input()` or `print()`** | Modules should be silent — the caller decides what to display | Return data; let `main.py` print it |
| **Docstrings on every function** | Others (and future you) need to know what it does | `"""Clean and validate an email."""` |
| **Safe defaults** | Handle `None`, empty strings, and wrong types gracefully | `if not isinstance(text, str): return ""` |

> Go to **Exercise 6 (Data Cleaning Module)** where you will build a full reusable cleaning module and use it to process a messy dataset.

---

## Quick Reference Cheat Sheet

```
┌──────────────────────────────────────────────────────────┐
│  STRING METHODS                                          │
│    .strip()  .split()  .join()  .replace()               │
│    .startswith()  .endswith()  .find()  .count()         │
│    .isdigit()  .isalpha()  .isalnum()  .isspace()       │
│    .ljust()  .rjust()  .center()  .zfill()              │
│                                                          │
│  REGEX (import re)                                       │
│    re.match(pattern, string)    → match at start         │
│    re.search(pattern, string)   → first match anywhere   │
│    re.findall(pattern, string)  → all matches            │
│    re.sub(pattern, repl, string)→ find & replace         │
│    re.split(pattern, string)    → split by pattern       │
│                                                          │
│  REGEX PATTERNS                                          │
│    \d  digit    \D  non-digit    \w  word char           │
│    \s  space    \S  non-space    .   any char            │
│    +   one+     *   zero+        ?   optional            │
│    {n} exact    {n,m} range      ^   start   $  end     │
│    ()  group    []  char class   |   or                  │
│                                                          │
│  DATA CLEANING PATTERNS                                  │
│    " ".join(text.split())        → normalize whitespace  │
│    re.sub(r"\D", "", phone)      → extract digits        │
│    str.strip().lower()           → normalize case        │
│    LOOKUP_MAP.get(key, default)  → standardize categories│
│                                                          │
│  ENCODING                                                │
│    open(f, encoding="utf-8")     → always specify        │
│    errors="ignore" / "replace"   → handle bad bytes      │
│    unicodedata.normalize("NFC")  → normalize Unicode     │
│                                                          │
│  MODULES                                                 │
│    from cleaners import func     → import your module    │
│    if __name__ == "__main__":    → guard test code       │
└──────────────────────────────────────────────────────────┘
```

---
