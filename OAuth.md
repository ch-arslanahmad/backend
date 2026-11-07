# OAuth

OAuth is an authorization framework that allows a third-party application to obtain limited access to a service without giving it the user's credentials. It enables users to grant access to their resources on one site to another site without sharing their passwords.

Instead of handing out credentials, you get a **token** with a limited scope and lifetime.

## Problem

For example, a third-party application needs to access your Google Drive files for some task. Instead of giving the application your Google username and password, you can use OAuth to grant the application access to your files without sharing your credentials.

## Authentication vs Authorization

- Authentication is the process of verifying the identity of a user or system. In other words, answering **who are you**. Used in login systems via passkey, password, or other methods.

- Authorization (_Authz_) is the process of deciding whether a user has permission to access a resource or perform an action. In other words, answering **what can you do**. Used in access control systems via roles (RBAC), permissions, and policies.

They are separate questions: a valid token proves who you are, not what you may touch. You can be authenticated and still not authorized, a GitHub user is signed in but cannot open another user's private repository.

Authorization does not always require authentication. A public resource is authorized for everyone, so the policy has no identity to check; protected resources need a principal, so authentication must run first there.
