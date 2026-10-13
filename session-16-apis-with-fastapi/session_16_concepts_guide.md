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

**How to read this example:** Route decorators register handlers, returned dictionaries become JSON, and Uvicorn serves the app. FastAPI uses the same declarations to generate interactive documentation.

```bash
$ uv add fastapi uvicorn
$ uvicorn app:app --reload
# Open http://127.0.0.1:8000/docs  (auto-generated API docs!)
```

---

## 2. Path and Query Parameters

**How to read this example:** Path parameters identify a resource, while ordinary arguments can represent query options. Type hints parse and validate values and become part of the public contract.

```python
@app.get("/expenses/{expense_id}")
def get_expense(expense_id: int):
    return {"id": expense_id}

@app.get("/expenses")
def list_expenses(category: str = None, limit: int = 10):
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

**How to read this example:** `HTTPException` returns a deliberate status and detail. A missing record should be a 404 rather than an error object with a success status.

```python
from fastapi import HTTPException

@app.get("/expenses/{eid}")
def get_expense(eid: int):
    expense = find_expense(eid)
    if not expense:
        raise HTTPException(status_code=404, detail="Expense not found")
    return expense
```

---

## 6. FastAPI vs Flask

| Feature | Flask | FastAPI |
|---------|-------|---------|
| Speed | Good | Excellent (async) |
| Validation | Manual | Automatic (Pydantic) |
| API Docs | Manual | Auto-generated |
| Type Hints | Optional | Required |
| Best For | Web apps + APIs | Pure APIs |

---

## Quick Reference

**How to read this example:** This is an index of FastAPI syntax; the earlier examples explain how methods, parameters, models, errors, and documentation work together.

```
INSTALL     uv add fastapi uvicorn
RUN         uvicorn app:app --reload
GET         @app.get("/path")
POST        @app.post("/path")
PATH PARAM  /items/{item_id}
QUERY PARAM def func(limit: int = 10)
VALIDATION  Pydantic BaseModel
DOCS        /docs (Swagger UI) or /redoc
ERROR       raise HTTPException(status_code=404)
```

---


---

## 7. FastAPI Design Patterns

FastAPI becomes especially powerful when you design endpoints around clearly defined models and data validation rules.

### Example

**How to read this example:** This example applies the concept of Example. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Expense(BaseModel):
    category: str
    amount: float

@app.post("/expenses")
def add_expense(expense: Expense):
    return {"status": "created", "expense": expense.model_dump()}
```

This pattern prevents invalid payloads before your business logic runs.

> Go to **Exercise 6 (Expense API with Validation)** to build a stronger API design.

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

For example, a request body like this:

**How to read this example:** The Pydantic model defines the request body shape, and the endpoint receives a validated object. Its return value forms a JSON response. These declarations support both runtime validation and generated documentation.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Expense(BaseModel):
    category: str
    amount: float

@app.post("/expenses")
def add_expense(expense: Expense):
    return {"message": "saved", "total": expense.amount}
```

Now the server validates the payload before the function body runs. If a client sends a string where a float is expected, FastAPI can reject it automatically.

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

A path parameter is part of the URL itself. In `/expenses/{expense_id}`, the value identifies which expense the client wants:

**How to read this example:** The path placeholder is converted to an integer by its annotation. A URL ending in `/12` passes integer 12; nonnumeric input fails validation before the handler uses it as an identifier.

```python
@app.get("/expenses/{expense_id}")
def get_expense(expense_id: int):
    return {"id": expense_id}
```

The annotation `int` tells FastAPI to parse and validate the path value. If a client requests `/expenses/not-a-number`, FastAPI returns a validation error rather than passing arbitrary text to code that expects an integer.

### Query Parameters Filter or Shape a Collection

Query parameters follow the `?` in a URL. They are useful for optional filters, pagination, and sorting:

**How to read this example:** Category and limit are query parameters because they are function arguments outside the path. A URL such as `/expenses?category=Food&limit=5` requests a filtered subset; defaults make these options optional.

```python
@app.get("/expenses")
def list_expenses(category: str | None = None, limit: int = 10):
    return {"category": category, "limit": limit}
```

A request such as `/expenses?category=Food&limit=5` asks for a filtered, limited collection. Defaults communicate what should happen when the client omits an option. Production APIs should also place sensible bounds on values such as `limit` so one request cannot ask for an unreasonable amount of data.

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

This is useful because it puts validation at the application's boundary. Internal code can work with an `ExpenseCreate` object rather than repeatedly checking whether arbitrary dictionary keys exist. Validation is not a replacement for business rules, though: the application may still need to check whether a category is allowed or whether the user has permission to create the record.

### Response Models Document and Filter Output

A response model describes the fields the endpoint returns:

**How to read this example:** The response model declares the public fields the endpoint promises. FastAPI validates and documents the result; missing required fields reveal a mismatch between the implementation and its contract.

```python
from pydantic import BaseModel

class ExpenseResponse(BaseModel):
    id: int
    amount: float
    category: str
```

Using `response_model=ExpenseResponse` documents the shape and helps validate the returned data. It can also prevent internal fields from unintentionally being exposed, provided the response model includes only public fields.

Request and response models often differ. A client may submit a category and amount, while the server assigns an ID and timestamp. Separate models communicate that difference clearly.

### Status Codes and HTTP Exceptions Communicate Outcomes

Use status codes to distinguish outcomes:

- `200 OK` commonly indicates a successful read or update.
- `201 Created` is a clear choice when a new resource has been created.
- `404 Not Found` tells the client that the requested identifier has no matching resource.
- `422 Unprocessable Entity` is commonly returned automatically when request validation fails.

If an item is absent, raise an `HTTPException` rather than returning a success-shaped object:

**How to read this example:** When lookup finds no expense, `HTTPException` returns status 404 and a detail message. Clients can distinguish a missing resource from a successful result without interpreting message text.

```python
from fastapi import HTTPException

if expense is None:
    raise HTTPException(status_code=404, detail="Expense not found")
```

The `detail` field gives the client a useful explanation. Do not return `200` with `{"error": ...}` for a failed operation unless an API contract explicitly requires that pattern; a correct status makes client behavior and monitoring clearer.

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
