# JWT

JWT means JSON Web Token...

As we know JSON is a universally used format to transfer data.

While JWT is signed token provided by a server with a secret key.

Only the holder of the signing key (the server's secret, or its private key) can produce a JWT that verifies. Without it, any forgery fails at verification.

The server holds the SECRET key & produces the JWT. Client only stores and sends it.

A JWT contains three things:

- one is the header,
- the second is the payload (the main content),
- the third is the signature.

Hence called self-contained: the claims, the identity, travel inside the token, so the server needs no session lookup. The signature adds no data of its own, it only proves the claims were not altered.

A JWT is a credential, a signed container of claims; the application decides how to use it.

Keep in mind you make it a requirement for most authorized requests to include a token. While some are subject to exceptions like login/signup etc.

The actual flow,

```text
1. LOGIN
   Client: "email + password"
   Server: Validates credentials
   Server: **PRODUCES** the JWT (using secret key)
   Server: Sends JWT to client

2. CLIENT STORES
   Client: Receives JWT
   Client: Stores it (localStorage, cookie, memory)

3. EVERY REQUEST
   Client: **SENDS** the JWT (does NOT create new ones)
   Server: **VERIFIES** the JWT
   Server: Processes request
```

## Caution

The JWT **is** the authentication. Whoever holds it can make requests as that user.

By design, if anyone has your JWT, they can impersonate you. To mitigate this:

1. **HTTPS only**, tokens are encrypted while traveling.
2. **Short expiration**, tokens become useless quickly.
3. **httpOnly cookies**, prevents Cross-Site Scripting (XSS) attacks from stealing tokens. Also helps prevent Cross-Site Request Forgery (CSRF).

Additionally, use two types of tokens:

| Token Type  | Purpose                                                                |
| ----------- | ---------------------------------------------------------------------- |
| **Access**  | Short-lived; grants access to a resource                               |
| **Refresh** | Long-lived; used to obtain new access tokens without re-authentication |

These are just common types. There are many others (ID tokens, bearer tokens, session tokens, client tokens, one-time tokens) and not all of them use JWT format. JWT is a way of encoding a token, not the token itself.
