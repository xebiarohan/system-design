# Single Logout (SLO)

**Single Logout (SLO)** is the counterpart to SSO.

SSO means:

> **Log in once → access multiple applications.**

SLO means:

> **Log out once → terminate the user's sessions across multiple applications.**

---

## 1. The problem SLO solves

Suppose a user has logged into:

```text
             Identity Provider
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      App A       App B       App C
```

Each application may have its **own local session**:

```text
IdP       → authenticated
App A     → logged in
App B     → logged in
App C     → logged in
```

Now the user clicks **Logout** in App A.

Without SLO:

```text
App A → Logged out ✅

App B → Still logged in ❌
App C → Still logged in ❌
```

That's potentially surprising and undesirable from a security perspective.

With SLO:

```text
Logout
  ↓
IdP + App A + App B + App C
  ↓
All sessions terminated
```

---

# 2. The important architecture

Think of the Identity Provider as the central authority:

```text
                 Identity Provider
                       │
                 IdP Session
                       │
          ┌────────────┼────────────┐
          │            │            │
        App A        App B        App C
       Session       Session       Session
```

There are actually **multiple sessions** involved.

### IdP session

The user is logged into the Identity Provider.

### Application sessions

Each application may maintain its own session.

For example:

```text
Browser
 ├── IdP session
 ├── App A session
 ├── App B session
 └── App C session
```

SLO is about coordinating the termination of these sessions.

---

# 3. SLO with OIDC

Since you're learning OAuth/OIDC, this is the important part.

OIDC defines a mechanism called **RP-Initiated Logout**.

Here:

* **OP** = OpenID Provider / Identity Provider
* **RP** = Relying Party / Application

So:

```text
User
 ↓
RP (Application)
 ↓
OP (Identity Provider)
```

The application can initiate logout at the IdP.

---

# 4. RP-Initiated Logout

Suppose the user clicks:

```text
Logout
```

in App A.

App A redirects the browser to the IdP's logout endpoint.

Conceptually:

```text
App A
  │
  │ Logout request
  ▼
Identity Provider
```

The IdP terminates the user's IdP session.

Then the IdP can redirect the user back to the application.

Conceptually:

```text
App A
  │
  │ 1. Logout
  ▼
IdP
  │
  │ 2. Destroy IdP session
  │
  │ 3. Logout processing
  ▼
App A
```

The application should also invalidate its **own local session**.

---

# 5. But how does App B know?

This is where SLO becomes more interesting.

Imagine:

```text
                 IdP
              /   |   \
             /    |    \
           App A App B App C
```

User logs out from App A.

The IdP knows:

```text
User's IdP session = terminated
```

But App B might still have:

```text
App B session = active
```

There are different mechanisms for coordinating logout.

---

# 6. Front-Channel Logout

With **front-channel logout**, the IdP tells the applications to perform logout through the user's browser.

Conceptually:

```text
                 IdP
                  │
         Logout notification
                  │
          ┌───────┼───────┐
          ↓       ↓       ↓
        App A   App B   App C
          │       │       │
       Logout   Logout   Logout
```

The browser participates in the logout process.

Think:

> "IdP tells each application: please terminate this user's session."

---

# 7. Back-Channel Logout

**Back-channel logout** works differently.

The IdP directly communicates with the applications' backend servers.

```text
                 IdP
              /   |   \
             /    |    \
            ↓     ↓     ↓
          App A  App B  App C
          Backend Backend Backend
```

No browser interaction is required for the notification.

For example:

```text
IdP
 │
 │ Logout token
 │
 ├──────────────→ App A backend
 │
 ├──────────────→ App B backend
 │
 └──────────────→ App C backend
```

Each application receives the logout notification and invalidates the corresponding user session.

---

# 8. Front-channel vs Back-channel

This distinction is worth remembering:

|                       | Front-channel             | Back-channel            |
| --------------------- | ------------------------- | ----------------------- |
| Communication         | Through browser           | Server-to-server        |
| Browser involved      | Yes                       | No                      |
| Reliability           | More dependent on browser | Generally more reliable |
| Backend communication | Indirect                  | Direct                  |
| Security              | Good                      | Stronger architecture   |

A simple mental model:

```text
Front-channel:

IdP → Browser → Apps


Back-channel:

IdP ─────────→ App backends
```

---

# 9. Logout Token

In OIDC back-channel logout, the IdP can send a **Logout Token** to an application.

Conceptually:

```json
{
  "iss": "https://idp.example.com",
  "aud": "my-client-id",
  "events": {
    "http://schemas.openid.net/event/backchannel-logout": {}
  },
  "sid": "abc123"
}
```

The important part is `sid`.

`sid` represents the user's **session identifier**.

The application can use it to identify which session should be terminated.

---

# 10. Why not simply delete all sessions?

Because an application may have many users.

Imagine App B has:

```text
User A → Session A
User B → Session B
User C → Session C
```

If User A logs out:

```text
❌ Don't delete all sessions
```

Instead:

```text
Logout User A
      ↓
Find User A's session
      ↓
Invalidate Session A
```

The logout/session identifiers help applications know **which user's session** should be terminated.

---

# 11. SLO flow you should remember

A simplified OIDC SLO flow:

```text
User
 │
 │ Click Logout
 ▼
Application (RP)
 │
 │ RP-Initiated Logout
 ▼
Identity Provider (OP)
 │
 │ Terminate IdP session
 │
 ├──── Front-channel ────→ Applications
 │
 └──── Back-channel ─────→ Application backends
                              │
                              ↓
                       Invalidate sessions
```

Result:

```text
IdP session     ❌
App A session   ❌
App B session   ❌
App C session   ❌
```

---

# 12. One important real-world caveat

**SSO is easier than SLO.**

Logging in is centralized naturally:

```text
User → IdP → authenticated
```

But applications can maintain independent sessions:

```text
IdP ── App A session
   ├─ App B session
   └─ App C session
```

So logout requires **coordination** between the IdP and RPs.

That's why OIDC provides different logout mechanisms rather than simply assuming that destroying the IdP session automatically destroys every application session.

---

## What to remember for your OAuth roadmap

```text
SSO
 │
 └── Login once
       ↓
     Multiple applications


SLO
 │
 └── Logout once
       ↓
     Multiple applications
```

And the OIDC logout mechanisms:

```text
RP-Initiated Logout
        │
        ├── Front-Channel Logout
        │
        └── Back-Channel Logout
```

**Most important concept:** terminating the **IdP session** and terminating each application's **local session** are separate things; SLO coordinates them.
