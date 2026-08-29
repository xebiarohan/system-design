# Phase 6 — API Security

**Goal:** Understand how to protect APIs from common attacks and how authentication mechanisms such as cookies, sessions, JWTs, and OAuth fit into API security.

At this point, you've already learned **authentication, tokens, JWT, OAuth 2.0, OIDC, and Identity Providers**.

Now we move from:

> **"How does a client prove who it is?"**

to:

> **"How do we securely expose and protect an API?"**

---

## 1. HTTPS / TLS

This is the foundation of API security.

Before authentication, tokens, JWTs, etc., you need to protect the **communication channel** between the client and server.

Without HTTPS:

```text
Client
   |
   |  HTTP
   |  Authorization: Bearer eyJ...
   ↓
Server
```

An attacker positioned on the network could potentially intercept the request.

With HTTPS:

```text
Client
   |
   |  HTTPS
   |  🔒 Encrypted
   ↓
Server
```

### What TLS provides

TLS provides three important properties:

### 1. Confidentiality

Attackers cannot easily read the data being transmitted.

```text
Client → 🔒 encrypted → Server
```

For example:

```http
Authorization: Bearer eyJhbGciOi...
```

is encrypted while traveling over the network.

---

### 2. Integrity

An attacker cannot silently modify the request or response without detection.

For example:

```text
Original:

amount=100
```

An attacker shouldn't be able to change it to:

```text
amount=10000
```

without the connection detecting tampering.

---

### 3. Server authentication

TLS certificates help the client verify that it is actually communicating with the intended server.

For example:

```text
https://api.example.com
```

The certificate helps establish that the server is really associated with `api.example.com`.

---

### HTTP vs HTTPS

```text
HTTP
 ↓
Data sent without TLS protection
 ↓
Unsafe for sensitive APIs
```

vs.

```text
HTTPS
 ↓
TLS encryption
 ↓
Confidentiality + Integrity + Server authentication
```

### Important rule

**Never send passwords, access tokens, refresh tokens, session cookies, or other sensitive information over plain HTTP.**

---

# 2. CORS

CORS stands for:

**Cross-Origin Resource Sharing**

This is primarily a **browser security mechanism**.

It becomes important when your frontend and backend are hosted on different origins.

For example:

```text
Frontend
https://app.example.com

        ↓ API request

Backend
https://api.example.com
```

These are different origins because the hostnames differ.

---

## What is an origin?

An origin consists of:

```text
scheme + host + port
```

For example:

```text
https://example.com:443
```

Changing any of these creates a different origin.

Examples:

```text
https://example.com
https://api.example.com
http://example.com
https://example.com:8080
```

These are different origins.

---

## Why does CORS exist?

Browsers enforce the **Same-Origin Policy**.

Imagine a user is logged into:

```text
https://bank.com
```

and then visits:

```text
https://evil.com
```

Without browser protections, `evil.com` could potentially make requests to `bank.com` using the user's browser context.

CORS allows the server to explicitly say which origins are allowed to interact with it from browser JavaScript.

Example:

```http
Access-Control-Allow-Origin: https://app.example.com
```

Meaning:

> JavaScript running from `app.example.com` is allowed to access this response.

---

## Important distinction

CORS does **not** authenticate users.

It does **not** protect your API from:

```text
curl
Postman
another backend
mobile application
```

CORS is mainly enforced by **web browsers**.

Think:

```text
CORS
 ↓
Browser security policy
```

not:

```text
CORS
 ↓
API authentication
```

---

# 3. CSRF

CSRF stands for:

**Cross-Site Request Forgery**

This is especially important when authentication uses **cookies**.

Suppose:

```text
User
 ↓
Login to bank.com
 ↓
Session cookie stored in browser
```

The browser automatically sends the cookie with requests to `bank.com`.

Now imagine the user visits:

```text
evil.com
```

That website attempts:

```http
POST https://bank.com/transfer
```

The browser may automatically attach the bank's authentication cookie.

The attacker's website is attempting to make the user's browser perform an authenticated action.

That's CSRF.

---

## Simplified attack

```text
User logs into bank.com
        ↓
Session Cookie
        ↓
User visits evil.com
        ↓
evil.com creates malicious request
        ↓
Browser sends request to bank.com
        ↓
Cookie may be attached
        ↓
Bank thinks request came from user
```

---

## CSRF protection

Common approaches include:

### CSRF tokens

Server generates a token:

```text
CSRF Token
```

The client must include it with state-changing requests.

```http
POST /transfer

X-CSRF-TOKEN: abc123
```

The server verifies it.

---

### SameSite cookies

Cookies can be configured using:

```http
SameSite=Strict
```

or

```http
SameSite=Lax
```

or

```http
SameSite=None; Secure
```

This controls when browsers send cookies in cross-site contexts.

---

## Important relationship

A common rule of thumb:

```text
Cookie-based authentication
        ↓
Think about CSRF
```

Whereas:

```text
Authorization: Bearer <token>
        ↓
CSRF is generally not the primary concern
```

because browsers do not automatically attach arbitrary `Authorization` headers to cross-site requests.

However, JWT/API-token systems still have plenty of other security concerns.

---

# 4. XSS

XSS stands for:

**Cross-Site Scripting**

XSS occurs when an attacker manages to inject malicious JavaScript into a web page that is then executed in another user's browser.

Example:

```html
<script>
    maliciousCode();
</script>
```

---

## Why XSS matters for authentication

Suppose you store an access token in:

```text
localStorage
```

and an attacker successfully executes JavaScript on your website.

That JavaScript may be able to read:

```javascript
localStorage.getItem("accessToken")
```

The attacker could potentially steal the token.

Then:

```text
XSS
 ↓
Steal access token
 ↓
Attacker
 ↓
API
 ↓
Authenticated as victim
```

That's why token storage and XSS protection are closely related.

---

## HttpOnly cookies

A cookie can be marked:

```http
HttpOnly
```

JavaScript cannot access an HttpOnly cookie using:

```javascript
document.cookie
```

So:

```text
XSS
 ↓
JavaScript
 ↓
Cannot directly read HttpOnly cookie
```

But remember:

**HttpOnly does not prevent XSS itself.**

An XSS vulnerability is still dangerous because malicious JavaScript may perform actions as the user through the browser.

---

## XSS prevention

Learn:

* Output encoding
* Input validation
* Content Security Policy (CSP)
* Framework escaping
* Avoiding unsafe HTML injection
* Safe handling of user-generated content

---

# 5. SameSite Cookies

SameSite is a cookie attribute that controls when browsers send cookies in cross-site requests.

Example:

```http
Set-Cookie: sessionId=abc123;
            Secure;
            HttpOnly;
            SameSite=Lax
```

There are three important values:

```text
Strict
Lax
None
```

---

## SameSite=Strict

Cookie is sent only in same-site contexts.

```text
example.com
    ↓
example.com

Cookie ✓
```

Cross-site situations are heavily restricted.

Security is stronger, but some application flows may become inconvenient.

---

## SameSite=Lax

A more practical default for many session cookies.

It allows some cross-site navigation while restricting many cross-site requests.

---

## SameSite=None

Cookie can be sent in cross-site contexts.

But it requires:

```text
Secure
```

So:

```http
SameSite=None; Secure
```

is required.

This is commonly relevant when legitimate cross-site cookie usage is necessary.

---

## Relationship between cookies and security

A typical secure session cookie might look like:

```http
Set-Cookie:
    sessionId=abc123;
    Secure;
    HttpOnly;
    SameSite=Lax
```

Each attribute has a purpose:

```text
Secure
  ↓
Only HTTPS

HttpOnly
  ↓
JavaScript cannot directly read cookie

SameSite
  ↓
Controls cross-site cookie sending
```

---

# 6. API Keys

API keys are one of the simplest ways to authenticate an API client.

Example:

```http
GET /api/products

X-API-Key: abc123
```

or:

```http
Authorization: ApiKey abc123
```

The server checks:

```text
API Key
   ↓
Valid?
   ↓
Yes → process request
No  → 401 Unauthorized
```

---

## Where API keys are useful

API keys are commonly useful for:

* Server-to-server integrations
* Internal services
* Developer APIs
* Identifying applications
* Simple integrations

Example:

```text
Your application
       ↓
X-API-Key
       ↓
Third-party API
```

---

## API keys vs OAuth

API keys generally represent an **application/client credential**.

OAuth can provide:

```text
User authorization
+
Scopes
+
Access tokens
+
Refresh tokens
+
Delegated permissions
```

For example:

```text
API Key

"Which application is calling me?"
```

while OAuth can answer:

```text
"Which application is calling?"
+
"On whose behalf?"
+
"What is it allowed to access?"
```

---

## API key security

Treat API keys like secrets.

Do not put sensitive API keys in:

```text
Git repositories
Frontend JavaScript
Public URLs
Logs
Screenshots
```

Use:

```text
Environment variables
Secret managers
Secure configuration
```

---

# 7. Rate Limiting

Rate limiting controls how many requests a client can make within a period.

For example:

```text
100 requests / minute
```

If the client exceeds the limit:

```http
HTTP/1.1 429 Too Many Requests
```

---

## Why rate limiting matters

### 1. Prevent abuse

```text
Attacker
   ↓
10 million requests
   ↓
API
   ↓
💥
```

Rate limiting reduces the impact.

---

### 2. Brute-force protection

For example:

```text
POST /login
```

An attacker shouldn't be able to try millions of passwords.

Rate limiting can enforce:

```text
5 login attempts / minute
```

---

### 3. Protect resources

Expensive endpoints may require stricter limits.

For example:

```text
GET /products
    ↓
1000 requests/minute

POST /generate-report
    ↓
10 requests/minute
```

---

## Common rate-limiting algorithms

Learn these:

* Fixed Window
* Sliding Window
* Token Bucket
* Leaky Bucket

For example, Token Bucket:

```text
        Tokens
      ┌─────────┐
      │ ● ● ● ● │
      └────┬────┘
           ↓
        Request
           ↓
      Token consumed
```

---

# 8. Authentication vs API Security

This distinction is **very important**.

Authentication is only one part of API security.

Think of API security as:

```text
                 API Security
                      |
        ┌─────────────┼──────────────┐
        ↓             ↓              ↓
 Authentication   Authorization   Transport
        |             |              |
     JWT/OAuth      Roles/Scopes     HTTPS
```

But there are additional layers:

```text
API Security
│
├── HTTPS/TLS
├── Authentication
├── Authorization
├── CORS
├── CSRF protection
├── XSS protection
├── Rate limiting
├── Input validation
├── Secure headers
├── Logging & monitoring
└── Secrets management
```

---

# 9. Cookies vs Sessions vs JWT vs OAuth

This is where everything you've learned starts coming together.

---

## Cookies

A cookie is a **browser storage/transmission mechanism**.

It is not itself an authentication protocol.

Example:

```http
Cookie: sessionId=abc123
```

Cookies can contain:

```text
Session ID
JWT
Other application data
```

---

## Sessions

Sessions are typically **server-side authentication state**.

```text
Browser
   ↓
Session Cookie
   ↓
Server
   ↓
Session Store
```

The cookie usually contains only an identifier:

```text
sessionId=abc123
```

The actual session data lives on the server.

---

## JWT

JWT is a **token format**.

Example:

```text
xxxxx.yyyyy.zzzzz
```

The token can contain claims:

```json
{
  "sub": "123",
  "role": "ADMIN",
  "exp": 1788000000
}
```

JWT does not automatically mean OAuth.

You can use JWT without OAuth.

---

## OAuth

OAuth is an **authorization framework**.

It defines how a client obtains authorization to access resources.

For example:

```text
User
 ↓
Authorization Server
 ↓
Access Token
 ↓
Client
 ↓
Resource Server
```

The access token could be a JWT, but it doesn't have to be.

---

# 10. When to Use Cookies

Cookies are particularly useful for:

```text
Browser
   ↓
Web application
   ↓
Backend
```

For example:

```text
React/Angular application
        ↓
Backend
        ↓
HttpOnly Secure Cookie
```

Advantages:

* Browser handles cookies automatically
* HttpOnly can prevent JavaScript from directly reading the cookie
* SameSite provides CSRF-related protection
* Works naturally with browser sessions

Need to think carefully about:

* CSRF
* SameSite
* XSS
* Secure flag
* Cookie scope

---

# 11. When to Use Sessions

Sessions are a good choice when:

```text
Browser
   ↓
Traditional web application
   ↓
Server
```

Example:

```text
Login
 ↓
Server creates session
 ↓
Session ID stored in cookie
 ↓
Browser sends cookie
 ↓
Server looks up session
```

Advantages:

* Easy revocation
* Server controls session state
* Simple logout
* Good fit for traditional web applications

Trade-off:

```text
Server
 ↓
Must maintain session state
```

---

# 12. When to Use JWT

JWTs are useful when you need a portable token containing claims.

For example:

```text
Client
   ↓
JWT
   ↓
API Gateway
   ↓
Microservice A
   ↓
Microservice B
```

Each service can verify the JWT signature without asking a central session store for every request.

This can be useful in distributed systems.

But remember:

> **JWT does not automatically make an application stateless, secure, or better than sessions.**

JWT introduces its own lifecycle and security considerations.

---

# 13. When to Use OAuth

Use OAuth when you need **delegated authorization**.

For example:

```text
User
 ↓
Application A
 ↓
Access Google resources
```

Instead of giving Application A the user's Google password:

```text
❌ Application A
      ↓
   Google password
```

OAuth provides:

```text
Application A
      ↓
OAuth Authorization
      ↓
Access Token
      ↓
Google API
```

OAuth is particularly important for:

* Third-party applications
* Delegated access
* Microservices
* APIs
* Enterprise identity systems

---

# 14. 401 vs 403

This is a very common API interview and real-world topic.

## 401 Unauthorized

Usually means:

> The request does not have valid authentication credentials.

Example:

```http
GET /api/profile

Authorization: Bearer invalid-token
```

Response:

```http
401 Unauthorized
```

Think:

```text
"Who are you?"
        ↓
"I don't know / your credentials aren't valid."
```

---

## 403 Forbidden

Authentication succeeded, but the client doesn't have permission.

Example:

```text
User = NORMAL_USER

GET /admin/users
```

Token is valid:

```text
Authentication ✓
```

Authorization fails:

```text
Permission ✗
```

Response:

```http
403 Forbidden
```

Think:

```text
"I know who you are."
        ↓
"But you're not allowed to do this."
```

---

# 15. API Security Request Flow

Let's put everything together.

Imagine:

```http
GET /api/orders
Authorization: Bearer eyJ...
```

The request travels like this:

```text
                HTTPS
Client ─────────────────────→ API
                              │
                              ↓
                       TLS connection
                              │
                              ↓
                    Authentication
                              │
                              ↓
                     Verify token
                              │
                              ↓
                    Authorization
                              │
                              ↓
                     Check scopes
                              │
                              ↓
                       Rate limiting
                              │
                              ↓
                       Business logic
                              │
                              ↓
                         Response
```

For a JWT-based OAuth API:

```text
Client
   │
   │ HTTPS
   │ Authorization: Bearer JWT
   ↓
API Gateway
   │
   ├── TLS ✓
   ├── Token validation ✓
   ├── Signature ✓
   ├── exp ✓
   ├── iss ✓
   ├── aud ✓
   ├── Scope ✓
   └── Rate limit ✓
          │
          ↓
      Backend API
```

---

# 16. A Real-World Example

Suppose you have:

```text
React Frontend
       ↓
API Gateway
       ↓
Order Service
       ↓
Payment Service
```

Authentication is handled using OAuth/OIDC.

The flow could be:

```text
User
 ↓
Identity Provider
 ↓
Login
 ↓
Access Token
 ↓
React
 ↓
Authorization: Bearer <access_token>
 ↓
API Gateway
```

The gateway validates:

```text
HTTPS ✓
Token signature ✓
Token not expired ✓
Issuer ✓
Audience ✓
Scopes ✓
Rate limit ✓
```

Then:

```text
API Gateway
     ↓
Order Service
```

The Order Service may perform additional authorization:

```text
Does user have:

orders:read
       ↓
      YES
       ↓
Return orders
```

---

# 17. Common API Security Mistakes

You should be able to recognize these immediately.

### ❌ Sending tokens over HTTP

```text
http://api.example.com
```

Use:

```text
https://api.example.com
```

---

### ❌ Putting secrets in frontend code

```javascript
const API_KEY = "super-secret-key";
```

Anything delivered to the browser should be considered accessible to the user.

---

### ❌ Storing sensitive tokens carelessly

For example:

```text
localStorage
```

can become particularly dangerous in the presence of XSS.

---

### ❌ Trusting JWT payload without verification

Never do:

```text
Decode JWT
 ↓
Read role=ADMIN
 ↓
Trust it
```

Instead:

```text
JWT
 ↓
Verify signature
 ↓
Validate claims
 ↓
Authorize
```

---

### ❌ Thinking CORS protects your API

CORS protects browser behavior.

It is **not an authentication mechanism**.

---

### ❌ Thinking authentication means authorization

Valid token:

```text
Authentication ✓
```

doesn't necessarily mean:

```text
Authorization ✓
```

A normal user may have a valid token but still be forbidden from:

```text
DELETE /admin/users/123
```

---

# 18. The Big Picture

At this stage, you should mentally separate the different security layers:

```text
                    API Security
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ↓                ↓                ↓
   Transport       Authentication    Authorization
        │                │                │
      HTTPS         OAuth/JWT        Roles/Scopes
      TLS           Sessions          Permissions
                         │
                         ↓
                  Request Protection
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
        CSRF            XSS          CORS
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                   Abuse Protection
                         │
                         ↓
                  Rate Limiting
```

---

# 19. What You Should Be Able to Explain

After completing this topic, you should confidently answer:

### HTTPS/TLS

* Why should APIs use HTTPS?
* What does TLS protect?
* What are confidentiality, integrity, and authentication?

### CORS

* What is an origin?
* Why does CORS exist?
* Is CORS authentication?
* Does CORS protect APIs from Postman/curl?

### CSRF

* What is CSRF?
* Why are cookies vulnerable to CSRF?
* How do SameSite cookies help?
* What is a CSRF token?

### XSS

* What is XSS?
* Why is XSS dangerous for token-based authentication?
* Why does HttpOnly help?
* Does HttpOnly prevent XSS?

### Cookies

* What do `Secure`, `HttpOnly`, and `SameSite` mean?
* When should cookies be used?

### API Keys

* What is an API key?
* When should API keys be used?
* How are API keys different from OAuth access tokens?

### Rate Limiting

* Why do APIs need rate limiting?
* What does HTTP `429` mean?
* What are Token Bucket and Sliding Window?

### Authentication vs Authorization

You should be able to explain:

```text
Authentication
      ↓
"Who are you?"

Authorization
      ↓
"What are you allowed to do?"
```

---

# 20. Most Important Mental Model

Don't think of API security as one technology.

It's a **collection of layers**:

```text
HTTPS
  ↓
Protect communication

Authentication
  ↓
Identify the caller

Authorization
  ↓
Determine permissions

CORS / CSRF / XSS
  ↓
Protect browser-based applications

Rate Limiting
  ↓
Protect against abuse

Input Validation
  ↓
Protect application logic

Logging / Monitoring
  ↓
Detect attacks
```

And your OAuth knowledge fits into this architecture:

```text
                 Identity Provider
                        │
                        │ OAuth / OIDC
                        ↓
                   Access Token
                        │
                        ↓
Client ───────────────→ API
          HTTPS         │
                        ↓
                 Token Validation
                        │
                        ↓
                   Authorization
                        │
                        ↓
                    API Logic
```

### The key takeaway

**OAuth/JWT solves only part of API security.**

A production API still needs:

```text
HTTPS
+ Authentication
+ Authorization
+ CORS/CSRF/XSS considerations
+ Secure cookies/token handling
+ Rate limiting
+ Input validation
+ Monitoring
```

That's the big picture you want to have in your head before moving to **Phase 7 — Token Lifecycle**.
