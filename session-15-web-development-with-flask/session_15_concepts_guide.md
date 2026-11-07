# Session XV: Concepts Guide
## Web Development with Flask

Build web apps with routes, templates, forms using Flask.

---

## 1. What Is Flask?

Flask is a lightweight Python web framework. It maps **URLs to Python functions**.

**How to read this example:** `Flask(__name__)` creates the application and its route registry. Route functions return browser responses, and the main guard starts the local development server only when the file is run directly.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "<h1>Hello, World!</h1>"

if __name__ == "__main__":
    app.run(debug=True)
```

**How to read this example:** The application registers a Python function for a URL and returns its result as an HTTP response. The main guard starts the local server only when the module runs directly.

```bash
$ uv add flask
$ uv run app.py
# Open http://127.0.0.1:5000
```

---

## 2. Routes and Dynamic URLs

**How to read this example:** The decorator maps a URL to a handler; dynamic parts become function arguments, and converters such as `int` validate a path segment.

```python
@app.route("/hello/<name>")
def greet(name):
    return f"<h1>Hello, {name}!</h1>"

@app.route("/expense/<int:expense_id>")
def show_expense(expense_id):
    return f"Expense #{expense_id}"
```

---

## 3. Templates with Jinja2

**How to read this example:** Python supplies records to Jinja, whose loop renders HTML. Separating markup from routing logic makes both easier to maintain.

```
project/
├── app.py
├── templates/
│   ├── base.html
│   ├── home.html
│   └── expenses.html
└── static/
    └── style.css
```

**How to read this example:** Python supplies records to Jinja, whose loop renders HTML. Separating markup from routing logic makes both easier to maintain.

```python
from flask import render_template

@app.route("/expenses")
def expenses():
    data = [{"category": "Food", "amount": 25}, ...]
    return render_template("expenses.html", expenses=data)
```

**How to read this example:** Python supplies records to Jinja, whose loop renders HTML. Separating markup from routing logic makes both easier to maintain.

```html
<!-- templates/expenses.html -->
{% extends "base.html" %}
{% block content %}
<table>
  {% for e in expenses %}
  <tr><td>{{ e.category }}</td><td>${{ e.amount }}</td></tr>
  {% endfor %}
</table>
{% endblock %}
```

---

## 4. Forms and POST Requests

**How to read this example:** GET displays a form; POST submits fields as strings. Convert and validate them before saving, then redirect after success to avoid duplicate submissions on refresh.

```python
from flask import request, redirect, url_for

@app.route("/add", methods=["GET", "POST"])
def add_expense():
    if request.method == "POST":
        category = request.form["category"]
        amount = float(request.form["amount"])
        # Save to database/file...
        return redirect(url_for("expenses"))
    return render_template("add.html")
```

---

## 5. JSON API Endpoints

**How to read this example:** `jsonify()` returns structured data for a programmatic client, unlike a rendered HTML page. Stable keys and meaningful status codes make the API predictable.

```python
from flask import jsonify

@app.route("/api/expenses")
def api_expenses():
    data = load_expenses()
    return jsonify(data)
```

---

## Quick Reference

**How to read this example:** This example applies the concept of Quick Reference. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```
INSTALL     uv add flask
RUN         app.run(debug=True)
ROUTE       @app.route("/path")
TEMPLATE    render_template("file.html", var=value)
FORM        request.form["field"]  (POST)
REDIRECT    redirect(url_for("function_name"))
JSON API    return jsonify(data)
```

---


---

## 7. Flask Patterns That Scale Better

A small Flask app is easy to write, but a larger app becomes chaotic without structure. Keep your code separated by role:

- routes handle HTTP requests
- templates render the UI
- helper functions manage data logic
- configuration stays in one place

### Example

**How to read this example:** This example applies the concept of Example. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("index.html", title="Budget App")
```

This keeps the app readable and easier to extend.

> Go to **Exercise 6 (Form + Redirect Flow)** to practice a realistic user flow.

---

## 8. Debugging and Validation in Flask

When you build a form, check the request method and validate the submitted values before saving data. This prevents invalid entries and confusing behavior.

**How to read this example:** This POST-only route reads an optional form field, trims whitespace, and returns status 400 when required input is blank. Validation at the request boundary prevents invalid data from reaching storage or business logic.

```python
from flask import request

@app.route("/submit", methods=["POST"])
def submit():
    name = request.form.get("name", "").strip()
    if not name:
        return "Name is required", 400
    return f"Hello {name}!"
```

This is exactly the type of defensive code that turns a demo app into a usable one.


---

## Deep Dive: What Makes Flask Useful?

Flask is powerful because it allows you to turn Python functions into web endpoints. This means a developer can build a working web app without writing front-end code from scratch.

A simple Flask route might look like this:

**How to read this example:** This route combines request handling, validation, template rendering, and redirecting. A GET displays a page; a POST reads text, rejects an empty category, or redirects on success. Each helper participates in the browser request-response flow.

```python
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

@app.route("/expenses", methods=["GET", "POST"])
def expenses():
    if request.method == "POST":
        category = request.form.get("category", "").strip()
        if not category:
            return "Category is required", 400
        return redirect(url_for("expenses"))
    return render_template("expenses.html")
```

This example shows a realistic flow:

- receive a form submission
- validate the input
- decide whether to continue or reject it
- redirect the user to a useful next page

That is a website in a very practical sense: request in, validation, response out.

The real strength of Flask is not the syntax alone — it is the way it encourages a clean separation between:

- routing logic
- templates for presentation
- helper functions for data processing

This is exactly the kind of architecture you will see again in larger app projects.


---

## Understanding the Web Request Flow in Flask

Flask connects browser requests to Python functions. The browser requests a URL, Flask finds a matching route, the route function runs, and the function returns a response. That response may be HTML, a redirect, JSON, or an error status.

### Routes Map Addresses to Behavior

The `@app.route(...)` decorator registers a URL pattern. Flask calls the decorated function when a request matches it:

**How to read this example:** The decorator registers the home function for `/`, and its returned string becomes the response body. This is fine for a tiny demonstration; templates are easier to maintain when the page contains more HTML.

```python
@app.route("/")
def home():
    return "<h1>Home</h1>"
```

The browser does not call `home()` directly. It sends an HTTP request for `/`, and Flask dispatches that request to the function. A dynamic segment such as `/hello/<name>` captures part of the URL and passes it as a function argument. A converter such as `<int:expense_id>` asks Flask to parse that segment as an integer; a non-integer path does not match that route.

Use route names and URL patterns that describe resources or actions clearly. For example, `/expenses/42` can represent expense 42, while a route for adding a record may accept a form submission at `/expenses/new`.

### Templates Separate Presentation from Python Logic

`render_template("expenses.html", expenses=data)` passes values from Python into a Jinja template. The template is responsible for the HTML presentation; the route is responsible for selecting and preparing the data.

**How to read this example:** The route passes records into Jinja, whose loop creates one row for each expense. Python prepares the data while the template controls presentation, so the HTML can change without changing data retrieval.

```python
@app.route("/expenses")
def expenses():
    records = [
        {"category": "Food", "amount": 25.00},
        {"category": "Transport", "amount": 12.50},
    ]
    return render_template("expenses.html", expenses=records)
```

**How to read this example:** Python supplies records to Jinja, whose loop renders HTML. Separating markup from routing logic makes both easier to maintain.

```html
{% for expense in expenses %}
  <p>{{ expense.category }}: ${{ "%.2f"|format(expense.amount) }}</p>
{% endfor %}
```

The loop in the template repeats presentation for each record. This is clearer than constructing a large HTML string in the route function. Flask's template engine also escapes ordinary variable output by default, which helps prevent user-provided text from being interpreted as markup. Avoid marking untrusted text as safe HTML.

### GET and POST Have Different Roles

`GET` is commonly used to display a page or retrieve information. `POST` is commonly used to submit information that may change server state, such as adding an expense. A route can accept both methods and branch on `request.method`:

**How to read this example:** GET displays the form; POST receives submitted values as strings. Convert and validate those values before saving, then redirect after success so refreshing the browser does not resubmit the same transaction.

```python
@app.route("/add", methods=["GET", "POST"])
def add_expense():
    if request.method == "POST":
        category = request.form.get("category", "").strip()
        amount_text = request.form.get("amount", "").strip()
        # Validate and save the submitted values here.
        return redirect(url_for("expenses"))
    return render_template("add.html")
```

On the initial `GET`, the app displays the form. On `POST`, the browser sends submitted fields in `request.form`. Using `.get()` allows the route to handle a missing field explicitly instead of immediately raising `KeyError`. The amount still needs numeric conversion and domain validation: form values arrive as strings.

### Redirect After a Successful Form Submission

After processing a successful `POST`, a redirect asks the browser to make a new `GET` request to another route. This is known as the Post/Redirect/Get pattern. It avoids resubmitting the form if the user refreshes the resulting page.

**How to read this example:** `url_for()` finds the URL registered for the `expenses` endpoint, and `redirect()` asks the browser to navigate there. Endpoint-based links avoid duplicating hard-coded paths and make route changes safer.

```python
return redirect(url_for("expenses"))
```

`url_for()` uses the endpoint function name to build the URL, so the route can change without hard-coding its path in every redirect. If validation fails, show a clear message and preserve safe form values where appropriate so the user can correct the entry.

### Return Helpful Status Codes

A route's response includes a status code as well as its body. Successful page requests generally return `200`; an invalid form submission can return `400`; and a missing resource can return `404`.

**How to read this example:** The return tuple contains both a readable explanation and HTTP status 400. Clients and tests can recognize invalid input from the status, while the message explains what needs correction.

```python
if not category:
    return "Category is required.", 400
```

Status codes help browsers, API clients, and automated tests understand the result without interpreting a sentence. The message should still explain what the user can do next.

### JSON Routes Serve Programs, Not HTML Pages

`jsonify()` converts Python dictionaries and lists into a JSON response. This is useful when the caller is another program, a JavaScript frontend, or a data tool:

**How to read this example:** `jsonify()` turns Python records into structured JSON rather than HTML. This is intended for another program or frontend; stable field names let clients consume the result predictably.

```python
@app.route("/api/expenses")
def api_expenses():
    records = load_expenses()
    return jsonify(records)
```

An HTML route is designed for people using a browser; a JSON route is designed for clients that want structured fields. Keep the data shape stable and document fields so API clients can rely on them.

### Development Server and Debug Mode

`app.run(debug=True)` is convenient during local development because Flask can show detailed errors and reload code changes. Debug mode must not be exposed in a public production deployment: detailed tracebacks reveal implementation information, and the development server is not designed to serve production traffic.

For real deployment, use a production WSGI server and configure secrets and environment-specific settings outside the source code.

### Following One Expense Through the App

When someone submits a form, the complete flow is:

1. A `GET` request displays a template containing the form.
2. The browser sends a `POST` with string values.
3. The route validates required fields and converts numeric text.
4. Valid data is saved; invalid data gets a helpful response.
5. A successful request redirects to a page that displays the updated list.

Knowing this sequence helps you debug whether a problem belongs to routing, form parsing, validation, storage, or rendering.
