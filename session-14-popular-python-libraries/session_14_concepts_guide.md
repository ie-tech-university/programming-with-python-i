# Session XIV: Concepts Guide
## Leveraging Popular Python Libraries

requests, pandas basics, duckdb basics, matplotlib intro, beautifulsoup4.

---

## 1. requests — HTTP for Humans

**How to read this example:** The HTTP request retrieves JSON, and a dictionary lookup selects the desired value. Add a timeout and status check before parsing so an error response is not treated as valid data.

```python
import requests
r = requests.get("https://api.github.com/repos/python/cpython")
data = r.json()
print(data["stargazers_count"])
```

---

## 2. pandas — Data Manipulation

**How to read this example:** A DataFrame is a table; previewing, filtering, deriving columns, grouping, and sorting are separate transformations. Inspect types and intermediate results before interpreting the summary.

```python
import pandas as pd

df = pd.read_csv("expenses.csv")
print(df.head())
print(df.describe())
print(df.groupby("category")["amount"].sum())
```

### Key Operations

**How to read this example:** This example applies the concept of Key Operations. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
# Filter
expensive = df[df["amount"] > 100]

# Add column
df["tax"] = df["amount"] * 0.21

# Sort
df.sort_values("amount", ascending=False)

# Group and aggregate
df.groupby("category").agg({"amount": ["sum", "mean", "count"]})
```

---

## 3. duckdb — SQL in Python

**How to read this example:** The SQL filters input rows and aggregates groups. Converting with `.df()` returns a DataFrame that can continue through a pandas workflow.

```python
import duckdb

result = duckdb.sql("SELECT * FROM 'expenses.csv' WHERE amount > 100")
print(result.df())

# Query a DataFrame
duckdb.sql("SELECT category, SUM(amount) FROM df GROUP BY category")
```

---

## 4. matplotlib — Basic Visualization

**How to read this example:** The plot compares values and uses a title and axis label to explain them. Saving and showing the figure are different operations; labels and units make a chart interpretable.

```python
import matplotlib.pyplot as plt

categories = ["Food", "Transport", "Housing"]
amounts = [450, 150, 800]

plt.bar(categories, amounts)
plt.title("Expenses by Category")
plt.ylabel("Amount ($)")
plt.savefig("chart.png")
plt.show()
```

---

## 5. beautifulsoup4 — Web Scraping

**How to read this example:** The parser turns HTML into a tree, `find_all()` selects matching tags, and text extraction reads their content. Prefer an official API when available and follow site rules.

```python
import requests
from bs4 import BeautifulSoup

page = requests.get("https://example.com")
soup = BeautifulSoup(page.text, "html.parser")
titles = soup.find_all("h2")
for t in titles:
    print(t.text)
```

---

## Quick Reference

**How to read this example:** This example applies the concept of Quick Reference. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```
requests    HTTP calls: get, post, json(), raise_for_status()
pandas      DataFrames: read_csv, groupby, filter, merge
duckdb      SQL queries: duckdb.sql("SELECT...").df()
matplotlib  Charts: plt.bar, plt.plot, plt.pie, plt.savefig
bs4         Scraping: BeautifulSoup, find_all, .text
```

---


---

## 7. How to Learn a New Python Library

A library is easiest to use when you follow a predictable workflow:

1. Read the official quickstart or examples
2. Install it in the project environment
3. Try a tiny example in a notebook or scratch script
4. Check the returned shapes and types
5. Build a small real-world function around it

### Example

**How to read this example:** This example applies the concept of Example. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
import pandas as pd

df = pd.DataFrame({"city": ["Lisbon", "Madrid"], "population": [500000, 330000]})
print(df)
print(df["population"].sum())
```

This approach avoids the trap of copying code you do not understand.

> Go to **Exercise 6 (Library Pipeline Challenge)** to practice the same workflow with a real dataset.

---

## 8. Common Technical Pitfalls

- Calling `.head()` on a `None` result from an API or parser
- Mixing string values and numeric values in a DataFrame without cleaning
- Assuming all CSV files have the same columns
- Forgetting to check internet access before scraping
- Not reading the library docs for missing parameters

The main lesson is simple: if a library returns a structure you do not understand, inspect it before building logic around it.


---

## Deep Dive: Learning a Library by Reading Its Shapes

The goal of a library is not to make you memorize syntax. The goal is to understand the data it returns and the operations it supports.

Take `pandas` as an example. A DataFrame is just a table, but the most important thing is not the table itself — it is knowing how to filter, transform, and aggregate it.

**How to read this example:** The records become a DataFrame; grouping partitions rows by category, selecting `amount` chooses the measurement, and `sum()` totals each group. This small example reveals pandas data shape and aggregation before moving to a larger dataset.

```python
import pandas as pd

sales = pd.DataFrame({
    "month": ["Jan", "Feb", "Jan"],
    "category": ["Food", "Food", "Travel"],
    "amount": [120, 90, 200]
})

print(sales.groupby("category")["amount"].sum())
```

This single example teaches several patterns at once:

- a dict can become a DataFrame
- a column can be selected as a Series
- grouping creates a summary by category
- the output is easy to read and easy to plot

When you work with a new library, you should ask three questions:

1. What structure does it return?
2. Which columns or methods are relevant?
3. What kind of transformation does the library make easy?

That mindset is much more valuable than memorizing code snippets.


---

## What Each Library Does and When to Reach for It

The libraries in this session solve different kinds of problems. Knowing their roles helps you choose a tool intentionally rather than trying to use one library for every task.

### `requests`: Communicate with a Web Service

`requests` handles the HTTP details of contacting a server. It builds requests, sends query parameters and headers, and gives you a response object to inspect.

**How to read this example:** The request retrieves a repository record, checks the HTTP status, and parses JSON. The star count is selected by key. A timeout and status check matter because a remote service can be slow or return an error document.

```python
import requests

response = requests.get(
    "https://api.github.com/repos/python/cpython",
    timeout=10,
)
response.raise_for_status()
repository = response.json()
print(repository["stargazers_count"])
```

The timeout prevents an unresponsive server from holding the program forever; `raise_for_status()` prevents an error page from being mistaken for valid repository data. `requests` is useful when your program needs information from a web API, but it does not decide how your application should handle outages or missing fields—that remains your responsibility.

### `pandas`: Work with Tabular Data

A DataFrame is a labeled table: rows represent observations, columns represent fields, and each column can have its own data type. This makes pandas useful for loading CSV files, cleaning columns, selecting subsets, and calculating summaries.

**How to read this example:** A DataFrame stores records as rows and fields as columns. Preview and descriptive statistics inspect it; filters select rows, derived columns add calculations, and groupby totals amounts per category. Inspect types before arithmetic because CSV values may be text.

```python
import pandas as pd

expenses = pd.DataFrame({
    "category": ["Food", "Transport", "Food"],
    "amount": [25.0, 12.0, 40.0],
})

food = expenses[expenses["category"] == "Food"]
print(food["amount"].sum())  # 65.0
```

The filtering expression creates a Boolean mask: each row is tested, and only rows where the condition is true remain. When you create a new column, pandas applies the operation to the column's values efficiently:

**How to read this example:** A DataFrame stores records as rows and fields as columns. Preview and descriptive statistics inspect it; filters select rows, derived columns add calculations, and groupby totals amounts per category. Inspect types before arithmetic because CSV values may be text.

```python
expenses["tax"] = expenses["amount"] * 0.21
```

Before calculating, inspect `.head()`, `.shape`, and `.dtypes`. Data imported from CSV may contain numbers stored as text, missing values, or inconsistent labels; analysis is only meaningful when you understand those details.

### `duckdb`: Ask Relational Questions with SQL

DuckDB is an embedded analytical database. It lets you use SQL to filter and aggregate data, including a CSV file, without first loading the entire workflow into a database server.

**How to read this example:** The SQL filters source rows with `WHERE`, groups matching records, and calculates totals. This describes a relational question without a manual Python loop; `.df()` converts the result back to pandas for further work.

```python
import duckdb

summary = duckdb.sql("""
    SELECT category, SUM(amount) AS total
    FROM 'expenses.csv'
    GROUP BY category
    ORDER BY total DESC
""").df()
```

This query says: read the CSV rows, group rows with the same category, add the amount in each group, and sort the summaries from highest to lowest. DuckDB can be a comfortable choice when the question naturally sounds like a SQL query or the dataset is large enough that a database-style workflow is useful.

Pandas and DuckDB can work together. A DuckDB query can return a DataFrame using `.df()`, so you can use SQL for a query and pandas for subsequent Python-oriented transformations.

### `matplotlib`: Control the Details of a Plot

Matplotlib creates charts from data arrays or sequences. An `Axes` object represents the plotting area; labels and titles explain what the chart measures.

**How to read this example:** `subplots()` creates a figure and plotting axes; the bar plot compares amounts, while labels explain the measurement. Accurate values can still be misunderstood without a title and units.

```python
import matplotlib.pyplot as plt

categories = ["Food", "Transport", "Housing"]
amounts = [450, 150, 800]

fig, ax = plt.subplots()
ax.bar(categories, amounts)
ax.set_title("Monthly expenses by category")
ax.set_ylabel("Amount ($)")
fig.tight_layout()
plt.show()
```

The chart is not automatically an explanation. A title should describe what is being compared, and axis labels should include units where appropriate. A chart without these details may display correct numbers but still mislead its reader.

### Beautiful Soup: Parse HTML Structure

Beautiful Soup converts HTML text into a navigable tree. You can find elements by tag or attribute and extract their text:

**How to read this example:** Beautiful Soup parses HTML into a tree, `find()` selects a heading, and `get_text(strip=True)` extracts readable text. The conditional handles a missing heading. Live markup can change, so use a documented API when available.

```python
from bs4 import BeautifulSoup

html = "<article><h2>Python news</h2><p>New release notes</p></article>"
soup = BeautifulSoup(html, "html.parser")
heading = soup.find("h2")

if heading:
    print(heading.get_text(strip=True))
```

The HTML parser is useful for pages where the content is available in the returned markup. A live website may block automated requests, render content with JavaScript, or change its HTML structure. Check the site's terms and use a documented API when one is provided; web scraping is not a promise that every page can or should be collected.

### Choosing Between Tools

Imagine you need a monthly category chart from an online expense feed:

1. Use `requests` to retrieve the feed and handle the response.
2. Convert the returned records into a pandas DataFrame.
3. Clean the columns and aggregate categories with pandas or DuckDB.
4. Use Matplotlib to create a chart with labels and context.

Each library has a distinct job in that pipeline. Keeping the boundaries clear makes it easier to replace one tool later—for instance, changing the API source does not need to change the plotting code if both sources produce the same clean table.

### Learn by Inspecting Intermediate Results

Do not write a long chain of transformations before checking what each step returns. Inspect a small sample:

**How to read this example:** These calls inspect sample rows, exact column names, and inferred types before analysis. They can reveal missing fields or amounts loaded as text. Checking early helps find source-data issues before they affect a chart.

```python
df = pd.read_csv("expenses.csv")
print(df.head())
print(df.columns.tolist())
print(df.dtypes)
```

Then make one transformation and inspect it again. This reveals wrong column names, unexpected types, and missing values close to where they enter the workflow. It also helps you build an accurate mental model of what the library is doing.
