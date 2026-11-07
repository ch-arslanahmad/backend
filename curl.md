# cURL

curl (Client URL) is a command-line tool on Linux for transferring data to or from a server.

It supports many protocols, including `HTTP`, `HTTPS`, `FTP`, `SFTP`, and `SMTP`. Developers and system administrators use it for downloading files, _testing APIs_, and automating network tasks.

```bash
curl [options] [url]
```

## Common options

| Flag | Meaning                                                                |
| ---- | ---------------------------------------------------------------------- |
| `-i` | Include response headers in the output (normal method, GET by default) |
| `-I` | Send a HEAD request: headers only, no body                             |
| `-X` | Specify the HTTP method (`POST`, `PUT`, …)                             |
| `-H` | Add a request header                                                   |
| `-d` | Send a request body                                                    |
| `-b` | Send cookies from a file or manually                                   |
| `-c` | Save cookies to a file or manually                                     |
| `-x` | Use a proxy (for example `-x http://localhost:8080`)                   |
| `-L` | Follow redirects                                                       |
| `-v` | Verbose output                                                         |

Other options not covered above:

- `-o` Write the response to a file
- `-k` Skip TLS certificate verification (testing only)

## Basic GET

```bash
curl https://api.example.com/api/items
```

This will return the body of the response, which is usually JSON.

## Headers only

```bash
curl -I https://api.example.com/api/items
```

Example of a captured response from a Spring Boot app when the request is not allowed:

```http
HTTP/1.1 403
Allow: POST
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Date: Wed, 21 Oct 2026 14:20:29 GMT
```

`-i` shows the same headers plus the body.

## Headers and body

```bash
curl -i https://api.example.com/api/items
```

## Method and body

`-X` specifies the HTTP method for request, it's short for `--request`

- `-d` specifies the request body, it's short for `--data`.
- `-H` specifies the request header, it's short for `--header`. The `Content-Type` header is required to tell the server what type of data is being sent, the default is not `application/json`, even though JSON is what most APIs use.

```bash
curl -X POST https://api.example.com/api/items \
  -H "Content-Type: application/json" \
  -d '{"name": "example"}'
```

> [!note]
> The `-d` option automatically sets the request method to `POST`, so you can omit `-X POST` in this case.
>
> ```bash
> curl -d '{"name": "example"}' -H "Content-Type: application/json" https://api.example.com/api/items
> ```
>
> This is equivalent to the above.

## Cookies

Save the cookies from a response, then send them on later requests:

- `-c` specifies the cookie file to save cookies to, it's short for `--cookie-jar`, to save the cookies received from the server.
- `-b` specifies the cookie file to send cookies from, it's short for `--cookie`, to send the saved cookie to the server.

```bash
curl -c cookies.txt -X POST https://api.example.com/api/auth/login \
  -H "Content-Type: application/json" -d '{"email": "a@example.com", "password": "..."}'
```

```bash
curl -b cookies.txt https://api.example.com/api/auth/me
```

This is why a browser works while curl returns `401`: the browser manages the cookie jar automatically, curl does not.

## Proxy

```bash
curl -x http://localhost:8080 https://example.com
```

You basically make the request to the proxy, which then makes the request to the target server. This is useful for debugging and testing.

To learn more about curl, see the [official documentation](https://curl.se/docs/manpage.html).

However to practice curl, you can use the [httpbin](https://httpbin.org/) service, which is a simple HTTP request and response service. It allows you to test various HTTP methods, headers, cookies, and more.
