# Session XVII: Concepts Guide
## Data Analysis with pandas and duckdb

Load, filter, group, aggregate, merge data with pandas and DuckDB SQL.

---

## 1. pandas Basics

**How to read this example:** `read_csv()` creates a DataFrame. Shape, dtypes, a sample, and descriptive statistics answer different inspection questions and help catch source-data issues before analysis. The sample file `expenses.csv` is in this session's folder.

```python
import pandas as pd

df = pd.read_csv("expenses.csv")
print(df.shape)      # (rows, cols)
print(df.dtypes)     # column types
print(df.head())     # first 5 rows
print(df.describe()) # statistics
```

> Go to **Exercise 1 (Load and Explore)**, where you will build the sample DataFrame and inspect it with `shape`, `dtypes`, `head()` and `describe()`.

---

## 2. Filtering & Selecting

**How to read this example:** Selecting columns narrows fields and Boolean masks choose matching rows. Parentheses with `&` or `|` are important when combining pandas conditions.

```python
# Select columns
df[["category", "amount"]]

# Filter rows
expensive = df[df["amount"] > 100]
food = df[df["category"] == "Food"]

# Multiple conditions
big_food = df[(df["category"] == "Food") & (df["amount"] > 50)]
```

---

## 3. Grouping & Aggregation

**How to read this example:** `groupby()` partitions rows and aggregations summarize each group. Counts exclude missing values in the selected column, so check what is being counted.

```python
# Group by category
grouped = df.groupby("category")["amount"].sum()

# Multiple aggregations
summary = df.groupby("category").agg(
    total=("amount", "sum"),
    average=("amount", "mean"),
    count=("amount", "count")
).sort_values("total", ascending=False)
```

> Go to **Exercise 3 (Group & Aggregate)**, where you will summarize spending per category and find the top spender.

---

## 4. Adding & Transforming Columns

**How to read this example:** Derived columns apply operations to a Series; date conversion enables date components. Inspect unparseable values rather than silently accepting incomplete results.

```python
df["tax"] = df["amount"] * 0.21
df["total"] = df["amount"] + df["tax"]
df["month"] = pd.to_datetime(df["date"]).dt.month
```

> Go to **Exercise 2 (Filter & Transform)**, where you will filter Food expenses over $40 and add tax and month columns.

---

## 5. Merging DataFrames

**How to read this example:** Spending is summed per category first, so each total is compared with that category's budget rather than with single transactions. The left merge keeps categories that have no budget; their budget is `NaN` and `>` quietly returns `False`, so the code lists them explicitly instead of letting them pass as within budget.

```python
spent = df.groupby("category", as_index=False)["amount"].sum()

categories = pd.DataFrame({
    "category": ["Food", "Transport"],
    "budget": [500, 200]
})

merged = spent.merge(categories, on="category", how="left")
merged["over_budget"] = merged["amount"] > merged["budget"]

# NaN budget makes over_budget False: report those categories
no_budget = merged[merged["budget"].isna()]
print("No budget set for:", no_budget["category"].tolist())
```

> Go to **Exercise 5 (Merge & Report (Capstone))**, where you will compare each category's total spending with its budget and print a report.

---

## 6. DuckDB — SQL in Python

**How to read this example:** SQL expresses filtering, grouping, aggregation, and ordering as clauses. Read each clause to understand which records contribute to the summary.

```python
import duckdb

# Query a CSV directly
result = duckdb.sql("""
    SELECT category, SUM(amount) as total
    FROM 'expenses.csv'
    GROUP BY category
    ORDER BY total DESC
""")
print(result.df())  # Returns a pandas DataFrame

# Query a pandas DataFrame
duckdb.sql("SELECT * FROM df WHERE amount > 100").show()
```

> Go to **Exercise 4 (DuckDB Queries)**, where you will repeat the filters and summaries of Exercises 2 and 3 in SQL.

---

## 7. pandas vs DuckDB

| Task | pandas | DuckDB |
|------|--------|--------|
| Load CSV | `pd.read_csv()` | `FROM 'file.csv'` |
| Filter | `df[df["x"]>5]` | `WHERE x > 5` |
| Group | `df.groupby("x")["y"].sum()` | `SUM(y) … GROUP BY x` |
| Join | `df.merge()` | `JOIN` |
| Syntax | Python methods | SQL |

---

## Quick Reference

**How to read this example:** This block is an index mapping common analysis questions to pandas or SQL operations; the earlier examples explain their behavior.

```
PANDAS      pd.read_csv  df.head  df.groupby  df.merge  df.sort_values
DUCKDB      duckdb.sql("SELECT...").df()  .show()
FILTER      df[df["col"] > val]  /  WHERE col > val
GROUP       df.groupby("col").agg(...)  /  GROUP BY col
MERGE       df.merge(df2, on="key")  /  JOIN ... ON
```

---


---

## 8. A Better Data Analysis Workflow

Real analysis is not only about plotting charts. It is also about asking the right questions and validating the data before making assumptions.

### A Typical Workflow

**How to read this example:** The workflow moves from inspection through cleaning and exploration to communication. Decisions such as dropping missing rows change the conclusion and should be explained.

```python
# 1. Load data
# 2. Inspect structure and missing values
# 3. Clean and normalize columns
# 4. Explore trends and distributions
# 5. Summarize and communicate findings
```

A good data analyst should always check:
- are there missing values?
- are columns typed correctly?
- are there duplicates or inconsistent labels?
- do the results match the domain reality?

> Go to **Exercise 6 (Data Cleaning + Summary Pipeline)**, where you will practice a complete workflow.

---

## 9. Common Analysis Pitfalls

- Plotting raw numbers before checking duplicates or missing values
- Using `mean()` on data that has outliers or missing entries
- Mixing inconsistent labels such as `USA` and `United States`
- Reporting a chart without context or explanation
- Forgetting that data quality matters more than fancy visuals

The best analysis is honest, repeatable, and easy to explain.


---

## Deep Dive: Data Analysis Is Mostly Data Cleaning

A lot of beginners think data analysis starts with charts. In reality, the best analysis usually starts much earlier: with cleaning, validating, and understanding the raw data.

**How to read this example:** The raw table mixes capitalization, a missing category, and numeric text. The pipeline standardizes labels, converts values, removes unusable rows, and then groups totals. Each cleaning decision affects the result and should fit the analysis question.

```python
import pandas as pd

raw = pd.DataFrame({
    "category": ["Food", "food", "Transport", "Food", None],
    "amount": [12, 15, "40", 18, 9]
})

cleaned = raw.copy()
cleaned["category"] = cleaned["category"].fillna("Unknown").str.strip().str.lower()
cleaned["amount"] = pd.to_numeric(cleaned["amount"], errors="coerce")
cleaned = cleaned.dropna().drop_duplicates()
print(cleaned.groupby("category")["amount"].sum())
```

This example teaches a core lesson: analysis is not just “visualize interesting numbers.” It is “make the input trustworthy before drawing conclusions.”

When you clean data, you are doing engineering work:

- filling missing values
- standardizing labels
- converting types
- removing invalid rows
- checking for duplicates

Only after that are you ready to plot, compare, or explain patterns.


---

## Understanding the DataFrame Operations

Pandas gives you ways to describe operations over rows and columns. The key is to understand the shape of the result after every step and to check whether the operation answers the question you intended to ask.

### Inspect Before You Analyze

After loading data, inspect its dimensions, column names, types, and a few rows:

**How to read this example:** `shape`, `columns`, `dtypes`, and `head()` reveal table size, available fields, interpreted types, and sample values. Together they show whether the input matches expectations before calculations begin.

```python
print(df.shape)
print(df.columns.tolist())
print(df.dtypes)
print(df.head())
```

`shape` tells you how many rows and columns you are working with. `dtypes` can reveal that an amount column was read as text (`str` in pandas 3, `object` in older versions) because one row contains text such as `"unknown"`. `head()` provides a small sample; it is a quick inspection, not proof that every row is correct.

For a fuller quality check, count missing values with `df.isna().sum()` and look for repeated rows with `df.duplicated().sum()`. These checks help you discover data issues before they distort a calculation.

### Selecting Columns and Filtering Rows

Selecting columns answers “Which fields do I need?” Filtering answers “Which observations meet my condition?”

**How to read this example:** Selecting two columns focuses the table on needed fields; the Boolean mask keeps rows whose amounts exceed 100. Separating field selection from row filtering makes each operation easier to understand.

```python
selected = df[["category", "amount"]]
large_expenses = df[df["amount"] > 100]
```

The expression `df["amount"] > 100` creates a Boolean value for each row. Pandas uses that Boolean series as a mask and keeps the rows where the condition is true.

For multiple conditions, use `&` for AND and `|` for OR, and put parentheses around each comparison:

```python
large_food = df[(df["category"] == "Food") & (df["amount"] > 50)]
```

The parentheses matter because Python operator precedence determines how an expression is grouped. Use `&` and `|` for pandas Series rather than Python's scalar `and` and `or`.

### Grouping and Aggregation Answer Summary Questions

`groupby()` divides rows into groups that share a value. An aggregation then calculates one or more summaries for each group:

**How to read this example:** `groupby()` partitions rows by category, and named aggregations calculate total, average, and count. Sorting ranks categories by total. Count ignores missing values in the measured column, so it can differ from raw row count.

```python
summary = df.groupby("category").agg(
    total=("amount", "sum"),
    average=("amount", "mean"),
    transaction_count=("amount", "count"),
)
```

This asks three distinct questions per category: how much was spent in total, what was the average transaction size, and how many non-missing amounts were counted. The `count` aggregation ignores missing values in the selected column, so it may differ from the total number of rows in a group.

Sorting the result by `total` makes comparisons easier. If you need to merge the summary later, consider resetting the index or using `as_index=False` so the group key remains a regular column.

### Transforming Columns Creates New Information

A derived column is computed from existing values:

**How to read this example:** The assignments use vectorized arithmetic to calculate tax and tax-inclusive totals for each row without an explicit Python loop. Ensure the amount column is numeric so the multiplication has the intended meaning.

```python
df["tax"] = df["amount"] * 0.21
df["total_with_tax"] = df["amount"] + df["tax"]
```

This is called vectorized computation: pandas applies the arithmetic across the column without requiring a Python loop over each row. It is concise and usually efficient.

Dates need explicit conversion before accessing date components:

```python
df["date"] = pd.to_datetime(df["date"], errors="coerce")
df["month"] = df["date"].dt.month
```

With `errors="coerce"`, unparseable date values become `NaT`, pandas' missing datetime value. Inspect those results and decide whether to correct, exclude, or report them; conversion should not silently erase the underlying data-quality issue.

### Cleaning Values Requires a Domain Decision

Type conversion and missing-value handling are not merely technical chores. They encode decisions about what the data means.

**How to read this example:** `to_numeric()` converts numeric strings and marks invalid values as missing when coercion is enabled. It does not decide whether to correct, exclude, or report those rows; that decision depends on the analysis purpose.

```python
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")
```

This converts numeric text such as `"40"` to a number and marks invalid entries as missing. You then need to decide what a missing amount means. Dropping an invalid row may be appropriate for a narrow summary, but it can hide how much data was discarded. Filling it with zero (`fillna(0)`) is only correct when “missing” genuinely means “no amount”; often it does not.

Standardizing labels is similarly meaningful: `"Food"`, `"food"`, and `" Food "` may refer to the same category, but normalization should reflect the actual domain rather than merging distinct labels carelessly.

### Merging DataFrames Adds Related Information

A merge combines rows using a shared key, similar to a relational database join:

**How to read this example:** The merge matches category-level spending with budget values. A left join preserves unmatched spending categories, while duplicate budget keys can multiply rows. Compare budgets and spending at the same time and category granularity.

```python
spent = df.groupby("category", as_index=False)["amount"].sum()
merged = spent.merge(
    categories,
    on="category",
    how="left",
)
```

A left join keeps every row of the left table, here one spending total per category, and attaches a budget where one matches. Categories without a budget receive missing values (`NaN`). An inner join would discard them, which may be undesirable if you want to find unbudgeted spending.

Before merging, check that the key columns use compatible formats and that the lookup table has the expected number of rows per key. Duplicate keys on the right side can multiply rows and inflate totals.

### DuckDB Expresses the Same Questions in SQL

DuckDB allows the same kind of filtering, grouping, and joining with SQL:

**How to read this example:** The query reads a CSV, excludes zero or negative amounts, groups the remaining rows by category, sums them, and sorts totals descending. Each SQL clause defines which records contribute to the answer, paralleling pandas filtering and aggregation.

```sql
SELECT category, SUM(amount) AS total
FROM 'expenses.csv'
WHERE amount > 0
GROUP BY category
ORDER BY total DESC;
```

Read the query in stages: select the output fields, choose the source rows, group them, calculate the aggregate, and sort the result. This can be easier to read when the analysis involves several joins or when the analyst already knows SQL.

Pandas and DuckDB are not competing answers to every question. Use pandas when the work is naturally expressed as Python transformations or needs close interaction with Python objects; use SQL when relational filtering and aggregation make the question clearer. You can use both in one workflow.

### Interpret Results, Do Not Just Print Them

A summary table is an intermediate result, not the conclusion. If Housing is the largest expense category, ask whether the dataset covers the same time period for every category and whether the amounts are comparable. Include units and context when reporting results:

> “Housing accounts for $1,200 of the $1,850 recorded expenses in January.”

That statement is more useful than “Housing is highest” because it names the period and gives scale. Good analysis includes enough context for another person to understand what was measured and what limitations remain.
