# 6. API Authentication (Detailed)

This topic is the bridge between **authentication** (proving identity) and **JWT/OAuth**. Once you understand API authentication, JWTs and OAuth become much easier.

---

# What is API Authentication?

API authentication is the process of **proving to an API that the client making the request is allowed to use it.**

Unlike websites, APIs don't have a user interface. They only receive HTTP requests.

Example:

```
Mobile App
      │
      │ HTTP Request
      ▼
REST API
```

The API has to answer:

> "Who sent this request?"

If it cannot identify the client, it should reject the request.

---

# Why APIs Need Authentication

Imagine a banking API.

Endpoints:

```
GET /balance

POST /transfer

GET /transactions
```

Without authentication, anyone could call

```
GET /balance
```

and view your account.

Authentication prevents this.

---

# Client and Server

Every API authentication system has two sides.

```
Client
    │
    │ Request
    ▼
Server(API)
```

The client could be:

* Web application
* Mobile app
* Backend server
* Desktop application
* IoT device

The server checks whether the client is authenticated.

---

# HTTP Requests Refresher

Every API request looks like:

```
GET /users/profile HTTP/1.1

Host: api.example.com

Authorization: Bearer eyJhbGciOi...
```

The request contains:

* Method
* URL
* Headers
* Body (optional)

Authentication information is usually sent in **headers**.

---

# What are HTTP Headers?

Headers are metadata about the request.

Example:

```
GET /profile

Host: api.example.com

User-Agent: Chrome

Accept: application/json

Authorization: Bearer TOKEN
```

Each header gives extra information.

Examples:

```
Host
```

Which server to contact.

```
Content-Type
```

What format the body uses.

```
Authorization
```

Who is making the request.

---

# The Authorization Header

The most common authentication header is

```
Authorization
```

Example:

```
Authorization: Bearer abc123
```

This tells the server:

> "Here is my authentication token."

The server reads this header before processing the request.

---

# Authentication Flow

Suppose you log in.

```
POST /login

Email
Password
```

Server checks credentials.

If correct:

```
Access Token
```

is returned.

Now every request includes it.

```
GET /profile

Authorization: Bearer TOKEN
```

---

Complete flow:

```
User

↓

Login

↓

Server verifies password

↓

Server issues token

↓

Client stores token

↓

Client sends token

↓

API verifies token

↓

API returns data
```

---

# What is a Bearer Token?

A bearer token is simply a token that grants access to whoever **bears (possesses)** it.

Example:

```
Authorization: Bearer eyJhbGc...
```

Think of it like a movie ticket.

The ticket doesn't know who is holding it.

Anyone holding it can enter.

That is why bearer tokens **must be protected**.

---

# Why is it Called "Bearer"?

Because the server assumes:

```
If you possess this token,

you are authenticated.
```

No password is sent with every request.

Instead:

```
Password

↓

Login once

↓

Receive token

↓

Use token repeatedly
```

---

# Example Request

Without authentication:

```
GET /profile
```

Server:

```
401 Unauthorized
```

With authentication:

```
GET /profile

Authorization: Bearer TOKEN
```

Server:

```
200 OK
```

---

# How the Server Verifies the Token

When a request arrives:

```
GET /orders

Authorization: Bearer TOKEN
```

Server:

1. Read Authorization header.
2. Extract token.
3. Verify token.
4. Check expiration.
5. Identify user.
6. Process request.

Diagram:

```
Request

↓

Authorization Header

↓

Extract Token

↓

Verify Token

↓

Authenticated User

↓

Return Response
```

---

# HTTP Status Codes

## 200 OK

Authenticated successfully.

```
200 OK
```

---

## 401 Unauthorized

Authentication failed.

Reasons:

* No token
* Invalid token
* Expired token

Example:

```
401 Unauthorized
```

---

## 403 Forbidden

Authenticated, but not allowed.

Example:

```
User is logged in

↓

Access admin route

↓

403 Forbidden
```

Difference:

```
401

↓

You are NOT authenticated.
```

vs

```
403

↓

You ARE authenticated

but

You cannot access this resource.
```

---

# Bearer Authentication Example

Request:

```
GET /users/me

Authorization: Bearer abcxyz123
```

Server:

```
Validate token

↓

User = Alice

↓

Return profile
```

Response:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

---

# Token Verification

The server checks:

```
Token exists?

↓

Signature valid?

↓

Expired?

↓

Revoked?

↓

User exists?
```

If every check passes:

```
Authenticated
```

---

# Stateless Authentication

One major reason tokens became popular is that they support **stateless authentication**.

With sessions:

```
Browser

↓

Session ID

↓

Server Memory
```

The server must store session data.

With tokens:

```
Browser

↓

Token

↓

Server verifies token

↓

No session lookup
```

The token contains enough information (or references it) for the server to validate the request.

Benefits:

* Easier to scale
* Works well with microservices
* No centralized session store required (for self-contained tokens like JWTs)

---

# Sessions vs Bearer Tokens

## Session Authentication

```
Browser

↓

Cookie

↓

Server

↓

Session Database
```

Server stores login state.

---

## Bearer Authentication

```
Client

↓

Bearer Token

↓

Server verifies token

↓

No session database required
```

The authentication data travels with the request.

---

# Where Does the Token Come From?

Usually from:

```
POST /login
```

Server response:

```json
{
  "access_token": "eyJhbGc..."
}
```

Client stores it.

Future requests:

```
Authorization: Bearer eyJhbGc...
```

---

# Example API

### Login

```
POST /login
```

Body:

```json
{
    "email":"alice@example.com",
    "password":"password123"
}
```

Response:

```json
{
    "access_token":"eyJhbGc..."
}
```

---

### Protected Endpoint

```
GET /profile
```

Header:

```
Authorization: Bearer eyJhbGc...
```

Response:

```json
{
    "name":"Alice"
}
```

---

# Common Authentication Methods for APIs

## 1. API Key

```
x-api-key: abc123
```

Used for:

* Public APIs
* Service identification
* Simple integrations

Not ideal for user authentication because it usually identifies an application, not an individual user.

---

## 2. Basic Authentication

```
Authorization: Basic base64(username:password)
```

The username and password are Base64-encoded (not encrypted), so **HTTPS is essential**. It's simple but generally not recommended for modern user-facing APIs.

---

## 3. Bearer Token

```
Authorization: Bearer TOKEN
```

Most common for modern REST APIs.

---

## 4. Mutual TLS (mTLS)

Both client and server authenticate each other using TLS certificates. This is common in enterprise environments and service-to-service communication where stronger identity guarantees are required.

---

# Common Mistakes

### Sending Tokens in URLs

❌ Bad:

```
GET /profile?token=abc123
```

URLs can end up in browser history, logs, and analytics systems.

Use:

```
Authorization: Bearer TOKEN
```

---

### Using HTTP Instead of HTTPS

Never send tokens over plain HTTP.

Always use:

```
HTTPS
```

HTTPS encrypts the request while it's in transit, protecting the token from interception.

---

### Forgetting Expiration

Access tokens should be short-lived.

Example:

```
Access Token

15 minutes
```

Long-lived tokens increase the impact if they are stolen.

---

# Real-World Example

Suppose you open a food delivery app.

### Step 1

Login:

```
Email

Password
```

---

### Step 2

Server:

```
Credentials correct

↓

Generate Access Token
```

---

### Step 3

Client stores token.

---

### Step 4

Client requests:

```
GET /orders

Authorization: Bearer TOKEN
```

---

### Step 5

API verifies the token and returns your orders.

---

# How This Fits into Your Roadmap

API Authentication is the foundation for the next topics:

```
Sessions
        ↓
Tokens
        ↓
API Authentication
        ↓
JWT
        ↓
JWT Signing
        ↓
JWT Security
        ↓
OAuth 2.0
```

After mastering this topic, you'll already understand:

* Why the `Authorization` header exists.
* What a Bearer token is.
* How clients authenticate to APIs.
* Why APIs return `401 Unauthorized` and `403 Forbidden`.
* Why HTTPS is mandatory.
* The difference between session-based and token-based authentication.

That sets you up to dive into **JWT**, where you'll learn what the token actually contains, how it's signed, and how servers verify it securely.
