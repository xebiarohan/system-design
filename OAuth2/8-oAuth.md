# Why OAuth Exists

## 1. The fundamental problem

Imagine you have an application called **PhotoApp**.

You want PhotoApp to access a user's photos stored on Google.

Without OAuth, you might design it like this:

```text
User
 │
 │ gives Google username + password
 ↓
PhotoApp
 │
 │ uses those credentials
 ↓
Google
```

This is a terrible security model.

The user has effectively said:

> "Here is my Google password. You can log into Google as me."

PhotoApp now potentially has the same power as the user.

It could potentially:

* Read emails
* Access files
* Modify account information
* Delete data
* Change the password
* Access other Google services

And Google has no good way to distinguish:

```text
User using Google
```

from

```text
PhotoApp using Google's password
```

That is the fundamental problem OAuth addresses.

---

# 2. OAuth's basic idea

Instead of giving PhotoApp your Google password, you give PhotoApp a **limited authorization**.

For example:

> "PhotoApp is allowed to read my photos."

Not:

> "PhotoApp is allowed to access my entire Google account."

So the architecture becomes:

```text
                ┌──────────────────────┐
                │       Google         │
                │                      │
                │ Authorization Server │
                └──────────┬───────────┘
                           │
                           │ Access Token
                           ↓
User ───────→ PhotoApp ─────────→ Google API
              │                    │
              │                    │
              └────────────────────┘
```

The important difference is:

**PhotoApp never receives the user's Google password.**

Instead, Google gives PhotoApp a credential representing a particular authorization.

---

# 3. OAuth is about delegated authorization

This is the key sentence to remember:

> **OAuth allows a user to delegate limited access to their resources to another application without sharing their credentials.**

There are several important words here.

### Delegate

The user is allowing another application to perform something on their behalf.

### Limited

The application doesn't necessarily get unlimited access.

For example:

```text
Read photos
```

could be allowed while:

```text
Delete photos
```

isn't.

### Resources

These are things belonging to the user.

For example:

```text
Photos
Files
Contacts
Calendar events
Messages
```

### Without sharing credentials

The application doesn't receive:

```text
username
password
```

Instead it receives an OAuth credential, usually an **access token**.

---

# 4. Before OAuth vs after OAuth

This comparison is extremely important.

## Before OAuth

Suppose you build a third-party application that needs access to a user's Google account.

You might have:

```text
User
 │
 │ Google password
 ↓
Third-party application
 │
 │ Google password
 ↓
Google
```

The third-party application possesses the user's credential.

That's dangerous.

---

## With OAuth

The architecture becomes:

```text
User
 │
 │
 ↓
Third-party Application
 │
 │ "I need permission to read photos"
 ↓
Authorization Server
 │
 │ User authenticates
 │
 │ User approves
 │
 ↓
Authorization Code
 │
 ↓
Application
 │
 │ exchanges code
 ↓
Access Token
 │
 ↓
Resource Server
```

The user's password stays between:

```text
User ↔ Authorization Server
```

The application receives:

```text
Access Token
```

instead.

---

# 5. Why not simply give the application the password?

There are several problems.

## Problem 1 — The application gets too much power

Suppose an application only needs:

```text
Read calendar
```

but you give it your account password.

The application may potentially be able to do:

```text
Read calendar
Read email
Delete files
Change account settings
Delete account
```

The password doesn't naturally express:

> "You may do X but not Y."

OAuth allows much more granular authorization.

---

# 6. Problem 2 — You can't easily revoke one application's access

Imagine you gave your Google password to 20 applications.

Later you want to stop one application.

You have a problem.

If you change your password:

```text
Password changed
     ↓
Potentially affects everything
```

OAuth solves this much more cleanly.

You can conceptually have:

```text
Application A → Access Token A
Application B → Access Token B
Application C → Access Token C
```

Then revoke Application B's authorization without necessarily affecting A and C.

---

# 7. Problem 3 — Password sharing violates separation of responsibility

Consider these two systems:

### Password model

```text
Third-party App
      │
      │ user's password
      ↓
     Google
```

The third-party application has become responsible for protecting Google's credential.

That's undesirable.

---

### OAuth model

```text
Third-party App
      │
      │ OAuth token
      ↓
Google API
```

The application gets a credential specifically intended for API access.

The user's actual authentication credential remains with the authorization server.

---

# 8. OAuth introduces the concept of scopes

This is one of the most important OAuth concepts.

A **scope** represents what the client is requesting permission to do.

Imagine an application asks for:

```text
photos.read
```

The user authorizes that scope.

The application gets an access token associated with that authorization.

Conceptually:

```text
Access Token
     │
     ├── User: Alice
     │
     ├── Client: PhotoApp
     │
     └── Scope: photos.read
```

The application isn't necessarily getting:

```text
everything Alice can access
```

It gets something closer to:

```text
what Alice authorized PhotoApp to access
```

---

# 9. OAuth separates three things

This is where OAuth starts connecting with what you've already learned in Phases 1 and 2.

There are three different concepts:

```text
Authentication
Authorization
Resource Access
```

Let's separate them.

---

## Authentication

Authentication asks:

> Who is this user?

For example:

```text
Alice enters Google credentials
        ↓
Google verifies Alice
        ↓
Google knows the user is Alice
```

That's authentication.

---

## Authorization

Authorization asks:

> What is Alice allowing PhotoApp to do?

For example:

```text
Alice
 │
 └── allows PhotoApp
          │
          └── read photos
```

That's authorization.

---

## Resource access

Finally:

```text
PhotoApp
    │
    │ Access Token
    ↓
Google Photos API
```

The API checks whether the token permits the requested operation.

---

# 10. OAuth does not mean "Login with Google"

This is an extremely important distinction.

You will often see:

```text
Login with Google
```

and people say:

> "That's OAuth."

That's only partially correct.

OAuth itself is fundamentally about:

```text
Delegated Authorization
```

not standardized user authentication.

For authentication/identity, you normally use:

**OpenID Connect (OIDC)**.

You will study that in Phase 4.

Think of it like:

```text
OAuth 2.0
    │
    └── "Can this application access something?"

OpenID Connect
    │
    └── "Who is this user?"
```

OAuth can be involved in a login experience, but OAuth alone doesn't define a standard identity assertion about the user.

---

# 11. A real-world example

Imagine you build:

```text
CalendarApp
```

A user wants CalendarApp to access their Google Calendar.

Without OAuth:

```text
CalendarApp:
"Give me your Google password."
```

Obviously bad.

With OAuth:

```text
CalendarApp
     │
     │ Request authorization
     ↓
Google Authorization Server
     │
     │ User authenticates
     ↓
User sees:
"CalendarApp wants to:
 read your calendar"
     │
     │ User approves
     ↓
Authorization Code
     │
     ↓
CalendarApp
     │
     │ exchanges code
     ↓
Access Token
     │
     ↓
Google Calendar API
```

Now CalendarApp can make authorized requests.

For example:

```http
GET /calendar/events
Authorization: Bearer ACCESS_TOKEN
```

Google's API can inspect the token and determine whether that token is authorized for the requested resource.

---

# 12. The important actors

OAuth introduces a vocabulary that you need to memorize.

There are four major roles.

```text
Resource Owner
       │
       ↓
     Client
       │
       ↓
Authorization Server
       │
       ↓
 Resource Server
```

Don't worry about mastering them yet—you have a dedicated topic for this next.

But you should understand the basic idea:

### Resource Owner

Usually:

```text
The user
```

The person who owns the data.

### Client

The application requesting access.

For example:

```text
PhotoApp
CalendarApp
MobileApp
Backend service
```

### Authorization Server

The system that handles authorization and issues tokens.

For example:

```text
Google authorization server
Microsoft authorization server
Auth0
Keycloak
```

### Resource Server

The API containing the protected resources.

For example:

```text
Google Photos API
Google Calendar API
Microsoft Graph API
```

---

# 13. Authorization Server vs Resource Server

This distinction is particularly important.

Suppose Google has:

```text
Authorization Server
        │
        │ issues tokens
        ↓
   Access Token
        │
        ↓
Resource Server
        │
        │ validates/uses token
        ↓
    User Data
```

The **Authorization Server** is responsible for issuing tokens.

The **Resource Server** hosts the protected API/resource.

They can be separate systems, although in some architectures they may be operated by the same organization or even be closely integrated.

---

# 14. Why use an access token?

Now connect this with your Phase 2 knowledge.

You learned:

```text
Authorization: Bearer TOKEN
```

OAuth provides a standardized way for a client to obtain an access token representing delegated authorization.

So instead of:

```text
Authorization: Bearer GOOGLE_PASSWORD
```

you have:

```http
Authorization: Bearer ACCESS_TOKEN
```

The token might represent something conceptually like:

```text
User: Alice
Client: PhotoApp
Scopes:
    photos.read
Expiration:
    1 hour
```

The exact representation depends on the authorization server.

And importantly:

**An OAuth access token does not have to be a JWT.**

It could be an opaque token:

```text
8f72ab91c...
```

or a JWT:

```text
eyJhbGciOiJSUzI1NiIs...
```

OAuth defines how authorization is obtained and used; it does not require access tokens to have the JWT format.

That's an important connection between your **JWT** topic and OAuth.

---

# 15. OAuth solves the "delegation" problem

Imagine you are Alice.

You have:

```text
Alice
 │
 ├── Google account
 │
 ├── Google Photos
 │
 └── Google Calendar
```

You install:

```text
PhotoApp
```

You want PhotoApp to access your photos.

You are effectively saying:

> "I authorize PhotoApp to access this resource on my behalf."

That's **delegation**.

OAuth gives us a standardized protocol for expressing that delegation.

Conceptually:

```text
Alice
 │
 │ delegates limited authority
 ↓
PhotoApp
 │
 │ uses delegated authority
 ↓
Google API
```

---

# 16. OAuth is not primarily about protecting APIs from users

This is another common misunderstanding.

OAuth isn't simply:

> "A way to authenticate API requests."

Its deeper purpose is:

> **Allow a client to obtain delegated authorization to access protected resources.**

The access token is the mechanism that allows the client to demonstrate that authorization to the resource server.

So think:

```text
OAuth
 ↓
Delegation
 ↓
Authorization
 ↓
Access Token
 ↓
Protected Resource
```

---

# 17. OAuth vs API keys

You will later learn API keys, so it's useful to see the distinction now.

An API key generally identifies an application/client:

```text
Application
     │
     │ API Key
     ↓
    API
```

OAuth can represent:

```text
User
  │
  │ authorizes
  ↓
Application
  │
  │ Access Token
  ↓
Protected API
```

So OAuth becomes particularly useful when:

> **An application needs to access a user's protected resources with that user's authorization.**

---

# 18. OAuth vs sessions

You've already learned sessions.

A session typically looks like:

```text
Browser
   │
   │ Session Cookie
   ↓
Your Application
   │
   ↓
Session Store
```

OAuth solves a different problem.

OAuth typically looks more like:

```text
Your Application
      │
      │ Access Token
      ↓
Another Service's API
```

For example:

```text
Your application
      │
      │ OAuth
      ↓
Google API
```

So:

### Sessions

Usually answer:

> "Is this browser logged into my application?"

### OAuth

Usually answers:

> "Has this application been authorized to access resources from another service on behalf of a user?"

These can exist together.

For example:

```text
User
 │
 │ logs into your application
 ↓
Your App
 │
 │ OAuth authorization
 ↓
Google
```

Your application might use a normal session for the user's login while separately using OAuth to access Google's APIs.

---

# 19. OAuth is useful because passwords don't scale for delegation

Imagine there are 1,000 applications integrating with a service.

Without OAuth:

```text
User password
     ↓
App A
App B
App C
...
App 1000
```

That's a security nightmare.

With OAuth:

```text
                  ┌── App A → Token A
                  │
User → Authorization
                  ├── App B → Token B
                  │
                  ├── App C → Token C
                  │
                  └── App D → Token D
```

Each application can receive a separate authorization.

This gives you:

* Limited permissions
* Separate credentials/tokens
* Revocation
* Expiration
* Better auditing
* No password sharing

---

# 20. The big mental model

I recommend remembering OAuth using this model:

```text
                    USER
                     │
                     │ owns
                     ↓
                  RESOURCE
                     │
                     │
              "I authorize this"
                     │
                     ↓
                  CLIENT
                     │
                     │ receives
                     ↓
                ACCESS TOKEN
                     │
                     │
                     ↓
              RESOURCE SERVER
```

And somewhere in the middle:

```text
Authorization Server
        │
        │ issues access token
        ↓
    Access Token
```

So:

```text
User
 ↓
Authorizes Client
 ↓
Authorization Server
 ↓
Access Token
 ↓
Resource Server
 ↓
Protected Resource
```

---

# 21. The problem OAuth actually solves

If you remember only one thing from this topic, remember this:

### Without OAuth

```text
"I need access to your data."

"Okay, here's my password."
```

Bad.

### With OAuth

```text
"I need access to your data."

"Okay, I'll authorize you to access
these specific resources."

        ↓

Authorization Server

        ↓

Access Token

        ↓

Client accesses protected API
```

That's the fundamental reason OAuth exists.

---

# 22. How this connects to the rest of your roadmap

You're now at:

```text
Phase 2
   │
   ├── Tokens
   ├── Access Tokens
   ├── Refresh Tokens
   ├── JWT
   └── JWT Security
          │
          ↓
Phase 3
   │
   └── Why OAuth Exists  ← YOU ARE HERE
          │
          ↓
      OAuth Roles
          │
          ↓
   Authorization Code Flow
          │
          ↓
         PKCE
          │
          ↓
 Client Credentials
          │
          ↓
   Refresh Token Flow
          │
          ↓
 Device Authorization
          │
          ↓
Phase 4
   │
   └── OpenID Connect
```

This order is actually very useful.

You've already learned what a token is.

Now you're learning **why a standardized system for obtaining and using those tokens is necessary**.

Then you'll learn exactly **who participates in that system**.

Then you'll learn **how the authorization flow works**.

---

## The 5 questions you should be able to answer

Before moving to **Topic 11 — OAuth Roles**, make sure you can answer these:

1. **What problem does OAuth solve?**

   → Delegated authorization without sharing the user's credentials.

2. **Why shouldn't a third-party application receive the user's password?**

   → It gives the application excessive control and creates credential-security and revocation problems.

3. **What does OAuth give the client instead?**

   → An access token representing delegated authorization.

4. **What is the difference between authentication and OAuth authorization?**

   → Authentication establishes who the user is; OAuth authorization determines what a client is allowed to access on the user's behalf.

5. **Does OAuth require JWT access tokens?**

   → No. OAuth access tokens can be opaque strings or JWTs.

If those five answers make sense, you're ready for **Topic 11: OAuth Roles — Resource Owner, Client, Authorization Server, and Resource Server**.
