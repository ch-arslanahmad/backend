# API

API (Application Programming Interface) are standard ways programs talk to each other without knowing internal details.

Think of a restaurant: you (client) tell the waiter (API) your request. He processes it, gives it to the kitchen, and returns your food (response).

- **Client**, who gives the request
- **Server**, sends back the response

```
Browser: "Give me all users"
Server:  "[{\"name\": \"Alice\", \"email\": \"alice@example.com\"}]"
```

The format for API request & response is mostly JSON.

---

## HTTP Methods

| Method  | Use                                                   |
| ------- | ----------------------------------------------------- |
| GET     | Retrieve data                                         |
| POST    | Send data                                             |
| PUT     | Update existing data                                  |
| PATCH   | Update partially (one field)                          |
| DELETE  | Delete data                                           |
| OPTIONS | Ask which methods a URL supports (no body, no change) |

### OPTIONS

This request asks the server what methods are allowed on a given URL. The server responds with an `Allow` header listing the supported methods. This is often used in CORS preflight requests to check permissions before making the actual request.

For example,

```bash
curl -i -X OPTIONS http://localhost:3000/api/events
HTTP/1.1 200 OK
Allow: GET, POST, OPTIONS
Content-Length: 0

```

The list comes from the routes that exist, not from a permission grant. Spring MVC generates it for any mapped path; no handler needed. Safe and idempotent: no data, no side effect.

**CORS preflight.** Before a cross-origin request (different origin/port), the browser sends the OPTIONS itself, with `Origin` + `Access-Control-Request-Method`; server answers `Access-Control-Allow-*`. That preflight **never carries cookies**, so demanding auth on it only breaks the requests it exists to enable: a `401` here blocks the real request before it is sent. Fix: run CORS before auth (`http.cors(...)` in Spring Security), or permit preflights only in the security config.

Same-origin prod (nginx serves SPA and proxies `/api`) = no CORS = no preflight. Dev origin split (`:5173` → `:8080`) is where it shows up.

## Status Codes

| Code    | Meaning      | Common Use                 |
| ------- | ------------ | -------------------------- |
| **200** | OK           | Successful GET, PUT, PATCH |
| **201** | Created      | Successful POST            |
| **204** | No Content   | Successful DELETE          |
| **400** | Bad Request  | Invalid data sent          |
| **401** | Unauthorized | Not logged in              |
| **403** | Forbidden    | No permission              |
| **404** | Not Found    | Resource doesn't exist     |
| **500** | Server Error | Something broke            |

Quick mental model: `2xx` = success, `4xx` = your bad request, `5xx` = server problem.

---

## Anatomy of a Request

| Part         | Description     | Example                           |
| ------------ | --------------- | --------------------------------- |
| Method       | What to do      | GET, POST, PUT, DELETE            |
| URL/Endpoint | Where to send   | `http://localhost:7000/api/users` |
| Headers      | Metadata        | `Content-Type: application/json`  |
| Body         | Data (optional) | `{"name": "Alice"}`               |

## Anatomy of a Response

| Part        | Description     | Example                          |
| ----------- | --------------- | -------------------------------- |
| Status Code | Result          | `200 OK`, `404 Not Found`        |
| Headers     | Metadata        | `Content-Type: application/json` |
| Body        | Data (optional) | `{"id": 1, "name": "Alice"}`     |

---

## URL Breakdown

```
https://example.com/products/shoes/123?color=red&size=10
│        │                │      │         │
│        │                │      │         └─ Query params
│        │                │      └─────────── Path param
│        │                └────────────────── Endpoint
│        └─────────────────────────────────── Base URL
└──────────────────────────────────────────── Protocol
```

| Type              | Example              | Use Case                       |
| ----------------- | -------------------- | ------------------------------ |
| Path param `{id}` | `/api/students/5`    | Identifies a specific resource |
| Query `?key=val`  | `/api/students?id=5` | Optional filters/searching     |

Path params for unique resource ID. Query params for optional filters, also called **query string**.

### GET requests and the query string

In a GET request, there is nothing in the request body by convention (read-only, idempotent), and hence we fit every filter and all the info needed in the query string.

This introduces security risks: query strings are visible in logs and history, so don't put sensitive values there. The query string is the part after `?`, and query parameters are the `key=value` pairs in it.

If we need to send data, we use POST, PUT, or PATCH depending on what the request does.

---

## HTTP vs API Requests

Not every HTTP request is an API request. A normal page load returns HTML for browsers. An API request returns structured data (JSON) for programs.

### Example: Reddit format switching

```
https://www.reddit.com/r/mac/comments/... (returns HTML)
https://www.reddit.com/r/mac/comments/....json (returns JSON)
```

This is URL-based format switching (same resource, different output). It behaves like an API but isn't a dedicated endpoint. True API request:

```bash
curl https://jsonplaceholder.typicode.com/posts/1
# Returns: {"userId": 1, "id": 1, "title": "...", "body": "..."}
```

### How HTTP Works

1. Client sends request to server (page load or API call)
2. Server processes, interacts with database if needed
3. Server sends back response (e.g. `200 OK` with data)
4. Client receives and uses it

### HTTP vs HTTPS

|          | HTTP       | HTTPS               |
| -------- | ---------- | ------------------- |
| Security | Plain text | Encrypted (TLS/SSL) |
| Port     | 80         | 443                 |

---

## Web System Architecture

```
Client → Request/Response → Server → Database
```

- **Client** sends requests
- **Server** handles logic and defines APIs
- **Database** stores data, closely coupled with server

---

### Common Headers

| Header           | Meaning                         | Example value           |
| ---------------- | ------------------------------- | ----------------------- |
| `Content-Type`   | Data format                     | `application/json`      |
| `Authorization`  | Authentication information(key) | `Bearer token123`       |
| `Accept`         | Expected response format        | `application/json`      |
| `User-Agent`     | User agent string               | `Mozilla/5.0 (browser)` |
| `Cache-Control`  | Can this be cached?             | `max-age=3600` (1 hour) |
| `Content-Length` | Data length                     | `1024` (bytes)          |

### Principle: Statelessness

Each API call stands alone. The server doesn't remember your last call. No session, no context carried over. Every request must include everything the server needs to process it (auth tokens, IDs, etc.).

**Why it matters:** Don't expect the server to remember something from a previous request. This is why auth tokens must be sent with every request, not just at login.

## API Types

### 1. REST APIs

- Most popular. Uses HTTP methods (GET, POST, PUT, DELETE)
- Data in JSON, stateless (each request independent)

### 2. GraphQL APIs

- Single endpoint, ask for exactly the data you need
- One request can get data from multiple sources

### 3. WebSocket APIs

- Two-way continuous connection
- Both sides send and receive data in real-time

> [!important]
> REST is the most common. GraphQL for complex data fetching. WebSocket for real-time apps.

---

## Documenting APIs

- **OpenAPI** (formerly Swagger) is a standard machine-readable spec (JSON/YAML) for REST APIs: endpoints, parameters, schemas, auth, status codes.

- **Auto-generation:** FastAPI produces OpenAPI from type hints. Serves `/openapi.json`, `/docs` (Swagger UI), `/redoc`.

- **Hand-authored:** Write spec manually

---

> See [api-intermediate](api-intermediate.md) for authentication patterns, API testing, and CRUD best practices.
