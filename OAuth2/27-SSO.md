# Single Sign-On (SSO)

SSO is one of the most important concepts around **OAuth 2.0 + OpenID Connect (OIDC)**.

The basic idea is:

> **Authenticate once with an Identity Provider (IdP), then access multiple applications without logging in separately to each one.**

---

## 1. Without SSO

Imagine a company has three applications:

```text
Application A → Login
Application B → Login
Application C → Login
```

The user might have to authenticate three times:

```text
User
 │
 ├── Login → App A
 │
 ├── Login → App B
 │
 └── Login → App C
```

That's annoying.

---

# 2. With SSO

With SSO, there is a central **Identity Provider (IdP)**.

Examples include:

* Microsoft Entra ID
* Okta
* Keycloak
* Auth0

The architecture becomes:

```text
                 Identity Provider
                       (IdP)
                         │
              ┌──────────┼──────────┐
              │          │          │
             App A     App B      App C
```

The user authenticates with the IdP.

After that, other applications can reuse that authentication session.

```text
User
 │
 │ Login
 ▼
Identity Provider
 │
 │ authenticated
 ▼
App A

Later...

User → App B
          │
          │ "Already authenticated?"
          ▼
      Identity Provider
          │
          │ Yes
          ▼
        App B
```

The user doesn't have to enter their username/password again.

---

# 3. Important distinction: SSO ≠ OAuth

This is important for your roadmap.

**OAuth 2.0** primarily deals with:

> **Authorization / delegated access**

**OIDC** adds:

> **Authentication / identity**

SSO is generally achieved using **OIDC or SAML**, with OIDC being highly relevant to your OAuth roadmap.

Think:

```text
OAuth 2.0
   │
   └── Authorization

OIDC
   │
   └── Authentication + Identity

SSO
   │
   └── User authenticates once
       and accesses multiple applications
```

---

# 4. How SSO works with OIDC

Let's say you have:

```text
Browser
   │
   ├── Application A
   ├── Application B
   └── Application C

Identity Provider
```

The user first opens **App A**.

App A redirects the browser to the IdP:

```text
Browser
   │
   │ 1. Access App A
   ▼
App A
   │
   │ 2. Redirect to IdP
   ▼
Identity Provider
```

The IdP asks the user to authenticate:

```text
Username
Password
   ↓
Identity Provider
```

After successful authentication, the IdP creates an **authentication session** for the user.

Then it sends the browser back to App A with an authorization code:

```text
IdP
 │
 │ authorization code
 ▼
App A
```

App A exchanges the code for tokens:

```text
Authorization Code
       ↓
     Token
       ↓
 ID Token + Access Token
```

App A now knows:

> "This user is authenticated."

---

# 5. Now comes the SSO magic

The user opens App B.

App B doesn't know whether the user is authenticated.

So it redirects to the IdP:

```text
Browser
   │
   ▼
App B
   │
   ▼
Identity Provider
```

But the user already has an IdP session.

Therefore:

```text
IdP
 │
 │ "This browser already has
 │  an authenticated session."
 │
 ▼
No login required
```

The IdP immediately sends the user back to App B with a new authorization code.

```text
App B
  ↓
Authorization Code
  ↓
Tokens
  ↓
User authenticated
```

**That's SSO.**

---

# 6. Very important: the applications don't share tokens

A common misunderstanding is:

> "Does App A give its access token to App B?"

**No.**

Instead:

```text
             Identity Provider
                 /        \
                /          \
             App A        App B
```

App A gets its own tokens.

App B gets its own tokens.

For example:

```text
App A
 └── Access Token A

App B
 └── Access Token B
```

This is much safer.

An access token issued for App A shouldn't automatically be usable by App B.

---

# 7. What actually gets shared?

The important thing shared across applications is the **authentication session at the IdP**.

Conceptually:

```text
Browser
   │
   │ IdP Session Cookie
   ▼
Identity Provider
```

That session tells the IdP:

> "This user has already authenticated."

Therefore:

```text
App A → IdP → Login required
                   ↓
              IdP Session
                   ↓
App B → IdP → Login NOT required
                   ↓
App C → IdP → Login NOT required
```

---

# 8. SSO vs Single Logout

There's a related concept called **Single Logout (SLO)**.

SSO:

```text
Login once
   ↓
Access many applications
```

SLO:

```text
Logout once
   ↓
Logout from many applications
```

However, logout is more complicated than login because each application may have its own local session.

---

# 9. SSO vs Federation

These terms are related but aren't identical.

### SSO

Focuses on:

> One authentication session → multiple applications.

### Identity Federation

Focuses on:

> One identity provider establishing trust with applications/services, potentially across organizational boundaries.

For example:

```text
Company IdP
    │
    ├── Internal HR
    ├── Internal Jira
    ├── Internal Git
    └── External SaaS application
```

---

# 10. OIDC SSO flow — remember this

For your OAuth roadmap, this is the flow I'd memorize:

```text
                ┌─────────────────┐
                │ Identity        │
                │ Provider        │
                │ (OIDC)          │
                └────────┬────────┘
                         │
                    IdP Session
                         │
          ┌──────────────┼──────────────┐
          │              │              │
        App A           App B          App C
          │              │              │
       Tokens          Tokens         Tokens
          │              │              │
       Session         Session        Session
```

The **IdP authentication session** is what enables the SSO experience.

---

## 11. The key distinction

Don't think:

> **SSO = one token for everything**

Think:

> **SSO = one central authentication session that allows multiple applications to authenticate the same user without repeatedly asking for credentials.**

And in modern OAuth-based systems:

```text
OIDC
  ↓
Authentication
  ↓
Identity Provider session
  ↓
SSO across applications
```

Since you've already covered **OIDC, ID Tokens, UserInfo, scopes, JWK/JWKS, JWS, JWE and PoP**, SSO is basically where many of those concepts come together in a real-world architecture.
