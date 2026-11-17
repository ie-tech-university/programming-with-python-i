# Session XII: Concepts Guide
## Working with APIs and JSON Data

Connect to REST APIs, process JSON responses, handle errors and authentication.

> Runnable examples use two free services that need no key: the currency endpoint `https://open.er-api.com/v6/latest/USD` and the course dataset `data/countries.json`. Addresses under `api.example.com` and variables such as `url` are **placeholders**: replace them with an endpoint from your API's documentation.

---

## 1. What Is an API?

An **API** (Application Programming Interface) is a contract between two programs. A **REST API** uses HTTP to let you request and send data over the web. The **method** tells the server what kind of action the client wants:

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Read data | A dashboard requests the current exchange rate |
| `POST` | Send new data | A form sends a new expense record |
| `PUT` | Replace or update data at a known address | Edit a profile |
| `DELETE` | Remove data | Delete a record |

These method names are an HTTP convention, not Python keywords; the API documentation says which methods each endpoint accepts. A `GET` should not create or delete data, because tools and users expect it to be safe to repeat.

An API call has two halves. The **request** carries a method, a URL with optional query parameters, headers (for example `Authorization`), an optional JSON body and, on the Python side, a `timeout`. The **response** carries a status code, headers and a body that `.json()` parses. Status codes come in families: `2xx` success, `4xx` a problem with the request (400 bad request, 401, 404, 429 too many requests) and `5xx` a problem on the server. The exceptions to catch where you can respond usefully are `Timeout`, `ConnectionError`, `HTTPError` and their parent `RequestException`.

---

## 2. The `requests` Library

One function call sends a request; the `Response` object it returns holds the answer.

### Your First `GET` Request

Install the library in your own project with `uv add requests` (the course project already includes it).

**How to read this example:** `timeout=10` stops waiting after 10 seconds, and `raise_for_status()` turns an HTTP error status into an exception before `.json()` parses the body into a dictionary.

```python
import requests

url = "https://open.er-api.com/v6/latest/USD"
response = requests.get(url, timeout=10)
response.raise_for_status()
data = response.json()

print(response.status_code)  # 200
print(data["base_code"])     # USD
print(data["rates"]["EUR"])  # e.g. 0.895793 (changes daily)
```

> Go to **Exercise 1 (First API Call)**, where you will print the status, the base currency and five rates, and handle request errors.

### The `Response` Object

`requests.get()` returns a `Response`; each attribute answers a different question:

- `status_code` / `ok`: the HTTP result (`ok` is `True` below 400).
- `headers`: metadata such as the content type. It is a `CaseInsensitiveDict`, so `"Content-Type"` and `"content-type"` find the same header.
- `text`: the raw body as a string.
- `json()`: the parsed body, a `dict` or a `list` depending on whether the JSON starts with `{` or `[`.

`.json()` raises an error if the body is not JSON (a server may answer with an HTML error page), so check the status first.

```python
response = requests.get(url, timeout=10)
print(response.status_code)              # 200
print(response.ok)                       # True
print(response.headers["content-type"])  # application/json
print(response.text[:20])                # {"result":"success",
data = response.json()
print(type(data).__name__)               # dict
```

---

## 3. HTTP Status Codes

The first digit gives the family: `2xx` success, `4xx` a problem with the request, `5xx` a problem on the server. `raise_for_status()` turns every `4xx`/`5xx` into a `requests.exceptions.HTTPError`.

| Code | Meaning | What your code should do |
|------|---------|--------------------------|
| `200` | OK | Read the body |
| `201` | Created | A `POST` succeeded; read the new record |
| `400` | Bad Request | Fix the data or parameters you sent |
| `401` | Unauthorized | Credentials are missing or invalid |
| `404` | Not Found | Check the URL or the resource name |
| `429` | Too Many Requests | Slow down and retry later (section 9) |
| `500` | Internal Server Error | The server failed; retrying at once may not help |

### Errors Inside a `200` Response

The exact meaning of a code can vary by API, so read its documentation. Some APIs even report errors inside a successful response. The currency API answers an unknown currency with status `200` and a body whose `"result"` is `"error"`:

**How to read this example:** `raise_for_status()` would not complain here, because the HTTP request itself succeeded. Check `data.get("result") == "success"` (or the field your API documents) before using the rates.

```python
data = requests.get("https://open.er-api.com/v6/latest/XYZ",
                    timeout=10).json()
print(data)  # {'result': 'error', 'error-type': 'unsupported-code'}
```

---

## 4. Query Parameters

Query parameters are small options attached to a URL after `?`. Pass them as a dictionary with `params=` and `requests` builds and encodes the URL for you, including spaces and accented letters.

**How to read this example:** The dictionary becomes `?q=...&limit=10`; `response.url` shows the final address. Use parameters for filters, search terms and pagination documented by the API, so the server sends a subset instead of everything.

```python
params = {"q": "San José", "limit": 10}
response = requests.get("https://api.example.com/events",
                        params=params, timeout=10)
print(response.url)
# https://api.example.com/events?q=San+Jos%C3%A9&limit=10
```

`api.example.com` is a placeholder: use your API's real endpoint.

---

## 5. Headers & Authentication

Headers describe the request: the format the client accepts, the content type, or a token proving the client's identity.

Never commit an API key into a script or notebook. Read it from an environment variable and fail visibly when it is missing, instead of silently sending a fake key.

**How to read this example:** The key comes from the environment, not the source code, and the program stops with a clear message if it is absent. The service's documentation defines the exact header format; environment variables still need protecting on the machine that runs the app.

```python
import os
import requests

api_key = os.environ.get("MY_API_KEY")
if not api_key:
    raise RuntimeError("Set the MY_API_KEY environment variable first.")

headers = {"Authorization": f"Bearer {api_key}"}
response = requests.get(url, headers=headers, timeout=10)
```

---

## 6. Error Handling

Network code can wait forever unless you pass a **timeout**, which bounds the wait and lets the program report the problem. The exceptions you will meet, all in `requests.exceptions`:

- `Timeout`: the server did not answer within the allowed time.
- `ConnectionError`: no connection could be made or kept, for example no network, an unknown host name, or a refused connection.
- `HTTPError`: an error status, raised by `raise_for_status()`.
- `RequestException`: the parent of all of them, for any other request failure.

Catch exceptions only where you can respond usefully. If a function returns `None` after a failure, callers must check for `None` before using the result; otherwise the failure resurfaces later as a less clear error.

### A Reusable `fetch_data()` Function

**How to read this example:** The timeout bounds waiting, `raise_for_status()` converts error statuses into `HTTPError`, and each handler prints a message that tells the user what actually went wrong. Every failure path returns `None`.

```python
def fetch_data(url, params=None):
    try:
        response = requests.get(url, params=params, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.Timeout:
        print("The server did not respond in time.")
    except requests.exceptions.HTTPError as exc:
        print(f"HTTP error: {exc.response.status_code}")
    except requests.exceptions.ConnectionError:
        print("Could not connect: check the network and the host name.")
    except requests.exceptions.RequestException as exc:
        print(f"Request failed: {exc}")
    return None

rates = fetch_data("https://open.er-api.com/v6/latest/USD")
print(rates["base_code"])  # USD
fetch_data("https://no-such-host.invalid/data")
# Could not connect: check the network and the host name.
```

> Go to **Exercise 2 (Currency Converter)**, where you will wrap a rate request in this kind of error handling and return `None` when the conversion fails.

---

## 7. Navigating JSON Responses

JSON objects become dictionaries and JSON arrays become lists, so you navigate a response with keys and indexes.

**How to read this example:** Direct indexing suits **required** keys: a missing key raises `KeyError`. `.get()` suits **optional** keys; the default `{}` keeps the second lookup working when `"rates"` is absent. Check for `None` instead of inventing a value.

```python
data = {
    "base_code": "USD",
    "rates": {"EUR": 0.9, "GBP": 0.76, "JPY": 153.2},
}

eur_rate = data["rates"]["EUR"]    # required: KeyError if missing
print(eur_rate)                    # 0.9

rates = data.get("rates", {})      # optional: {} if missing
cad_rate = rates.get("CAD")
if cad_rate is None:
    print("No CAD rate in this response.")
# No CAD rate in this response.

for currency, value in rates.items():
    print(f"1 USD = {value} {currency}")
# 1 USD = 0.9 EUR
# 1 USD = 0.76 GBP
# 1 USD = 153.2 JPY
```

Avoid plausible-looking fallbacks for essential data: a missing rate replaced by `0` or `"N/A"` would break a financial calculation silently.

### Lists of Records and Nested Fields

Many APIs return a **list** of records, each with nested dictionaries and lists. The course countries dataset is a list where `name` is a dictionary and `capital` is a list that can be empty.

**How to read this example:** Loop over the list, chain keys to reach nested values (`country["name"]["common"]`), and guard fields that may be empty before indexing them with `[0]`.

```python
countries = [  # same shape as the course countries.json
    {"name": {"common": "Portugal", "official": "Portuguese Republic"},
     "capital": ["Lisbon"], "region": "Europe", "population": 10804871},
    {"name": {"common": "Macau", "official": "Macao SAR of China"},
     "capital": [], "region": "Asia", "population": 685900},
]
print(type(countries).__name__)  # list

for country in countries:
    name = country["name"]["common"]
    capitals = country.get("capital") or ["no capital listed"]
    print(f"{name}: {capitals[0]}")
# Portugal: Lisbon
# Macau: no capital listed
```

> Go to **Exercise 3 (Country Data Fetcher)**, where you will search this list for a country by name and return its capital, population and region.

---

## 8. Caching API Responses

Calling the same endpoint again and again wastes time and can get you blocked. A cache saves a response to a file and reuses it while it is fresh.

**How to read this example:** The file's modification time gives its age. The cache stores the URL next to the data, so a file written for USD is never returned for EUR. Only a successful fetch is saved, so a failure never overwrites good cached data. `fetch_data()` is the function from section 6.

```python
import json
import os
import time

def cached_fetch(url, cache_file, max_age=3600):
    if os.path.exists(cache_file):
        age = time.time() - os.path.getmtime(cache_file)
        if age < max_age:
            with open(cache_file, encoding="utf-8") as f:
                cached = json.load(f)
            if cached.get("url") == url:  # same request?
                return cached["data"]
    data = fetch_data(url)  # the function from section 6
    if data is not None:    # never overwrite with a failure
        with open(cache_file, "w", encoding="utf-8") as f:
            json.dump({"url": url, "data": data}, f, indent=2)
    return data

base = "https://open.er-api.com/v6/latest/"
usd = cached_fetch(base + "USD", "api_cache.json")
eur = cached_fetch(base + "EUR", "api_cache.json")
print(usd["base_code"], eur["base_code"])  # USD EUR
```

### Caching: Save Work, but Define Freshness

A cache makes a dashboard faster and reduces the number of requests sent to an API. The right `max_age` depends on the data: a currency rate that changes during the day should not be cached indefinitely; a list of country names may be safe to cache for much longer.

For a reliable cache, also decide what happens when the cache file is malformed, and whether stale data should be used when the network is unavailable. Stale results can be better than no result for some applications, but financial or safety-sensitive decisions may require fresh data.

> Go to **Exercise 4 (API Caching)**, where you will build `cached_fetch()` yourself and check that a second call reuses the saved response.

---

## 9. Rate Limits and Retries

Most APIs limit how many requests a client may send per minute or per day. Over the limit, the server answers `429 Too Many Requests`, often with a `Retry-After` header saying how many seconds to wait.

**How to read this example:** On a `429` the function waits and tries again, at most `max_attempts` times. It uses `Retry-After` when it is a number of seconds; otherwise it **backs off**, doubling the wait each time (2 s, 4 s, …). Any other status goes through `raise_for_status()` as usual.

```python
import time
import requests

def get_with_retry(url, max_attempts=3):
    for attempt in range(1, max_attempts + 1):
        response = requests.get(url, timeout=10)
        if response.status_code != 429:
            response.raise_for_status()
            return response.json()
        if attempt == max_attempts:
            break
        retry_after = response.headers.get("Retry-After", "")
        wait = int(retry_after) if retry_after.isdigit() else 2 ** attempt
        print(f"429 Too Many Requests: waiting {wait} s")
        time.sleep(wait)
    raise RuntimeError("Still rate limited; try again later.")

# Server answers 429 with "Retry-After: 1" twice, then 200:
data = get_with_retry(url)
# 429 Too Many Requests: waiting 1 s
# 429 Too Many Requests: waiting 1 s
```

The best defense is not to hit the limit: cache responses (section 8) and never call an API inside a tight loop.

---

## 10. POST Requests

With `json=`, `requests` serializes a Python dictionary as JSON and sets the `Content-Type` header. A `POST` needs a timeout just like a `GET`.

**How to read this example:** The status `201` means the server created the record. Read the returned body for server-generated information, such as the new `id`, instead of assuming it equals what you sent.

```python
new_item = {"name": "Widget", "price": 9.99}
response = requests.post(url, json=new_item,
                         headers=headers, timeout=10)
response.raise_for_status()
print(response.status_code)  # 201
created_item = response.json()
print(created_item["id"])    # e.g. 101 (assigned by the server)
```

The server decides which fields are required and may return a normalized record or validation details. Do not send private information unless the service is trusted and the connection is protected (HTTPS).

---

## 11. A Real API Workflow

APIs are useful only when you treat them as unreliable systems.

### From the Happy Path to Robust Code

APIs are not magical: they are networked interfaces that return data in a predictable format. The main challenge is handling uncertainty. A first version often covers only the happy path:

**How to read this example:** Send a GET, check the status, parse the JSON and select the EUR rate. This works when the network is healthy and the response is exactly what we expect.

```python
import requests

url = "https://open.er-api.com/v6/latest/USD"
response = requests.get(url, timeout=10)
response.raise_for_status()

payload = response.json()
print(payload["rates"]["EUR"])  # e.g. 0.895793
```

But real code must also handle slow servers, bad connections, server-side errors, rate limits and missing keys in the JSON.

A more robust version:

**How to read this example:** The same request now fails politely: a timeout, an HTTP error or any other request problem prints a clear message, and a missing EUR rate is reported instead of crashing with `KeyError` or printing a fake value.

```python
import requests

url = "https://open.er-api.com/v6/latest/USD"
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    eur_rate = data.get("rates", {}).get("EUR")
    if eur_rate is None:
        print("The response has no EUR rate.")
    else:
        print(f"1 USD = {eur_rate} EUR")  # e.g. 1 USD = 0.895793 EUR
except requests.exceptions.Timeout:
    print("The API did not respond in time.")
except requests.exceptions.HTTPError as exc:
    print(f"API error: {exc.response.status_code}")
except requests.exceptions.RequestException as exc:
    print(f"Something went wrong: {exc}")
```

The user experience is much better when your app fails politely instead of crashing in the middle of a report.

### Why This Matters

A production-quality script should:
- fail gracefully when the API is down
- handle missing or empty JSON calmly
- avoid re-fetching API data too often, and back off on `429`
- keep secrets out of source code

> Go to **Exercise 5 (Multi-Currency Report)**, where you will group transactions by their original currency and compare them through EUR totals.

> Go to **Exercise 6 (Cached Currency Lookup)**, where you will combine a timeout, error handling and a per-currency cache in one function.

---

## Where to Practice: the Exercises

- Exercise 1 (First API Call)
- Exercise 2 (Currency Converter)
- Exercise 3 (Country Data Fetcher)
- Exercise 4 (API Caching)
- Exercise 5 (Multi-Currency Report)
- Exercise 6 (Cached Currency Lookup)

---

## Quick Reference

**How to read this example:** This block collects common request options and response properties as an index; use the sections above to choose the correct options for a particular API.

```
INSTALL    uv add requests
GET        requests.get(url, params=, headers=, timeout=10)
POST       requests.post(url, json=, headers=, timeout=10)
CHECK      response.raise_for_status()    4xx/5xx -> HTTPError
RESPONSE   .status_code  .ok  .headers  .text  .json()
ERRORS     Timeout  ConnectionError  HTTPError  RequestException
AUTH       headers={"Authorization": f"Bearer {api_key}"}
JSON       data["required"]   data.get("optional")  -> None
CACHE      save {"url", "data"}; reuse while mtime is fresh
429        wait Retry-After seconds (or 2, 4, 8 s...), retry
```
