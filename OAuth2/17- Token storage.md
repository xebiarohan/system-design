# Phase 8 — Token Storage

**Goal:** Understand where authentication tokens should be stored in browser-based applications and the security trade-offs of each storage mechanism.

This topic is extremely important because **having a secure JWT is not enough**. If an attacker can steal the token from the browser, they can often use that valid token as the user.

The central question is:

> **If my application has an access token or refresh token, where should I put it?**

---

## 1. Why Token Storage Matters

Suppose your application performs this flow:

```text
User Login
    ↓
Authorization Server
    ↓
Access Token
    ↓
Browser
    ↓
API
```

The access token is effectively a **credential**.

For example:

```text
eyJhbGciOiJSUzI1NiIs...
```

If an attacker obtains that token:

```text
Attacker
    ↓
Stolen Access Token
    ↓
API
```

the API may treat the attacker as the legitimate user until the token expires or is otherwise invalidated.

Therefore:

```text
Secure Token
       +
Secure Storage
       +
Secure Transmission
       =
Better Authentication Security
```

Token storage is especially important for browser applications because JavaScript running in the application can potentially access browser storage.

The current OAuth guidance for browser-based applications recommends avoiding persistent browser storage such as `localStorage` for tokens and considers **in-memory storage** or, preferably for many architectures, a **Backend-for-Frontend (BFF)** using `HttpOnly` cookies. ([RFC Editor][1])

---

# 2. Types of Browser Storage

The main mechanisms you should understand are:

```text
Browser
│
├── Cookies
│
├── Memory
│
├── localStorage
│
└── sessionStorage
```

They have very different security characteristics.

---

# 3. HttpOnly Cookies

An **HttpOnly cookie** is a cookie that JavaScript cannot read.

Example:

```http
Set-Cookie: __Host-session=abc123;
             Secure;
             HttpOnly;
             SameSite=Strict;
             Path=/
```

The browser stores the cookie.

When making a request to the appropriate server:

```text
Browser
   │
   │ Cookie: __Host-session=abc123
   ↓
Server
```

JavaScript does **not** need to manually read the cookie.

The `HttpOnly` attribute prevents JavaScript from accessing the cookie through APIs such as `document.cookie`. ([MDN Web Docs][2])

---

## Why HttpOnly is important

Imagine you have:

```javascript
const token = localStorage.getItem("access_token");
```

An XSS vulnerability could allow malicious JavaScript to execute:

```javascript
localStorage.getItem("access_token");
```

and steal the token.

With:

```http
HttpOnly
```

JavaScript cannot directly read the cookie.

Therefore:

```text
XSS
 ↓
JavaScript executes
 ↓
Cannot read HttpOnly cookie
```

This significantly reduces the ability to **exfiltrate the credential itself**. ([OWASP Cheat Sheet Series][3])

But there is an important nuance:

> `HttpOnly` does **not** make an XSS vulnerability harmless.

If malicious JavaScript executes inside your application, the browser may still send the cookie along with requests initiated by that script. `HttpOnly` primarily protects the **confidentiality of the cookie value**. ([OWASP Cheat Sheet Series][3])

---

# 4. Secure Cookies

`Secure` means:

> Send the cookie only over HTTPS.

Example:

```http
Set-Cookie: session=abc123;
             Secure;
             HttpOnly;
             SameSite=Strict;
```

Without `Secure`, a sensitive cookie could potentially be transmitted over an insecure HTTP connection.

With:

```text
Secure
```

the browser restricts the cookie to HTTPS requests, helping protect against network interception. ([MDN Web Docs][2])

So for authentication cookies, you generally want:

```text
Secure
+
HttpOnly
```

---

# 5. SameSite Cookies

`SameSite` controls when the browser sends a cookie with cross-site requests.

The main values are:

```text
Strict
Lax
None
```

---

## SameSite=Strict

```http
SameSite=Strict
```

The browser is highly restrictive about sending the cookie in cross-site contexts.

This provides strong protection against certain CSRF scenarios.

---

## SameSite=Lax

```http
SameSite=Lax
```

This is less restrictive and allows some cross-site navigation scenarios.

It is commonly useful when `Strict` would interfere with legitimate application flows.

---

## SameSite=None

```http
SameSite=None; Secure
```

This allows the cookie to be sent in cross-site contexts.

`SameSite=None` requires `Secure`. ([MDN Web Docs][2])

---

## Important

`SameSite` is **not a complete CSRF defense**.

Think of it as:

```text
SameSite
    +
CSRF protection
    +
Origin/Referer validation where appropriate
```

rather than:

```text
SameSite = CSRF completely solved
```

OWASP specifically recommends treating SameSite as defense-in-depth rather than automatically replacing CSRF protection. ([OWASP Cheat Sheet Series][3])

---

# 6. Browser Memory

Another option is to keep the access token only in application memory.

For example:

```javascript
let accessToken = null;
```

Or in a React application:

```text
React Application
       ↓
Application Memory
       ↓
Access Token
```

The token is not stored in:

```text
localStorage
sessionStorage
Cookie
IndexedDB
```

Instead, it exists only while the application is running.

---

## Advantage

If the page is refreshed:

```text
Page Refresh
     ↓
JavaScript memory cleared
     ↓
Access token disappears
```

This reduces the lifetime of the stored token.

The OAuth browser-app guidance specifically describes in-memory storage as limiting exposure to the current execution context, although it also notes that this does not eliminate the risks of malicious JavaScript running in the application. ([RFC Editor][1])

---

## Disadvantage

The user may have to obtain a new access token after:

```text
Page refresh
Browser restart
Application reload
```

A common approach is therefore:

```text
Access Token
     ↓
Memory
```

and use a separate mechanism to obtain a new access token when necessary.

For example:

```text
Page loads
   ↓
Application authenticates
   ↓
Access Token
   ↓
Memory
```

---

# 7. localStorage

`localStorage` provides persistent browser storage.

Example:

```javascript
localStorage.setItem(
    "access_token",
    token
);
```

The token remains available after page reloads.

```text
Browser
   ↓
localStorage
   ↓
access_token
```

---

## Advantage

Very easy to use.

```javascript
const token =
    localStorage.getItem("access_token");
```

The token survives:

```text
Page refresh
     ↓
Token still exists
```

---

## Major Security Problem

JavaScript can read `localStorage`.

Therefore:

```text
Application
    ↓
JavaScript
    ↓
localStorage
    ↓
Access Token
```

If an attacker manages to execute JavaScript through an XSS vulnerability:

```text
XSS
 ↓
Malicious JavaScript
 ↓
localStorage.getItem(...)
 ↓
Token stolen
```

This is why OWASP recommends **not storing authentication tokens, session IDs, JWTs, refresh tokens, or other credentials in `localStorage`**. ([OWASP Cheat Sheet Series][3])

---

# 8. sessionStorage

`sessionStorage` is similar to `localStorage`, but its lifetime is associated with the browser tab.

Example:

```javascript
sessionStorage.setItem(
    "access_token",
    token
);
```

Conceptually:

```text
Tab A
 └── sessionStorage
       └── token


Tab B
 └── separate sessionStorage
```

It is not shared between multiple tabs in the same way `localStorage` is. ([RFC Editor][1])

---

## Advantage

The token does not normally persist after the tab's lifetime.

---

## Problem

The fundamental security issue remains:

```text
JavaScript
    ↓
sessionStorage
    ↓
Token
```

Therefore, XSS can still access it.

So:

```text
localStorage
     ❌

sessionStorage
     ❌
```

are generally poor choices for storing authentication credentials in browser applications. OWASP explicitly advises against storing authentication tokens in either. ([OWASP Cheat Sheet Series][3])

---

# 9. localStorage vs sessionStorage

| Feature                           | localStorage | sessionStorage |
| --------------------------------- | ------------ | -------------- |
| JavaScript accessible             | Yes          | Yes            |
| Survives page reload              | Yes          | Yes            |
| Survives tab close                | Usually yes  | No             |
| Shared across tabs                | Yes          | No             |
| Vulnerable to token theft via XSS | Yes          | Yes            |
| Recommended for auth tokens       | ❌            | ❌              |

The important distinction is **lifetime and sharing**, not protection against XSS.

---

# 10. Cookie vs localStorage

This is one of the most important comparisons.

### localStorage

```text
JavaScript
    ↓
localStorage
    ↓
Token
```

JavaScript can directly access the token.

---

### HttpOnly Cookie

```text
Browser
   ↓
HttpOnly Cookie
   ↓
Server
```

JavaScript cannot directly read the cookie.

Therefore:

```text
localStorage

Token
  ↑
  │
JavaScript can read
```

versus:

```text
HttpOnly Cookie

Token
  ↑
  │
JavaScript cannot directly read
```

That difference is extremely important when defending against token exfiltration through XSS. ([OWASP Cheat Sheet Series][3])

---

# 11. But Cookies Introduce CSRF

There is a catch.

Cookies are automatically attached by the browser when their cookie rules match the request.

For example:

```text
Browser
   │
   │ Cookie automatically attached
   ↓
https://api.example.com
```

This means cookies introduce an important concern:

```text
CSRF
```

For example:

```text
Attacker Website
      ↓
Victim's Browser
      ↓
Request to Your API
      ↓
Cookie automatically included
```

Therefore, cookie-based authentication should be designed together with CSRF protections and appropriate `SameSite` settings. ([OWASP Cheat Sheet Series][3])

This gives us an important trade-off:

```text
localStorage

+ JavaScript accessible
- Easier token theft through XSS


HttpOnly Cookie

+ JavaScript cannot read token
- Browser automatically sends cookie
- Must consider CSRF
```

---

# 12. The Important Difference: Token Storage vs Authentication Architecture

This is where many developers get confused.

You might hear:

> "JWT should be stored in HttpOnly cookies."

That statement is incomplete.

There are actually different architectures.

### Architecture A — SPA directly uses access token

```text
React / Angular
       ↓
OAuth Authorization Server
       ↓
Access Token
       ↓
Browser Memory
       ↓
API
Authorization: Bearer TOKEN
```

The browser application directly handles the access token.

Modern OAuth guidance recommends **Authorization Code + PKCE** for browser applications, with tokens preferably kept in memory rather than persistent browser storage. ([oauth.net][4])

---

### Architecture B — Backend-for-Frontend (BFF)

A stronger architecture for many applications is:

```text
Browser
   ↓
HttpOnly Cookie
   ↓
BFF
   ↓
Access Token
   ↓
API
```

Here:

```text
Browser
    ❌ does not need access token

BFF
    ✅ holds/uses access token
```

The browser instead has a session represented by an `HttpOnly` cookie.

The BFF communicates with the authorization/resource servers on behalf of the browser.

This can keep OAuth tokens entirely out of browser JavaScript. ([RFC Editor][1])

---

# 13. Recommended Cookie Configuration

For a sensitive authentication/session cookie, you will commonly see something like:

```http
Set-Cookie: __Host-session=abc123;
             Path=/;
             Secure;
             HttpOnly;
             SameSite=Strict
```

Meaning:

```text
__Host-
   ↓
Restrict cookie to the host

Secure
   ↓
HTTPS only

HttpOnly
   ↓
JavaScript cannot read it

SameSite=Strict
   ↓
Restrict cross-site sending

Path=/
   ↓
Available across the host's paths
```

The `__Host-` prefix has additional browser-enforced requirements: `Secure`, no `Domain`, and `Path=/`. ([MDN Web Docs][2])

---

# 14. Access Token vs Refresh Token Storage

Do not automatically treat both tokens identically.

Remember:

```text
Access Token
    ↓
Used frequently
    ↓
Usually short-lived
```

while:

```text
Refresh Token
    ↓
Used to obtain new access tokens
    ↓
Usually longer-lived
    ↓
More sensitive
```

Therefore, a stolen refresh token can be particularly dangerous.

A common secure architecture is:

```text
Browser
   │
   │ HttpOnly Cookie
   ↓
Backend / BFF
   │
   ├── Access Token
   │
   └── Refresh Token
```

The browser does not need to directly handle either OAuth token.

The OAuth browser-app guidance specifically discusses architectures where persistent browser token storage is unnecessary because the browser can use a cookie-based session to obtain access tokens from a backend. ([RFC Editor][1])

---

# 15. Token Storage Comparison

| Storage               |                       JS Can Read? |         Persists? |                      XSS Token Theft |        CSRF Concern | General Recommendation               |
| --------------------- | ---------------------------------: | ----------------: | -----------------------------------: | ------------------: | ------------------------------------ |
| `localStorage`        |                                  ✅ |                 ✅ |                              🔴 High |              🟢 Low | ❌ Avoid for auth tokens              |
| `sessionStorage`      |                                  ✅ |      Tab lifetime |                              🔴 High |              🟢 Low | ❌ Avoid for auth tokens              |
| Memory                |                 ✅ Application code |                 ❌ |                    🟠 Depends on XSS |              🟢 Low | ✅ Good for short-lived access tokens |
| HttpOnly Cookie       |                                  ❌ |           Depends | 🟢 Better against token exfiltration |              🔴 Yes | ✅ Strong option                      |
| BFF + HttpOnly Cookie | ❌ Browser doesn't hold OAuth token | Session dependent |                            🟢 Strong | 🟠 Must handle CSRF | ⭐ Excellent architecture             |

The table is a simplification: no storage mechanism makes an XSS-compromised application completely safe. An attacker who can execute code as your application may still be able to make authenticated requests or interact with your backend. ([RFC Editor][1])

---

# 16. Why "JWT in localStorage" Is a Bad Default

You will frequently see code like:

```javascript
localStorage.setItem(
    "jwt",
    accessToken
);
```

followed by:

```javascript
const token =
    localStorage.getItem("jwt");

fetch("/api/orders", {
    headers: {
        Authorization: `Bearer ${token}`
    }
});
```

It works.

But security-wise:

```text
XSS
 ↓
JavaScript executes
 ↓
localStorage.getItem("jwt")
 ↓
JWT stolen
 ↓
Attacker
 ↓
API
```

The problem isn't that JWT itself is insecure.

The problem is:

```text
JWT
+
Unsafe Storage
=
Credential Theft Risk
```

JWT signing protects the **integrity** of the token.

It does not protect the token from being stolen from browser storage.

---

# 17. A Better SPA Approach

For a browser SPA such as:

```text
React
Angular
Vue
```

a modern approach is:

```text
Authorization Code + PKCE
          ↓
     Access Token
          ↓
       Memory
          ↓
Authorization: Bearer TOKEN
          ↓
          API
```

When the page reloads:

```text
Page Reload
    ↓
Memory cleared
    ↓
Token unavailable
    ↓
Authenticate / obtain token again
```

This sacrifices persistence in exchange for reducing the exposure of long-lived browser storage.

Current OAuth browser-app guidance recommends **Authorization Code + PKCE**, discourages the old Implicit flow, and recommends in-memory token storage rather than `localStorage` when the SPA directly handles tokens. ([oauth.net][4])

---

# 18. An Even Stronger Architecture — BFF

For many production web applications:

```text
                 ┌─────────────────┐
                 │ Authorization   │
                 │ Server          │
                 └────────┬────────┘
                          │
                          │ Tokens
                          ↓
┌──────────┐       ┌──────────────┐       ┌─────────────┐
│ Browser  │──────>│ BFF / Backend│──────>│ Resource    │
│          │ Cookie│              │ Token │ Server/API  │
└──────────┘       └──────────────┘       └─────────────┘
```

The browser gets:

```text
HttpOnly Cookie
```

The BFF handles:

```text
Access Token
Refresh Token
OAuth communication
```

Therefore:

```text
Browser JavaScript
        ↓
   No OAuth token
```

This can substantially reduce the consequences of token theft from browser storage and is a recommended architecture to consider for browser-based applications. ([RFC Editor][1])

---

# 19. Important Security Principle

Never think:

> "Where can I hide the token from the attacker?"

Instead think:

> **"What capabilities does malicious JavaScript have if XSS occurs?"**

For example:

```text
localStorage
     ↓
Attacker can read token
```

With:

```text
HttpOnly Cookie
     ↓
Attacker cannot directly read token
```

But:

```text
XSS
 ↓
Attacker's JavaScript runs as your application
 ↓
Can potentially make authenticated requests
```

So token storage is only **one layer** of browser security.

You still need:

```text
XSS Protection
+
CSRF Protection
+
HTTPS
+
Secure Cookies
+
Short Token Lifetimes
+
Refresh Token Rotation
+
Content Security Policy
+
Input/Output Security
```

---

# 20. What Should You Remember?

The most important rules are:

### Rule 1

**Don't put authentication tokens in `localStorage` by default.**

```text
localStorage
     ↓
JavaScript accessible
     ↓
XSS can steal token
```

OWASP explicitly recommends against storing authentication tokens and credentials there. ([OWASP Cheat Sheet Series][3])

---

### Rule 2

**`sessionStorage` doesn't solve the XSS problem.**

```text
sessionStorage
      ↓
JavaScript accessible
      ↓
XSS can still steal token
```

Its main difference is lifetime/tab isolation, not protection against malicious JavaScript. ([RFC Editor][1])

---

### Rule 3

**HttpOnly cookies protect the cookie value from JavaScript.**

```text
HttpOnly
   ↓
document.cookie
   ↓
Cannot access authentication cookie
```

([MDN Web Docs][2])

---

### Rule 4

**Secure cookies should use HTTPS.**

```text
Secure
   ↓
HTTPS only
```

([MDN Web Docs][2])

---

### Rule 5

**Cookie authentication requires CSRF considerations.**

```text
HttpOnly Cookie
       +
SameSite
       +
CSRF protection
```

([OWASP Cheat Sheet Series][3])

---

### Rule 6

**For SPAs directly handling OAuth tokens, prefer memory over persistent storage.**

```text
Authorization Code + PKCE
           ↓
      Access Token
           ↓
         Memory
```

([oauth.net][4])

---

### Rule 7

**For many production web applications, consider a BFF.**

```text
Browser
   ↓
HttpOnly Cookie
   ↓
BFF
   ↓
OAuth Tokens
   ↓
APIs
```

This keeps OAuth tokens out of the browser application itself. ([RFC Editor][1])

---

# 21. Mental Model

Remember this simple model:

```text
                 TOKEN STORAGE
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
   localStorage   Memory        Cookie
        │             │             │
        ↓             ↓             ↓
   JS can read    JS can use    HttpOnly?
        │             │             │
        ↓             ↓             ↓
   XSS can steal  Short-lived   Better protection
                                  │
                                  ↓
                              CSRF concern
```

And for modern browser applications:

```text
SPA
 │
 ├── Authorization Code + PKCE
 │
 ├── Access Token → Memory
 │
 └── API → Authorization: Bearer TOKEN
```

or, where appropriate:

```text
Browser
   │
   └── HttpOnly Secure SameSite Cookie
              │
              ↓
             BFF
              │
              ├── Access Token
              │
              └── Refresh Token
              │
              ↓
             API
```

---

# Key Takeaway

The most important distinction is:

```text
JWT security
     ≠
Token storage security
```

A perfectly signed JWT can still be stolen.

For browser applications, think in terms of:

```text
                 Best default thinking

Persistent browser storage
(localStorage / sessionStorage)
                ↓
              Avoid

Direct SPA token handling
                ↓
       Authorization Code + PKCE
                ↓
          Token in memory

Web application with backend
                ↓
       BFF + HttpOnly Cookie
                ↓
       Tokens stay server-side
```

The key trade-off to remember is:

```text
localStorage
    ↓
Easy
    ↓
But JavaScript can steal token


Memory
    ↓
Less persistent
    ↓
Better exposure profile


HttpOnly Cookie
    ↓
JavaScript cannot read token
    ↓
But CSRF must be considered


BFF + HttpOnly Cookie
    ↓
OAuth tokens stay away from browser JS
    ↓
Strong production architecture
```

That mental model will make the next topics—**RBAC, scopes, claims, OAuth, token revocation, and advanced API security**—much easier to reason about.

[1]: https://www.rfc-editor.org/rfc/rfc10017.html?utm_source=chatgpt.com "RFC 10017: OAuth 2.0 for Browser-Based Applications"
[2]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie?utm_source=chatgpt.com "Set-Cookie header - HTTP | MDN"
[3]: https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html?utm_source=chatgpt.com "Session Management - OWASP Cheat Sheet Series"
[4]: https://oauth.net/2/browser-based-apps/?utm_source=chatgpt.com "OAuth 2.0 for Browser-Based Apps — RFC 10017"
