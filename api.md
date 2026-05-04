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

## Prerequisite Terminology for REST APIs

### Architecture of Web System,

The communication flow in a web system is:

```
Client -> API (Request/Response) -> Server
```

A typical web system has,

- Client & Server (which defines APIs)
- Database (closely aligned with the Server).

### HTTP & HTTPS

HyperText Transfer Protocol (HTTP) is a web protocol to transfer web sites data between server & client. HTTPS is a secure (encrypted) version of HTTP.

This is why almost all sites have, `https://www.google.com`

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


### Testing APIs

You can test APIs with

- `curl` directly
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
