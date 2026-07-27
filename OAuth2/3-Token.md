This is one of the most important topics in modern backend development. Once you understand **what a token is**, JWT, OAuth, OpenID Connect, and API security become much easier.

---

# Topic 5: What is a Token?

## Definition

A **token** is a piece of data issued by an authentication system that represents a user's identity and/or permissions.

Instead of sending your username and password with every request, you authenticate **once**, receive a token, and then send that token with future requests.

Think of it like a **movie ticket**.

---

### Without a token

Every time you want to enter a movie hall:

```
Customer
    ↓
Show Aadhaar
Show PAN
Show payment receipt
Verify identity
```

This would be slow and repetitive.

---

### With a token

After buying the ticket:

```
Customer
    ↓
Movie Ticket
    ↓
Enter Hall
```

The ticket proves you've already been verified.

A token works the same way.

---

# Real Example

Suppose you log into Amazon.

```
Username
Password
```

Amazon verifies your credentials.

Instead of asking for your password every time you click:

* Orders
* Wishlist
* Cart
* Profile

Amazon gives your browser a token.

Now every request looks like

```
GET /orders

Authorization: Bearer eyJhbGciOiJIUzI1Ni...
```

The server verifies the token.

If valid:

```
User = Rohan
```

No password required again.

---

# Why Do We Need Tokens?

Imagine there were no tokens.

Every API request would look like

```
POST /orders

Username: rohan
Password: abc123
```

Problems:

* Password travels over the network repeatedly.
* If intercepted, the attacker gets the actual password.
* Database lookup is needed on every request.
* Very inefficient.

Instead:

```
Login once
↓

Receive token
↓

Use token everywhere
```

---

# Authentication Flow

```
           Login
              │
              ▼
     Username + Password
              │
              ▼
      Authentication Server
              │
      Verify credentials
              │
              ▼
       Generate Token
              │
              ▼
          Client stores it
              │
              ▼
    Send token with every request
```

---

# What Does a Token Contain?

Depending on the implementation, it may contain:

```
User ID

Username

Roles

Permissions

Expiration Time

Issuer

Audience
```

For example:

```
{
    "userId": 104,
    "role": "ADMIN",
    "expires": "2026-12-31"
}
```

For a JWT, this information is encoded and digitally signed.

---

# Why Not Just Store User ID?

Imagine the client sends:

```
GET /profile

UserId: 104
```

What's stopping an attacker from changing it?

```
UserId: 105
```

Now they become another user.

A token prevents this because it is digitally signed.

If modified:

```
Token Verification

↓

FAILED
```

The request is rejected.

---

# Stateless Authentication

This is one of the biggest reasons tokens became popular.

## Session-based authentication

```
Browser
     │
Cookie(SessionID)
     │
Server
     │
Session Store
```

The server stores login information.

Example:

```
Session ID

ABC123
```

Server memory:

```
ABC123

↓

User 104

↓

Role ADMIN
```

Every request requires looking up the session.

---

## Token-based authentication

```
Browser

↓

Token

↓

Server
```

The server simply verifies the token.

No database lookup is required (assuming the token contains the needed claims and hasn't been revoked).

The server doesn't need to remember anything.

Hence:

```
Stateless Authentication
```

The token itself carries the necessary information.

---

# Sessions vs Tokens

| Sessions                        | Tokens                                            |
| ------------------------------- | ------------------------------------------------- |
| Server stores session           | Client stores token                               |
| Stateful                        | Stateless                                         |
| Requires session store          | No session store required                         |
| Easy logout                     | Logout is more complex                            |
| Common for traditional web apps | Common for APIs, SPAs, mobile apps, microservices |

---

# What is a Bearer Token?

This is often misunderstood.

A bearer token means:

> Whoever **bears (possesses)** the token can use it.

The server doesn't ask:

* password
* PIN
* fingerprint

It simply checks:

```
Is this token valid?
```

If yes:

```
Access Granted
```

This is why bearer tokens **must be protected**. If an attacker steals one before it expires, they can often use it as if they were the legitimate user.

The HTTP header looks like:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

The word `Bearer` is an authentication scheme defined by the HTTP authentication framework.

---

# Access Token

This is the token sent with every API request.

Example:

```
Login

↓

Access Token

↓

GET /orders

↓

GET /profile

↓

GET /cart
```

Characteristics:

* Short-lived (often 5–30 minutes, though it varies by system)
* Sent on every API request
* Contains identity and authorization claims
* If compromised, the attacker only has access until it expires

---

# Refresh Token

A refresh token is **not** sent with every API request.

It exists to obtain a new access token after the current one expires.

```
Login

↓

Access Token (15 min)

+

Refresh Token (30 days)

↓

Access Token expires

↓

Use Refresh Token

↓

Receive New Access Token
```

This improves security by keeping access tokens short-lived while avoiding frequent logins.

You'll study refresh tokens in depth later in your roadmap.

---

# Complete Example

```
Step 1

Client

↓

POST /login

↓

username
password

--------------------------------

Step 2

Server verifies password

--------------------------------

Step 3

Server returns

Access Token

Refresh Token

--------------------------------

Step 4

Client stores them

--------------------------------

Step 5

GET /orders

Authorization: Bearer ACCESS_TOKEN

--------------------------------

Step 6

Server verifies token

↓

Valid

↓

Return Orders

--------------------------------

Step 7

15 minutes later

Access Token expired

--------------------------------

Step 8

POST /refresh

Refresh Token

↓

New Access Token

--------------------------------

Step 9

Continue making API requests
```

---

# Why Did the Industry Move from Sessions to Tokens?

As applications evolved, architectures changed:

```
Web Browser
↓

One Server
```

became

```
Web
Mobile
Tablet
Smart TV

↓

API Gateway

↓

20 Microservices
```

With sessions:

* Every server needs access to the shared session store.
* Scaling requires distributed session management.
* Cross-service authentication becomes harder.

With tokens:

* Any service that can verify the token can authenticate the user.
* No shared session store is needed for normal request processing.
* This fits distributed systems and microservices much better.

---

# Common Interview Questions

### 1. Why do we use tokens instead of sending passwords?

Because the user authenticates once, receives a signed token, and then uses that token for subsequent requests. This avoids repeatedly sending credentials and improves both security and efficiency.

---

### 2. What is stateless authentication?

The server does not store session information. Authentication is performed by validating the token supplied with each request.

---

### 3. What is a bearer token?

A bearer token grants access to whoever possesses it. If it is stolen, it can typically be used until it expires or is revoked.

---

### 4. Can a token be modified by the client?

A client can change the token's contents, but if it's a signed token like a JWT, the signature verification will fail and the server will reject it.

---

### 5. Is a token encrypted?

Not necessarily. A JWT is typically **encoded** (Base64URL) and **digitally signed**, but its payload is usually **not encrypted**. Anyone holding the token can decode the payload, but they cannot modify it without invalidating the signature. Encryption is a separate concept (e.g., JWE), which you'll encounter later in your roadmap.

---

## Key Takeaways

* **Token**: A credential issued after successful authentication.
* **Bearer token**: Anyone who possesses it can use it, so it must be protected.
* **Access token**: Used on every API request; typically short-lived.
* **Refresh token**: Used only to obtain a new access token after expiration.
* **Stateless authentication**: The server validates the token instead of looking up a server-side session.
* **Sessions vs tokens**: Sessions are stateful and server-managed; tokens are client-held and better suited for APIs and distributed systems.

Once you're comfortable with these concepts, the next topic—**HTTP Authentication and the `Authorization: Bearer <token>` header**—will feel much more intuitive.
