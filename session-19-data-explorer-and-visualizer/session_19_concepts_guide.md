# Session XIX: Concepts Guide
## Advanced Lab — Data Explorer and Visualizer with API Integration

This is the **capstone lab** of the entire Python I course. You will combine **every skill** you have learned across Sessions I–XVIII into a single, portfolio-grade project: a data exploration tool that fetches live data from a public API, stores it locally, analyzes it with DuckDB, and visualizes it through an interactive Streamlit dashboard.

There is **little new theory** in this session. The few tools the exercises need that earlier sessions did not cover (`io.StringIO`, `st.download_button`, `st.expander`, `st.error` with `st.stop`, and DuckDB's `ILIKE` and `UNNEST`) each get a short section where they are used. Otherwise, this guide serves as an **architecture reference** — a blueprint for how all the pieces fit together.

---

## 1. Project Overview

What you will build, and the four layers that turn API data into a dashboard.

### What You Will Build

A complete **Data Explorer and Visualizer** application that:

1. **Fetches** live data from a public REST API using `requests`
2. **Stores** the retrieved data in JSON or CSV files for offline reuse
3. **Loads and analyzes** data using DuckDB (filtering, aggregation, joins)
4. **Visualizes** insights through an interactive Streamlit dashboard
5. **Packages** all functionality into clean, reusable modules

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│ STREAMLIT DASHBOARD · app.py                            │
│  ┌───────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Sidebar   │  │   Charts &   │  │   Data tables    │  │
│  │ filters   │  │   metrics    │  │   & details      │  │
│  └───────────┘  └──────────────┘  └──────────────────┘  │
│                           ▲                             │
├───────────────────────────┼─────────────────────────────┤
│ ANALYSIS LAYER (DuckDB) · analyzer.py                   │
│  SQL queries: GROUP BY, WHERE, ORDER BY, JOIN           │
│                           ▲                             │
├───────────────────────────┼─────────────────────────────┤
│ DATA STORAGE LAYER · processor.py                       │
│  data/raw/countries.json → data/processed/countries.csv │
│                           ▲                             │
├───────────────────────────┼─────────────────────────────┤
│ DATA FETCHING LAYER · fetcher.py                        │
│  requests.get() → parse JSON → validate list → cache    │
│                           ▲                             │
│                       Public API                        │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Recommended Project Structure

Organize your project using the `src/` layout you learned in Session I:

```
data-explorer/
├── pyproject.toml
├── README.md
├── .gitignore
├── data/
│   ├── raw/               # countries.json cached by fetcher.py
│   │   └── .gitkeep
│   └── processed/         # countries.csv written by processor.py
│       └── .gitkeep
├── src/
│   ├── __init__.py
│   ├── fetcher.py         # Download, validate and cache the JSON
│   ├── processor.py       # Flatten records, write CSV
│   └── analyzer.py        # DuckDB queries
├── app.py                 # Streamlit entry point
└── tests/
    └── test_fetcher.py    # Optional: basic tests
```

### Why This Structure Matters

| Directory/File | Purpose | Sessions Used |
|---------------|---------|---------------|
| `src/fetcher.py` | API calls with error handling, JSON cache | S12 (APIs), S2 (error handling) |
| `src/processor.py` | Flatten nested records, write CSV | S3 (nested data), S5 (File I/O) |
| `src/analyzer.py` | DuckDB SQL queries | S14, S17 (DuckDB) |
| `app.py` | Main dashboard | S18 (Streamlit) |
| `data/` | Persistent storage | S5 (File I/O), S7 (automation) |

---

## 3. Layer 1 — Data Fetching

`fetcher.py` downloads the countries once, checks them and caches them on disk.

### Pattern: Fetch → Validate → Return

```python
import requests


def fetch_countries(
    url: str, params: dict | None = None
) -> list[dict]:
    """Fetch the list of countries from a JSON API.

    Raises:
        requests.RequestException: On network or HTTP errors.
        ValueError: If the response is not JSON or not a list.
    """
    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        data = response.json()
    except requests.exceptions.Timeout:
        print("Error: Request timed out.")
        raise
    except requests.exceptions.HTTPError as e:
        print(f"Error: HTTP {response.status_code} — {e}")
        raise
    except requests.exceptions.JSONDecodeError:
        print("Error: Response is not valid JSON.")
        raise

    # A 200 response may still be an error message
    if not isinstance(data, list):
        kind = type(data).__name__
        raise ValueError(f"Expected a list, got a {kind}")
    return data
```

### Recommended Public APIs (No Auth Required)

| API | URL | Data Type |
|-----|-----|-----------|
| Countries (course dataset, REST Countries format) | `https://raw.githubusercontent.com/ie-tech-university/programming-with-python-i/main/data/countries.json` | Country info |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | Weather data |
| CoinGecko | `https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur` | Crypto prices |
| JSONPlaceholder | `https://jsonplaceholder.typicode.com/` | Mock data |
| Universities | `http://universities.hipolabs.com/search` | University list |

### Caching Pattern: Don't Re-Fetch

```python
import json
import os


def fetch_with_cache(url: str, cache_path: str, force_refresh: bool = False) -> list[dict]:
    """Fetch from the API, or load the copy saved by an earlier run."""
    if os.path.exists(cache_path) and not force_refresh:
        print(f"Loading from cache: {cache_path}")
        with open(cache_path, "r", encoding="utf-8") as f:
            return json.load(f)

    print(f"Fetching from API: {url}")
    data = fetch_countries(url)  # a validated list, or an exception

    # dirname() is "" for a bare file name such as "countries.json"
    os.makedirs(os.path.dirname(cache_path) or ".", exist_ok=True)
    with open(cache_path, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
    return data
```

> This pattern is critical for Streamlit apps — without caching, every user interaction triggers a new API call, making the app slow and hitting rate limits.

Because `fetch_countries()` validates the data before returning it, an error response is never written to the cache: the next run simply tries the API again.

> Go to **Exercise 1: Data Fetcher Module (`src/fetcher.py`)**, where you will write `fetch_countries()` and `fetch_with_cache()` with type hints, docstrings and separate error handling.

---

## 4. Layer 2 — Data Storage

`processor.py` flattens the raw JSON into rows and saves them as CSV.

### Flattening Nested Records (`processor.py`)

`.get()` with a default keeps one incomplete record from crashing the pipeline; `process_all()` skips and reports records that still fail:

```python
def flatten_country(country: dict) -> dict:
    """Turn one nested country record into a flat row."""
    capitals = country.get("capital", [])
    languages = country.get("languages", {})
    return {
        "name": country.get("name", {}).get("common", "Unknown"),
        "region": country.get("region", "Unknown"),
        "population": country.get("population", 0),
        "capital": capitals[0] if capitals else "N/A",
        "languages": ", ".join(sorted(languages.values())) or "N/A",
        # Exercise 2 adds official_name, subregion, area, currencies
    }


def process_all(raw_data: list[dict]) -> list[dict]:
    """Flatten every record; skip and report the ones that fail."""
    records = []
    for country in raw_data:
        try:
            records.append(flatten_country(country))
        except (AttributeError, IndexError, TypeError) as e:
            print(f"Warning: skipped a record ({e})")
    return records


spain = {"name": {"common": "Spain"}, "region": "Europe",
         "population": 47351567, "capital": ["Madrid"],
         "languages": {"spa": "Spanish"}}
rows = process_all([spain, "not a dict"])
# Warning: skipped a record ('str' object has no attribute 'get')
print(len(rows), rows[0]["capital"], rows[0]["languages"])
# 1 Madrid Spanish
```

### Saving the Flat Rows as CSV (`processor.py`)

One dictionary per country becomes one CSV row; DuckDB queries this file in the next layer:

```python
import csv
import os


def save_processed_csv(records: list[dict], filepath: str) -> None:
    """Write the flat records to a CSV file, one row per country."""
    if not records:
        print("Warning: No records to save.")
        return

    os.makedirs(os.path.dirname(filepath) or ".", exist_ok=True)
    with open(filepath, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=records[0].keys())
        writer.writeheader()
        writer.writerows(records)
    print(f"Saved {len(records)} records to {filepath}")
```

> Go to **Exercise 2: Data Processing Module (`src/processor.py`)**, where you will flatten all nine fields with `.get()` defaults and save the rows with `csv.DictWriter`.

### When to Use JSON vs CSV

| Feature | JSON | CSV |
|---------|------|-----|
| Nested data | ✅ Natural | ❌ Must flatten |
| Human-readable | ✅ Good | ✅ Great (spreadsheets) |
| DuckDB loading | ✅ Supported | ✅ Supported |
| File size | Larger | Smaller |
| Best for | API responses, configs | Tabular data, exports |

---

## 5. Layer 3 — Analysis with DuckDB

### Loading and Querying Files Directly

DuckDB can query JSON and CSV files **without loading them into Python first**:

```python
import duckdb


def query_data(filepath: str, sql: str) -> list[tuple]:
    """Run a SQL query against a data file."""
    con = duckdb.connect()
    result = con.execute(sql, [filepath]).fetchall()
    con.close()
    return result


# Example: query a CSV file directly
results = query_data(
    "data/processed/countries.csv",
    """
    SELECT region, COUNT(*) as count, ROUND(AVG(population), 0) as avg_pop
    FROM read_csv_auto(?)
    WHERE population > 1000000
    GROUP BY region
    ORDER BY count DESC
    """
)
```

### Dynamic Queries Based on User Input

```python
def get_filtered_data(filepath: str, region: str | None = None, min_pop: int = 0) -> list[tuple]:
    """Query data with optional filters (values go in as ? parameters)."""
    conditions, params = ["population >= ?"], [min_pop]
    if region:
        conditions.append("region = ?")   # never paste user input into SQL
        params.append(region)
    where_clause = " AND ".join(conditions)

    sql = f"""
        SELECT name, region, population, area
        FROM read_csv_auto(?)
        WHERE {where_clause}
        ORDER BY population DESC
        LIMIT 50
    """
    con = duckdb.connect()
    result = con.execute(sql, [filepath, *params]).fetchall()
    con.close()
    return result
```

### Common Aggregation Patterns

```python
# Count by category
"""SELECT region, COUNT(*) as count FROM 'data.csv' GROUP BY region"""

# Average by group
"""SELECT region, AVG(population) as avg_pop FROM 'data.csv' GROUP BY region"""

# Top N records
"""SELECT name, population FROM 'data.csv' ORDER BY population DESC LIMIT 10"""

# Percentage of total
"""
SELECT region,
       COUNT(*) as count,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) as pct
FROM 'data.csv'
GROUP BY region
"""
```

### Searching and Splitting Text in SQL

`ILIKE` is `LIKE` without case (`%` matches any text); `string_split()` turns "English, French" into a list and `UNNEST()` gives each item its own row:

```python
import duckdb

con = duckdb.connect()
con.execute("""CREATE TABLE countries AS SELECT * FROM (VALUES
    ('Spain', 'Spanish'), ('Canada', 'English, French'),
    ('United States', 'English')) AS t(name, languages)""")

# ILIKE ignores case; % matches any text
query = "states"
sql = "SELECT name FROM countries WHERE name ILIKE ? ORDER BY name"
print(con.execute(sql, [f"%{query}%"]).fetchall())
# [('United States',)]

# string_split() makes a list; UNNEST() gives one row per item
print(con.execute("""
    SELECT lang, COUNT(*) AS country_count
    FROM (SELECT UNNEST(string_split(languages, ', ')) AS lang
          FROM countries)
    GROUP BY lang
    ORDER BY country_count DESC, lang
""").fetchall())
# [('English', 2), ('French', 1), ('Spanish', 1)]
```

> Go to **Exercise 3: Analysis Module (`src/analyzer.py`)**, where you will write region summaries, top-N rankings, a name search and language statistics in SQL.

---

## 6. Layer 4 — Visualization with Streamlit

`app.py` turns the analysis into an interactive dashboard.

### Dashboard Structure Pattern

```python
import streamlit as st

# ── Page configuration ──
st.set_page_config(page_title="Data Explorer", layout="wide")
st.title("🌍 Data Explorer Dashboard")

# ── Sidebar: Filters ──
with st.sidebar:
    st.header("Filters")
    selected_region = st.selectbox("Region", ["All"] + regions)
    min_population = st.slider("Min Population", 0, 1_000_000_000, 0)
    refresh = st.button("🔄 Refresh Data")

# ── Main area: Metrics ──
col1, col2, col3 = st.columns(3)
col1.metric("Total Countries", total_count)
col2.metric("Avg Population", f"{avg_pop:,.0f}")
col3.metric("Total Area (km²)", f"{total_area:,.0f}")

# ── Charts ──
tab1, tab2 = st.tabs(["📊 Bar Chart", "📋 Data Table"])

with tab1:
    st.bar_chart(chart_data)

with tab2:
    st.dataframe(table_data)
```

### Using `@st.cache_data` to Avoid Re-Fetching

```python
@st.cache_data(ttl=3600)  # Cache for 1 hour
def load_country_data() -> list[dict]:
    """Fetch and cache country data."""
    url = ("https://raw.githubusercontent.com/ie-tech-university/"
           "programming-with-python-i/main/data/countries.json")
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    if not isinstance(data, list):
        raise ValueError("Expected a list of countries")
    return data
```

> If the function raises an exception, `st.cache_data` stores nothing and the next rerun tries again. Do not catch the error inside and return `[]`: that empty list would be cached for an hour.

> Go to **Exercise 4: Streamlit Dashboard (`app.py`)**, where you will wire the three modules into a dashboard with filters, metrics, tabs and an expander.

### Common Streamlit Chart Functions

| Function | Use Case |
|----------|----------|
| `st.bar_chart(df)` | Quick bar chart from a DataFrame |
| `st.line_chart(df)` | Time series or trends |
| `st.map(df)` | Geographic data (needs `lat`/`lon` columns) |
| `st.dataframe(df)` | Interactive, sortable table |
| `st.metric(label, value, delta)` | KPI cards |

### Exporting Data as a CSV Download

- `io.StringIO()` is a text “file” that lives in memory. `csv.DictWriter` writes to it exactly as it writes to a file opened with `open()`.
- `buffer.getvalue()` returns everything written so far as one string.
- `st.download_button()` sends that string to the browser as a file; `file_name` and `mime` tell the browser what it is.
- `st.divider()` draws a horizontal line to separate parts of a page or sidebar.

```python
import csv
import io

import streamlit as st

rows = [{"name": "Spain", "region": "Europe"},
        {"name": "Japan", "region": "Asia"}]

buffer = io.StringIO()  # a text "file" that lives in memory
writer = csv.DictWriter(buffer, fieldnames=["name", "region"])
writer.writeheader()
writer.writerows(rows)
csv_text = buffer.getvalue()  # the whole CSV as one string

st.divider()  # a horizontal line between sections
st.download_button(
    label="📥 Download CSV",
    data=csv_text,
    file_name="countries.csv",
    mime="text/csv",
)
```

> Go to **Exercise 5: Data Export Feature**, where you will write `records_to_csv_string()` and add a download button for the filtered countries.

### Errors, Refresh and Collapsible Details

- `st.error()` shows a red message box instead of a traceback.
- `st.stop()` ends the current run: nothing after it is drawn, so the rest of the page never sees missing data.
- `st.cache_data.clear()` empties the cache, so the next call to `load_data()` really loads again: useful behind a “Refresh” button.
- `with st.expander("…"):` puts secondary content in a section that starts collapsed.

```python
if st.sidebar.button("🔄 Refresh data"):
    st.cache_data.clear()  # forget all cached results

try:
    data = load_data()
except Exception as e:  # no network, invalid JSON, wrong shape...
    st.error(f"Could not load the data: {e}")
    st.stop()  # end this run here: nothing below is drawn

st.metric("Countries", len(data))
with st.expander("Show the first records"):  # starts collapsed
    st.write(data[:3])
```

---

## 7. Putting It All Together — The Complete Workflow

How the modules connect, from the API call to the browser.

### Step-by-Step Integration

```
1. FETCH:    fetcher.py    →  fetch_countries(): call the API, check for a list
2. STORE:    fetcher.py    →  fetch_with_cache(): keep data/raw/countries.json
3. CLEAN:    processor.py  →  process_all(): flatten the nested records
             processor.py  →  save_processed_csv(): data/processed/countries.csv
4. ANALYZE:  analyzer.py   →  get_region_summary(), get_top_countries(), …
5. DISPLAY:  app.py        →  filters, metrics, charts, download button
```

### The `app.py` Entry Point Pattern

Two caches, two jobs: the JSON file on disk survives restarts and works offline; `@st.cache_data` keeps the flat records in memory so reruns skip reading and flattening:

```python
"""Data Explorer — Main Streamlit Application."""
import os

import streamlit as st

from src.analyzer import get_region_summary, get_top_countries
from src.fetcher import fetch_with_cache
from src.processor import process_all, save_processed_csv

# ── Configuration ──
st.set_page_config(page_title="Data Explorer", layout="wide")
COUNTRIES_URL = ("https://raw.githubusercontent.com/"
                 "ie-tech-university/programming-with-python-i/"
                 "main/data/countries.json")
RAW_PATH = "data/raw/countries.json"
CSV_PATH = "data/processed/countries.csv"


# ── Data pipeline ──
@st.cache_data(ttl=3600)  # errors are not cached
def load_data() -> list[dict]:
    return process_all(fetch_with_cache(COUNTRIES_URL, RAW_PATH))


try:
    data = load_data()
except Exception as e:
    st.error(f"Could not load the data: {e}")
    st.stop()
if not os.path.exists(CSV_PATH):
    save_processed_csv(data, CSV_PATH)  # read by analyzer.py

# ── Dashboard ──
st.title("🌍 World Data Explorer")
summary = get_region_summary(CSV_PATH)
st.bar_chart({r["region"]: r["country_count"] for r in summary})
st.table(get_top_countries(CSV_PATH, n=5))
# ... sidebar filters, metrics, tabs, download button ...
```

### Running the Application

```bash
# From the project root
$ uv run streamlit run app.py
```

> Go to **Exercise 6: Complete Integration and Git Packaging (Capstone)**, where you will connect every module, handle errors with `st.error()`, and publish the project with a README, a `.gitignore` and meaningful commits.

---

## 8. Skills Checklist — Everything You Have Learned

Before starting the lab, confirm you are comfortable with each skill:

| Skill | Session | Confidence? |
|-------|---------|:-----------:|
| Variables, types, f-strings | S1 | ☐ |
| Conditionals and loops | S2 | ☐ |
| Dictionaries and nested structures | S3 | ☐ |
| List comprehensions, `zip`, `lambda` | S4 | ☐ |
| File I/O: JSON and CSV | S5 | ☐ |
| String cleaning and regex | S6 | ☐ |
| CLI scripts and `argparse` | S7 | ☐ |
| OOP: classes, inheritance, polymorphism | S8–S9 | ☐ |
| REST APIs with `requests` | S12 | ☐ |
| Functions, modules, `__init__.py` | S13 | ☐ |
| Libraries: pandas, DuckDB, matplotlib | S14 | ☐ |
| Flask web routes | S15 | ☐ |
| FastAPI endpoints | S16 | ☐ |
| pandas and DuckDB queries | S17 | ☐ |
| Streamlit dashboards | S18 | ☐ |

---

## Quick Reference — Key Imports

```python
# ── Standard Library ──
import csv
import io
import json
import os

# ── Third Party ──
import requests          # uv add requests
import duckdb            # uv add duckdb
import streamlit as st   # uv add streamlit
```

---

## Quick Reference — File Operations Cheat Sheet

```
JSON SAVE      json.dump(data, f, indent=2)
JSON LOAD      data = json.load(f)
CSV SAVE       csv.DictWriter(f, fieldnames).writerows()
CSV LOAD       list(csv.DictReader(f))
CSV STRING     io.StringIO() … buffer.getvalue()
DUCKDB QUERY   duckdb.connect().execute(sql, params).fetchall()
API FETCH      requests.get(url, timeout=10).json()
ST CACHE       @st.cache_data(ttl=3600)
ST EXPORT      st.download_button(label, data, file_name)
ST RUN         uv run streamlit run app.py
```

---
