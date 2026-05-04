# **API**

API (Application Programming Interface) are basically standard ways programs talk to each other, without knowing internal details.

For example, In a restaurant,

- You (Client), tell the waiter (API) your request.

He processes it and gives it to the restaurant and returns you the food (response) you asked.

So basically,

- **Client:** who gives the request.
- **Server:** sends back the response to request

You cannot go to the kitchen directly, you communicate via the waiter (API).

```
Browser: "Give me all users"
Server:  "[{\"name\": \"Alice\", \"email\": \"alice@example.com\"}]"
```

The format for the API request & response is in `JSON`.

## HTTP Fundamentals

### HTTP vs HTTPS

HyperText Transfer Protocol (HTTP) is a web protocol to transfer web sites data between server & client. HTTPS is a secure (encrypted) version of HTTP.

HTTP uses port 80.
HTTPS uses port 443

This is why almost all sites have, `https://www.google.com`


### How HTTP works

HTTP works on a client-server model:

1. Client (browser) sends a request to the server (e.g. `GET /users`, a page rending request, or an API request)
2. Server processes the request, interacts with database if needed
3. Server sends back a response (e.g. `200 OK` with user data in JSON)
4. Client receives response and renders it (e.g. shows user list on page)

Keep in mind, not every request is an API request, it can be a normal page load, generally called a HTTP request.

- API requests are a specific type of HTTP request that usually return data in structured format like JSON (most-common), XML etc.

If you go to a post in reddit, for example,
https://www.reddit.com/r/mac/comments/1nv5vcu/any_opinions_of_the_mac_mini/

It opens a webpage, because it sends a HTTP request (page-load request) and gets back HTML.

However if you add `.json` at the end of the URL, Reddit returns the same content as JSON data instead of HTML:

https://www.reddit.com/r/mac/comments/1nv5vcu/any_opinions_of_the_mac_mini.json

This is URL-based format switching, same resource, different output format. It behaves like an API request because it returns JSON, but it's not a separate API endpoint. True API requests hit dedicated endpoints like:

```bash
# Proper API request (dedicated endpoint, returns JSON)
curl https://jsonplaceholder.typicode.com/posts/1

# Returns: {"userId": 1, "id": 1, "title": "...", "body": "..."}
```

This is a real API request, separate endpoint, structured JSON, meant for programs.

To load a webpage, the browser sends a HTTP request to the server, which returns HTML. This is a normal HTTP request called a page load request, not an API request.

The thing with API is that they are formats meant to be processed for a user output and are meant to call for a task to be done by another program rather than a simple page rendering, as API mean Application Programming Interface, they are meant to be used by other programs, not just for page rendering.

### HTTPS Methods

To communicate & transfer data, HTTPs uses **HTTPS Methods**:

| Methods | Use                          |
| ------- | ---------------------------- |
| POST    | Send Data                    |
| DELETE  | Delete Data from Server      |
| PUT     | Update Existing Data         |
| GET     | Retrieve Data                |
| PATCH   | Update partially (one field) |

### HTTPS Status Codes

| Code    | Meaning      | Common Use Case                                |
| ------- | ------------ | ---------------------------------------------- |
| **200** | OK           | Successful GET, PUT, PATCH                     |
| **201** | Created      | Successful POST (new resource created)         |
| **204** | No Content   | Successful DELETE                              |
| **400** | Bad Request  | You sent invalid data (missing required field) |
| **401** | Unauthorized | Not logged in                                  |
| **403** | Forbidden    | Logged in but no permission                    |
| **404** | Not Found    | Resource doesn't exist                         |
| **500** | Server Error | Something broke on the server                  |
**To Summarize:**

- 2xx = success
- 4xx = bad request by you
- 5xx = server problem

> [!note]
> Keep in mind, these are the most common, you have from 200-599 total status codes.

> [!important]
> Come back here after learning:
>
> - Async vs sync
> - Background jobs / queues
> - Webhooks or polling

## Prerequisite Terminology for REST APIs

### Architecture of Web System,

The communication flow in a web system is:

```
Client -> Request/Response -> Server
```

The request/response may be handled via an API or may not be.

A typical web system has,

- Client & Server (which defines APIs)
- Database (closely aligned with the Server).

## Anatomy of an API Request

| Part         | Description          | Example                                         |
| ------------ | -------------------- | ----------------------------------------------- |
| Method       | What to do           | GET, POST, PUT, DELETE                          |
| URL Endpoint | Where to send        | http://localhost:7000/api/users                 |
| Headers      | Metadata             | Content-Type: application/json                  |
| Body         | Data sent (optional) | {"name": "Alice", "email": "alice@example.com"} |

## Anatomy of an API Response

| Part        | Description              | Example                                                  |
| ----------- | ------------------------ | -------------------------------------------------------- |
| Status Code | Result of the request    | 200 OK, 201 Created, 400 Bad Request, 404 Not Found      |
| Headers     | Response metadata        | Content-Type: application/json, Content-Length: 123      |
| Body        | Data returned (optional) | {"id": 1, "name": "Alice", "email": "alice@example.com"} |

### Difference between a **URLs** & **Endpoints**

- URL is the where the whole web page lies, like HTML etc.
- endpoint is where only the data lies, in JSON, it is a URL path for a specific API resource

```bash
https://example.com/products/shoes/123?color=red&size=10
│        │                │      │         │
│        │                │      │         └─ Query params
│        │                │      └─────────── Path param
│        │                └────────────────── Endpoint
│        └─────────────────────────────────── Base URL
└──────────────────────────────────────────── Protocol
```

Here,

- `https://example.com/products/shoes/123?color=red&size=10` is the actual URL.
- `/products/shoes/123` is the endpoint (path to specific resource)
- path param is `123` (specific shoe ID)
- `color=red&size=10` are query parameters (additional filters or options)

### Testing APIs

You can test APIs with

- `curl` directly.
- `httpie` for a a friendlier curl
- Postman for large complex API endpoints, enterprise standard

## Type of APIs

### 1. REST APIs

- Most popular API type.
- Uses HTTPS methods: GET, POST, PUT, DELETE
- Data in JSON
- Stateless (each request is independent)

### 2. GraphQL APIs

- single endpoint
- Lets you ask for exactly the type of data needed.
- single request can get data from multiple sources.

### 3. WebSocket API

- Two-way communication, continuous connection between client & server.
- both sides can send and receive data

> [!note]
> The most popular & common API type is REST API, however each has its own usage like GraphQL for complex data fetching and WebSocket for real-time applications.
