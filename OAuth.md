# OAuth

OAuth is an authorization framework that allows a third-party application to obtain limited access to a service without giving it the user's credentials. It enables users to grant access to their resources on one site to another site without sharing their passwords.

Instead of handing out credentials, you get a **token** with a limited scope and lifetime.

OAuth can be called a delegated authorization.

## Problem

For example, a third-party application needs to access your Google Drive files for some task. Instead of giving the application your Google username and password, you can use OAuth to grant the application access to your files without sharing your credentials.

## Authentication vs Authorization

- Authentication is the process of verifying the identity of a user or system. In other words, answering **who are you**. Used in login systems via passkey, password, or other methods.

- Authorization (_Authz_) is the process of deciding whether a user has permission to access a resource or perform an action. In other words, answering **what can you do**. Used in access control systems via roles (RBAC), permissions, and policies.

They are separate questions: a valid token proves who you are, not what you may touch. You can be authenticated and still not authorized, a GitHub user is signed in but cannot open another user's private repository.

Authorization does not always require authentication. A public resource is authorized for everyone, so the policy has no identity to check; protected resources need a principal, so authentication must run first there.

## 4 Roles

| Roles           | Who                                                                      | Access                                                                       |
| --------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| Resource Owner  | Main user (you/me)                                                       | Owns data. Says yes/no to access.                                            |
| Client          | App asking for access                                                    | Requests access to resource server on behalf of resource owner.              |
| Auth Server     | Authenticates the main user, validates the client & obtains user consent | Issues tokens to clients after successful authentication and authorization.  |
| Resource Server | API holding data                                                         | Checks token before serving. Validates token and scopes to determine access. |

The auth server does the authentication. OAuth's main purpose is authorization. The resource server checks the token, not the user.

## Requests and endpoints

OAuth is not a single endpoint. It is a protocol: roles, a few endpoints, and the parameters each one accepts. A login is a sequence of HTTP requests to those endpoints.

The auth server owns the endpoints. The client owns the callback.

| Endpoint                  | Method | Channel          | Purpose                                                    |
| ------------------------- | ------ | ---------------- | ---------------------------------------------------------- |
| `/authorize`              | GET    | browser redirect | user logs in and consents, auth server returns a `code`    |
| `/token`                  | POST   | server-to-server | trade the `code` (+ verifier) for tokens                   |
| `/revoke` / `/introspect` | POST   | server-to-server | optional: revoke a token, or ask whether it is still valid |

For the general shape of a request (method, URL, headers, body), see [api.md](api.md#anatomy-of-a-request).

Everything after `?` is a parameter, not part of the endpoint:

```http
GET https://auth.example.com/authorize
    ?response_type=code
    &client_id=...
    &redirect_uri=https://app.example.com/callback
    &scope=pods:read
    &state=...
    &code_challenge=...
```

`redirect_uri` is a parameter whose value points at an endpoint, but that endpoint is on the client, not the auth server. `state` is just an opaque value, not a URI at all.

`response_type=code` selects the flow, what to send back from auth server to the client i.e., `code`.

## Tokens

The tokens can be of different types, but the most common are:

- **Access Token**: A token that grants access to a specific resource for a limited time. It is usually short-lived and is used to authenticate API requests.
- **Refresh Token**: A token that can be used to obtain a new access token without user interaction, more long-lived than access tokens. It is used to maintain a user's session without requiring them to log in again.

The access token is usually a [**JWT**](JWT.md) (JSON Web Token), they are self-contained, cheap to verify and distribute, but can't be revoked before they expire.

Alternatively, the access or refresh token can be an opaque token, which is a random string (e.g., a UUID) that has no meaning outside the authorization server. The resource server must check with the authorization server to validate it, requires a database lookup, and is more expensive to verify, but can be revoked at any time, sessions use opaque tokens for this reason.

### ID Token (OIDC)

The ID token is an OpenID Connect (OIDC) extension on top of OAuth 2.0 to authenticate the user. It is issued by the authorization server as a signed JWT and carries identity claims (`sub`, email, name, etc.).

- Client verifies it to know the user's identity, then uses the access token to act on their behalf.
- Unlike access tokens (which can be opaque), an ID token is always a JWT.

### Sending the Access Token

- API requests carry the access token in the `Authorization` header, as a bearer token:
  `Authorization: Bearer <token>`
- Resource server validates the token before serving.

> [!note]
> The choice between JWT and opaque tokens depends on the specific use case and requirements of the application.
>
> - JWTs are more efficient for verification but cannot be revoked
> - opaque tokens can be revoked but require additional server-side logic.
>   They may be used together, with the access token being a JWT and the refresh token being opaque.

## Client Types

There are two kinds of OAuth clients, split by whether they can keep a secret.

1. **Confidential Client**: can securely store a secret (e.g., server-side applications). It uses the `client_secret` to authenticate itself to the authorization server.
2. **Public Client**: cannot securely store a secret (e.g., mobile apps, single-page applications). It has no `client_secret` and must rely on PKCE for security.

Providers require PKCE for public clients. It may also be used with confidential clients, and using it everywhere is the recommended approach.

## Flow: Authorization Code + PKCE

This is the most common flow for web and mobile applications.

It involves the following steps:

<img alt="Authorization Code + PKCE Flow" src="https://developers.google.com/static/identity/images/branding_guide_android_web_spec.png" width="300">

1. The **client** redirects the user to the _authorization server’s authorization endpoint_, sending a PKCE code challenge plus other params (client ID, redirect URI, scopes, etc.).

2. _User authenticates_ & _consents at the auth server_, the user grants access to the client.

<img alt="account-chooser-and-consent" src="https://uploads.strapi.logto.io/customize_google_prompt_59a998bf66.png">

3. The **authorization server** redirects the _user_ back to the client with an authorization_code.
4. The **client** posts the `authorization_code` and `code_verifier` to the auth server’s token endpoint.
5. The **auth server** validates the _code_ and verifier, then issues an access_token (and optionally a refresh_token) to the client.

If already logged in Google, there may be implicit or explicit consent, so the user is not prompted to log in or consent again.

<img src="https://developers.google.com/static/identity/gsi/web/images/one-tap-ui-fedcm.png" width="400">

<img src="https://developers.google.com/static/identity/gsi/web/images/auto-sign-in-fedcm.png" width="400">

Hence why it only asks once and then doesn't ask every time user logs in. The user can revoke access at any time from their Google account settings.

> PKCE (`code_challenge`, `code_verifier`) is explained under [Grants](#1-authorization-code-grant-with-pkce).

### Callback checks (state + redirect_uri)

The callback is a URL on the client (`redirect_uri`). It is a **new** HTTP request that arrives after an ordinary `302` chain (the same redirect `curl -L` follows), so nothing is in memory when it lands (HTTP is stateless). `state` is the only link back to the flow that started it, and the value compared against it lives in the session cookie.

To start the flow, the client sends a request to the auth server:

```
GET https://provider.com/authorize?client_id=abc&redirect_uri=https://myapp.com/callback&state=xyz123
```

The auth server checks `redirect_uri` here, before anything else happens: it must exactly match the URI registered for that `client_id`. A mismatch fails the request, so no login page and no code.

After the user logs in and consents, the auth server redirects back to that same `redirect_uri` with the code and the unchanged `state`:

```
HTTP/1.1 302 Found
Location: https://myapp.com/callback?code=authcode123&state=xyz123
```

2 requests happen due to a redirect, monitor via `curl -L` which will show the redirect chain.

- `state` prevents login-CSRF: without it, an attacker can silently complete a login with their own account and feed the victim's browser `callback?code=ATTACKER_CODE`, binding the victim's session to the attacker's account.
- `redirect_uri` prevents code leakage: without an exact match, an attacker could send `/authorize` with `redirect_uri=https://evil.com` and your public `client_id`, and the victim's code would arrive at the attacker's server.

Code theft happens when the code ends up anywhere other than the client that requested it: a hijacked `redirect_uri`, an open redirect on the client, browser history, or proxy logs. Exact matching stops the code from being delivered to another host, but cannot stop leakage from the real callback. PKCE covers that gap: redeeming a stolen code still requires the `code_verifier`, which never left the client.

Callback adds statefulness to the flow.

## Grants

Access tokens exist to allow the clients to access resources or do operations on behalf of the resource owner, while a new access token is provided by the refresh token after it expires.

The way the client obtains an access token is called a **grant**.

The most common grant types are:

### 1. **Authorization Code Grant** (with PKCE)

This is the way Google, GitHub, and other OAuth providers recommend for web and mobile apps. It is the most secure way to obtain an access token.

#### PKCE (Proof Key for Code Exchange)

PKCE is an extension of the Authorization Code flow. It stops a stolen authorization code from being exchanged for a token: the client sends a hashed `code_challenge` up front, then proves that it still holds the original `code_verifier` when it redeems the code. The verifier never appears in the browser redirect.

The real issue is that the `client_secret` is static and reusable. It’s the same value for every login, for as long as it’s valid. If it leaks once, the attacker can use it again and again until someone notices and rotates it. Any stolen code paired with it works.

#### With and without PKCE

Both versions follow the same sequence. They differ only in setup and in the token request.

| Step                            | Confidential, no PKCE                                                                                                                                 | Confidential + PKCE                                                                                                  |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| 1. Setup                        | Client holds a long-lived `client_secret` in its config                                                                                               | Same, plus client generates a random `code_verifier` in memory and computes `code_challenge = SHA256(code_verifier)` |
| 2. Authorization request        | Client redirects user to auth server with `client_id`, `redirect_uri`, `response_type=code`                                                           | Same, plus `code_challenge` and `code_challenge_method=S256`                                                         |
| 3. Login                        | User logs in and consents. Auth server issues a `code`                                                                                                | Same, plus auth server records `code_challenge` against that `code`. Only the `code` is returned to the client       |
| 4. Redirect (front channel)     | Auth server redirects browser to `redirect_uri?code=...`. The code is visible in the browser and may appear in logs                                   | Same. The verifier is never sent here                                                                                |
| 5. Token request (back channel) | Client sends `code`, `client_id`, `client_secret`, `redirect_uri`                                                                                     | Same, plus `code_verifier`                                                                                           |
| 6. Verification                 | Auth server checks the code is valid, unused, and unexpired, that it belongs to this `client_id` and `redirect_uri`, and that `client_secret` matches | Same checks, plus auth server hashes `code_verifier` and compares it to the stored `code_challenge`                  |
| 7. Result                       | Tokens issued if all checks pass                                                                                                                      | Tokens issued only if all checks pass, including the challenge match                                                 |

PKCE adds a check on top of the secret; it does not replace it. For a public client there is no secret to add it to, so PKCE is the only proof that the token request came from the client that started the login.

### 2. Implicit Grant (deprecated)

The access token is returned directly in the front‑channel redirect (the URL fragment or query string). No separate token request step.

For example,

```
GET https://auth.example.com/authorize
    ?response_type=token
    &client_id=...
    &redirect_uri=https://app.example.com/callback
    &scope=pods:read
```

It is deprecated and not used in modern apps due to the security risks of **exposing the access token** in the browser and the lack of refresh tokens. It is recommended to use the Authorization Code Grant with PKCE instead.

### 3. Resource Owner Password Credentials Grant [ROPC] (deprecated)

The client collects the user's credentials (username and password) and sends them to the authorization server to obtain an access token. This flow is not recommended due to security risks, as it requires the client to handle sensitive user credentials directly.

It is no different from a normal login, and the client can impersonate the user. It is only used in legacy applications where other flows are not feasible.

> [!important]
> The implicit/password are dead in modern apps due to being insecure.

### 4. Client Credentials Grant

The client authenticates with its own `client_secret` and gets an access token with **no user involved** — no login, no browser, no consent screen. The token represents the client itself, not a user, so it can't carry claims like `email` or `role`.

The authorization in this case is based on the client’s identity and its permissions decided by the authorization server, not on a user’s identity. The access token can be used to access resources or perform operations that are allowed for the client itself.

This is used for server-to-server communication, for example

- An MCP server accesses external APIs without acting on behalf of a user.
- Backend-for-frontend (BFF) talking to a microservice
- Backend-to-backend communication (third-party service talking to your API),

For example, a MCP server can do this to get an access token to call a protected API:

```http
POST /token
Content-Type: application/x-www-form-urlencoded
client_id=service-id
&client_secret=service-secret
&grant_type=client_credentials
&scope=pods:read
```

### Grants Summary

| Grant                      | Has user? | Has Browser?            | How token is handed over  |
| -------------------------- | --------- | ----------------------- | ------------------------- |
| Authorization Code (+PKCE) | yes       | yes (redirect to login) | POST `/token` backchannel |
| Implicit                   | yes       | yes (redirect)          | in URL fragment (leaked)  |
| Resource Owner Password    | yes       | no                      | POST `/token` with creds  |
| Client Credentials         | no        | no                      | POST `/token` with secret |

After the authorization code grant, a refresh token lets you mint new access tokens without re-logging in (`grant_type=refresh_token`).

> [!note]
> Refresh isn’t one of the four ways to get a first token. It comes after one. An initial grant, usually authorization code, gives you an access token and a refresh token. Later, `grant_type=refresh_token` exchanges the refresh token for a new access token, so the session continues without a new login.
> The spec lists `refresh_token` as a grant type value, which is why it’s called a “refresh grant”. However, it can’t start a session on its own, hence also called **maintenance grant**. It is not a replacement for the four main grants, it is a continuation of one of them.

## Scopes & Consent

A scope is a named permission string the client asks for, such as `pods:read` or `reports:write`, following [resource:permission] format. It describes what the token is allowed to do.

For example a GET request,

```
GET /authorize?client_id=my-app&scope=pods:read&redirect_uri=...&response_type=code
```

Scopes keep access narrow. An app that only displays your pods doesn’t need permission to delete them.

1. The client sends the scopes it needs in the authorization request (`scope=pods:read`).

2. The authorization server shows the user what the client(app) is asking for, and the user approves or denies. This is **consent**.

3. The authorization server issues an access token with the approved scopes.

4. The access token carries the approved scopes in its `scope` claim.

The resource server validates the scopes on every request and rejects any call that requires a scope not present in the token.

A user can revoke the refresh token, which prevents further access‑token issuance and effectively invalidates the session’s access to protected resources.

> [!important]
> With the client‑credentials grant there is no user, so no consent screen appears; the scopes are defined by the client’s registration and the scopes it requests.

## Token Lifecycle & Refresh

When the access token expires, the client exchanges the refresh token silently:

```
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA&client_id=my-spa-client
```

There is no Authorization header and no secret (as in confidential apps). The `client_id` in the body tells the server which app is asking.

If valid, the server responds with a new access token and maybe a new `refresh_token` depending on the rotation policy (see below).

### When the refresh token expires

Server returns `invalid_grant`. The user must go through `/authorize` again.

A refresh token stops working when:

- Lifetime passed or idle timeout hit.
- User revoked access or changed their password.
- Server detected reuse of an already-used token.

If a stolen refresh token is used, rotation detects it.

## Rotation: replacing the refresh token

- **Non-rotating:** The refresh token stays the same. Each refresh returns only a new access token, and the app keeps the original refresh token until it expires. This is simpler, but a stolen refresh token keeps working for its whole lifetime.

- **Rotating:** Each refresh returns a new refresh token and invalidates the old one. The app must save the new one immediately and discard the old one.

Rotation is required for most modern apps due to them being public as it has no secret to protect the refresh token so the server has to rely on rotation to limit damage if the token is stolen. If someone presents an old token that was already rotated out, the server knows a copy was stolen.

It can then revoke the whole chain of tokens from that login, which forces a fresh login.

### Operational Problem with Rotation

Rotation has one practical trap. If your app sends two refresh requests at the same time (common when several API calls fail together), the second request uses a token that was already rotated. The server may read that as theft and revoke everything as stated (may vary by policy).

To avoid this, make sure:

- only one refresh runs at a time.
- Other requests should wait for it to finish and then use the new access token.
- Save the new refresh token before you use the new access token.
