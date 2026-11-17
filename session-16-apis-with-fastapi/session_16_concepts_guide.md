# Session XVI: Concepts Guide
## APIs with FastAPI

Create modern APIs with FastAPI, Pydantic models, OpenAPI docs.

---

## 1. Why FastAPI?

FastAPI is a modern, high-performance web framework for building APIs. It uses **type hints** for automatic validation and documentation.

**How to read this example:** `FastAPI()` creates the application to which routes attach. The root handler returns a dictionary that becomes JSON; Uvicorn serves the app, and `/docs` displays the contract generated from routes and types.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello, World!"}
```

**How to read this example:** In your own project, `uv add "fastapi[standard]"` installs FastAPI together with Uvicorn and the `fastapi` command (the course repository already declares it, so `uv sync` is enough there). `fastapi dev app.py` serves the app and reloads it when the file changes; `uv run uvicorn app:app --reload` does the same, where `app:app` means file `app.py`, variable `app`.

```bash
$ uv add "fastapi[standard]"
$ uv run fastapi dev app.py
# or run Uvicorn directly:
$ uv run uvicorn app:app --reload
# Open http://127.0.0.1:8000/docs  (auto-generated API docs!)
```

> Go to **Exercise 1 (Hello FastAPI)**, where you will build an app with a root and a `/health` endpoint and explore its `/docs` page.

---

## 2. Path and Query Parameters

**How to read this example:** Path parameters identify a resource, while ordinary arguments can represent query options. Type hints parse and validate values and become part of the public contract.

```python
@app.get("/expenses/{expense_id}")
def get_expense(expense_id: int):
    return {"id": expense_id}

@app.get("/expenses")
def list_expenses(
    category: str | None = None,
    limit: int = 10,
):
    # category and limit are query params
    return {"category": category, "limit": limit}
```

---

## 3. Pydantic Models — Request Validation

**How to read this example:** The model defines the JSON body shape and validates it before the endpoint runs. Domain rules such as an allowed category may still need explicit business validation.

```python
from pydantic import BaseModel

class ExpenseCreate(BaseModel):
    amount: float
    category: str
    description: str = ""

@app.post("/expenses", status_code=201)
def create_expense(expense: ExpenseCreate):
    # expense is automatically validated!
    return {"created": expense.model_dump()}
```

---

## 4. Response Models

**How to read this example:** A response model documents and validates public output. Separate request and response models clarify server-generated values such as IDs.

```python
class ExpenseResponse(BaseModel):
    id: int
    amount: float
    category: str

@app.get("/expenses/{eid}", response_model=ExpenseResponse)
def get_expense(eid: int):
    return {"id": eid, "amount": 25.50, "category": "Food"}
```

---

## 5. Error Handling

**How to read this example:** `find_expense()` returns the matching dictionary or `None`. The handler checks `expense is None` and raises `HTTPException`, which FastAPI turns into a 404 response with a `detail` message. A missing record should be a 404 rather than an error object with a success status.

```python
from fastapi import HTTPException

EXPENSES = [{"id": 1, "amount": 25.5, "category": "Food"}]

def find_expense(eid: int) -> dict | None:
    for expense in EXPENSES:
        if expense["id"] == eid:
            return expense
    return None

@app.get("/expenses/{eid}")
def get_expense(eid: int):
    expense = find_expense(eid)
    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")
    return expense
```

The update and delete endpoints in section 7 reuse `EXPENSES` and `find_expense()`.

> Go to **Exercise 4 (Error Handling)**, where you will return 404 for unknown IDs and 400 for amounts that are not positive.

---

## 6. FastAPI vs Flask

| Feature | Flask | FastAPI |
|---------|-------|---------|
| Speed | Good; `async` views since Flask 2.0 | Very good; built for `async def` (plain `def` also works, as in this session) |
| Validation | Manual | Automatic (Pydantic) |
| API Docs | Manual | Auto-generated |
| Type Hints | Optional | Central to validation and docs |
| Best For | Web apps + APIs | Pure APIs |

---

## Quick Reference

**How to read this example:** This is an index of FastAPI syntax; the earlier examples explain how methods, parameters, models, errors, and documentation work together.

```
INSTALL     uv add "fastapi[standard]"
RUN         uv run fastapi dev app.py
GET         @app.get("/path")
POST        @app.post("/path", status_code=201)
PUT/DELETE  @app.put("/path/{id}")  @app.delete("/path/{id}")
PATH PARAM  /items/{item_id}
QUERY PARAM def func(limit: int = Query(default=10, le=100))
VALIDATION  Pydantic BaseModel, Field(gt=0)
DOCS        /docs (Swagger UI) or /redoc
ERROR       raise HTTPException(status_code=404)
```

---


---

## 7. FastAPI Design Patterns

FastAPI becomes especially powerful when you design endpoints around clearly defined models and data validation rules.

### Updating and Deleting Resources

A full CRUD API also changes and removes records. Every field in `ExpenseUpdate` is optional, so a client can send only what changes, such as `{"amount": 30}`.

**How to read this example:** `model_dump(exclude_none=True)` keeps only the fields the client sent, and `dict.update()` changes just those keys, so omitted fields are not erased. Both endpoints reuse `find_expense()` from section 5 and return 404 for an unknown ID.

```python
class ExpenseUpdate(BaseModel):
    amount: float | None = None
    category: str | None = None

@app.put("/expenses/{eid}")
def update_expense(eid: int, changes: ExpenseUpdate):
    expense = find_expense(eid)
    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")
    expense.update(changes.model_dump(exclude_none=True))
    return expense

@app.delete("/expenses/{eid}")
def delete_expense(eid: int):
    expense = find_expense(eid)
    if expense is None:
        raise HTTPException(status_code=404, detail="Expense not found")
    EXPENSES.remove(expense)
    return {"deleted": eid}
```

HTTP reserves `PATCH` for partial updates; `@app.patch` works the same way.

> Go to **Exercise 5 (Full CRUD API (Capstone))**, where you will combine GET, POST, PUT and DELETE in one expense API.

---

## 8. Good API Hygiene

A professional API should:
- validate request bodies early
- return clear status codes
- keep endpoint logic simple
- document behavior with descriptive routes and models
- avoid mixing business rules with transport code

This is why FastAPI is so popular: it makes many of these habits easy to follow.


---

## Deep Dive: FastAPI as a Contract

FastAPI is not just “Flask but with type hints.” It is a framework that turns code contracts into actual validation and documentation.

The request model from section 3 is checked before the function body runs. Pydantic v2 validates in *lax* mode by default: a JSON string such as `"12.5"` is converted to the float 12.5, and only text that is not a number, such as `"twelve"`, is rejected with a 422 response.

**How to read this example:** `check()` builds each model and reports whether validation passed. `Field(strict=True)` (or the `StrictFloat` type) accepts only real numbers, so the string `"12.5"` is rejected too.

```python
from pydantic import BaseModel, Field, ValidationError

class Expense(BaseModel):
    amount: float                       # lax (default)

class StrictExpense(BaseModel):
    amount: float = Field(strict=True)  # numbers only

def check(model, value):
    try:
        return model(amount=value).amount
    except ValidationError:
        return "rejected (422 in FastAPI)"

print(check(Expense, "12.5"))        # 12.5
print(check(Expense, "twelve"))      # rejected (422 in FastAPI)
print(check(StrictExpense, "12.5"))  # rejected (422 in FastAPI)
print(check(StrictExpense, 12.5))    # 12.5
```

This is a powerful idea because it moves validation to the boundary of the application. That means your internal code can be simpler and more trustworthy. The API becomes a clear contract between client and server:

- one side sends exactly the expected shape
- the other side returns a consistent response
- invalid data is rejected early and clearly

This is one reason FastAPI is so popular in modern Python backends.


---

## Understanding FastAPI's API Contract

An API contract describes what a client may send and what the server promises to return. FastAPI uses route declarations, type hints, and Pydantic models to make much of that contract explicit.

### Routes and HTTP Methods Define the Operation

`@app.get("/expenses")` registers a handler for reading expense information. `@app.post("/expenses")` registers a handler for creating a new record. Choosing the correct method helps clients understand what an endpoint does and makes behavior more predictable.

**How to read this example:** The GET decorator registers a collection endpoint. Returning an `items` field gives clients a predictable response shape, including when the collection is empty; storage can later populate the list without changing the route contract.

```python
@app.get("/expenses")
def list_expenses():
    return {"items": []}
```

The function's return value is converted into a JSON response. Keep route functions focused on handling the HTTP boundary—reading validated input, calling application logic, and selecting an appropriate response.

### Path Parameters Identify a Specific Resource

A path parameter is part of the URL itself. In `/expenses/{expense_id}` (the section 2 example), the value identifies which expense the client wants.

The annotation `int` tells FastAPI to parse and validate the path value: `/expenses/12` passes the integer 12 to the handler. If a client requests `/expenses/not-a-number`, FastAPI returns a 422 validation error rather than passing arbitrary text to code that expects an integer.

### Query Parameters Filter or Shape a Collection

Query parameters follow the `?` in a URL. They are useful for optional filters, pagination, and sorting. A request such as `/expenses?category=Food&limit=5` asks for a filtered, limited collection; defaults say what happens when the client omits an option.

**How to read this example:** `category` and `limit` are query parameters because they are function arguments that are not in the path. `Query(default=10, ge=1, le=100)` keeps the default and adds bounds, so `/expenses?limit=500` gets a 422 response instead of an unreasonably large answer.

```python
from fastapi import Query

@app.get("/expenses")
def list_expenses(
    category: str | None = None,
    limit: int = Query(default=10, ge=1, le=100),
):
    result = EXPENSES
    if category is not None:
        result = [e for e in result if e["category"] == category]
    return result[:limit]
```

> Go to **Exercise 3 (Path & Query Parameters)**, where you will fetch one expense by ID and filter the list with `?category=Food&min_amount=10`.

### Pydantic Models Define and Validate Request Bodies

An HTTP request body can contain structured JSON. A Pydantic model describes the required fields and their types:

**How to read this example:** The model declares required fields and a default for the optional description. FastAPI parses JSON into this model and rejects missing or incompatible values before the endpoint runs. Field constraints can express rules such as a positive amount.

```python
from pydantic import BaseModel, Field

class ExpenseCreate(BaseModel):
    category: str
    amount: float = Field(gt=0)
    description: str = ""
```

The annotation `Field(gt=0)` adds a constraint: an amount must be greater than zero. If the incoming body is missing a required field or violates a constraint, FastAPI returns a validation response before the endpoint's business logic executes.

> Go to **Exercise 6 (Expense API with Validation)**, where you will combine `Field(gt=0)` with a business rule in one endpoint.

This is useful because it puts validation at the application's boundary. Internal code can work with an `ExpenseCreate` object rather than repeatedly checking whether arbitrary dictionary keys exist. Validation is not a replacement for business rules, though: the application may still need to check whether a category is allowed or whether the user has permission to create the record.

### Response Models Document and Filter Output

A response model describes the fields the endpoint returns. For a collection, wrap the model in `list[...]`:

**How to read this example:** `response_model=list[ExpenseResponse]` promises a list whose items all have the `ExpenseResponse` shape from section 4. FastAPI validates each item, documents the shape in `/docs` and drops fields the model does not declare, so `internal_note` never reaches the client.

```python
STORED = [
    {"id": 1, "amount": 25.5, "category": "Food",
     "internal_note": "bank import"},
]

@app.get("/expenses", response_model=list[ExpenseResponse])
def list_expenses():
    return STORED

# GET /expenses returns:
# [{"id": 1, "amount": 25.5, "category": "Food"}]
```

A response model is a safety net for internal fields only if it lists just the public ones.

> Go to **Exercise 2 (Pydantic Models)**, where you will create expenses with POST and list them with GET.

Request and response models often differ. A client may submit a category and amount, while the server assigns an ID and timestamp. Separate models communicate that difference clearly.

### Status Codes and HTTP Exceptions Communicate Outcomes

Use status codes to distinguish outcomes:

- `200 OK` commonly indicates a successful read or update.
- `201 Created` is a clear choice when a new resource has been created: `@app.post("/expenses", status_code=201)`.
- `404 Not Found` tells the client that the requested identifier has no matching resource.
- `422 Unprocessable Entity` is commonly returned automatically when request validation fails.

If an item is absent, raise an `HTTPException` (section 5) rather than returning a success-shaped object; its `detail` field gives the client a useful explanation. Do not return `200` with `{"error": ...}` for a failed operation unless an API contract explicitly requires that pattern; a correct status makes client behavior and monitoring clearer.

### Automatic Documentation Comes from the Contract

FastAPI reads route declarations and models to generate OpenAPI documentation. The interactive `/docs` page lets a developer inspect endpoints and submit sample requests; `/redoc` presents another view of the same schema.

This documentation is only as clear as the names, types, descriptions, and response models in the code. Descriptive field names and explicit status codes make the API easier to use without a separate hand-maintained list of endpoint details.

### A Request's Journey

For a `POST /expenses` call, the process is:

1. FastAPI matches the method and route.
2. The JSON body is parsed into the request model.
3. Type and field constraints are validated.
4. The endpoint applies remaining business rules.
5. The application saves the resource.
6. A response model and status code communicate the result.

If a request fails, identify which step failed before changing code. A malformed amount is a request-validation issue; a valid amount in a disallowed category is a business-rule issue; an unknown ID during a read is a resource-lookup issue.

### Type Hints Are Part of the Interface

In ordinary Python, type hints mostly help readers and tools. In FastAPI, annotations also influence how parameters are interpreted and validated. That makes hints part of the public API behavior: changing a parameter from a path value to a query value, or changing its type, can affect clients.

Use accurate types and defaults. Avoid annotating a value as `int` if the endpoint will actually return text, and make optionality explicit. Accurate annotations improve editor support, generated documentation, and client expectations together.
