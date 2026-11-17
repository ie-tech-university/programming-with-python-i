# Session XIV: Concepts Guide
## Leveraging Popular Python Libraries

This session covers five widely used libraries: `requests` for HTTP, pandas for tables, DuckDB for SQL queries, Matplotlib for charts, and Beautiful Soup for HTML, plus a workflow for learning any new library.

---

## 1. requests — HTTP for Humans

Send HTTP requests and read the JSON a web service returns.

### requests: Communicate with a Web Service

`requests` handles the HTTP details of contacting a server: it builds the request, sends query parameters and headers, and gives you a response object to inspect.

**How to read this example:** `timeout=10` stops an unresponsive server from holding the program forever, and `raise_for_status()` raises an `HTTPError` for a 4xx or 5xx answer, so an error page is never mistaken for repository data. Only then does `.json()` parse the body into a dictionary.

```python
import requests

response = requests.get(
    "https://api.github.com/repos/python/cpython",
    timeout=10,
)
response.raise_for_status()
repo = response.json()
print(repo["full_name"], repo["stargazers_count"])
```

`requests` gets the data; deciding how your program handles outages or missing fields remains your responsibility.

### Query Parameters and Rate Limits

```python
import requests

params = {"q": "language:python", "sort": "stars", "per_page": 3}
response = requests.get(
    "https://api.github.com/search/repositories",
    params=params,
    timeout=10,
)
if response.status_code in (403, 429):
    print("Rate limit reached: wait and try again later.")
else:
    response.raise_for_status()
    for repo in response.json()["items"]:
        print(repo["full_name"], repo["stargazers_count"])
```

**How to read this example:** `params=` builds the query string for you and encodes special characters, so you never glue URLs together by hand: `response.url` is `…/search/repositories?q=language%3Apython&sort=stars&per_page=3`.

> **Warning:** Without a token, GitHub allows **60 requests per hour per IP address** (search: 10 per minute). A whole class behind one university IP shares that quota, so expect `403` (or `429`) answers: check the status code and wait instead of retrying in a loop.

> Go to **Exercise 1 (requests — GitHub API)**, where you will fetch the five most-starred Python repositories with `params=` and handle errors.

---

## 2. pandas — Data Manipulation

Load, inspect, filter, and summarize tables.

### pandas: Work with Tabular Data

A **DataFrame** is a labeled table: rows are observations, columns are fields, and each column has its own data type.

`expenses.csv` (seven sample expenses) is in this session's folder; run the code from there.

**How to read this example:** Inspect before calculating: `.shape` gives the (rows, columns) count, `.head()` shows sample rows, and `.dtypes` the type pandas inferred for each column. CSV data may contain numbers stored as text, missing values, or inconsistent labels.

```python
import pandas as pd

df = pd.read_csv("expenses.csv")
print(df.shape)     # (7, 4)
print(df.head(3))
print(df.dtypes)
```

### Key Operations

```python
df = pd.read_csv("expenses.csv")

food = df[df["category"] == "Food"]     # boolean mask
print(food["amount"].sum())             # 145.5

df["tax"] = df["amount"] * 0.21         # adds a column to df

by_amount = df.sort_values("amount", ascending=False)
print(by_amount.head(3))

summary = df.groupby("category")["amount"].agg(["sum", "mean", "count"])
print(summary)
```

**How to read this example:** The mask `df["category"] == "Food"` tests every row and keeps those where it is `True`. `df["tax"] = …` changes `df` itself, but `sort_values()` and `groupby()` return **new** objects and leave `df` unchanged, so assign or print their result.

> Go to **Exercise 2 (pandas — CSV Analysis)**, where you will filter, group, add a tax column, and sort a sample expenses DataFrame.

### Cleaning Column Names with rename()

```python
import pandas as pd

raw = pd.DataFrame({" Category ": ["Food", "Rent", "Fun"],
                    "AMOUNT": [80, 800, -5]})

clean = raw.rename(columns=lambda col: col.strip().lower())
print(clean.columns.tolist())   # ['category', 'amount']

valid = clean[clean["amount"] > 0]
print(valid)
#   category  amount
# 0     Food      80
# 1     Rent     800
```

**How to read this example:** `rename(columns=…)` takes a function applied to every column name, or a dictionary for specific columns: `raw.rename(columns={"AMOUNT": "amount"})`. Like most pandas methods it returns a new DataFrame. A boolean mask then drops the invalid row.

### From Nested JSON to a DataFrame

APIs often return nested records, like the course dataset `data/countries.json`. Pick the fields you need into flat dictionaries, then build the DataFrame:

**How to read this example:** `c["name"]["common"]` reads a value inside a nested dictionary, and `.get("population", 0)` supplies a default when a key is missing. `nlargest(2, "population")` returns the two rows with the largest values, already sorted.

```python
import pandas as pd

countries = [
    {"name": {"common": "Spain"}, "population": 49355143},
    {"name": {"common": "Portugal"}, "population": 10804871},
    {"name": {"common": "France"}, "population": 68720337},
]
rows = [
    {"name": c["name"]["common"],
     "population": c.get("population", 0)}
    for c in countries
]
df = pd.DataFrame(rows)
print(df.nlargest(2, "population"))
#      name  population
# 2  France    68720337
# 0   Spain    49355143
```

---

## 3. duckdb — SQL in Python

DuckDB is an embedded analytical database: it runs SQL inside your Python process, with no server. `FROM` can name a CSV file in quotes or a pandas DataFrame by its variable name.

**How to read this example:** The first query filters rows with `WHERE`; the second groups by category, names the sum with `AS total`, and sorts with `ORDER BY`. `.df()` converts a result to a pandas DataFrame for further work; `.show()` just prints it as a table.

```python
import duckdb
import pandas as pd

big = duckdb.sql(
    "SELECT * FROM 'expenses.csv' WHERE amount > 100"
).df()
print(big)

df = pd.read_csv("expenses.csv")
summary = duckdb.sql("""
    SELECT category, SUM(amount) AS total
    FROM df
    GROUP BY category
    ORDER BY total DESC
""").df()
print(summary)
```

> Go to **Exercise 3 (duckdb — SQL Queries)**, where you will ask the same questions of the expenses DataFrame in SQL and compare the results with pandas.

---

## 4. matplotlib — Basic Visualization

Turn numbers into labeled charts.

### matplotlib: Bar Charts

`plt.subplots()` creates a figure (`fig`, the whole image) and axes (`ax`, the plotting area); you draw on the axes.

**How to read this example:** The bar chart compares amounts; the title says what is compared and the axis label gives the unit. `fig.savefig()` writes the image to a file and `plt.show()` displays it; save first, because showing can discard the figure.

A chart is not automatically an explanation: without a title and units it may display correct numbers and still mislead its reader.

```python
import matplotlib.pyplot as plt

categories = ["Food", "Transport", "Housing"]
amounts = [450, 150, 800]

fig, ax = plt.subplots()
ax.bar(categories, amounts)
ax.set_title("Monthly expenses by category")
ax.set_ylabel("Amount ($)")
fig.tight_layout()
fig.savefig("chart.png")
plt.show()
```

### Pie Charts Show Shares of a Whole

```python
# same categories and amounts as the bar chart
fig, ax = plt.subplots()
ax.pie(amounts, labels=categories, autopct="%1.0f%%")
ax.set_title("Share of monthly expenses")
plt.show()
```

A pandas Series can draw itself on an axes too: `totals.plot(kind="pie", ax=ax)` or `kind="bar"`.

**How to read this example:** `ax.pie()` draws each value as a slice of the total; `labels=` names the slices and `autopct="%1.0f%%"` writes each share as a whole percentage (`%%` is a literal percent sign): Food 32%, Transport 11%, Housing 57%.

Use a pie only for a few categories that add up to a meaningful total; a bar chart compares values more precisely.

> Go to **Exercise 4 (matplotlib — Basic Charts)**, where you will draw a bar chart and a pie chart of expenses by category.

---

## 5. beautifulsoup4 — Web Scraping

Parse HTML pages when no API offers the data.

### Beautiful Soup: Parse HTML Structure

Beautiful Soup converts HTML text into a navigable tree. You can find elements by tag or attribute and extract their text.

**How to read this example:** `find()` returns the first matching tag, or `None` if there is none, so check it before use; `find_all()` returns a list of every match; `get_text(strip=True)` extracts the readable text without surrounding spaces.

```python
from bs4 import BeautifulSoup

html = """
<article>
  <h2>Python news</h2>
  <ul><li>New release</li><li>Security fix</li></ul>
</article>
"""
soup = BeautifulSoup(html, "html.parser")
heading = soup.find("h2")
if heading:
    print(heading.get_text(strip=True))   # Python news
for item in soup.find_all("li"):
    print(item.get_text(strip=True))
# New release
# Security fix
```

### Scraping a Live Page

```python
import requests
from bs4 import BeautifulSoup

page = requests.get("https://books.toscrape.com/", timeout=10)
page.raise_for_status()
soup = BeautifulSoup(page.text, "html.parser")
for h3 in soup.find_all("h3")[:3]:
    link = h3.find("a")
    print(link["title"])
# A Light in the Attic
# Tipping the Velvet
# Soumission
```

**How to read this example:** books.toscrape.com is a sandbox site built for scraping practice. The request gets the same `timeout` and `raise_for_status()` as any API call; each book title is in an `<h3>`, and `link["title"]` reads an attribute of the `<a>` tag.

A live website may block automated requests, render content with JavaScript, or change its HTML. Check the site's terms and use a documented API when one is provided.

---

## 6. How to Learn a New Python Library

Follow a predictable workflow: docs, install, tiny example, inspect, build.

A library is easiest to use when you follow a predictable workflow:

1. Read the official quickstart or examples
2. Install it in the project environment: `uv add requests pandas duckdb matplotlib beautifulsoup4`
3. Try a tiny example in a notebook or scratch script
4. Check the returned shapes and types
5. Build a small real-world function around it

> The course repository already lists these libraries in its `pyproject.toml`, so `uv sync` installs them; use `uv add` in your own projects.

### Read the Shapes a Library Returns

The goal is not to memorize syntax but to understand the data a library returns and the operations it supports. Take pandas: a DataFrame is just a table, and what matters is knowing how to filter, transform, and aggregate it.

**How to read this example:** The dictionary becomes a DataFrame; grouping partitions rows by category, selecting `amount` chooses the measurement, and `sum()` totals each group. The result is a Series indexed by category.

```python
import pandas as pd

sales = pd.DataFrame({
    "month": ["Jan", "Feb", "Jan"],
    "category": ["Food", "Food", "Travel"],
    "amount": [120, 90, 200],
})
print(sales.groupby("category")["amount"].sum())
# category
# Food      210
# Travel    200
# Name: amount, dtype: int64
```

This single example teaches several patterns at once:

- a dict can become a DataFrame
- a column can be selected as a Series
- grouping creates a summary by category
- the output is easy to read and easy to plot

Do not write a long chain of transformations before checking each step: print `df.head()`, `df.columns.tolist()`, and `df.dtypes`, make one transformation, and inspect again.

When you work with a new library, ask three questions:

1. What structure does it return?
2. Which columns or methods are relevant?
3. What kind of transformation does the library make easy?

This approach avoids the trap of copying code you do not understand.

> Go to **Exercise 6 (Library Pipeline Challenge)**, where you will clean, filter, and group a small list of records that includes an invalid row.

---

## 7. Putting the Libraries Together

One pipeline, one job per library.

### Choosing Between Tools

Imagine you need a monthly category chart from an online expense feed:

1. Use `requests` to retrieve the feed and handle the response.
2. Convert the returned records into a pandas DataFrame.
3. Clean the columns and aggregate categories with pandas or DuckDB.
4. Use Matplotlib to create a chart with labels and context.

Each library has a distinct job in that pipeline. Keeping the boundaries clear makes it easier to replace one tool later—for instance, changing the API source does not need to change the plotting code if both sources produce the same clean table.

> Go to **Exercise 5 (Combined Pipeline)**, where you will fetch the course countries dataset, build a DataFrame, query it with DuckDB, and chart the ten most populous countries.

---

## 8. Common Technical Pitfalls

- Calling `.head()` on a `None` result from an API or parser
- Mixing string values and numeric values in a DataFrame without cleaning
- Assuming all CSV files have the same columns
- Forgetting to check internet access before scraping
- Ignoring rate limits: unauthenticated GitHub API calls stop with `403` after 60 requests per hour per IP
- Not reading the library docs for missing parameters

The main lesson is simple: if a library returns a structure you do not understand, inspect it before building logic around it.

---

## Quick Reference

```
requests    get(url, params={...}, timeout=10)
            raise_for_status()  .json()  .status_code
pandas      pd.read_csv  .head()  .shape  .dtypes
            df[df["amount"] > 0]  sort_values  groupby().agg
            rename(columns=...)  nlargest(n, "col")
duckdb      duckdb.sql("SELECT ... FROM df").df()  /  .show()
matplotlib  fig, ax = plt.subplots()  ax.bar  ax.pie
            ax.set_title  fig.savefig("chart.png")  plt.show()
bs4         BeautifulSoup(html, "html.parser")
            find  find_all  get_text(strip=True)  tag["attr"]
install     uv add requests pandas duckdb matplotlib beautifulsoup4
```
