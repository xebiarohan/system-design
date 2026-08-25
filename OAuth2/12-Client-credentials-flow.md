# OAuth Flows — Client Credentials Flow

The **Client Credentials Flow** is the OAuth flow used when an application needs to authenticate **itself** to another application/API.

The key difference from the flows you've already learned:

> **There is no user involved.**

You can think of it as:

```text
Application A
     ↓
"Here are my credentials"
     ↓
Authorization Server
     ↓
Access Token
     ↓
Application B / API
```

This is commonly used for **machine-to-machine (M2M)** communication.

---

## 12.1 When Do We Use Client Credentials?

Suppose you have two backend services:

```text
Order Service
      ↓
Payment Service
```

The Order Service needs to call:

```http
POST /payments
```

But there is no human user making this request.

The request is being made by:

```text
Order Service
```

So an Authorization Code Flow doesn't make sense.

There is no:

```text
User
  ↓
Login
  ↓
Consent
```

Instead:

```text
Order Service
      ↓
Authenticate itself
      ↓
Authorization Server
      ↓
Access Token
      ↓
Payment Service
```

This is exactly what Client Credentials Flow is designed for.

---

# 12.2 The Most Important Concept

Compare the flows you've already learned.

### Authorization Code

```text
User
  ↓
Client Application
  ↓
Authorization Server
  ↓
User authenticates
  ↓
Authorization Code
  ↓
Access Token
```

The **user** is the resource owner.

---

### PKCE

```text
User
  ↓
Client Application
  ↓
Authorization Server
  ↓
User authenticates
  ↓
Authorization Code + PKCE
  ↓
Access Token
```

Again, a **user** is involved.

---

### Client Credentials

```text
Client Application
      ↓
Authorization Server
      ↓
Access Token
      ↓
Resource Server
```

There is **no user**.

The client itself is being authenticated.

---

# 12.3 The Four OAuth Roles

You already learned the OAuth roles.

For Client Credentials:

### Client

The application/service requesting access.

Example:

```text
Order Service
```

### Authorization Server

The server that authenticates the client and issues the access token.

Example:

```text
Keycloak
Auth0
Okta
Microsoft Entra ID
```

### Resource Server

The API containing the protected resources.

Example:

```text
Payment API
```

### Resource Owner

Usually the user in OAuth.

But in Client Credentials Flow:

```text
There is no user involved.
```

The client is accessing resources **on its own behalf**.

---

# 12.4 Client ID and Client Secret

Before the flow can happen, the client needs to be registered with the Authorization Server.

For example:

```text
Client ID:

order-service
```

And a secret:

```text
Client Secret:

abc123-secret-xyz
```

Think of them roughly as:

```text
Client ID
    ↓
Who are you?

Client Secret
    ↓
Prove that you are really that client
```

The Authorization Server uses these credentials to authenticate the application.

---

# 12.5 Complete Flow

The complete flow looks like this:

```text
                    Authorization Server
                           │
                           │
              Client ID + Client Secret
                           │
                           │
                           ↓
                    Authenticate Client
                           │
                           ↓
                     Access Token
                           │
                           │
                           ↓
Client ─────────────────────────────→ Resource Server
                                      │
                                      │
                              Validate Access Token
                                      │
                                      ↓
                                 API Response
```

There is no browser redirect.

There is no login page.

There is no authorization code.

There is no PKCE.

---

# 12.6 Step 1 — Register the Client

First, the service is registered with the Authorization Server.

For example:

```text
Client:

order-service

Client ID:

order-service-123

Client Secret:

super-secret-value
```

You can also configure allowed scopes:

```text
payments:read
payments:write
orders:read
```

---

# 12.7 Step 2 — Request an Access Token

The client sends a request to the token endpoint.

For example:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&scope=payments:read
```

The client also authenticates itself.

One common approach is:

```http
Authorization: Basic base64(client_id:client_secret)
```

Conceptually:

```text
Client ID
    +
Client Secret
    ↓
Authorization Server
```

---

# 12.8 Why `grant_type=client_credentials`?

OAuth needs to know which flow the client is requesting.

So the request contains:

```text
grant_type=client_credentials
```

This tells the Authorization Server:

> "I am a machine/service. I want an access token using my client credentials."

Compare this with Authorization Code:

```text
grant_type=authorization_code
```

The grant type identifies the OAuth grant being used.

---

# 12.9 Step 3 — Authorization Server Authenticates the Client

The Authorization Server verifies:

```text
Client ID
+
Client Secret
```

For example:

```text
Client ID = order-service-123
           ↓
Does this client exist?

Client Secret = abc123
           ↓
Is the secret correct?

Requested scope = payments:read
           ↓
Is this client allowed to request this scope?
```

If everything is valid:

```text
Client authenticated
       +
Scope allowed
       ↓
Issue Access Token
```

---

# 12.10 Step 4 — Access Token Is Returned

The Authorization Server responds with something like:

```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "payments:read"
}
```

Important fields:

### `access_token`

The token the client will use when calling the API.

### `token_type`

Usually:

```text
Bearer
```

### `expires_in`

How long the token is valid.

For example:

```text
3600 seconds
```

means:

```text
1 hour
```

### `scope`

The permissions granted to the token.

---

# 12.11 Step 5 — Call the API

Now the Order Service can call the Payment API.

```http
POST /payments
Authorization: Bearer eyJhbGciOi...
```

The flow is now:

```text
Order Service
      │
      │ Bearer Access Token
      ↓
Payment API
      │
      ↓
Validate token
      │
      ↓
Allow / Deny
```

---

# 12.12 What Does the Resource Server Validate?

The Payment API needs to determine whether the token is valid.

Depending on the token implementation, it may verify:

```text
Signature
Expiration
Issuer
Audience
Scopes
```

For example:

```text
Token

sub = order-service-123
aud = payment-api
scope = payments:write
exp = future timestamp
```

The API can then determine:

```text
Is this token valid?
        ↓
Is it intended for me?
        ↓
Is it expired?
        ↓
Does it have payments:write?
        ↓
Allow request
```

---

# 12.13 There Is No Refresh Token

This is an important difference.

In the Client Credentials Flow, you generally don't get a refresh token.

Instead:

```text
Access Token expires
        ↓
Client requests another access token
        ↓
Authorization Server authenticates client
        ↓
New Access Token
```

So:

```text
Client Credentials

Access Token
     ↓
Expires
     ↓
Request new Access Token
```

Not:

```text
Access Token
     ↓
Refresh Token
     ↓
New Access Token
```

The reason is simple:

> The client already has its credentials and can authenticate again.

---

# 12.14 Client Credentials vs Authorization Code

This distinction is extremely important.

|                       | Authorization Code         | Client Credentials        |
| --------------------- | -------------------------- | ------------------------- |
| User involved         | Yes                        | No                        |
| User login            | Yes                        | No                        |
| User consent          | Usually                    | No                        |
| Client authentication | Yes                        | Yes                       |
| Authorization Code    | Yes                        | No                        |
| PKCE                  | Commonly used              | No                        |
| Access Token          | Yes                        | Yes                       |
| Refresh Token         | Possible                   | Typically no              |
| Typical use           | User accessing application | Service accessing service |

Think:

```text
Authorization Code

User → Application → API
```

versus:

```text
Client Credentials

Service → API
```

---

# 12.15 Real-World Example

Imagine an e-commerce system:

```text
Order Service
Payment Service
Inventory Service
Notification Service
```

When an order is created:

```text
Order Service
      │
      ├────→ Inventory Service
      │
      ├────→ Payment Service
      │
      └────→ Notification Service
```

There isn't necessarily a human user logging into each service.

Instead, services authenticate themselves.

For example:

```text
Order Service
      ↓
Client Credentials
      ↓
Access Token
      ↓
Inventory API
```

And:

```text
Order Service
      ↓
Client Credentials
      ↓
Access Token
      ↓
Payment API
```

This is one of the most common real-world uses of Client Credentials.

---

# 12.16 Client Credentials with Microservices

This becomes especially useful when you have many microservices.

For example:

```text
                 Authorization Server
                         │
              ┌──────────┼──────────┐
              │          │          │
              ↓          ↓          ↓
           Service A  Service B  Service C
              │          │          │
              ↓          ↓          ↓
           API A       API B       API C
```

Each service can have its own identity.

For example:

```text
order-service
payment-service
inventory-service
```

Each can have its own:

```text
client_id
client_secret
scopes
```

This is much better than sharing one global credential between all services.

---

# 12.17 Service Identity

One of the biggest concepts to understand here is:

> **Client Credentials provides an identity for the application/service, not for a human user.**

For example:

```text
User:

Rohan
```

is different from:

```text
Service:

order-service
```

Authorization Code can result in a token representing access associated with a user/client interaction.

Client Credentials represents:

```text
order-service
```

acting as itself.

This distinction becomes very important when designing microservice security.

---

# 12.18 Scopes Are Still Important

Client Credentials does not mean:

> "The service gets unlimited access."

The client can be restricted using scopes.

For example:

```text
order-service
```

might be allowed:

```text
payments:read
payments:create
```

but not:

```text
payments:refund
```

So:

```text
Client
   ↓
Requested scopes
   ↓
Authorization Server
   ↓
Allowed scopes
   ↓
Access Token
```

Example:

```json
{
  "scope": "payments:read payments:create"
}
```

The Payment API can then enforce these permissions.

---

# 12.19 Client Credentials Is NOT a Login Flow

This is worth emphasizing.

Suppose you have:

```text
My Backend
      ↓
Google API
```

Your backend can use Client Credentials **if that API supports this model**.

But this does NOT mean:

```text
User logged into Google
```

There is no user authentication happening.

The token represents:

```text
My Backend
```

not:

```text
A specific human user
```

This is one of the most important differences between OAuth flows.

---

# 12.20 Security of the Client Secret

The Client Secret is extremely sensitive.

It should be stored on the backend:

```text
Backend
   ↓
Environment Variable / Secret Manager
   ↓
Client Secret
```

Never put it into:

```text
Browser JavaScript
SPA
Mobile application
```

Why?

Because those environments cannot reliably keep a secret.

For example, if you put:

```text
client_secret=abc123
```

inside a React application, users can inspect the application and obtain it.

Therefore:

```text
Confidential Backend
        ↓
Client Secret
        ↓
Client Credentials
```

is the typical model.

---

# 12.21 Client Credentials and PKCE

You might wonder:

> "I learned PKCE. Why don't we use PKCE here?"

Because PKCE solves a different problem.

PKCE protects the **authorization code** exchanged in a user-based flow.

Client Credentials has:

```text
No user
No browser redirect
No authorization code
```

Therefore:

```text
PKCE
  ↓
Not needed for Client Credentials
```

The client authenticates directly with the Authorization Server.

---

# 12.22 Client Credentials Request — Complete Example

A typical request looks like:

```http
POST https://auth.example.com/oauth2/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic BASE64(client_id:client_secret)

grant_type=client_credentials&
scope=payments:write
```

Response:

```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "payments:write"
}
```

Then:

```http
POST https://api.example.com/payments
Authorization: Bearer eyJhbGciOi...
```

---

# 12.23 Complete Mental Model

You should be able to visualize the entire flow without looking at documentation:

```text
                 CLIENT
              Order Service
                   │
                   │
                   │ Client ID
                   │ Client Secret
                   ↓
          ┌─────────────────────┐
          │ Authorization       │
          │ Server              │
          └─────────────────────┘
                   │
                   │ Authenticate client
                   │ Check scopes
                   ↓
              Access Token
                   │
                   │
                   ↓
          ┌─────────────────────┐
          │ Resource Server     │
          │ Payment API         │
          └─────────────────────┘
                   │
                   │ Validate token
                   ↓
                Response
```

Notice what is **missing**:

```text
❌ User
❌ Login page
❌ Password
❌ Authorization Code
❌ PKCE
❌ User consent
```

---

# 12.24 Client Credentials vs API Keys

You may wonder:

> "If a service just needs to authenticate itself, why not use an API key?"

There is some overlap, but OAuth gives you a more standardized authorization model.

API key:

```text
Client
   ↓
API Key
   ↓
API
```

OAuth Client Credentials:

```text
Client
   ↓
Client Authentication
   ↓
Authorization Server
   ↓
Short-lived Access Token
   ↓
API
```

OAuth gives you concepts such as:

* Scopes
* Token expiration
* Centralized authorization
* Standard token issuance
* Separation between Authorization Server and Resource Server

So in larger systems, OAuth can provide a much richer security model.

---

# 12.25 Common Mistakes

### Mistake 1 — Thinking a user is involved

Wrong:

```text
User → Login → Client Credentials
```

Correct:

```text
Service → Authorization Server
```

---

### Mistake 2 — Putting the client secret in a frontend

Wrong:

```text
React App
    ↓
Client Secret
```

Correct:

```text
Backend
    ↓
Client Secret
```

---

### Mistake 3 — Thinking the access token represents a user

In Client Credentials:

```text
Access Token
      ↓
Represents the client/service
```

Not necessarily:

```text
Access Token
      ↓
Represents a human user
```

---

### Mistake 4 — Expecting a refresh token

Typically:

```text
Access Token expires
       ↓
Request another Access Token
```

---

# 12.26 What You Should Remember

If you remember only **five things**, remember these:

### 1. No user

```text
Client → Authorization Server
```

---

### 2. Used for machine-to-machine communication

```text
Service → Service
```

---

### 3. Client authenticates itself

Usually using:

```text
Client ID
+
Client Secret
```

---

### 4. Access token represents the client

```text
order-service
     ↓
Access Token
```

rather than a human user.

---

### 5. No PKCE / authorization code

Because there is:

```text
No browser
No user
No authorization code
```

---

# 12.27 Final Comparison

At this point, your three flows should look like this:

```text
AUTHORIZATION CODE

User
 ↓
Client
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Access Token
 ↓
API
```

```text
PKCE

User
 ↓
Client
 ↓
Authorization Server
 ↓
Authorization Code + PKCE
 ↓
Access Token
 ↓
API
```

```text
CLIENT CREDENTIALS

Client
 ↓
Client Authentication
 ↓
Authorization Server
 ↓
Access Token
 ↓
API
```

The easiest way to remember the purpose is:

```text
Authorization Code
        ↓
"Let this application access resources
on behalf of a user."

Client Credentials
        ↓
"Let this service access resources
as itself."
```

---

## Practical Exercise

Since you've already learned Authorization Code and PKCE, don't just read this one. Build a tiny M2M example.

Create:

```text
Order Service
      ↓
Authorization Server
      ↓
Payment API
```

Implement:

1. Register `order-service` as an OAuth client.
2. Give it a client ID and secret.
3. Request an access token using `client_credentials`.
4. Give the client a scope such as `payments:read`.
5. Call the Payment API with the access token.
6. Make the API reject requests without the token.
7. Make the API reject a token without the required scope.
8. Let the access token expire and request a new one.

If you can build this and explain **why there is no user, authorization code, PKCE, or refresh token**, you've understood the Client Credentials Flow properly.
