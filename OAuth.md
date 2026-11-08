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

## Tokens

The tokens can be of different types, but the most common are:

- **Access Token**: A token that grants access to a specific resource for a limited time. It is usually short-lived and is used to authenticate API requests.
- **Refresh Token**: A token that can be used to obtain a new access token without user interaction, more long-lived than access tokens. It is used to maintain a user's session without requiring them to log in again.

The access token is usually a [**JWT**](JWT.md) (JSON Web Token), they are self-contained, cheap to verify and distribute, but can't be revoked before they expire.

Alternatively, the access or refresh token can be an opaque token, which is a random string (e.g., a UUID) that has no meaning outside the authorization server. The resource server must check with the authorization server to validate it, requires a database lookup, and is more expensive to verify, but can be revoked at any time, sessions use opaque tokens for this reason.

### ID Token

The ID token is a special token to authenticate the user. It is issued by the authorization server and must carry identity claims (`sub`, email, name, etc.).

- Client verifies it to know the user's identity, then uses the access token to act on their behalf.
- It is usually a JWT; an opaque ID token is possible.

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

Hence why it only asks once and then doesnt ask everytime user logs in. The user can revoke access at any time from their Google account settings.

> PKCE (`code_challenge`, `code_verifier`) is explained under [Grants](#1-authorization-code-grant-with-pkce).

## Grants

Access tokens exist to allow the clients to access resources or do operations on behalf of the resource owner, while a new access token is provided by the refresh token after it expires.

The way the client obtains an access token is called a **grant**,

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
