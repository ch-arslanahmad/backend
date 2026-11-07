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

Alternatively, the access token can be an opaque token, which is a random string (e.g., a UUID) that has no meaning outside the authorization server. The resource server must check with the authorization server to validate it, requires a database lookup, and is more expensive to verify, but can be revoked at any time, sessions use opaque tokens for this reason.

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
