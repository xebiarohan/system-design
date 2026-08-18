

# Phase 1 — Authentication Fundamentals

**Goal:** Understand how a user proves their identity.

## 1. Authentication vs Authorization

Learn the difference between:

* Authentication → "Who are you?"
* Authorization → "What are you allowed to do?"

Example:

```
Login with username/password
        ↓
Authentication

Access Admin Dashboard
        ↓
Authorization
```

---

## 2. Identity

Learn:

* User identity
* User accounts
* Credentials
* User IDs
* Roles
* Permissions

Understand that authentication establishes identity, while authorization determines access.

---

## 3. Password Authentication

Topics:

* Password hashing
* Salt
* Pepper
* Why passwords are never stored in plain text

Algorithms:

* bcrypt
* Argon2
* scrypt

Understand:

```
Password
    ↓
Hash
    ↓
Store hash only
```

---

## 4. Sessions

Before learning tokens, understand sessions.

Learn:

* Session IDs
* Cookies
* Server-side session storage
* Session expiration
* Logout
* Session hijacking

Architecture:

```
Browser
    ↓
Cookie(SessionID)
    ↓
Server
    ↓
Session Store
```

Questions:

* Why does a browser stay logged in?
* Where is login information stored?

---

# Phase 2 — Token-Based Authentication

Now you'll understand why tokens became popular.

---

## 5. What is a Token?

Learn:

* Access token
* Refresh token
* Bearer token

Questions:

* Why use tokens?
* Why not sessions?
* Stateless authentication

---

## 6. API Authentication

Learn how APIs authenticate clients.

Examples:

```
Authorization: Bearer TOKEN
```

Understand:

* HTTP headers
* Authorization header
* Bearer authentication

---

## 7. JWT (JSON Web Token)

This is one of the most important topics.

Learn:

Structure:

```
Header

Payload

Signature
```

Example:

```
xxxxx.yyyyy.zzzzz
```

Understand:

* Claims
* Standard claims (`sub`, `exp`, `iat`, `iss`, `aud`)
* Custom claims
* Expiration
* Signature verification

---

## 8. JWT Signing

Learn:

* HMAC
* RSA
* Public/private keys

Difference:

```
HS256

Shared secret
```

vs

```
RS256

Private key signs

Public key verifies
```

---

## 9. JWT Security

Very important.

Learn:

* Never trust payload without verification
* Token expiration
* Token theft
* Replay attacks
* Secure storage
* Token revocation

---

# Phase 3 — OAuth 2.0

Now OAuth will make much more sense.

---

## 10. Why OAuth Exists

Problem:

Instead of giving your password to another app,

```
GitHub

↓

Google

↓

Facebook
```

OAuth lets users authorize access without sharing their password.

---

## 11. OAuth Roles

Know these roles:

* Resource Owner
* Client
* Authorization Server
* Resource Server

Be able to identify each role in an example.

---

## 12. OAuth Flow

Study the main flows:

### Authorization Code Flow

Most important.

```
User

↓

Client

↓

Authorization Server

↓

Access Token
```

---

### PKCE

Essential for:

* Mobile apps
* Single-page applications (SPAs)

Understand why it's needed.

---

### Client Credentials Flow

Used for:

```
Server

↓

Server
```

No user involved.

---

### Refresh Token Flow

```
Access Token expires

↓

Refresh Token

↓

New Access Token
```

---

### Device Authorization Flow

Used for:

* TVs
* Smart devices
* Consoles

---

# Phase 4 — OpenID Connect (OIDC)

Many developers confuse OAuth with login.

OAuth **does not define user authentication**; it defines delegated authorization. **OpenID Connect (OIDC)** builds on OAuth 2.0 to provide standardized user authentication and identity information.

Learn:

* ID Token
* UserInfo endpoint
* Scopes (`openid`, `profile`, `email`)

Understand:

```
OAuth

↓

Authorization

OpenID Connect

↓

Authentication
```

---

# Phase 5 — Identity Providers

Learn how real-world providers implement OAuth/OIDC.

Examples:

* Google
* GitHub
* Microsoft
* Auth0
* Okta
* Keycloak

Study:

* Login with Google
* Login with GitHub

---

# Phase 6 — API Security

Learn how APIs are protected.

Topics:

* HTTPS/TLS
* CORS
* CSRF
* XSS
* SameSite cookies
* API keys
* Rate limiting

Know when to use:

* Cookies
* Sessions
* JWT
* OAuth

---

# Phase 7 — Token Lifecycle

Understand the complete lifecycle.

```
Login

↓

Verify credentials

↓

Issue Access Token

↓

API Requests

↓

Token expires

↓

Refresh Token

↓

New Access Token

↓

Logout

↓

Revoke Refresh Token
```

---

# Phase 8 — Token Storage

Where should tokens be stored?

Compare:

* HttpOnly cookies
* Secure cookies
* Browser memory
* Local Storage
* Session Storage

Understand the security trade-offs of each.

---

# Phase 9 — Authorization Models

Authentication identifies the user; authorization determines what they can do.

Learn:

* Role-Based Access Control (RBAC)
* Attribute-Based Access Control (ABAC)
* Scopes
* Permissions
* Claims-based authorization

---

# Phase 10 — Advanced Topics

Once you're comfortable with the basics, explore:

* OAuth 2.1
* Token introspection
* Token revocation
* JSON Web Keys (JWK/JWKS)
* JSON Web Signature (JWS)
* JSON Web Encryption (JWE)
* Proof-of-Possession (PoP) tokens
* Mutual TLS (mTLS)
* Single Sign-On (SSO)
* Single Logout (SLO)
* Multi-Factor Authentication (MFA)
* Passkeys and WebAuthn
* SAML (for enterprise systems)

---

# Practical projects

Theory sticks much better when you build something. Progress through these in order:

1. **Basic login system**

   * Email/password
   * Password hashing
   * Sessions

2. **JWT authentication API**

   * Login endpoint
   * Generate JWT
   * Protect routes
   * Verify JWT

3. **Refresh token implementation**

   * Short-lived access tokens
   * Refresh token rotation
   * Logout

4. **Role-based authorization**

   * Admin vs User
   * Route protection

5. **"Login with Google"**

   * OAuth 2.0 + OIDC
   * Exchange authorization code for tokens
   * Retrieve user profile

6. **Microservices authentication**

   * API Gateway
   * JWT verification
   * Service-to-service authentication

---

## Recommended learning order

```
Authentication Basics
        ↓
Passwords & Hashing
        ↓
Sessions & Cookies
        ↓
HTTP Authentication
        ↓
Tokens
        ↓
JWT
        ↓
Access & Refresh Tokens
        ↓
Authorization (RBAC/ABAC)
        ↓
OAuth 2.0
        ↓
OpenID Connect
        ↓
Identity Providers
        ↓
API Security
        ↓
Advanced OAuth & Identity
```

This progression builds from first principles to production-grade identity systems, making it much easier to understand why each technology exists and when to use it.
