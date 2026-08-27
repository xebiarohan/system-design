# Phase 5 — Identity Providers

**Goal:** Understand how real-world Identity Providers (IdPs) implement OAuth 2.0 and OpenID Connect (OIDC), and how your application integrates with them for login and authorization.

At this stage, you already understand OAuth 2.0 and OIDC conceptually. Now the goal is to understand what happens when you use an actual provider such as Google, GitHub, Microsoft, Auth0, Okta, or Keycloak.

---

## 1. What is an Identity Provider?

An **Identity Provider (IdP)** is a system that manages user identities and can authenticate users on behalf of your application.

Examples:

* Google
* Microsoft Entra ID
* GitHub
* Auth0
* Okta
* Keycloak

Your application does **not** necessarily need to manage the user's password itself.

Instead:

```text
Your Application

      ↓

Identity Provider

      ↓

User authenticates

      ↓

Identity Provider

      ↓

Your Application
```

For example:

```text
Your App
   ↓
Google
   ↓
User logs in with Google
   ↓
Google authenticates user
   ↓
Google sends authorization response
   ↓
Your App
```

The important idea is:

> Your application delegates authentication to an Identity Provider.

---

# 2. Why Use an Identity Provider?

Without an IdP, your application may need to implement:

* User registration
* Password hashing
* Password reset
* Email verification
* MFA
* Account recovery
* Login
* Session management
* Suspicious-login detection
* Social login
* Enterprise SSO

That's a lot of security-sensitive infrastructure.

An IdP can provide many of these capabilities.

Instead of:

```text
Your App
   ↓
Username/password
   ↓
Your Database
   ↓
Password verification
```

you can have:

```text
Your App
   ↓
Identity Provider
   ↓
User Authentication
```

Your application receives a trusted authentication result rather than handling the user's password directly.

---

# 3. Identity Provider vs OAuth Authorization Server

These terms are related but not exactly identical.

An **OAuth Authorization Server** issues OAuth tokens.

An **Identity Provider** provides identity/authentication capabilities.

With OIDC, the authorization server also participates in authentication and issues an **ID Token**.

Conceptually:

```text
OAuth

Authorization Server
        ↓
   Access Token
        ↓
Resource Server
```

OIDC adds:

```text
Identity Provider
        ↓
Authentication
        ↓
ID Token
        ↓
Your Application
```

So when you use:

```text
Login with Google
```

you're generally using **OpenID Connect on top of OAuth 2.0**, not simply OAuth authorization.

---

# 4. The Main Identity Providers

You don't need to memorize every provider-specific detail.

Instead, understand the common concepts and then learn how each provider implements them.

---

## Google

Google provides OAuth 2.0 and OpenID Connect capabilities.

Typical use case:

```text
Login with Google
```

Your application redirects the user to Google.

```text
Your App
   ↓
Google
   ↓
User logs in
   ↓
Google
   ↓
Your App
```

Google can provide identity information through OIDC.

Typical information includes:

* Subject identifier
* Email
* Name
* Profile information

The exact claims available depend on the scopes and provider configuration.

---

## GitHub

GitHub provides OAuth-based authorization.

A common example is:

```text
Login with GitHub
```

The important distinction is that GitHub's OAuth implementation should not automatically be treated as equivalent to a full OIDC identity provider.

For login, your application may need to obtain the user's profile information through GitHub's APIs after receiving an access token.

Conceptually:

```text
Your App
   ↓
GitHub Authorization
   ↓
Authorization Code
   ↓
Access Token
   ↓
GitHub API
   ↓
User Profile
```

This is an important comparison with OIDC providers.

---

## Microsoft

Microsoft provides identity services through Microsoft Entra ID.

Common use cases include:

* Microsoft account login
* Enterprise applications
* Organization login
* Microsoft 365 environments
* Enterprise SSO

A typical enterprise scenario is:

```text
Employee
   ↓
Your Company Application
   ↓
Microsoft Entra ID
   ↓
Company Login
   ↓
Your Application
```

---

## Auth0

Auth0 is an identity platform that can act as an authentication and authorization layer for applications.

It can integrate with multiple identity sources.

For example:

```text
Your Application
       ↓
     Auth0
       ↓
 ┌─────┼─────────┐
 ↓     ↓         ↓
Google GitHub  Enterprise IdP
```

This means your application can integrate primarily with Auth0 instead of implementing separate integrations for every identity provider.

---

## Okta

Okta is heavily used in enterprise identity and access management.

Typical scenarios include:

* Enterprise SSO
* Workforce identity
* OIDC
* OAuth
* SAML
* MFA

Example:

```text
Employee
   ↓
Company Application
   ↓
Okta
   ↓
Company Login
   ↓
Application
```

---

## Keycloak

Keycloak is an open-source identity and access management system.

It is particularly useful for learning because you can run your own identity server.

You can use Keycloak to experiment with:

* OAuth 2.0
* OIDC
* Clients
* Users
* Roles
* Groups
* Scopes
* Access tokens
* Refresh tokens
* ID tokens
* SSO

Conceptually:

```text
Your Application
       ↓
   Keycloak
       ↓
     User
```

For learning OAuth/OIDC, running Keycloak locally can be extremely useful.

---

# 5. The Common Identity Provider Architecture

Although providers differ, the architecture is usually similar.

```text
                    Identity Provider
                  ┌───────────────────┐
                  │                   │
User ────────────>│ Authentication    │
                  │                   │
                  │ Authorization     │
                  │                   │
                  │ Token Issuance     │
                  │                   │
                  └─────────┬─────────┘
                            │
                            ↓
                       Your App
```

Your application registers itself with the provider.

The provider gives your application configuration such as:

* Client ID
* Client secret, when applicable
* Authorization endpoint
* Token endpoint
* Issuer
* JWKS endpoint
* Redirect URI configuration

---

# 6. Registering Your Application

Before using an IdP, you normally create an application/client in the provider's dashboard.

For example:

```text
Identity Provider
       ↓
Create Application
       ↓
Client ID
Client Secret
Redirect URI
       ↓
Your Application
```

The provider needs to know:

> Which application is requesting authentication?

This is what the **client registration** represents.

---

# 7. Client ID

A **Client ID** identifies your application to the Identity Provider.

Example:

```text
client_id = my-web-application
```

It is generally not considered a password or secret.

Think:

```text
Client ID

"What application are you?"
```

rather than:

```text
Client ID

"What is your password?"
```

---

# 8. Client Secret

A **Client Secret** is a credential used by confidential clients.

For example:

```text
client_id
client_secret
```

The secret must be protected.

Never put a confidential client secret into browser JavaScript.

Bad:

```text
React Application
      ↓
client_secret = "SUPER_SECRET"
```

Users can inspect browser code.

Instead:

```text
Browser
   ↓
Your Backend
   ↓
Identity Provider
```

The backend can securely store the secret.

This is one reason you need to understand the difference between:

* Confidential clients
* Public clients

---

# 9. Redirect URI

The **Redirect URI** is one of the most important concepts when integrating an IdP.

Suppose your application starts login:

```text
https://myapp.com/login
```

The user is redirected to the provider:

```text
Your App
   ↓
Google
```

After authentication, Google needs to know where to send the user.

For example:

```text
https://myapp.com/oauth/callback
```

So:

```text
User
  ↓
Google
  ↓
Authentication
  ↓
https://myapp.com/oauth/callback
```

The redirect URI normally needs to be registered with the provider.

This prevents an attacker from changing the callback destination to an unauthorized location.

---

# 10. Authorization Code Flow with an Identity Provider

This is the flow you should understand extremely well.

Suppose:

```text
Alice
```

wants to log into:

```text
My Application
```

using Google.

The flow looks like:

```text
Alice
  │
  ↓
My Application
  │
  ↓
Google
  │
  ↓
Alice authenticates
  │
  ↓
Google
  │
  ↓
Authorization Code
  │
  ↓
My Application
  │
  ↓
Token Endpoint
  │
  ↓
Access Token + ID Token
```

More precisely:

```text
1. User clicks "Login with Google"

2. Application creates authorization request

3. Browser redirects to Google

4. Google authenticates the user

5. Google asks for consent if necessary

6. Google redirects browser back to application

7. Application receives authorization code

8. Application exchanges code for tokens

9. Provider returns tokens

10. Application validates the result

11. Application establishes its own application session
```

---

# 11. The Authorization Request

The application redirects the browser to the provider's authorization endpoint.

Conceptually:

```text
GET /authorize

?client_id=...
&redirect_uri=...
&response_type=code
&scope=openid profile email
&state=...
```

Important parameters:

### `client_id`

Identifies your application.

### `redirect_uri`

Where the provider should send the browser afterward.

### `response_type`

For Authorization Code Flow:

```text
response_type=code
```

### `scope`

Defines what access/identity information is being requested.

For OIDC:

```text
scope=openid
```

is fundamental.

Common additional scopes:

```text
profile
email
```

### `state`

Used to protect against request forgery and to correlate the login request with the callback.

---

# 12. What Happens at the Identity Provider?

The user is now interacting with the IdP.

For example:

```text
Google Login Page
```

The user might:

```text
Enter email
      ↓
Enter password
      ↓
Complete MFA
      ↓
Approve requested access
```

Your application should not receive the user's Google password.

Instead:

```text
User
   ↓
Google
   ↓
Authentication
```

This is one of the fundamental benefits of delegated authentication.

---

# 13. Authorization Code

After authentication, the provider redirects the browser back to your application.

For example:

```text
https://myapp.com/oauth/callback?code=ABC123&state=XYZ
```

The important value is:

```text
code=ABC123
```

The authorization code is temporary.

It is **not** the access token.

This distinction is extremely important.

```text
Authorization Code

        ↓

Exchange

        ↓

Access Token
```

---

# 14. Why Not Send the Access Token Directly?

Modern OAuth Authorization Code Flow intentionally separates:

```text
Authorization Code
```

from:

```text
Access Token
```

The application can exchange the code at the token endpoint.

With PKCE, the authorization request also contains a proof mechanism that binds the authorization response to the client that initiated the request.

Conceptually:

```text
Browser
   ↓
Authorization Code
   ↓
Application
   ↓
Token Endpoint
   ↓
Tokens
```

This is safer than exposing access tokens directly in the authorization response.

---

# 15. PKCE

You learned PKCE in Phase 3.

Now connect it to Identity Providers.

The client creates:

```text
code_verifier
```

Then derives:

```text
code_challenge
```

The authorization request contains:

```text
code_challenge
```

Later, the token request contains:

```text
code_verifier
```

The provider verifies that they match.

Conceptually:

```text
Client

code_verifier
      ↓
code_challenge
      ↓
Authorization Request
```

Later:

```text
Authorization Code
      +
code_verifier
      ↓
Token Endpoint
      ↓
Tokens
```

PKCE is especially important for public clients such as:

* Mobile applications
* Browser-based applications

Modern OAuth guidance generally favors Authorization Code + PKCE rather than older implicit-flow patterns.

---

# 16. ID Token vs Access Token

This is one of the most important things to understand in Phase 5.

An OIDC provider may return:

```text
Access Token
ID Token
Refresh Token
```

They have different purposes.

---

## Access Token

Used to access a resource/API.

```text
Application
    ↓
Access Token
    ↓
API
```

The access token answers:

> What can this client access?

---

## ID Token

Used by the client to obtain information about the authenticated user.

```text
Identity Provider
       ↓
    ID Token
       ↓
   Application
```

The ID token answers:

> Who authenticated?

It contains identity claims.

For example:

```json
{
  "iss": "https://example-idp.com",
  "sub": "123456789",
  "aud": "my-client",
  "exp": 1234567890,
  "iat": 1234567800
}
```

---

# 17. Never Use the ID Token as an API Access Token

This is a very common mistake.

Wrong:

```text
ID Token
   ↓
API
```

Correct:

```text
Access Token
   ↓
API
```

And:

```text
ID Token
   ↓
Client/Application
   ↓
Identity information
```

Think:

```text
ID Token

"Who is the user?"
```

versus:

```text
Access Token

"What resource/API may be accessed?"
```

---

# 18. Understanding the `sub` Claim

OIDC commonly uses:

```text
sub
```

as the subject identifier.

Example:

```json
{
  "sub": "248289761001"
}
```

This identifies the user within the issuer's identity namespace.

A common mistake is to use email as the permanent identity key:

```text
email = alice@example.com
```

Instead, understand the identity as something like:

```text
issuer + subject
```

Conceptually:

```text
iss + sub
```

This is important because email addresses can change.

---

# 19. Issuer (`iss`)

The:

```text
iss
```

claim identifies who issued the token.

Example:

```json
{
  "iss": "https://accounts.example.com"
}
```

Your application should verify that the issuer is the expected provider.

Conceptually:

```text
Token

iss = expected Identity Provider
```

If your application expects:

```text
https://idp.example.com
```

but receives:

```text
https://evil.example.com
```

the token should not be accepted.

---

# 20. Audience (`aud`)

The:

```text
aud
```

claim identifies the intended audience.

For an ID token, this is generally your client/application.

Example:

```json
{
  "aud": "my-client-id"
}
```

Your application should verify that the token was actually issued for it.

Think:

```text
iss

Who issued this?
```

and:

```text
aud

Who is this token intended for?
```

---

# 21. Token Signature Verification

Identity Providers commonly sign JWTs.

For example:

```text
ID Token
   ↓
JWT
   ↓
Signature
```

Your application must verify:

* Signature
* Issuer
* Audience
* Expiration
* Appropriate token type/context
* Other relevant claims

Do not simply decode the JWT and trust its payload.

Remember:

```text
Decode ≠ Verify
```

---

# 22. JWKS

Identity Providers commonly publish public signing keys through a **JWKS endpoint**.

JWKS means:

```text
JSON Web Key Set
```

Conceptually:

```text
Identity Provider
       ↓
Private Key
       ↓
Signs JWT

Identity Provider
       ↓
Public Key
       ↓
JWKS Endpoint
```

Your application can retrieve the provider's public keys and use them to verify JWT signatures.

Architecture:

```text
             Identity Provider
                    │
             ┌──────┴──────┐
             │             │
        Private Key     JWKS Endpoint
             │             │
             ↓             ↓
          Sign JWT     Public Keys
                           │
                           ↓
                      Your Backend
                           │
                     Verify JWT
```

This is why JWKS becomes important when integrating with real providers.

---

# 23. OIDC Discovery

OIDC providers can expose metadata describing their endpoints and capabilities.

A common discovery mechanism is:

```text
/.well-known/openid-configuration
```

The discovery document can tell your application where to find things such as:

* Authorization endpoint
* Token endpoint
* UserInfo endpoint
* JWKS URI
* Issuer
* Supported scopes
* Supported response types
* Supported algorithms

Conceptually:

```text
Your Application
       ↓
OIDC Discovery
       ↓
Provider Metadata
       ↓
Authorization Endpoint
Token Endpoint
JWKS Endpoint
UserInfo Endpoint
```

This is much better than hardcoding every endpoint manually.

---

# 24. UserInfo Endpoint

OIDC can provide a:

```text
UserInfo
```

endpoint.

The application can use an access token to request user information.

Conceptually:

```text
Access Token
     ↓
UserInfo Endpoint
     ↓
User Information
```

For example:

```json
{
  "sub": "123456",
  "name": "Alice",
  "email": "alice@example.com"
}
```

The exact claims depend on the provider and requested scopes.

---

# 25. Scopes

Scopes define what the application is requesting.

Common OIDC scopes:

```text
openid
profile
email
```

For example:

```text
scope=openid profile email
```

Think:

```text
openid
   ↓
Use OpenID Connect

profile
   ↓
Basic profile information

email
   ↓
Email-related information
```

OAuth scopes can also represent API permissions.

For example:

```text
read:orders
write:orders
```

So don't think of scopes as being exclusively about identity.

---

# 26. Consent

An Identity Provider may ask the user to approve requested permissions.

For example:

```text
My Application wants permission to:

✓ View your basic profile
✓ View your email address
```

The user can approve or deny the request.

Conceptually:

```text
Application
     ↓
Requested Scopes
     ↓
Identity Provider
     ↓
User Consent
     ↓
Authorization
```

Authentication and consent are different concepts.

The user may successfully authenticate but deny requested permissions.

---

# 27. Login vs Consent

These are often confused.

### Authentication

```text
"Who are you?"
```

### Authorization

```text
"What is this application allowed to access?"
```

For example:

```text
User logs into Google
        ↓
Authentication

User approves access to Google Drive
        ↓
Authorization
```

OIDC allows the authentication portion to be standardized.

OAuth handles delegated authorization.

---

# 28. Provider-Specific Differences

One of the most important skills in this phase is learning to distinguish:

```text
OAuth/OIDC standard
```

from:

```text
Provider-specific behavior
```

The standard defines concepts such as:

* Authorization endpoint
* Token endpoint
* Access tokens
* ID tokens
* Scopes
* Claims
* Redirect URIs
* PKCE

But providers may differ in:

* Available scopes
* User claims
* API permissions
* Token lifetimes
* Refresh-token behavior
* Account identifiers
* Consent behavior
* Endpoint URLs
* Supported algorithms
* Enterprise features

Therefore:

> Learn the protocol first, then learn the provider's implementation.

---

# 29. Login with Google — Mental Model

The complete architecture should now look like this:

```text
                    ┌─────────────────┐
                    │     Google      │
                    │                 │
                    │ Authentication  │
                    │ Authorization   │
                    │ Token Issuance  │
                    └────────┬────────┘
                             │
                             │
User ───────→ Your App ──────┘
                  │
                  │
                  ↓
          OAuth/OIDC Callback
                  │
                  ↓
             Your Backend
                  │
                  ↓
          Verify ID Token
                  │
                  ↓
          Find/Create User
                  │
                  ↓
        Application Session
```

Notice something important:

**Google authentication does not necessarily mean your application should use Google's tokens as its application session.**

Your application can establish its own session after successfully verifying the identity.

For example:

```text
Google Authentication
        ↓
Verified Identity
        ↓
Find/Create Local User
        ↓
Create Application Session
        ↓
Browser
```

---

# 30. Login with GitHub — Mental Model

GitHub commonly follows an OAuth authorization pattern:

```text
User
  ↓
Your App
  ↓
GitHub
  ↓
User authenticates
  ↓
Authorization Code
  ↓
Your App
  ↓
Access Token
  ↓
GitHub API
  ↓
User Information
```

This demonstrates an important lesson:

> OAuth by itself doesn't automatically give you a standardized identity token.

Your application needs to understand what the provider actually supports.

---

# 31. Mapping External Users to Local Users

This is an important real-world problem.

Suppose Alice logs in using Google:

```text
Google

sub = 12345
email = alice@example.com
```

Your database might contain:

```text
users

id
email
name
```

and:

```text
identity_accounts

user_id
provider
provider_subject
```

Conceptually:

```text
users
-----------------
id = 42
email = alice@example.com


identity_accounts
-----------------------------
user_id = 42
provider = google
subject = 12345
```

Now when Alice logs in again:

```text
Google
   ↓
sub = 12345
   ↓
Find identity_accounts
   ↓
user_id = 42
   ↓
Load local user
```

This allows your application to separate:

```text
External Identity

from

Local Application User
```

This distinction is extremely important in production systems.

---

# 32. Supporting Multiple Identity Providers

Suppose your application supports:

```text
Login with Google
Login with Microsoft
Login with GitHub
```

You could have:

```text
User
 │
 ├── Google identity
 │
 ├── Microsoft identity
 │
 └── GitHub identity
```

All three can map to one local account:

```text
                 Local User
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    Google      Microsoft     GitHub
    Identity     Identity     Identity
```

This is the basis for **account linking**.

---

# 33. Account Linking

Imagine:

```text
Alice signs up with Google
```

Later:

```text
Alice logs in with GitHub
```

Should these automatically become the same account?

**Not necessarily.**

Automatically linking accounts based only on matching email addresses can create security problems.

A safer design requires an authenticated account-linking process.

For example:

```text
Alice logs into existing account
        ↓
"Connect GitHub"
        ↓
GitHub authentication
        ↓
Verify GitHub identity
        ↓
Link GitHub identity
```

Then:

```text
One Local User
      │
 ┌────┴─────┐
 ↓          ↓
Google    GitHub
```

---

# 34. Multi-Tenant Identity

Enterprise applications often have multiple organizations.

For example:

```text
Your SaaS Application

Organization A
    ↓
Microsoft Entra ID

Organization B
    ↓
Okta

Organization C
    ↓
Google Workspace
```

Your application may need to determine:

```text
Which organization is this user from?
```

and:

```text
Which Identity Provider should authenticate them?
```

This leads to concepts such as:

* Tenants
* Organizations
* Domains
* Enterprise connections
* SSO configuration
* Identity federation

These become particularly important in B2B SaaS.

---

# 35. Identity Federation

Federation means trusting another identity system to authenticate users.

Conceptually:

```text
Your Application
       ↓
Identity Provider
       ↓
Another Identity Provider
       ↓
User
```

For example, a company may use:

```text
Your SaaS
    ↓
Okta
    ↓
Company Identity System
```

Your application doesn't need to know how the company's employees authenticate internally.

It trusts the configured identity provider.

---

# 36. Social Login vs Enterprise SSO

These are related but different use cases.

### Social Login

Examples:

```text
Login with Google
Login with GitHub
```

Usually aimed at consumer/developer applications.

### Enterprise SSO

Examples:

```text
Login with Microsoft Entra ID
Login with Okta
Login with corporate identity provider
```

Often involves:

* Organizations
* Employee accounts
* SAML
* OIDC
* SCIM
* MFA
* Centralized policies

The underlying identity concepts overlap, but enterprise identity introduces much more complexity.

---

# 37. SSO

**Single Sign-On (SSO)** allows a user to authenticate once and then access multiple applications.

Conceptually:

```text
                 Identity Provider
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
           App A      App B      App C
```

The user doesn't necessarily need to enter credentials separately into each application.

The IdP manages the central authentication experience.

---

# 38. MFA

An Identity Provider may also handle **Multi-Factor Authentication**.

For example:

```text
Username
   ↓
Password
   ↓
Authenticator / Security Key
   ↓
Authenticated
```

Your application may never directly handle the MFA process.

Instead:

```text
Your App
   ↓
Identity Provider
   ↓
MFA
   ↓
Authenticated User
   ↓
Your App
```

This is one reason organizations use centralized identity systems.

---

# 39. What Your Application Actually Trusts

This is perhaps the most important security concept in this phase.

Your application should not think:

```text
Google says this is Alice
    ↓
Trust everything
```

Instead, your application should verify the protocol response.

For OIDC, this typically includes checking things such as:

```text
Issuer
Audience
Signature
Expiration
Nonce
Authorization response
PKCE
Redirect URI
```

The exact checks depend on the flow and implementation.

The general principle is:

> Trust only after verification.

---

# 40. The `state` Parameter

Imagine:

```text
User starts login
```

Your application generates:

```text
state = RANDOM_VALUE
```

It sends:

```text
Your App
   ↓
IdP
   ↓
state=RANDOM_VALUE
```

The provider returns:

```text
Your App
   ↑
state=RANDOM_VALUE
```

Your application verifies:

```text
Returned state
       ==
Original state
```

This helps protect the authorization flow from request-forgery attacks.

Think:

```text
state

"Is this callback associated with the login request I started?"
```

---

# 41. The `nonce` Parameter

OIDC also uses a `nonce` to help protect against replay/substitution of ID tokens in the authentication flow.

Conceptually:

```text
Application
   ↓
Generate nonce
   ↓
Authorization Request
   ↓
Identity Provider
   ↓
ID Token containing nonce
   ↓
Application
   ↓
Verify nonce
```

Think:

```text
state

Protects/correlates the OAuth authorization flow
```

and:

```text
nonce

Binds the OIDC authentication response to the authentication request
```

You should understand both before implementing OIDC yourself.

---

# 42. Common Mistakes

These are worth explicitly remembering.

### Mistake 1 — Treating OAuth as authentication

```text
OAuth = Login
```

Not necessarily.

OIDC provides standardized authentication on top of OAuth.

---

### Mistake 2 — Using an ID token to call an API

Wrong:

```text
ID Token
   ↓
API
```

Usually:

```text
Access Token
   ↓
API
```

---

### Mistake 3 — Trusting a decoded JWT

Wrong:

```text
JWT
 ↓
Decode
 ↓
Trust
```

Correct:

```text
JWT
 ↓
Verify signature
 ↓
Validate claims
 ↓
Trust
```

---

### Mistake 4 — Storing client secrets in browser code

Wrong:

```text
Frontend
   ↓
client_secret
```

Secrets belonging to confidential clients must remain confidential.

---

### Mistake 5 — Not validating redirect URIs

Redirect URIs are security-sensitive.

Avoid:

```text
redirect_uri = anything supplied by user
```

Use explicitly registered and validated redirect URIs.

---

### Mistake 6 — Using email as the only identity key

Prefer a stable provider identity such as:

```text
issuer + subject
```

rather than assuming an email address is immutable.

---

### Mistake 7 — Automatically linking accounts

Don't blindly do:

```text
Google email == GitHub email
        ↓
Automatically merge accounts
```

Account linking should be an authenticated, deliberate operation.

---

# 43. Provider Comparison

A useful mental model is:

| Provider           | OAuth | OIDC                                           | Typical Use                      |
| ------------------ | ----- | ---------------------------------------------- | -------------------------------- |
| Google             | Yes   | Yes                                            | Social login / consumer identity |
| GitHub             | Yes   | Not traditionally the same OIDC role as Google | Developer applications           |
| Microsoft Entra ID | Yes   | Yes                                            | Enterprise identity              |
| Auth0              | Yes   | Yes                                            | Application identity platform    |
| Okta               | Yes   | Yes                                            | Enterprise identity              |
| Keycloak           | Yes   | Yes                                            | Self-hosted identity             |

The exact capabilities and terminology can evolve, so when integrating a provider, always read its current documentation rather than assuming all providers behave identically.

---

# 44. What You Should Be Able to Explain

Before leaving Phase 5, you should be able to answer:

### Identity Providers

* What is an Identity Provider?
* Why would an application use one?
* What does the IdP actually do?
* What is the difference between an IdP and your application?

### OAuth

* What is the authorization endpoint?
* What is the token endpoint?
* What is an authorization code?
* What is an access token?
* What is PKCE?
* What is `state`?

### OIDC

* What is an ID token?
* What is the difference between an ID token and access token?
* What is `openid` scope?
* What is `nonce`?
* What is the UserInfo endpoint?

### JWT

* How do you verify an ID token?
* What are `iss`, `sub`, `aud`, and `exp`?
* What is JWKS?
* How does your application obtain the provider's public keys?

### Users

* How do you map a Google user to a local user?
* Why shouldn't email necessarily be the identity key?
* How would you support Google + GitHub login?
* How would you implement account linking?

---

# 45. Practical Project — Login with Google

Now build it.

Don't start by writing your own OAuth implementation from scratch.

Use a well-maintained OAuth/OIDC library for your language/framework.

Your application should have:

```text
GET /login
```

and:

```text
GET /oauth/callback
```

Architecture:

```text
                 ┌──────────────┐
                 │    Google    │
                 └──────┬───────┘
                        │
                        │ OAuth/OIDC
                        │
┌──────────┐      ┌─────▼──────┐
│ Browser  │─────>│ Your App   │
└──────────┘      └─────┬──────┘
                        │
                        ↓
                  Local Database
```

Implement:

1. Register an application with the provider.
2. Configure the redirect URI.
3. Generate the authorization request.
4. Use Authorization Code Flow.
5. Use PKCE where appropriate.
6. Generate and validate `state`.
7. Generate and validate `nonce` for OIDC.
8. Receive the authorization code.
9. Exchange the code for tokens.
10. Validate the ID token.
11. Extract the stable user identity.
12. Find or create the local user.
13. Create your application's session.
14. Redirect the user into your application.

---

# 46. Practical Project — Login with GitHub

Build the same application using GitHub.

Compare:

```text
Google
```

with:

```text
GitHub
```

Pay attention to:

* Authorization endpoints
* Token endpoints
* Scopes
* User APIs
* Identity information
* Token format
* OIDC availability
* Provider-specific behavior

The purpose isn't just to make both buttons work.

The purpose is to understand:

> What does OAuth standardize, and what does each provider implement differently?

---

# 47. Practical Project — Keycloak Locally

Once Google and GitHub make sense, run Keycloak locally.

Create:

```text
Realm
   ↓
Client
   ↓
User
   ↓
Roles
   ↓
Scopes
```

Then connect your application:

```text
Your Application
       ↓
Keycloak
       ↓
OIDC
       ↓
Tokens
       ↓
Your Application
```

Inspect the actual:

* Authorization request
* Authorization code
* Token response
* ID token
* Access token
* JWT claims
* JWKS
* Discovery document

This will turn the abstract OAuth/OIDC concepts into something concrete.

---

# 48. The Complete Mental Model

At the end of this phase, you should be able to visualize:

```text
                         Identity Provider
                    ┌────────────────────────┐
                    │                        │
                    │ Authentication         │
                    │ Authorization          │
                    │ MFA                    │
                    │ User Identity          │
                    │ Token Issuance         │
                    │ JWKS                   │
                    │ OIDC Discovery         │
                    │                        │
                    └───────────┬────────────┘
                                │
                         OAuth / OIDC
                                │
                                ↓
                         Your Application
                                │
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
             Session       Local User       Permissions
                │               │               │
                └───────────────┼───────────────┘
                                ↓
                              APIs
```

The key separation is:

```text
Identity Provider
        ↓
Authenticates the user
        ↓
Issues protocol artifacts
        ↓
Your Application verifies them
        ↓
Your Application establishes its own identity/session
        ↓
Your Application authorizes actions
```

---

# Phase 5 Summary

You should now understand:

```text
Identity Provider
        ↓
User Authentication
        ↓
OAuth/OIDC Authorization
        ↓
Authorization Code
        ↓
Token Endpoint
        ↓
ID Token + Access Token
        ↓
Token Verification
        ↓
Stable User Identity
        ↓
Local User Account
        ↓
Application Session
        ↓
Application Authorization
```

The most important concepts to master are:

* Identity Provider
* Client registration
* Client ID
* Client secret
* Redirect URI
* Authorization endpoint
* Token endpoint
* Authorization Code
* PKCE
* `state`
* `nonce`
* Access Token
* ID Token
* UserInfo
* OIDC Discovery
* JWKS
* `iss`
* `sub`
* `aud`
* Token verification
* Local user mapping
* Account linking
* Social login
* Enterprise SSO
* Identity federation

Once these concepts are comfortable, **Phase 6 — API Security** becomes much easier because you'll understand exactly where tokens come from, who issues them, what they represent, and how your API should trust them.
