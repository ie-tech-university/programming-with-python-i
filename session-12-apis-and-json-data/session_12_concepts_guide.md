# Session XII: Concepts Guide
## Working with APIs and JSON Data

Connect to REST APIs, process JSON responses, handle errors and authentication.

---

## 1. What Is an API?

An **API** (Application Programming Interface) is a contract between two programs. A **REST API** uses HTTP to let you request and send data over the web.

### HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Retrieve data | Get currency rates |
| `POST` | Send new data | Create a transaction |
| `PUT` | Update data | Edit a profile |
| `DELETE` | Remove data | Delete a record |

---

## 2. The `requests` Library

**How to read this example:** The request returns a response object; its status and body are separate. Check the status and set a timeout before parsing JSON so network and HTTP failures are explicit.

```python
import requests

response = requests.get("https://api.exchangerate-api.com/v4/latest/USD")
print(response.status_code)  # 200
print(response.json())       # dict
```

### Response Object

**How to read this example:** This example applies the concept of Response Object. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
response.status_code   # int: 200, 404, 500
response.ok            # bool: True if < 400
response.text          # str: raw body
response.json()        # dict: parsed JSON
response.headers       # dict
```

---

## 3. HTTP Status Codes

| Code | Meaning |
|------|---------|
| `200` | OK |
| `201` | Created |
| `400` | Bad Request |
| `401` | Unauthorized |
| `404` | Not Found |
| `429` | Rate Limited |
| `500` | Server Error |

---

## 4. Query Parameters

**How to read this example:** The dictionary passed as `params` becomes encoded URL filters. This asks the server for a narrower result instead of fetching everything and discarding most of it locally.

```python
params = {"country": "ES", "limit": 10}
response = requests.get("https://api.example.com/data", params=params)
```

---

## 5. Headers & Authentication

**How to read this example:** Headers carry metadata and sometimes credentials. API keys belong in protected environment configuration, and the service docs specify the required authorization format.

```python
import os
API_KEY = os.environ.get("MY_API_KEY", "your-key")
headers = {"Authorization": f"Bearer {API_KEY}"}
response = requests.get(url, headers=headers)
```

---

## 6. Error Handling

**How to read this example:** A timeout bounds waiting; `raise_for_status()` converts HTTP error responses to exceptions. Separate handlers let the program explain different network failure types.

```python
def fetch_data(url, params=None):
    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.Timeout:
        print("Timed out")
    except requests.exceptions.HTTPError as e:
        print(f"HTTP Error: {e.response.status_code}")
    except requests.exceptions.ConnectionError:
        print("No internet")
    except requests.exceptions.RequestException as e:
        print(f"Error: {e}")
    return None
```

---

## 7. Navigating JSON Responses

**How to read this example:** JSON objects become dictionaries and arrays become lists. Direct indexing expects a required key; `.get()` is appropriate for optional keys, but avoid defaults that could be mistaken for valid essential data.

```python
data = response.json()
rate = data["rates"]["EUR"]
safe = data.get("rates", {}).get("CAD", "N/A")
for currency, rate in data["rates"].items():
    print(f"1 USD = {rate} {currency}")
```

---

## 8. Caching API Responses

**How to read this example:** This example applies the concept of 8. Caching API Responses. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
import json, os, time

def cached_fetch(url, cache_file, max_age=3600):
    if os.path.exists(cache_file):
        if time.time() - os.path.getmtime(cache_file) < max_age:
            with open(cache_file) as f:
                return json.load(f)
    data = fetch_data(url)
    if data:
        with open(cache_file, "w") as f:
            json.dump(data, f, indent=2)
    return data
```

---

## 9. POST Requests

**How to read this example:** The request returns a response object; its status and body are separate. Check the status and set a timeout before parsing JSON so network and HTTP failures are explicit.

```python
new_item = {"name": "Widget", "price": 9.99}
response = requests.post(url, json=new_item, headers=headers)
if response.status_code == 201:
    print(f"Created: {response.json()}")
```

---

## Quick Reference

**How to read this example:** This block collects common request options and response properties as an index; use the surrounding examples to choose the correct options for a particular API.

```
INSTALL     uv add requests
GET         requests.get(url, params=, headers=, timeout=)
POST        requests.post(url, json=, headers=)
RESPONSE    .status_code  .ok  .json()  .text
ERRORS      try/except RequestException
AUTH        headers={"Authorization": "Bearer TOKEN"}
CACHE       Save .json() to file, check mtime
```

---


---

## 10. Working with Real APIs Like a Professional

APIs are useful only when you treat them as unreliable systems. Requests can fail, responses can be slow, and data can change shape without warning.

### Better Request Pattern

**How to read this example:** This example applies the concept of Better Request Pattern. Follow each operation from its input to its result, and connect the syntax to the behavior the section describes; then predict how changing one input would alter the output.

```python
import requests

try:
    response = requests.get("https://api.example.com/data", timeout=10)
    response.raise_for_status()
    payload = response.json()
except requests.exceptions.Timeout:
    print("The server did not respond in time.")
except requests.exceptions.HTTPError as exc:
    print(f"Request failed with status {exc.response.status_code}")
except requests.exceptions.RequestException as exc:
    print(f"Unexpected request error: {exc}")
```

### Why This Matters

A production-quality script should:
- fail gracefully when the API is down
- handle missing or empty JSON calmly
- avoid re-fetching API data too often
- keep secrets out of source code

> Go to **Exercise 6 (Cached Currency Lookup)** to practice a stronger, more realistic API workflow.

---

## 11. Caching and Rate Limits

When you repeatedly call the same endpoint, you can waste time and get blocked. A simple cache can help:

**How to read this example:** The age check avoids another network call while the saved response remains fresh. After expiry, fetch an updated response; choose the lifetime based on how quickly the source data changes, and do not replace good cached data with a failed response.

```python
import json, os, time

CACHE_FILE = "cache.json"

if os.path.exists(CACHE_FILE):
    age = time.time() - os.path.getmtime(CACHE_FILE)
    if age < 3600:
        with open(CACHE_FILE) as f:
            data = json.load(f)
```

This is not a trick — it is a real-world engineering pattern.

---

## 12. Data Safety and API Design

When working with JSON, always confirm the structure before accessing nested values:

**How to read this example:** The chained dictionary lookups navigate from the rates object to one currency value. `.get()` handles optional keys, while required values should fail clearly rather than silently becoming plausible but incorrect numbers.

```python
rates = data.get("rates", {})
currency = rates.get("EUR", "N/A")
```

Use `.get()` when a field may be missing. This avoids a common `KeyError` and makes the code more resilient.


---

## Deep Dive: A Real API Workflow

The most important idea in this session is that APIs are not magical. They are just networked interfaces that return data in a predictable format. The main challenge is handling uncertainty.

A common real-world workflow looks like this:

**How to read this example:** The first request demonstrates the happy path: send GET, check status, parse JSON, and select the EUR rate. The second adds a timeout and exception handling because network availability and response status cannot be assumed.

```python
import requests

response = requests.get("https://api.exchangerate-api.com/v4/latest/USD", timeout=10)
response.raise_for_status()

payload = response.json()
print(payload["rates"]["EUR"])
```

This works when the network is healthy and the response is exactly what we expect. But real code must also handle:

- slow servers
- missing keys in JSON
- rate limits
- bad connections
- server-side errors

A more robust version is:

**How to read this example:** The first request demonstrates the happy path: send GET, check status, parse JSON, and select the EUR rate. The second adds a timeout and exception handling because network availability and response status cannot be assumed.

```python
import requests

try:
    response = requests.get("https://api.exchangerate-api.com/v4/latest/USD", timeout=10)
    response.raise_for_status()
    data = response.json()
    rate = data.get("rates", {}).get("EUR", "N/A")
    print(f"1 USD = {rate} EUR")
except requests.exceptions.Timeout:
    print("The API did not respond in time.")
except requests.exceptions.HTTPError as exc:
    print(f"API error: {exc.response.status_code}")
except requests.exceptions.RequestException as exc:
    print(f"Something went wrong: {exc}")
```

This kind of defensive programming is essential. The user experience is much better when your app fails politely instead of crashing in the middle of a report.


---

## Understanding Each Part of an API Request

An API request is a message sent from a client program to a server. The server interprets the message, performs an operation, and sends a response. Learning the parts of that exchange makes the `requests` examples much easier to adapt.

### HTTP Methods Describe the Intended Operation

The method tells the server what kind of action the client wants:

- `GET` asks to read information. For example, a dashboard can request the current exchange rate.
- `POST` submits new information. For example, a form can send a new expense record.
- `PUT` commonly replaces or updates a resource at a known address.
- `DELETE` asks to remove a resource.

These method names are a convention, not Python keywords. The API documentation defines which methods a particular endpoint accepts. A `GET` should generally not be used to create or delete data because tools and users expect it to be safe to repeat.

### A Response Has More Than a Status Code

`requests.get()` returns a `Response` object. Its fields answer different questions:

- `status_code` tells you the HTTP result, such as `200` or `404`.
- `headers` contains metadata, such as the content type or rate-limit information.
- `text` provides the response body as text.
- `json()` parses a JSON body into Python dictionaries, lists, strings, numbers, booleans, or `None`.

Calling `.json()` does not guarantee the response is valid JSON. A server might return an HTML error page, so JSON decoding can fail even if an HTTP request reached the server. Check the status and know the response format before treating it as the expected data structure.

**How to read this example:** The response exposes different information: status and `ok` describe HTTP success, `text` is the raw body, `json()` parses structured data, and headers contain metadata. These properties help diagnose whether the problem is the HTTP result or body format.

```python
response = requests.get(url, timeout=10)
response.raise_for_status()  # Raises HTTPError for unsuccessful status codes
data = response.json()       # Parse the body after checking the HTTP result
```

### Status Codes Tell You What Happened

Status codes are grouped by their first digit. For this course, the most useful distinction is between successful responses (`2xx`), client-side problems (`4xx`), and server-side problems (`5xx`).

- `200` means the request succeeded and often includes the requested data.
- `201` means a resource was created, which is a common response to a successful `POST`.
- `400` means the request was not acceptable as sent; the client may need to correct its input.
- `401` means authentication is missing or invalid.
- `404` means the requested resource or route was not found.
- `429` means the client is making requests too quickly and should respect the API's rate limits.
- `500` indicates a server-side failure; retrying immediately may not help.

The exact meaning can vary slightly by API, so use its documentation. `raise_for_status()` converts unsuccessful HTTP results into a `requests.exceptions.HTTPError`, making them straightforward to handle.

### Query Parameters Narrow a Request

Query parameters are small options attached to a URL. `requests` builds and encodes them safely when you pass a dictionary with `params=`:

**How to read this example:** The parameter dictionary becomes URL query values, asking the server for a country and limited results. Server-side filtering reduces downloaded data, and `params` safely encodes the query instead of manually constructing a URL.

```python
params = {"country": "ES", "limit": 10}
response = requests.get("https://api.example.com/events", params=params)
```

This asks for a subset of events rather than downloading everything and filtering it locally. Use query parameters for filters, search terms, pagination, and other options documented by the API. Passing a dictionary also handles characters such as spaces more reliably than manually concatenating a URL.

### Headers Carry Metadata and Credentials

Headers describe the request. They can tell a server which format the client accepts, identify the content type, or provide a token proving the client's identity.

An API key should not be committed into a script or notebook. Read it from an environment variable and fail visibly when the required setting is missing:

**How to read this example:** The credential comes from environment configuration rather than source code, reducing the risk of committing a secret. The missing-key check fails visibly; the API documentation determines the exact authorization header format.

```python
import os

api_key = os.environ.get("MY_API_KEY")
if not api_key:
    raise RuntimeError("Set the MY_API_KEY environment variable first.")

headers = {"Authorization": f"Bearer {api_key}"}
```

The exact header format is defined by the service. Environment variables keep credentials out of source control, but they still need to be protected in the machine or deployment environment where the application runs.

### JSON Is a Nested Data Structure

JSON objects become Python dictionaries; JSON arrays become Python lists. That means you can use dictionary keys and list indexes to navigate the response:

**How to read this example:** The sample dictionary mirrors nested JSON: the outer key selects rates, and the inner key selects a currency. Direct indexing is concise for required fields; `.get()` can handle optional fields, but missing values still need a deliberate policy.

```python
data = {
    "base": "USD",
    "rates": {"EUR": 0.92, "CAD": 1.36},
}

print(data["rates"]["EUR"])  # 0.92
```

Direct indexing is appropriate when the key is required and a missing value should be treated as an error. `.get()` is useful for optional fields:

**How to read this example:** The sample dictionary mirrors nested JSON: the outer key selects rates, and the inner key selects a currency. Direct indexing is concise for required fields; `.get()` can handle optional fields, but missing values still need a deliberate policy.

```python
rates = data.get("rates", {})
cad_rate = rates.get("CAD")
```

The default `{}` ensures the second lookup still operates on a dictionary when `"rates"` is absent. Avoid using a plausible-looking fallback for essential data: silently returning `0` for a missing exchange rate could produce an incorrect financial calculation.

### Timeouts and Exceptions Make Network Code Predictable

Network operations can wait indefinitely unless a timeout is specified. A timeout places a bound on how long the program waits and lets it recover or report the problem.

- A `Timeout` indicates the server did not respond within the allowed time.
- A `ConnectionError` indicates the client could not establish or maintain a connection.
- An `HTTPError` indicates an HTTP error status after `raise_for_status()`.
- The parent `RequestException` covers other request-related failures.

Catch exceptions only where the application can provide a useful response. If a function returns `None` after a failure, callers must check for it before using the result; otherwise the failure may simply become a less clear error later.

### Caching: Save Work, but Define Freshness

A cache stores a previous response so the application can reuse it instead of making a new network request. This can make a dashboard faster and reduce the number of requests sent to an API.

The file modification time is a simple freshness rule: calculate how many seconds have passed since the cache was written, then reuse it only if that age is below `max_age`. The right duration depends on the data. A currency rate that changes during the day should not be cached indefinitely; a list of country names may be safe to cache for much longer.

For a reliable cache, also consider malformed cache files, whether the cache belongs to the requested URL and parameters, and whether stale data should be used when the network is unavailable. These are design decisions: stale results can be better than no result for some applications, but financial or safety-sensitive decisions may require fresh data.

### POST: Submit Structured Data

With `json=`, `requests` serializes a Python dictionary as JSON and sets an appropriate content type:

**How to read this example:** The dictionary is encoded as a JSON request body with `json=`, and the status indicates whether the server created the resource. Parse the returned body for server-generated information such as an identifier instead of assuming it equals the submission.

```python
new_item = {"name": "Widget", "price": 9.99}
response = requests.post(url, json=new_item, timeout=10)
response.raise_for_status()
created_item = response.json()
```

The server decides which fields are required and whether the submitted values are valid. A successful status code does not remove the need to inspect the response: the server may return an identifier, a normalized version of the record, or additional validation details. Do not send private information unless the service is trusted and the connection is appropriately protected.
