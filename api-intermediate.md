# API Intermediate

Covers authentication, testing tools, and Service/DAO patterns. Assumes you know the basics from [api](api.md).

---

## Authentication

> See [JWT](JWT.md) for structure (header/payload/signature), login flow, access vs refresh tokens, and security cautions.

---

## API Testing: curl & HTTPie

### curl (Preinstalled on most systems)

### Basic GET Request

```bash
curl https://api.example.com/users
```

### GET with Query Parameters

```bash
curl "https://api.example.com/users?page=2&limit=10"
```

### POST Request with JSON Body

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}' \
  https://api.example.com/users
```

### Authentication (Bearer Token)

```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  https://api.example.com/protected-resource
```

### JWT Auth Request

JWT has three parts separated by dots: `header.payload.signature`

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.dozjgNryP4J3jVmNHl0w5N_XgL0n3I9PlFUP0THsR8U" \
  https://api.example.com/protected-resource
```

### Verbose Mode (Debugging)

```bash
curl -v https://api.example.com/resource
```

### Save Response to File

```bash
curl -o response.json https://api.example.com/resource
```

---

### HTTPie

Install on Arch: `sudo pacman -S httpie`.

You can [install](https://httpie.io/docs/cli/installation) if on any other OS

### Basic GET Request

```bash
http GET https://api.example.com/users
```

### GET with Query Parameters

```bash
http GET https://api.example.com/users page==2 limit==10
```

### POST Request with JSON Body

```bash
http POST https://api.example.com/users name="John" email="john@example.com"
```

### Authentication (Bearer Token)

```bash
http GET https://api.example.com/protected-resource Authorization:"Bearer YOUR_TOKEN_HERE"
```

### Save Response to File

```bash
http GET https://api.example.com/resource > response.json
```

---
