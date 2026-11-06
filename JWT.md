# JWT

JWT means JSON Web Token...

As we know JSON is a universally used format to transfer data.

While JWT is signed token provided by a server with a signing key (a shared secret, or a private key).

Only the holder of that key can produce a JWT that verifies; without it, any forgery fails at verification.

The client never creates one, it only stores the finished token and sends it back.

A JWT contains three things:

- one is the header,
- the second is the payload (the main content),
- the third is the signature.

Hence called self-contained: the claims, the identity, travel inside the token.

A JWT is a credential, a signed container of claims; the application decides how to use it.

A client only receives a JWT after it authenticates, through login or signup.

Protected endpoints require the token; unauthenticated endpoints such as `/login` and `/signup` are the exceptions.

## What JWT provides

By itself a JWT provides only:

- **Integrity**: the payload cannot be altered without invalidating the signature.
- **Authenticity**: a valid signature shows the token was issued by a holder of the signing key, so the server can trust the claims came from itself.
- **Statelessness**: the claims travel with the token, so no session lookup or DB query is needed to read them (`sub`, `role`, `exp`). This only covers identifying the caller. Anything the token cannot answer, such as the user's current role in the DB or who owns a row, still costs a query.

The two most common algorithms show the trade-off, who can verify:

- **HS256** signs and verifies with one shared secret, so a verifier can also forge and the secret must never leave the server.
- **RS256** uses a key pair: the private key signs, the public key verifies, so anyone can verify but only the key holder can issue.

It can carry both authentication and authorization data, but it is not a complete solution for either. A JWT is a format for a signed token: it defines signing, verification, and claims. The rest is up to the application.

## Payload & Claim

Payload is the entire JSON in the body of the JWT, while claim is single pair of key-value in the payload.

For example,

```json
{
  "sub": "2",
  "role": "student",
  "exp": 1730000000
}
```

This whole is the payload, while `sub`, `role`, and `exp` with their values are claims.

## Structure

The three parts are base64url-encoded and joined by dots: `header.payload.signature`.

| Part      | Holds                                                     | Example                                         |
| --------- | --------------------------------------------------------- | ----------------------------------------------- |
| Header    | `alg` (signing algorithm) and `typ` (token type, `"JWT"`) | `{"alg":"HS256","typ":"JWT"}`                   |
| Payload   | the claims, the actual data (id, role, expiry)            | `{"sub":"2","role":"student","exp":1730000000}` |
| Signature | proof the first two parts were signed with the secret     | `dozjgNryP4J3...`                               |

Header and payload are only _encoded_, not encrypted. Base64URL is just a convenient way to turn JSON into a string, so anyone can decode them and read the contents without any key. They are not secret, so never put sensitive data in the payload. Tamper-evidence comes from the signature over the header and payload, not from these two parts.

## Signing

Reading the payload is not the risk, changing it is. So the server _signs_ the token. Signing does not hide the payload, it only makes tampering detectable, and that is what makes the token trustworthy.

When the server creates a JWT, it:

1. Creates a header. (modifyable)
1. Creates a payload containing claims (e.g. user ID, role, expiration). (modifyable)
1. Signs the header and payload using a cryptographic signing algorithm. (modifyable but cannot create a valid signature without the secret key)

Concretely, it takes `base64url(header) + "." + base64url(payload)` and signs that string with the secret key:

```text
signature = sign(base64url(header) + "." + base64url(payload), secret)
```

Signature is computed over the header and payload (the signing input), so changing either invalidates it. It is not a hash of the payload alone, and it cannot be reused: you cannot take a signature from one token and attach it to a custom payload and header.

Change one character of the payload and the signature no longer matches, so the forgery is rejected.

## Claims

Claims are either registered by the spec or app-defined.

Some optional JWT standard claims are:

| Claim         | Meaning                                                |
| ------------- | ------------------------------------------------------ |
| `sub`         | subject, who the token is about (the user id)          |
| `iat`         | issued-at, when the token was made                     |
| `exp`         | expiry, after this the token is invalid                |
| `jti`         | unique token id, used for revocation                   |
| `iss` / `aud` | issuer / audience, who made it and who it is meant for |

`role` is not a registered claim. It is app-defined, a common name for the authorization data a token carries.

`sub`, `role` and `exp` are the ones most apps actually use; the rest is standard vocabulary.

## Verification

The server never trusts a token on sight. Verification means, in order:

1. **Signature.** Verify it with the key: recompute and compare for HMAC (HS256), or check it against the public key for RSA (RS256). Fails -> reject (forged or tampered).
2. **Expiry.** Check `exp` against the current time. Fails -> reject (expired).
3. **Claims.** Optionally check `iss`/`aud` (is it from the expected issuer and meant for this API?) and read `sub`/`role` to identify the caller.

Only after all pass does the request proceed. Crucially, verification is just a key check: no database or session lookup, which is exactly what keeps JWT auth stateless.

The moment you add a check the token alone cannot answer, such as looking up `jti` to revoke it, you have reintroduced state and given up that property.

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

The JWT **is** the credential the API trusts. Whoever holds it can make requests as that user.

By design, if anyone has your JWT, they can impersonate you.

The header says which algorithm signed the token, and the header is attacker-controlled. A server that trusts `alg` can be tricked into accepting `{"alg":"none"}` with no signature at all, or into checking an RS256 token with the public key treated as an HMAC secret. Pin the algorithm server-side and reject anything else.

To mitigate the rest:

1. **HTTPS only**, so the token cannot be sniffed or swapped in transit. The connection is encrypted, the token itself is not.
2. **Short expiration**, tokens become useless quickly.
3. **httpOnly cookies**, prevent Cross-Site Scripting (XSS) from stealing the token; JS can't read it.
4. **CSRF protection**, using `SameSite` (`Lax` or `Strict`) attributes and custom headers to prevent Cross-Site Request Forgery attacks.

A common pattern is two types of tokens:

| Token Type  | Purpose                                                                |
| ----------- | ---------------------------------------------------------------------- |
| **Access**  | Short-lived; grants access to a resource                               |
| **Refresh** | Long-lived; used to obtain new access tokens without re-authentication |

These are just common types. There are many others (ID tokens, session tokens, client tokens, one-time tokens), and not all of them use JWT format. Bearer is a separate axis: it describes how a token is sent, not what it is for, and access and refresh tokens are usually bearer tokens. JWT names the format; a single token that follows it is also called a JWT.
