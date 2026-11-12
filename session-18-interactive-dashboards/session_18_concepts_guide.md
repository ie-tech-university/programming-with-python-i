# Session XVIII: Concepts Guide
## Interactive Dashboards with Streamlit

Build interactive dashboards with widgets, charts, session state, caching.

---

## 1. What Is Streamlit?

Streamlit turns Python scripts into interactive web apps. No HTML, CSS, or JavaScript needed.

**How to read this example:** `import streamlit as st` uses the conventional short name, and display calls add components to the page. Streamlit reruns the script after widget interactions, so it behaves differently from a one-time terminal program.

```python
import streamlit as st

st.title("My Dashboard")
st.write("Hello, World!")
```

**How to read these commands:** `uv add streamlit` adds Streamlit to your project's dependencies; `uv run streamlit run app.py` starts the app inside the project environment and opens it in your browser.

```bash
$ uv add streamlit
$ uv run streamlit run app.py
```

---

## 2. Display Elements

**How to read this example:** Headings create hierarchy, metrics emphasize values, and tables show records. Label periods and units so the displayed numbers have context.

```python
st.title("Main Title")
st.header("Section Header")
st.subheader("Subsection")
st.write("Any text or data")
st.markdown("**Bold** and *italic*")
st.code("print('hello')", language="python")
st.dataframe(df)         # Interactive table
st.table(df)             # Static table
st.metric("Revenue", "$10,000", "+5%")
```

> Go to **Exercise 1 (Basic Dashboard)**, where you will build a page with a title, three metrics and a data table.

---

## 3. Input Widgets

**How to read this example:** Widgets return current user values that normal Python can use for filtering or validation. A button reports an interaction and lets the script respond on the rerun.

```python
name = st.text_input("Your name:")
age = st.number_input("Age:", min_value=0, max_value=120)
category = st.selectbox("Category:", ["Food", "Transport", "Housing"])
categories = st.multiselect("Filter:", ["Food", "Transport", "Housing"])
date = st.date_input("Date:")
amount = st.slider("Amount:", 0, 1000, 100)
agree = st.checkbox("I agree")
if st.button("Submit"):
    st.success("Done!")
```

---

## 4. Charts

**How to read this example:** Aggregate before plotting when the question concerns category totals; use ordered dates for a time-series line. Chart type and labels affect what viewers infer.

```python
import matplotlib.pyplot as plt

# Built-in charts: aggregate first, then plot
st.line_chart(df.groupby("date")["amount"].sum())
st.bar_chart(df.groupby("category")["amount"].sum())

# Matplotlib: build a figure, then pass it to st.pyplot
totals = df.groupby("category")["amount"].sum()
fig, ax = plt.subplots()
ax.bar(totals.index, totals.values)
ax.set_ylabel("Amount ($)")
st.pyplot(fig)
```

> Go to **Exercise 3 (Charts)**, where you will chart spending by category with bars and spending over time with a line.

---

## 5. Layout

**How to read this example:** Columns, sidebars, and tabs group related information and controls. Layout guides attention, so put controls near their results.

```python
# Columns
col1, col2 = st.columns(2)
with col1:
    st.metric("Income", "$3,000")
with col2:
    st.metric("Expenses", "$1,200")

# Sidebar
with st.sidebar:
    st.header("Filters")
    category = st.selectbox("Category:", options)

# Tabs
tab1, tab2 = st.tabs(["Overview", "Details"])
with tab1:
    st.write("Overview content")
```

---

## 6. Caching

**How to read this example:** The cache can reuse data for the same function inputs across script reruns. Consider how stale the result can be and when it should refresh.

```python
@st.cache_data
def load_data():
    return pd.read_csv("expenses.csv")

df = load_data()  # Only reads file once, then cached
```

---

## 7. Session State

**How to read this example:** Initialization creates a value only on the first run; session state retains it between reruns. It is not permanent storage: reloading the page or opening a new tab starts with empty session state.

```python
# Persist data across reruns
if "counter" not in st.session_state:
    st.session_state.counter = 0

if st.button("Increment"):
    st.session_state.counter += 1

st.write(f"Count: {st.session_state.counter}")
```

---

## Quick Reference

The commands and functions from this session at a glance:

```
INSTALL     uv add streamlit
RUN         uv run streamlit run app.py
DISPLAY     st.write  st.dataframe  st.metric  st.code
INPUT       st.text_input  st.selectbox  st.slider  st.button
FORMS       st.form  st.form_submit_button
CHARTS      st.line_chart  st.bar_chart  st.pyplot(fig)
LAYOUT      st.columns  st.sidebar  st.tabs
CACHE       @st.cache_data
STATE       st.session_state
```

---


---

## 8. Designing a Good Dashboard

A dashboard is not just a collection of charts. It should answer one question quickly:

- What matters most?
- What should the user change?
- What do I want the user to notice first?

### Good Dashboard Rules

- keep the first screen understandable
- show the most important metrics early
- let the user filter the data interactively
- make the layout clean and focused
- avoid clutter and unnecessary widgets

### A Sidebar Filter Pattern

**How to read this example:** `st.sidebar.selectbox()` and `st.sidebar.checkbox()` place controls in the sidebar without a `with` block. Their return values then filter what the main area shows.

```python
st.sidebar.header("Filters")
category = st.sidebar.selectbox(
    "Category", ["All", "Food", "Transport"]
)
only_large = st.sidebar.checkbox("Only amounts over $100")
```

This pattern gives users control without overwhelming them.

> Go to **Exercise 6 (Dashboard Hardening + Filters)** to practice a more polished, production-style dashboard.

---

## 9. Common Dashboard Mistakes

- Using too many charts on the same screen
- Displaying raw unfiltered data without context
- Recomputing expensive logic on every interaction
- Forgetting state management for multi-step workflows
- Hiding essential controls in hard-to-find places

A useful dashboard balances clarity and interactivity.

---

## 10. Professional Dashboard Thinking

When you build a dashboard, think like a product designer as much as a programmer:
- what does the user need to know?
- how will they explore the data?
- which metrics are actionable?
- how do I keep the app responsive?

The result should feel intuitive, not overwhelming.


---

## Deep Dive: A Dashboard Is a Decision Tool

A dashboard is not just a pretty collection of charts. It is a tool for helping a person make a decision faster.

For example, an expense dashboard is useful when it helps the user quickly answer questions like:

- Which category is spending the most?
- How much did I spend this month?
- Which records are above my usual threshold?

**How to read this example:** The records form a DataFrame; the metric shows overall spending, while the grouped chart reveals category composition. Together they explain scale and distribution. Label the period and whether income is included so users understand the total.

```python
import streamlit as st
import pandas as pd

expenses = pd.DataFrame({
    "category": ["Food", "Food", "Transport", "Housing"],
    "amount": [80, 45, 30, 1200]
})

st.title("Expense Dashboard")
st.metric("Total", f"${expenses['amount'].sum():.2f}")
st.bar_chart(expenses.groupby("category")["amount"].sum())
```

This is a good dashboard because it turns the records into one headline number and one comparison. With four rows or forty thousand, the user sees the important pattern immediately without reading raw rows.

A great dashboard usually follows a few rules:

- highlight the most important metric first
- keep filters near the top
- avoid too many charts on the same screen
- use session state to remember user choices
- make every interaction feel intentional and clear

This is where data science meets product thinking: the app should help the user understand, not overwhelm them.


---

## Understanding Streamlit's App Model

Streamlit lets you describe a web interface with Python statements. Its most important behavior to understand is that the script runs from top to bottom again when a user interacts with a widget. This rerun model explains how widgets, caching, and session state fit together.

### Display Functions Build the Page in Order

Calls such as `st.title()`, `st.write()`, `st.metric()`, and `st.dataframe()` add elements to the page as the script executes. The order of those calls is usually the order users see on the page.

Use the display function that fits the information:

- `st.metric()` emphasizes one key number and an optional comparison.
- `st.dataframe()` lets a user inspect and interact with a table.
- `st.table()` presents a small static table without the same interactive behavior.
- `st.write()` is flexible for quick output, but explicit components often communicate intent more clearly.

For a dashboard, a heading should tell users what they are looking at, while metric labels should include enough context to interpret values. “Total” is ambiguous; “Expenses this month” explains what the number represents.

### Widgets Return Values That Drive the App

A widget such as `st.selectbox()` returns the selected value. You use that value in ordinary Python logic to filter data:

**How to read this example:** The selectbox returns the chosen category, and the conditional uses that value to filter rows. The `All` option bypasses filtering. Streamlit reruns this logic when the selection changes, refreshing the page.

```python
category = st.selectbox("Category", ["All", "Food", "Transport"])

if category == "All":
    filtered = expenses
else:
    filtered = expenses[expenses["category"] == category]
```

When the selection changes, Streamlit reruns the script. It recreates `filtered` using the new selection and redraws the page. You generally do not need to write custom JavaScript event handlers for this simple interaction.

Choose widgets that match the decision the user needs to make. A `selectbox` works well when the user should choose one known option; a `multiselect` allows several; a `slider` suits a bounded numeric range; and a `date_input` is more understandable than asking users to type a date in an arbitrary format.

### Range and Multiple-Choice Filters

A few widget options turn single values into ranges:

- `st.set_page_config()` sets the browser tab title and a wide layout; call it once, at the top of the script.
- `default=` chooses what a `multiselect` starts with, here every option.
- A tuple as the value of `st.slider()` or `st.date_input()` makes it a range: it returns two values. While the user is picking dates, it briefly returns only one.
- `isin(list)` is `True` where the value is in the list; `between(low, high)` where it is in the range, ends included. `&` combines the conditions.

```python
from datetime import date

st.set_page_config(page_title="Expenses", layout="wide")

options = ["Food", "Transport", "Housing"]
cats = st.multiselect("Categories", options, default=options)
low, high = st.slider("Amount ($)", 0, 1000, (0, 1000))
days = st.date_input("Dates", (date(2026, 1, 1), date(2026, 1, 31)))

mask = df["category"].isin(cats) & df["amount"].between(low, high)
if len(days) == 2:  # only one date while the user is picking
    start, end = pd.to_datetime(days)
    mask &= df["date"].between(start, end)
filtered = df[mask]
```

> Go to **Exercise 2 (Interactive Filters)**, where you will filter expenses by category, amount range and date range from the sidebar.

### Charts Should Match the Kind of Comparison

A bar chart is useful for comparing categories. A line chart is useful for values changing over an ordered dimension such as date. The type of chart affects what patterns users can accurately see.

**How to read this example:** Grouping creates one total per category before the bar chart compares them. To show a time trend, aggregate by date and use a line chart. The visualization should answer the analytical question rather than simply decorate the page.

```python
category_totals = filtered.groupby("category")["amount"].sum()
st.bar_chart(category_totals)
```

This groups transactions before plotting, so each bar represents a category total rather than an individual transaction. A line chart should generally use a meaningful order such as time; connecting unrelated categories with a line can imply a trend that does not exist.

Include labels, units, and a clear title. If the underlying data is empty after filtering, show an explanatory message instead of displaying a blank chart that could look broken.

### Layout Controls Emphasis

`st.columns()` places related metrics side by side. `st.sidebar` is useful for controls that affect the main view, and tabs can separate distinct views when users need to switch between them.

Layout is a communication choice: put the most important information where users will notice it first, and group controls near the results they affect. Avoid placing every chart and filter on one screen just because the framework makes it easy.

### Caching Saves Repeated Work

Because the script reruns on interaction, expensive data loading or calculation can happen repeatedly. `@st.cache_data` tells Streamlit that a function's result can be reused for the same inputs:

**How to read this example:** The cache reuses a DataFrame for the same file path, avoiding repeated reads during widget reruns. The data may become stale if the file changes, so decide when to refresh and avoid sharing sensitive user-specific results.

```python
@st.cache_data
def load_expenses(path: str) -> pd.DataFrame:
    return pd.read_csv(path)
```

On a later rerun with the same `path`, Streamlit can return the cached DataFrame instead of reading the file again. Function arguments are part of the cache key, so changing the path can produce a different cached result.

Caching is for data that can safely be reused. If the data changes outside the app, a cached value may be stale; decide how users refresh it or how frequently it should be recomputed. Do not cache user-specific sensitive data in a way that could expose one user's result to another.

### Session State Preserves Values Across Reruns

Ordinary local variables are recreated when the script reruns. `st.session_state` stores values for a user's current session:

**How to read this example:** The initialization check creates the counter once; session state preserves it through later reruns. Clicking the button increments the saved value. A regular local variable would be recreated and lose its previous value.

```python
if "count" not in st.session_state:
    st.session_state.count = 0

if st.button("Add one"):
    st.session_state.count += 1

st.write(st.session_state.count)
```

The initialization check matters because the first run has no `"count"` key. On later runs, the saved value is reused. Session state is useful for a multi-step form, a counter, or selections that should persist between interactions; it is not a substitute for permanent storage because it lives only in one open connection to the app: reloading the page or opening a new tab starts with empty session state.

### Forms Group Related Inputs

If a page has several fields that should be submitted together, a Streamlit form groups them so that changes are collected and processed when the user presses the submit button. This avoids recalculating the submission flow after every individual field change.

**How to read this example:** Widgets inside `with st.form("add_expense"):` do not rerun the script while the user types. `st.form_submit_button()` returns `True` only on the rerun caused by the click, so the record is validated and saved once.

```python
if "expenses" not in st.session_state:
    st.session_state.expenses = []

with st.form("add_expense"):
    description = st.text_input("Description")
    amount = st.number_input("Amount ($)", min_value=0.0, step=1.0)
    submitted = st.form_submit_button("Add expense")

if submitted:
    if not description.strip():
        st.error("Please enter a description.")
    elif amount <= 0:
        st.error("The amount must be positive.")
    else:
        st.session_state.expenses.append(
            {"description": description, "amount": amount}
        )
        st.success(f"Added {description}: ${amount:.2f}")
```

Validate submitted values before adding a record. For example, reject blank descriptions and ensure an amount is positive. Then give immediate feedback with a success or error message so users know what happened.

The list of saved expenses lives in `st.session_state`, so it survives the reruns caused by later clicks and filters.

> Go to **Exercise 4 (Session State)**, where you will build an expense form that validates each entry, saves it in session state and shows a running total.

### Empty, Invalid, and Slow Data Need Visible States

A dashboard should explain what is happening when there is no data or a filter removes every row:

**How to read this example:** Filtering can legitimately produce no rows. Checking `.empty` lets the dashboard explain that state and suggest changing filters instead of showing a blank area that could look like an application failure.

```python
if filtered.empty:
    st.info("No expenses match these filters. Try a different category or amount.")
else:
    st.dataframe(filtered)
```

Similarly, show a clear error when a source file is missing or malformed, and consider a loading indicator for slow operations. A well-designed dashboard does not leave users guessing whether a blank area means “no matching records,” “still loading,” or “the app failed.”

### A User Interaction from Start to Finish

For a category filter, the complete rerun looks like this:

1. Streamlit runs the script and reads the expense data.
2. The category widget displays its current selection.
3. The user chooses a different category.
4. Streamlit reruns the script with the new widget value.
5. Python filters the DataFrame using that value.
6. The metrics and chart are rendered from the filtered result.

This mental model helps explain why local variables reset, why session state is needed for persistent interaction values, and why caching can prevent expensive repeated work.

> Go to **Exercise 5: Full Dashboard (Capstone)**, where you will combine cached loading, sidebar filters, metrics, charts in tabs and an expense form kept in session state.
