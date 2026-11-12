# Session XV: Concepts Guide
## Web Development with Flask

This session covers how to build a small web application with Flask: routes that map URLs to Python functions, Jinja templates for HTML, forms that are validated before saving, and JSON endpoints for other programs.

---

## 1. What Is Flask?

Flask is a lightweight Python web framework: it maps URLs to Python functions. The browser requests a URL, Flask finds the matching route, runs its function, and sends back what the function returns.

**How to read this example:** `Flask(__name__)` creates the application and its route registry. `@app.route("/")` registers `home()` for the address `/`, and its returned string becomes the response body. The main guard starts the development server only when the file is run directly.

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def home():
    return "<h1>Hello, World!</h1>"


if __name__ == "__main__":
    app.run(debug=True)
```

### Running the Development Server

```bash
$ uv add flask
$ uv run app.py
 * Running on http://127.0.0.1:5000
```

**How to read this example:** `uv add flask` records Flask in the project; `uv run app.py` runs the file, whose main guard calls `app.run()`. Open the printed address in a browser and stop the server with Ctrl+C.

Flask's own command does the same without the main guard:

```bash
$ uv run flask --app app run --debug
```

> **Warning:** On macOS, port 5000 is taken by the AirPlay Receiver. Choose another port: `app.run(debug=True, port=5001)` or `uv run flask --app app run --debug --port 5001`.

Debug mode shows detailed errors and reloads the code when you save. Use it **only during development**: its tracebacks reveal your code, and the development server is not built for production traffic.

> Go to **Exercise 1 (Hello Flask)**, where you will create an app with a home route and an `/about` route and run it.

---

## 2. Routes and Dynamic URLs

Map URLs to Python functions, including parts of the URL.

### Routes Map Addresses to Behavior

The browser never calls a function directly: it requests a URL and Flask dispatches it to the matching route. A dynamic segment such as `/hello/<name>` passes that part of the URL to the function as an argument.

**How to read this example:** The converter in `<int:expense_id>` makes Flask parse the segment as an integer; `/expenses/abc` does not match and returns 404. `escape()` turns user text into harmless characters, so `/hello/<script>…` cannot inject HTML.

Name URLs after resources: `/expenses` is the list, `/expenses/42` is expense 42, and `/expenses/new` is the form that adds one.

```python
from markupsafe import escape


@app.route("/hello/<name>")
def greet(name):
    # escape() blocks HTML injection
    return f"<h1>Hello, {escape(name)}!</h1>"


@app.route("/expenses/<int:expense_id>")
def show_expense(expense_id):
    return f"Expense #{expense_id}"
```

### Status Codes and 404 for Missing Records

A response has a **status code** as well as a body: `200` OK, `201` created, `400` invalid input, `404` not found. Browsers, API clients, and tests read the code without parsing a sentence.

**How to read this example:** If no expense has the requested id, `abort(404)` stops the function and Flask sends its standard "404 Not Found" page. To choose the message yourself, return a tuple instead: `return "Expense not found", 404`.

```python
from flask import abort

EXPENSES = [
    {"id": 1, "category": "Food", "amount": 25.0},
    {"id": 2, "category": "Transport", "amount": 12.5},
]


@app.route("/expenses/<int:expense_id>")
def show_expense(expense_id):
    for e in EXPENSES:
        if e["id"] == expense_id:
            return f"{e['category']}: ${e['amount']:.2f}"
    abort(404)  # no match: 404 Not Found
```

> Go to **Exercise 2 (Dynamic Routes)**, where you will add a greeting route and an expense detail route that returns 404 for an unknown id.

---

## 3. Templates with Jinja2

Keep HTML in template files and fill it with Python data.

### Templates Separate Presentation from Python Logic

```text
project/
├── app.py
├── templates/
│   ├── base.html
│   ├── expenses.html
│   └── add.html
└── static/
    └── style.css
```

**How to read this example:** `render_template("expenses.html", expenses=EXPENSES)` looks for the file in `templates/` and passes the list to it under the name `expenses`. The route prepares the data; the template decides how it looks.

```python
from flask import render_template


@app.route("/expenses")
def expenses():
    return render_template("expenses.html", expenses=EXPENSES)
```

`EXPENSES` is the list of expense dictionaries from the previous section. Building HTML in templates is clearer than assembling a large string in the route function.

### Template Inheritance and Loops

```html
{# templates/base.html #}
<!doctype html>
<html>
<head><title>Budget App</title></head>
<body>
  <h1>Budget App</h1>
  {% block content %}{% endblock %}
</body>
</html>
```

**How to read this example:** `base.html` holds the layout shared by every page and leaves a named `{% block content %}` empty. A child template `{% extends %}` it and fills only that block.

```html
{# templates/expenses.html #}
{% extends "base.html" %}
{% block content %}
<table>
  {% for e in expenses %}
  <tr>
    <td>{{ e.category }}</td>
    <td>${{ "%.2f"|format(e.amount) }}</td>
  </tr>
  {% endfor %}
</table>
{% endblock %}
```

`{% for %}` repeats a row per expense and `{{ … }}` prints a value; Jinja escapes printed text by default, so user input is not treated as markup.

> Go to **Exercise 3 (Templates)**, where you will render a list of expenses as an HTML table with a Jinja loop.

### Templates Without Files: render_template_string()

```python
from flask import render_template_string

FORM = """
<form method="post">
  <input name="name" placeholder="Name">
  <button type="submit">Send</button>
</form>
"""


@app.route("/contact")
def contact():
    return render_template_string(FORM)
```

**How to read this example:** `render_template_string()` renders a Jinja template stored in a Python string, which is handy for a one-file exercise; real applications keep templates in `templates/`.

> **Warning:** Never build the template string from user input. Pass values as variables instead, so Jinja escapes them: `render_template_string("<p>Hi {{ name }}</p>", name=name)`.

---

## 4. Forms and POST Requests

Receive form data, validate it, and redirect.

### GET Shows the Form, POST Submits It

```python
from flask import redirect, request, url_for


@app.route("/expenses/new", methods=["GET", "POST"])
def add_expense():
    if request.method == "GET":
        return render_template("add.html")
    category = request.form.get("category", "").strip()
    amount_text = request.form.get("amount", "").strip()
    if not category:
        return "Category is required.", 400
    try:
        amount = float(amount_text)
    except ValueError:
        return "Amount must be a number.", 400
    EXPENSES.append({"id": len(EXPENSES) + 1,
                     "category": category, "amount": amount})
    return redirect(url_for("expenses"))
```

**How to read this example:** GET displays the form in `add.html`; POST sends the fields in `request.form`, always as strings. `.get(…, "")` handles a missing field without a `KeyError`, and `try`/`except` turns `"abc"` into a clear 400 answer instead of a 500 crash. Only valid data is saved.

```html
{# templates/add.html #}
{% extends "base.html" %}
{% block content %}
<form method="post">
  <input name="category">
  <input name="amount">
  <button>Add</button>
</form>
{% endblock %}
```

> Go to **Exercise 4 (Forms)**, where you will build a form that adds expenses, validating the amount before saving.

### Redirect After a Successful Form Submission

After a successful POST, a redirect asks the browser to make a new GET request to another page. This **Post/Redirect/Get** pattern avoids submitting the form again when the user refreshes the page.

```python
return redirect(url_for("expenses"))
```

**How to read this example:** `url_for("expenses")` builds the URL of the route whose function is named `expenses`, and `redirect()` answers with status 302 and that address. Endpoint names keep working if the path changes later.

If validation fails, return a clear message with status 400 (or show the form again with the error) so the user can correct the entry.

> Go to **Exercise 6 (Form + Redirect Flow)**, where you will validate a name and email form and redirect to a success page.

---

## 5. JSON API Endpoints

Serve structured data to programs instead of HTML pages.

### JSON Routes Serve Programs, Not HTML Pages

`jsonify()` converts Python dictionaries and lists into a JSON response. An HTML route is designed for people using a browser; a JSON route is for another program, a JavaScript frontend, or a data tool.

**How to read this example:** The first route returns the whole list; the second returns one record, or a JSON error with status 404. Keep field names stable so clients can rely on them.

```python
from flask import jsonify


@app.route("/api/expenses")
def api_expenses():
    return jsonify(EXPENSES)


@app.route("/api/expenses/<int:expense_id>")
def api_expense(expense_id):
    for expense in EXPENSES:
        if expense["id"] == expense_id:
            return jsonify(expense)
    return jsonify({"error": "Expense not found"}), 404
```

### Receiving JSON: request.get_json()

```python
@app.route("/api/expenses", methods=["POST"])
def api_create_expense():
    data = request.get_json(silent=True) or {}
    category = data.get("category")
    amount = data.get("amount")
    if not category or not isinstance(amount, (int, float)):
        error = "category and a numeric amount are required"
        return jsonify({"error": error}), 400
    expense = {"id": len(EXPENSES) + 1,
               "category": category, "amount": amount}
    EXPENSES.append(expense)
    return jsonify(expense), 201
```

**How to read this example:** `request.get_json(silent=True)` parses the JSON body, or returns `None` when the body is not JSON. The route validates the fields, saves the record, and answers `201 Created` with the new expense.

A browser address bar only sends GET requests; try the POST from Python with Flask's test client:

```python
client = app.test_client()
r = client.post("/api/expenses",
                json={"category": "Gym", "amount": 30})
print(r.status_code, r.get_json())
# 201 {'amount': 30, 'category': 'Gym', 'id': 3}
```

> Go to **Exercise 5 (JSON API)**, where you will add endpoints that list, create, and look up expenses as JSON.

---

## 6. Flask Patterns That Scale Better

A small Flask app is easy to write, but a larger app becomes chaotic without structure. Keep your code separated by role:

- routes handle HTTP requests
- templates render the UI
- helper functions manage data logic
- configuration stays in one place

This is the same separation you will see again in larger app projects.

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/")
def home():
    return render_template("index.html", title="Budget App")
```

**How to read this example:** The route only chooses the template and the data it needs; the HTML lives in `templates/index.html`, so the page can be redesigned without touching Python. This keeps the app readable and easier to extend.

---

## 7. Debugging the Request Flow

Follow one request from the browser to the response.

### Following One Expense Through the App

When someone submits a form, the complete flow is:

1. A `GET` request displays a template containing the form.
2. The browser sends a `POST` with string values.
3. The route validates required fields and converts numeric text.
4. Valid data is saved; invalid data gets a helpful response.
5. A successful request redirects to a page that displays the updated list.

Knowing this sequence helps you debug whether a problem belongs to routing, form parsing, validation, storage, or rendering.

---

## Quick Reference

```
INSTALL     uv add flask
RUN         uv run app.py   or   uv run flask --app app run --debug
DEBUG       app.run(debug=True, port=5001)   (development only)
ROUTE       @app.route("/expenses/<int:expense_id>")
TEMPLATE    render_template("file.html", var=value)
FORM        request.form.get("field", "").strip()   (POST)
ERRORS      return "message", 400   /   abort(404)
REDIRECT    redirect(url_for("function_name"))
JSON API    return jsonify(data), 201   /   request.get_json()
```
