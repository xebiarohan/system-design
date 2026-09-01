
# 1. Start with OAuth 2.0

You already learned the basic OAuth model:

```text
Resource Owner
      │
      │ authorization
      ↓
Authorization Server
      │
      │ access token
      ↓
Client
      │
      │ access token
      ↓
Resource Server
      │
      ↓
Protected Resource
```

For example:

```text
Alice
  ↓
Google Authorization Server
  ↓
Access Token
  ↓
My Application
  ↓
Google API
```

OAuth 2.0 originally defined the framework in RFC 6749.

Over time, however, several security problems were identified, and additional specifications were created:

* PKCE
* OAuth Security BCP
* Native Apps guidance
* Browser-based Apps guidance
* Bearer Token Usage
* etc.

OAuth 2.1 essentially brings the modern recommendations together.

The current draft explicitly describes itself as consolidating OAuth 2.0 and several subsequent specifications. ([IETF Datatracker][1])

---

# 2. The Biggest Mental Model

Don't think:

```text
OAuth 2.0
    ↓
OAuth 2.1
    ↓
Completely different protocol
```

Think:

```text
OAuth 2.0
   +
PKCE
   +
Security Best Practices
   +
Modern redirect URI rules
   +
Removal of insecure flows
   +
Modern token handling
   ↓
OAuth 2.1
```

That's the key.

---

# 3. What Changed?

The most important changes are:

```text
OAuth 2.0                         OAuth 2.1
---------------------------------------------------------
Authorization Code               Authorization Code
Implicit Grant                   Removed
Password Grant                   Removed
PKCE                              Required/default
Loose redirect matching          Exact matching
Bearer token in URI              Removed
Refresh token security           Stronger
```

The current OAuth 2.1 draft explicitly lists these changes. ([IETF Datatracker][1])

Let's go through each one.

---

# 4. Authorization Code Flow Becomes the Main User Flow

OAuth 2.1 strongly centers around:

```text
Authorization Code Flow
```

The basic flow is:

```text
                 ┌──────────────────────┐
                 │ Authorization Server │
                 └──────────┬───────────┘
                            │
                            │
User                         │
 │                           │
 │ 1. Authorization request  │
 ├──────────────────────────>│
 │                           │
 │ 2. Login + consent        │
 │<─────────────────────────>│
 │                           │
 │ 3. Authorization code     │
 │<──────────────────────────┤
 │                           │
 └───────────────> Client    │
                         │
                         │ 4. Code + PKCE
                         ├───────────────>
                         │
                         │ 5. Access token
                         │<───────────────
                         │
                         ↓
                   Resource Server
```

The important idea is:

> **The authorization code is not the access token.**

The browser receives the short-lived code.

Then the client exchanges it for tokens.

---

# 5. Why Not Give the Access Token Directly?

This was essentially the idea behind the old **Implicit Flow**.

Old OAuth:

```text
Browser
   ↓
Authorization Server
   ↓
Access Token
   ↓
Browser
```

The access token traveled through the browser authorization response.

That created more opportunities for token leakage and replay.

OAuth 2.1 removes the Implicit Grant. ([IETF Datatracker][1])

Instead:

```text
Browser
   ↓
Authorization Server
   ↓
Authorization Code
   ↓
Client
   ↓
Token Endpoint
   ↓
Access Token
```

This is safer because the access token is obtained from the token endpoint rather than being directly returned through the authorization response.

---

# 6. PKCE Becomes Central

This is probably the **single most important OAuth 2.1 concept** to understand.

PKCE =

> **Proof Key for Code Exchange**

Suppose an attacker somehow intercepts:

```text
authorization_code = ABC123
```

Without additional protection, they might try:

```text
Attacker
   ↓
ABC123
   ↓
Token Endpoint
   ↓
Access Token
```

PKCE prevents this by binding the authorization code exchange to a secret that the legitimate client generated.

---

# 7. How PKCE Works

The client generates a random value:

```text
code_verifier

    ↓

"random-secret-value"
```

Then calculates:

```text
code_challenge =
    BASE64URL(
        SHA256(code_verifier)
    )
```

The client sends the challenge during the authorization request:

```text
/authorize

client_id=abc
response_type=code
redirect_uri=https://example.com/callback
code_challenge=XYZ
code_challenge_method=S256
```

The authorization server remembers:

```text
Authorization Code ABC123

associated with:

code_challenge = XYZ
```

Later, the client exchanges the code:

```http
POST /token
```

with:

```text
grant_type=authorization_code
code=ABC123
code_verifier=random-secret-value
```

The authorization server calculates:

```text
SHA256(code_verifier)
```

and checks:

```text
Does it equal the original code_challenge?
```

If:

```text
YES
```

→ issue tokens.

If:

```text
NO
```

→ reject.

---

# 8. The Security Benefit of PKCE

Imagine:

```text
Legitimate Client

code_verifier = SECRET

        ↓

code_challenge = HASH(SECRET)

        ↓

Authorization Server
```

Attacker steals:

```text
authorization_code
```

but doesn't know:

```text
code_verifier
```

Therefore:

```text
Attacker
   ↓
authorization_code
   +
wrong verifier
   ↓
Token Endpoint
   ↓
REJECT
```

The legitimate client:

```text
Legitimate Client
   ↓
authorization_code
   +
correct verifier
   ↓
Token Endpoint
   ↓
ACCESS TOKEN
```

This is why PKCE is so important.

The current OAuth 2.1 draft incorporates PKCE into the authorization-code grant, and the OAuth security guidance requires PKCE for public clients. ([IETF Datatracker][1])

---

# 9. PKCE Isn't Just for Mobile Apps Anymore

Historically, you may see:

> "PKCE is for mobile apps."

That's outdated as a learning model.

PKCE was originally particularly important for public/native clients, but modern OAuth security guidance applies it broadly, including browser-based applications. ([IETF Datatracker][2])

So your modern mental model should be:

```text
Authorization Code
        +
PKCE
```

rather than:

```text
Authorization Code
```

alone.

---

# 10. The Password Grant Is Removed

OAuth 2.0 had:

```text
Resource Owner Password Credentials Grant
```

Often called:

```text
ROPC
Password Grant
```

It looked roughly like:

```text
User
 ↓
Username + Password
 ↓
Client
 ↓
Authorization Server
```

For example:

```http
POST /token

grant_type=password
username=alice
password=secret
```

This is problematic because the application receives the user's password.

The whole point of OAuth is:

```text
User credentials

       ↓

Authorization Server

       ↓

Client receives authorization
```

not:

```text
User password
       ↓
Third-party application
```

OAuth 2.1 therefore omits the Resource Owner Password Credentials grant. ([IETF Datatracker][1])

---

# 11. Think About the Difference

### Old password approach

```text
User
  ↓
Password
  ↓
Client
  ↓
Authorization Server
```

The client sees the password.

### Modern OAuth

```text
User
  ↓
Authorization Server
  ↓
Authentication
  ↓
Consent
  ↓
Authorization Code
  ↓
Client
```

The client never needs the user's authorization-server password.

This is a fundamental OAuth security principle.

---

# 12. Implicit Flow Is Removed

OAuth 2.0's Implicit Grant looked roughly like:

```text
Browser
   ↓
Authorization Server
   ↓
Access Token
   ↓
Browser
```

For example:

```text
https://client.example/callback#access_token=XYZ
```

The token is exposed to the user-agent/browser environment.

Modern OAuth recommends:

```text
Authorization Code
       +
PKCE
```

instead.

OAuth 2.1 omits:

```text
response_type=token
```

from the framework. ([IETF Datatracker][1])

---

# 13. Redirect URI Matching Becomes Strict

This is another important security improvement.

Suppose the registered redirect URI is:

```text
https://example.com/callback
```

An authorization server shouldn't casually accept:

```text
https://example.com/callback2
```

or some attacker-controlled variation.

OAuth 2.1 requires exact redirect URI comparison according to the relevant security specification. ([IETF Datatracker][1])

Think:

```text
Registered:

https://app.example.com/oauth/callback

Request:

https://app.example.com/oauth/callback

        ↓

EXACT MATCH
        ↓
ALLOW
```

Not:

```text
"Looks similar"
```

---

# 14. Why Redirect URI Security Matters

Imagine:

```text
Authorization Server
       ↓
redirect_uri
       ↓
https://attacker.example/callback
```

If the authorization server sends the authorization code there:

```text
Authorization Code
       ↓
Attacker
```

Now the attacker may attempt to redeem it.

PKCE helps protect the code exchange, but redirect URI validation is still an important security boundary.

Therefore modern OAuth uses multiple defenses:

```text
Exact redirect URI
        +
PKCE
        +
Short-lived authorization code
        +
Single-use authorization code
```

---

# 15. Bearer Tokens Shouldn't Be Sent in URLs

OAuth 2.0 bearer tokens could historically appear in URI query parameters.

Something like:

```text
GET /api/orders?access_token=XYZ
```

This is a bad idea because URLs can leak through:

* browser history
* logs
* analytics
* proxies
* referrer information
* monitoring systems

Modern OAuth excludes bearer-token usage in URI query strings. ([IETF Datatracker][1])

Instead:

```http
GET /api/orders
Authorization: Bearer XYZ
```

Much better.

---

# 16. Refresh Tokens Get More Security Requirements

Suppose:

```text
Access Token
expires in 10 minutes
```

The client uses:

```text
Refresh Token
```

to obtain another access token.

But what if an attacker steals the refresh token?

That's dangerous because refresh tokens typically live longer.

OAuth 2.1 therefore adds stronger requirements around refresh tokens for public clients.

The current draft says refresh tokens for public clients must either be **sender-constrained** or **one-time use**. ([IETF Datatracker][1])

A common mechanism is:

```text
Refresh Token 1
      ↓
Refresh
      ↓
Access Token 2
+
Refresh Token 2
```

The old refresh token becomes invalid.

This is called:

> **Refresh Token Rotation**

Conceptually:

```text
RT1
 ↓
RT2
 ↓
RT3
 ↓
RT4
```

If someone tries to reuse:

```text
RT1
```

after it has already been consumed, the authorization server can detect suspicious reuse.

---

# 17. What OAuth 2.1 Does NOT Mean

This is very important.

OAuth 2.1 does **not** mean:

```text
OAuth 2.0 is completely obsolete
```

Nor does it mean:

```text
JWT is required
```

OAuth does not require JWT access tokens.

Your access token could be:

```text
opaque-random-string
```

or:

```text
JWT
```

OAuth defines the authorization framework; the access-token representation can vary.

---

# 18. OAuth 2.1 Does Not Define Login

This connects directly to your OIDC section.

OAuth:

```text
Authorization
```

OIDC:

```text
Authentication + Identity
```

For example:

```text
OAuth 2.1
     ↓
Can this application access
the user's resources?
```

OIDC:

```text
OpenID Connect
     ↓
Who is this user?
```

So:

```text
OAuth 2.1
    +
OpenID Connect
```

is a very common modern login architecture.

---

# 19. OAuth 2.1 Grant Types

The current draft defines three core grant types:

```text
Authorization Code
Client Credentials
Refresh Token
```

and provides an extension mechanism for additional grant types. ([IETF Datatracker][1])

### Authorization Code

For user-delegated authorization:

```text
User
 ↓
Authorization Server
 ↓
Code
 ↓
Client
 ↓
Access Token
```

with PKCE.

### Client Credentials

For machine-to-machine access:

```text
Service A
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B
```

No user interaction is required.

### Refresh Token

For obtaining a new access token:

```text
Refresh Token
      ↓
Authorization Server
      ↓
New Access Token
```

---

# 20. What About Device Authorization?

You previously listed:

> Device Authorization Flow

This is important in modern OAuth, especially for TVs, consoles, and devices with limited input.

But it is an **extension grant**, rather than one of the three core grant types defined in the current OAuth 2.1 draft.

Conceptually:

```text
TV
 ↓
Device Code
 ↓
User uses phone/computer
 ↓
Authorization Server
 ↓
TV receives token
```

So don't conclude:

> "OAuth 2.1 only supports three possible flows."

Better:

> **OAuth 2.1 defines three core grant types and allows extensions.**

---

# 21. OAuth 2.1 Authorization Code Flow

Let's put everything together.

Imagine:

```text
MyApp
```

wants access to:

```text
Google Calendar
```

The modern flow is approximately:

```text
                    ┌─────────────────────┐
                    │ Authorization      │
                    │ Server              │
                    └──────────┬──────────┘
                               │
                               │
MyApp                           │
 │                             │
 │ Generate code_verifier       │
 │                             │
 │ code_challenge = SHA256(...) │
 │                             │
 │──── authorization request ──>│
 │                             │
 │                             │
 │                         User Login
 │                             │
 │                         User Consent
 │                             │
 │<──── authorization code ─────│
 │                             │
 │                             │
 │──── code + verifier ────────>│
 │                             │
 │<──── access token ───────────│
 │                             │
 │                             │
 │──── Bearer token ────────────────> Calendar API
 │                                  │
 │<──────────── Protected data ─────│
```

This is the OAuth 2.1 mental model you should know.

---

# 22. What Is Actually Being Protected?

There are several security boundaries.

### Boundary 1 — User credentials

```text
User
   ↓
Authorization Server

Client never gets password
```

### Boundary 2 — Authorization code

```text
Authorization Code
       ↓
Protected by PKCE
```

### Boundary 3 — Redirect

```text
Authorization Code
       ↓
Exact redirect URI
```

### Boundary 4 — Access token

```text
Access Token
       ↓
HTTPS
       ↓
Resource Server
```

### Boundary 5 — Refresh token

```text
Refresh Token
       ↓
Rotation / sender constraint
```

OAuth 2.1 is essentially pushing the ecosystem toward these safer defaults.

---

# 23. OAuth 2.0 vs OAuth 2.1

Here's the table I'd put in your roadmap.

| Topic                                  | OAuth 2.0                | OAuth 2.1                                      |
| -------------------------------------- | ------------------------ | ---------------------------------------------- |
| Authorization Code                     | ✅                        | ✅                                              |
| PKCE                                   | Extension / separate RFC | Integrated into modern authorization-code flow |
| Implicit Grant                         | Defined                  | ❌ Removed                                      |
| Password Grant                         | Defined                  | ❌ Removed                                      |
| Client Credentials                     | ✅                        | ✅                                              |
| Refresh Token                          | ✅                        | ✅                                              |
| Exact redirect matching                | Security guidance        | Required modern behavior                       |
| Bearer token in URL                    | Historically specified   | ❌ Removed                                      |
| Public-client refresh-token protection | Less prescriptive        | Stronger requirements                          |
| Main philosophy                        | Framework                | Framework + modern security defaults           |

These differences are explicitly summarized in the current draft. ([IETF Datatracker][1])

---

# 24. OAuth 2.1 Is More About "Secure Defaults"

This is probably the best conceptual explanation.

OAuth 2.0 gave developers many choices.

Some were dangerous if implemented incorrectly:

```text
Implicit Flow
Password Grant
Weak redirect URI handling
No PKCE
Tokens in URLs
```

OAuth 2.1 says, essentially:

```text
Don't use those old patterns.

Use:

Authorization Code
        +
PKCE
        +
Strict redirect URI
        +
Secure token handling
        +
Secure refresh tokens
```

So OAuth 2.1 is less about inventing new OAuth functionality and more about **bringing the secure modern OAuth practices into one framework**.

---

# 25. Where JWT Fits

You might wonder:

> "If I'm using OAuth 2.1, do I need JWT?"

No.

You can have:

```text
OAuth 2.1
   ↓
Opaque Access Token
```

or:

```text
OAuth 2.1
   ↓
JWT Access Token
```

For example:

```text
Authorization Server
       ↓
JWT Access Token
       ↓
Resource Server
       ↓
Verify signature
       ↓
Check claims/scope
```

JWT is a **token format**, not OAuth itself.

Keep these separate:

```text
OAuth 2.1
    ↓
Authorization framework

JWT
    ↓
Token format

PKCE
    ↓
Authorization-code security mechanism

OIDC
    ↓
Authentication/identity layer
```

---

# 26. Where Scopes Fit

This connects directly to what we discussed earlier.

Suppose the application requests:

```text
scope=calendar.read calendar.write
```

The authorization server grants:

```text
scope=calendar.read
```

The access token represents that delegated authorization.

Then:

```text
Resource Server
      ↓
Validate token
      ↓
Check scope
      ↓
calendar.read?
      ↓
ALLOW / DENY
```

OAuth 2.1 does **not** replace your authorization model.

You can still have:

```text
Scopes
+
Roles
+
Claims
+
Permissions
+
ABAC policies
```

---

# 27. OAuth 2.1 + OIDC

A very common modern architecture looks like:

```text
                    User
                      │
                      ↓
             ┌─────────────────┐
             │ Authorization   │
             │ Server / IdP    │
             └────────┬────────┘
                      │
               OAuth 2.1
                      │
             Authorization Code
                      │
                    PKCE
                      │
                      ↓
                   Client
                      │
             ┌────────┴────────┐
             ↓                 ↓
         ID Token         Access Token
             │                 │
             ↓                 ↓
        Authentication     API Access
             │                 │
             ↓                 ↓
            App          Resource Server
```

This is a very useful architecture to understand before you start studying providers like:

* Google
* Microsoft
* Auth0
* Okta
* Keycloak

---

# 28. The Most Important Security Improvements

If you only remember six things from OAuth 2.1, remember these:

### 1. Authorization Code

```text
Use code → token exchange
```

instead of directly returning an access token.

### 2. PKCE

```text
Authorization Code
        +
code_verifier
```

protects the authorization-code exchange.

### 3. No Implicit Grant

```text
response_type=token
```

is out.

### 4. No Password Grant

```text
grant_type=password
```

is out.

### 5. Exact Redirect URI

```text
Registered URI
       ==
Requested URI
```

### 6. Protect Refresh Tokens

For public clients, use sender-constraining or one-time-use behavior as required by the current draft. ([IETF Datatracker][1])

---

# 29. One Diagram to Memorize

If you're studying for architecture interviews or trying to understand production OAuth, I'd memorize this:

```text
                    USER
                     │
                     │
                     ↓
             Authorization Server
                     │
             Login + Consent
                     │
                     ↓
             Authorization Code
                     │
                     │
                  + PKCE
                     │
                     ↓
                   CLIENT
                     │
                     │ code + verifier
                     ↓
             Token Endpoint
                     │
              ┌──────┴──────┐
              ↓             ↓
        Access Token    Refresh Token
              │
              │
              ↓
       Resource Server
              │
       ┌──────┴────────┐
       │               │
   Validate        Check scope
     token             │
       │               │
       └───────┬───────┘
               ↓
         Authorization
               ↓
          ALLOW / DENY
```

And the security principles around it:

```text
Authorization Code
       +
PKCE
       +
Exact Redirect URI
       +
HTTPS
       +
Short-lived Access Token
       +
Protected Refresh Token
       +
No Implicit
       +
No Password Grant
```

---

# 30. How I Recommend You Add OAuth 2.1 to Your Roadmap

Given the roadmap you showed me, I would **not** study OAuth 2.1 as an isolated topic.

I'd place it here:

```text
OAuth 2.0 Fundamentals
        ↓
OAuth Roles
        ↓
Authorization Code
        ↓
PKCE
        ↓
Scopes
        ↓
Access + Refresh Tokens
        ↓
OAuth Security Problems
        ↓
OAuth 2.1
        ↓
OIDC
        ↓
Identity Providers
        ↓
Advanced OAuth
```

And while learning OAuth 2.1, make sure you can answer these **without looking anything up**:

1. Why was the Implicit Flow removed?
2. Why was the Password Grant removed?
3. What problem does PKCE solve?
4. Why isn't PKCE just for mobile apps?
5. Why must redirect URIs be exact?
6. Why shouldn't access tokens be put in URLs?
7. Why are refresh tokens more sensitive than access tokens?
8. What is the difference between OAuth 2.0 and OAuth 2.1?
9. Does OAuth 2.1 require JWT?
10. Does OAuth 2.1 provide authentication/login?
11. What is the relationship between OAuth 2.1 and OIDC?
12. Where do scopes fit into the authorization decision?

If you can explain those 12 questions, you'll have a **solid conceptual understanding of OAuth 2.1**, rather than just memorizing its flow diagrams.

[1]: https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/14/?utm_source=chatgpt.com "draft-ietf-oauth-v2-1-14 - The OAuth 2.1 Authorization Framework"
[2]: https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-16?utm_source=chatgpt.com "draft-ietf-oauth-security-topics-16"
