# Session XIX: Concepts Guide
## Advanced Lab — Data Explorer and Visualizer with API Integration

This is the **capstone lab** of the entire Python I course. You will combine **every skill** you have learned across Sessions I–XVIII into a single, portfolio-grade project: a data exploration tool that fetches live data from a public API, stores it locally, analyzes it with DuckDB, and visualizes it through an interactive Streamlit dashboard.

There is **no new theory** in this session. Instead, this guide serves as an **architecture reference** — a blueprint for how all the pieces fit together.

---

## 1. Project Overview

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
│                    STREAMLIT DASHBOARD                   │
│  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Sidebar  │  │   Charts &   │  │   Data Tables    │  │
│  │ Filters  │  │   Metrics    │  │   & Details      │  │
│  └────┬─────┘  └──────┬───────┘  └────────┬─────────┘  │
│       │               │                    │            │
│       └───────────────┼────────────────────┘            │
│                       │                                 │
├───────────────────────┼─────────────────────────────────┤
│              ANALYSIS LAYER (DuckDB)                    │
│  ┌─────────────────────────────────────────────────┐    │
│  │  SQL queries: GROUP BY, WHERE, ORDER BY, JOIN   │    │
│  └──────────────────────┬──────────────────────────┘    │
│                         │                               │
├─────────────────────────┼───────────────────────────────┤
│               DATA STORAGE LAYER                        │
│  ┌──────────┐     ┌──────────┐     ┌──────────────┐    │
│  │ data.json│     │ data.csv │     │  cache/       │    │
│  └──────────┘     └──────────┘     └──────────────┘    │
│                         ▲                               │
├─────────────────────────┼───────────────────────────────┤
│              DATA FETCHING LAYER                        │
│  ┌─────────────────────────────────────────────────┐    │
│  │  requests.get() → parse JSON → validate → save  │    │
│  └─────────────────────────────────────────────────┘    │
│                         ▲                               │
│                    Public API                           │
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
│   ├── raw/               # Original API responses
│   │   └── .gitkeep
│   └── processed/         # Cleaned/transformed files
│       └── .gitkeep
├── src/
│   ├── __init__.py
│   ├── fetcher.py         # API fetching functions
│   ├── storage.py         # Save/load JSON and CSV
│   ├── analyzer.py        # DuckDB queries
│   └── visualizer.py      # Streamlit chart functions
├── app.py                 # Streamlit entry point
└── tests/
    └── test_fetcher.py    # Optional: basic tests
```

### Why This Structure Matters

| Directory/File | Purpose | Sessions Used |
|---------------|---------|---------------|
| `src/fetcher.py` | API calls with error handling | S12 (APIs), S2 (error handling) |
| `src/storage.py` | JSON/CSV read & write | S5 (File I/O) |
| `src/analyzer.py` | DuckDB SQL queries | S17 (pandas & DuckDB) |
| `src/visualizer.py` | Chart-building functions | S18 (Streamlit) |
| `app.py` | Main dashboard | S18 (Streamlit) |
| `data/` | Persistent storage | S5 (File I/O), S7 (automation) |

---

## 3. Layer 1 — Data Fetching

### Pattern: Fetch → Validate → Return

```python
import requests
from typing import Optional


def fetch_data(url: str, params: Optional[dict] = None) -> dict:
    """Fetch JSON data from a public API.

    Args:
        url: The API endpoint URL.
        params: Optional query parameters.

    Returns:
        Parsed JSON response as a dictionary.

    Raises:
        requests.exceptions.RequestException: On network errors.
        ValueError: If response is not valid JSON.
    """
    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.Timeout:
        print("Error: Request timed out.")
        raise
    except requests.exceptions.HTTPError as e:
        print(f"Error: HTTP {response.status_code} — {e}")
        raise
    except requests.exceptions.JSONDecodeError:
        print("Error: Response is not valid JSON.")
        raise
```

### Recommended Public APIs (No Auth Required)

| API | URL | Data Type |
|-----|-----|-----------|
| REST Countries | `https://restcountries.com/v3.1/all` | Country info |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | Weather data |
| CoinGecko | `https://api.coingecko.com/api/v3/coins/markets` | Crypto prices |
| JSONPlaceholder | `https://jsonplaceholder.typicode.com/` | Mock data |
| Universities | `http://universities.hipolabs.com/search` | University list |

### Caching Pattern: Don't Re-Fetch

```python
import os
import json


def fetch_with_cache(url: str, cache_path: str, force_refresh: bool = False) -> list:
    """Fetch from API or load from local cache."""
    if os.path.exists(cache_path) and not force_refresh:
        print(f"Loading from cache: {cache_path}")
        with open(cache_path, "r") as f:
            return json.load(f)

    print(f"Fetching from API: {url}")
    data = fetch_data(url)

    os.makedirs(os.path.dirname(cache_path), exist_ok=True)
    with open(cache_path, "w") as f:
        json.dump(data, f, indent=2)

    return data
```

> This pattern is critical for Streamlit apps — without caching, every user interaction triggers a new API call, making the app slow and hitting rate limits.

---

## 4. Layer 2 — Data Storage

### Saving and Loading JSON

```python
import json


def save_json(data: list | dict, filepath: str) -> None:
    """Save data to a JSON file."""
    with open(filepath, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
    print(f"Saved {len(data) if isinstance(data, list) else 1} records to {filepath}")


def load_json(filepath: str) -> list | dict:
    """Load data from a JSON file."""
    with open(filepath, "r", encoding="utf-8") as f:
        return json.load(f)
```

### Saving and Loading CSV

```python
import csv


def save_csv(data: list[dict], filepath: str) -> None:
    """Save a list of dictionaries to a CSV file."""
    if not data:
        print("Warning: No data to save.")
        return

    fieldnames = data[0].keys()
    with open(filepath, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)
    print(f"Saved {len(data)} rows to {filepath}")


def load_csv(filepath: str) -> list[dict]:
    """Load a CSV file as a list of dictionaries."""
    with open(filepath, "r", encoding="utf-8") as f:
        return list(csv.DictReader(f))
```

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
    result = con.execute(sql).fetchall()
    con.close()
    return result


# Example: query a CSV file directly
results = query_data(
    "data/countries.csv",
    """
    SELECT region, COUNT(*) as count, ROUND(AVG(population), 0) as avg_pop
    FROM 'data/countries.csv'
    WHERE population > 1000000
    GROUP BY region
    ORDER BY count DESC
    """
)
```

### Dynamic Queries Based on User Input

```python
def get_filtered_data(filepath: str, region: str = None, min_pop: int = 0) -> list[tuple]:
    """Query data with optional filters."""
    conditions = [f"population >= {min_pop}"]
    if region:
        conditions.append(f"region = '{region}'")

    where_clause = " AND ".join(conditions)

    sql = f"""
        SELECT name, region, population, area
        FROM '{filepath}'
        WHERE {where_clause}
        ORDER BY population DESC
        LIMIT 50
    """

    con = duckdb.connect()
    result = con.execute(sql).fetchall()
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

---

## 6. Layer 4 — Visualization with Streamlit

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
def load_country_data():
    """Fetch and cache country data."""
    url = "https://restcountries.com/v3.1/all"
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

### Common Streamlit Chart Functions

| Function | Use Case |
|----------|----------|
| `st.bar_chart(df)` | Quick bar chart from a DataFrame |
| `st.line_chart(df)` | Time series or trends |
| `st.map(df)` | Geographic data (needs `lat`/`lon` columns) |
| `st.dataframe(df)` | Interactive, sortable table |
| `st.metric(label, value, delta)` | KPI cards |

---

## 7. Putting It All Together — The Complete Workflow

### Step-by-Step Integration

```
1. FETCH:    fetcher.py   →  Call API, get raw JSON
2. STORE:    storage.py   →  Save to data/raw/countries.json
3. CLEAN:    storage.py   →  Flatten nested fields, save to data/processed/countries.csv
4. ANALYZE:  analyzer.py  →  Run DuckDB queries, return results
5. DISPLAY:  app.py       →  Streamlit reads results, renders charts
```

### The `app.py` Entry Point Pattern

```python
"""Data Explorer — Main Streamlit Application."""
import streamlit as st
from src.fetcher import fetch_with_cache
from src.storage import save_csv, flatten_countries
from src.analyzer import get_region_summary, get_top_countries

# ── Configuration ──
st.set_page_config(page_title="Data Explorer", layout="wide")
RAW_PATH = "data/raw/countries.json"
CSV_PATH = "data/processed/countries.csv"

# ── Data Pipeline ──
@st.cache_data(ttl=3600)
def load_data():
    raw = fetch_with_cache("https://restcountries.com/v3.1/all", RAW_PATH)
    flat = flatten_countries(raw)
    save_csv(flat, CSV_PATH)
    return flat

data = load_data()

# ── Dashboard ──
st.title("🌍 World Data Explorer")

# ... sidebar filters, metrics, charts, tables ...
```

### Running the Application

```bash
# From the project root
$ uv run streamlit run app.py
```

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
| Functions, modules, `__init__.py` | S13 | ☐ |
| REST APIs with `requests` | S12 | ☐ |
| Flask web routes | S15 | ☐ |
| FastAPI endpoints | S16 | ☐ |
| pandas and DuckDB queries | S17 | ☐ |
| Streamlit dashboards | S18 | ☐ |

---

## Quick Reference — Key Imports

```python
# ── Standard Library ──
import json
import csv
import os
from typing import Optional

# ── Third Party ──
import requests          # pip install requests
import duckdb            # pip install duckdb
import streamlit as st   # pip install streamlit
```

---

## Quick Reference — File Operations Cheat Sheet

```
┌──────────────────────────────────────────────────────────┐
│  JSON SAVE      json.dump(data, f, indent=2)             │
│  JSON LOAD      data = json.load(f)                      │
│  CSV SAVE       csv.DictWriter(f, fieldnames).writerows() │
│  CSV LOAD       list(csv.DictReader(f))                   │
│  DUCKDB QUERY   duckdb.connect().execute(sql).fetchall()  │
│  API FETCH      requests.get(url, timeout=10).json()      │
│  ST CACHE       @st.cache_data(ttl=3600)                  │
│  ST RUN         streamlit run app.py                      │
└──────────────────────────────────────────────────────────┘
```

---
