# Phase 4 — OpenID Connect (OIDC)

**Goal:** Understand how OAuth 2.0 is extended to provide **authentication and standardized user identity information**.

You have now completed OAuth 2.0 flows, so this is the perfect point to learn OIDC.

The biggest thing to understand is:

```text
OAuth 2.0
    ↓
Authorization
"Can this application access something?"

OIDC
    ↓
Authentication
"Who is the user?"
```

OIDC is **not a replacement for OAuth**.

It is an **identity layer built on top of OAuth 2.0**.

---

# 1. Why Does OIDC Exist?

This is the most important question to understand before learning the individual OIDC components.

Suppose you build an application:

```text
My Application
      ↓
Login with Google
```

You redirect the user to Google.

The user logs in:

```text
User
 ↓
Google
 ↓
Login
 ↓
Consent
```

Your application receives an OAuth access token.

Now you might think:

> "Great! I know who the user is."

But OAuth doesn't actually guarantee that.

OAuth primarily tells you:

```text
"This client has been granted access to some resources."
```

It doesn't standardize:

```text
"This is Rohan.
His email is rohan@example.com.
His unique Google user ID is 12345."
```

That's the gap OIDC fills.

---

# 2. OAuth vs OIDC

Let's make the distinction extremely clear.

## OAuth 2.0

OAuth answers:

> **"What is this client allowed to access?"**

Example:

```text
User
  ↓
Authorization Server
  ↓
"I authorize this application
to read my Google Calendar."
```

The application gets:

```text
Access Token
```

and uses it to call:

```text
Google Calendar API
```

---

## OIDC

OIDC answers:

> **"Who is the user?"**

Example:

```text
User
  ↓
Authorization Server
  ↓
Authentication
  ↓
ID Token
  ↓
Application
```

The application can receive information such as:

```text
User ID
Name
Email
```

---

# 3. The Key Difference

Remember this:

```text
OAuth 2.0

        "What can you access?"
                 ↓
            Access Token
                 ↓
              API
```

Whereas:

```text
OIDC

        "Who are you?"
                 ↓
             ID Token
                 ↓
           Application
```

This distinction will save you from a **lot** of confusion later.

---

# 4. OIDC Is Built on OAuth 2.0

OIDC doesn't create a completely separate authentication protocol.

Instead:

```text
                 OpenID Connect
                       │
                       ↓
                  OAuth 2.0
                       │
                       ↓
              Authorization Code
              PKCE
              Access Token
              Scopes
              etc.
```

OIDC adds standardized identity concepts on top.

The most important additions are:

* `openid` scope
* ID Token
* UserInfo endpoint
* Standard identity claims
* Authentication-related claims

---

# 5. OAuth + OIDC

Without OIDC:

```text
User
 ↓
OAuth Authorization
 ↓
Authorization Code
 ↓
Access Token
 ↓
Resource Server
```

With OIDC:

```text
User
 ↓
Authentication
 ↓
OAuth Authorization
 ↓
Authorization Code
 ↓
Access Token
 +
ID Token
 ↓
Application
```

The important addition is:

```text
ID Token
```

---

# 6. The `openid` Scope

This is the first OIDC concept you should understand.

In normal OAuth, you might request:

```text
scope=read:orders
```

OIDC introduces a special scope:

```text
scope=openid
```

When a client requests:

```text
openid
```

it is saying:

> **"I want to use OpenID Connect."**

For example:

```text
scope=openid profile email
```

means:

```text
openid
   ↓
Use OIDC

profile
   ↓
Request profile information

email
   ↓
Request email information
```

---

# 7. Why Is `openid` Special?

Because `openid` changes the nature of the OAuth request.

Without:

```text
openid
```

you're doing ordinary OAuth authorization.

With:

```text
openid
```

you're doing OIDC.

Conceptually:

```text
scope=orders:read

        ↓

OAuth
```

versus:

```text
scope=openid profile email

        ↓

OIDC
```

So remember:

> **`openid` is the scope that turns an OAuth request into an OIDC authentication request.**

---

# 8. The ID Token

The **ID Token** is the heart of OIDC.

It is a token containing information about the authenticated user and the authentication event.

It is usually a **JWT**.

You already learned JWT in your OAuth roadmap, so this should look familiar:

```text
Header.Payload.Signature
```

For example:

```text
eyJhbGciOiJSUzI1NiIs...
.
eyJpc3MiOiJodHRwczovL...
.
SflKxwRJSMeKKF2QT4fwp...
```

The important point is:

> **An ID Token is intended for the client application.**

An access token is intended for accessing APIs.

---

# 9. ID Token vs Access Token

This is probably the most important comparison in OIDC.

|                          | ID Token           | Access Token            |
| ------------------------ | ------------------ | ----------------------- |
| Purpose                  | Authentication     | API authorization       |
| Tells you                | Who authenticated  | What can be accessed    |
| Intended audience        | Client application | Resource server/API     |
| Usually JWT              | Yes                | Often, but not required |
| Contains identity claims | Yes                | Not necessarily         |
| Sent to API              | No, generally      | Yes                     |
| Defined by               | OIDC               | OAuth 2.0               |

Think:

```text
ID Token
    ↓
Application
    ↓
"Who is the user?"
```

while:

```text
Access Token
    ↓
API
    ↓
"What is this client allowed to do?"
```

---

# 10. Why Shouldn't I Send the ID Token to My API?

Because the ID Token isn't designed to authorize API requests.

Imagine:

```http
GET /api/orders
Authorization: Bearer ID_TOKEN
```

That's conceptually wrong.

The API should normally receive:

```http
GET /api/orders
Authorization: Bearer ACCESS_TOKEN
```

The ID Token is meant for the **client application** to understand the authentication result.

So:

```text
ID Token
    ↓
Client


Access Token
    ↓
Resource Server
```

---

# 11. What's Inside an ID Token?

Because the ID Token is normally a JWT, it contains claims.

Example:

```json
{
  "iss": "https://auth.example.com",
  "sub": "123456789",
  "aud": "my-client",
  "exp": 1780000000,
  "iat": 1779996400,
  "auth_time": 1779996300,
  "nonce": "abc123"
}
```

Let's understand these.

---

# 12. `iss` — Issuer

```json
{
  "iss": "https://auth.example.com"
}
```

`iss` means:

```text
Issuer
```

It identifies the Authorization Server / OpenID Provider that issued the token.

The application should verify that the issuer is the expected one.

Conceptually:

```text
ID Token
   ↓
Who issued this?
   ↓
iss
```

---

# 13. `sub` — Subject

This is one of the most important OIDC claims.

Example:

```json
{
  "sub": "123456789"
}
```

`sub` identifies the user.

But don't think of it as necessarily being:

```text
email@example.com
```

It is a stable identifier assigned by the OpenID Provider.

Your application might store:

```text
user_id = 123456789
```

and associate it with the local user account.

Think:

```text
sub
 ↓
Stable identity identifier
 ↓
"This specific user"
```

---

# 14. `aud` — Audience

Example:

```json
{
  "aud": "my-client"
}
```

`aud` tells you who the token is intended for.

For an ID Token, this should identify the client application.

Conceptually:

```text
ID Token
    ↓
Who is this token meant for?
    ↓
aud
```

Your application should verify that the audience matches its own client ID.

---

# 15. `exp` — Expiration

Example:

```json
{
  "exp": 1780000000
}
```

This tells you when the ID Token expires.

Your application must ensure:

```text
Current Time < exp
```

Otherwise:

```text
ID Token
    ↓
Expired
    ↓
Do not accept
```

---

# 16. `iat` — Issued At

Example:

```json
{
  "iat": 1779996400
}
```

This indicates when the ID Token was issued.

Conceptually:

```text
iat
 ↓
When was this token created?
```

---

# 17. `auth_time`

Example:

```json
{
  "auth_time": 1779996300
}
```

This indicates when the user authentication occurred.

This can be useful when an application needs to know how recently the user authenticated.

For example:

```text
User wants to change password
        ↓
Was the user recently authenticated?
        ↓
Check auth_time
```

---

# 18. `nonce`

You'll encounter `nonce` especially when working with browser-based OIDC flows.

Conceptually:

```text
Client
 ↓
Generate random nonce
 ↓
Authorization Request
 ↓
Authorization Server
 ↓
ID Token
 ↓
nonce returned
```

The client verifies that the nonce in the ID Token matches the one it originally generated.

This helps protect against certain replay and token-injection attacks.

Think:

```text
Client generated:
nonce = ABC123

ID Token contains:
nonce = ABC123

        ↓

Matches
        ↓
Good
```

If they don't match:

```text
ABC123
   ≠
XYZ999

   ↓

Reject
```

---

# 19. Standard Identity Claims

OIDC defines standard claims that can describe the user.

For example:

```json
{
  "sub": "123456789",
  "name": "Rohan Aggarwal",
  "given_name": "Rohan",
  "family_name": "Aggarwal",
  "email": "rohan@example.com"
}
```

Some common claims include:

* `sub`
* `name`
* `given_name`
* `family_name`
* `middle_name`
* `nickname`
* `preferred_username`
* `profile`
* `picture`
* `website`
* `email`
* `email_verified`
* `gender`
* `birthdate`
* `zoneinfo`
* `locale`

Not every provider will return every claim.

---

# 20. The `profile` Scope

The `profile` scope is used to request standard profile information.

For example:

```text
scope=openid profile
```

The provider may return claims such as:

```text
name
given_name
family_name
preferred_username
picture
locale
```

The exact claims available depend on the provider and its configuration.

Think:

```text
profile
   ↓
Basic user profile information
```

---

# 21. The `email` Scope

The `email` scope requests email-related information.

For example:

```text
scope=openid email
```

The provider may return:

```json
{
  "email": "rohan@example.com",
  "email_verified": true
}
```

The important distinction is:

```text
email
    ↓
User's email address

email_verified
    ↓
Whether the provider has verified that email
```

Don't assume that simply having an `email` claim means the email is verified.

Check:

```text
email_verified
```

when that distinction matters.

---

# 22. Combining Scopes

A typical OIDC request might contain:

```text
scope=openid profile email
```

Meaning:

```text
openid
   ↓
I want OIDC authentication

profile
   ↓
I want profile information

email
   ↓
I want email information
```

You might see this in an authorization URL:

```text
https://auth.example.com/authorize
    ?client_id=my-client
    &response_type=code
    &scope=openid%20profile%20email
    &redirect_uri=https://myapp.com/callback
```

---

# 23. OIDC Authorization Code Flow

Since you've already learned Authorization Code Flow, OIDC should now be easy to visualize.

Normal OAuth:

```text
User
 ↓
Client
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Access Token
```

OIDC:

```text
User
 ↓
Client
 ↓
OpenID Provider
 ↓
Authorization Code
 ↓
Access Token
 +
ID Token
```

The big addition is:

```text
ID Token
```

---

# 24. Complete OIDC Flow

Let's go through the entire thing.

### Step 1 — User wants to log in

```text
User
 ↓
My Application
 ↓
"Login with Google"
```

---

### Step 2 — Application redirects the user

The application sends the user to the OpenID Provider.

For example:

```text
https://accounts.example.com/authorize
```

with parameters such as:

```text
client_id
redirect_uri
response_type=code
scope=openid profile email
state
nonce
```

If using PKCE, you'll also have:

```text
code_challenge
code_challenge_method
```

---

### Step 3 — User authenticates

```text
User
 ↓
OpenID Provider
 ↓
Username / Password
 ↓
MFA
```

The provider authenticates the user.

---

### Step 4 — User authorizes

The user may see:

```text
"My Application wants access to your profile
and email."
```

The user approves.

---

### Step 5 — Authorization Code

The provider redirects the browser:

```text
OpenID Provider
      ↓
Authorization Code
      ↓
My Application
```

For example:

```text
https://myapp.com/callback?code=ABC123
```

---

### Step 6 — Exchange Code for Tokens

The application exchanges the authorization code.

The response may contain:

```json
{
  "access_token": "ACCESS_TOKEN",
  "id_token": "ID_TOKEN",
  "refresh_token": "REFRESH_TOKEN",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Depending on the flow and provider, a refresh token may or may not be returned.

---

# 25. What Does the Application Do With the ID Token?

The application validates it.

It should verify things such as:

```text
Signature
   ↓
Is the token authentic?

iss
   ↓
Was it issued by the expected provider?

aud
   ↓
Was it issued for my application?

exp
   ↓
Is it still valid?

nonce
   ↓
Does it match the original authentication request?
```

Only after appropriate validation should the application trust the identity information.

---

# 26. ID Token Validation

This is extremely important.

Never do this:

```text
Decode JWT
     ↓
Read "email"
     ↓
Trust it
```

Remember what you learned about JWT:

> **Decoding a JWT does not verify it.**

Correct approach:

```text
ID Token
    ↓
Parse
    ↓
Verify signature
    ↓
Verify issuer
    ↓
Verify audience
    ↓
Verify expiration
    ↓
Verify nonce when applicable
    ↓
Trust claims
```

---

# 27. Where Does the Public Key Come From?

You already learned about:

```text
RSA
Public Key
Private Key
```

OIDC providers publish their public signing keys through a **JWKS endpoint**.

Conceptually:

```text
OpenID Provider
      ↓
JWKS Endpoint
      ↓
Public Keys
      ↓
Application
      ↓
Verify ID Token Signature
```

For example:

```text
/.well-known/jwks.json
```

The exact URL depends on the provider.

This connects directly with the advanced OAuth topic in your roadmap:

```text
JWK / JWKS
```

So you'll get to revisit that later with much more context.

---

# 28. The UserInfo Endpoint

The second major OIDC concept is the **UserInfo endpoint**.

The UserInfo endpoint is an OAuth-protected endpoint that returns claims about the authenticated user.

Conceptually:

```text
Client
   ↓
Access Token
   ↓
UserInfo Endpoint
   ↓
User Information
```

Example:

```http
GET /userinfo
Authorization: Bearer ACCESS_TOKEN
```

Response:

```json
{
  "sub": "123456789",
  "name": "Rohan Aggarwal",
  "email": "rohan@example.com",
  "email_verified": true
}
```

---

# 29. ID Token vs UserInfo Endpoint

This is another important distinction.

### ID Token

Identity information is delivered:

```text
Authorization Server
       ↓
ID Token
       ↓
Client
```

### UserInfo

Identity information is retrieved:

```text
Client
   ↓
Access Token
   ↓
UserInfo Endpoint
   ↓
User Claims
```

So:

```text
ID Token
   ↓
Identity information delivered during token response
```

while:

```text
UserInfo
   ↓
Identity information retrieved through an API
```

---

# 30. Why Have Both?

Because they solve slightly different problems.

The ID Token contains information about:

```text
Authentication
+
Identity
```

The UserInfo endpoint provides:

```text
User claims
```

through an OAuth-protected API.

This can be useful when the application needs user information beyond what's included in the ID Token.

---

# 31. UserInfo Must Be Called With an Access Token

The request looks like:

```http
GET /userinfo
Authorization: Bearer ACCESS_TOKEN
```

Notice again:

```text
Access Token
      ↓
UserInfo
```

Not:

```text
ID Token
      ↓
UserInfo
```

The UserInfo endpoint is a protected resource.

---

# 32. `sub` Is Extremely Important

Suppose the UserInfo endpoint returns:

```json
{
  "sub": "123456789",
  "email": "rohan@example.com"
}
```

Your application should primarily identify the user using:

```text
sub
```

rather than blindly using:

```text
email
```

Why?

Because:

```text
sub
 ↓
Stable provider-specific identifier
```

Email addresses can change.

For example:

```text
Old:
rohan@example.com

New:
rohan@company.com
```

But the provider may continue identifying the same user using:

```text
sub = 123456789
```

---

# 33. Issuer + Subject

In real applications, you'll often think of a user's external OIDC identity as:

```text
issuer + subject
```

For example:

```text
iss:
https://accounts.example.com

sub:
123456789
```

Together:

```text
https://accounts.example.com
+
123456789
```

identify the user within that identity provider's namespace.

This becomes particularly important when your application supports multiple identity providers.

---

# 34. OIDC Provider

OIDC introduces the concept of an **OpenID Provider**, often abbreviated:

```text
OP
```

The OpenID Provider is the entity that:

* Authenticates the user
* Issues ID Tokens
* Issues access tokens
* Provides UserInfo
* Publishes OIDC configuration
* Publishes signing keys

Examples include:

```text
Google
Microsoft Entra ID
Auth0
Okta
Keycloak
```

Conceptually:

```text
                OpenID Provider
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
     Authentication  Tokens      UserInfo
          │            │            │
          ↓            ↓            ↓
        User        Client        Client
```

---

# 35. OAuth Roles vs OIDC Roles

You learned these OAuth roles:

```text
Resource Owner
Client
Authorization Server
Resource Server
```

OIDC adds terminology around the identity provider.

A useful mental model is:

```text
User
 ↓
Resource Owner
```

```text
OIDC Provider
 ↓
Authorization Server
+
Authentication / Identity Provider
```

The OIDC provider performs both authorization-server and identity-provider responsibilities.

---

# 36. Discovery Document

OIDC also provides a standardized way for clients to discover provider configuration.

Usually through:

```text
/.well-known/openid-configuration
```

The discovery document can tell the client things such as:

```text
authorization_endpoint
token_endpoint
userinfo_endpoint
jwks_uri
issuer
scopes_supported
response_types_supported
```

Conceptually:

```text
Client
   ↓
/.well-known/openid-configuration
   ↓
"Where are all the OIDC endpoints?"
```

This is extremely useful because the application doesn't have to hard-code every endpoint.

---

# 37. Example Discovery Document

A simplified example:

```json
{
  "issuer": "https://auth.example.com",
  "authorization_endpoint": "https://auth.example.com/authorize",
  "token_endpoint": "https://auth.example.com/token",
  "userinfo_endpoint": "https://auth.example.com/userinfo",
  "jwks_uri": "https://auth.example.com/.well-known/jwks.json"
}
```

Your application can use this information to configure OIDC.

---

# 38. OIDC Metadata

The discovery document is particularly useful when integrating multiple providers.

For example:

```text
Google
   ↓
Discovery

Microsoft
   ↓
Discovery

Keycloak
   ↓
Discovery
```

Your application can discover:

```text
Authorization endpoint
Token endpoint
UserInfo endpoint
JWKS endpoint
Issuer
```

automatically.

---

# 39. Authentication vs Authorization — Final Understanding

Now let's connect everything you've learned.

### Authentication

```text
Who are you?
```

OIDC helps answer this.

```text
User
 ↓
Identity Provider
 ↓
ID Token
 ↓
Application
```

---

### Authorization

```text
What are you allowed to access?
```

OAuth helps answer this.

```text
Client
 ↓
Access Token
 ↓
Resource Server
```

---

# 40. One User Login Can Produce Multiple Things

This is an important mental model.

After a successful OIDC flow, the client may receive:

```text
                  Token Endpoint
                       │
                       ↓
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Access Token   ID Token   Refresh Token
          │            │            │
          ↓            ↓            ↓
        API         Client       Token
                                 Endpoint
```

Each token has a different purpose.

---

# 41. Access Token

```text
Access Token
      ↓
Resource Server
      ↓
"Can this client access this resource?"
```

---

# 42. ID Token

```text
ID Token
      ↓
Client Application
      ↓
"Who authenticated?"
```

---

# 43. Refresh Token

```text
Refresh Token
      ↓
Authorization Server
      ↓
"Give me a new access token."
```

This three-token distinction is worth memorizing.

---

# 44. Example — Login With Google

Imagine your application has:

```text
Login with Google
```

The user clicks:

```text
Login
 ↓
Google
```

Google authenticates the user.

Your application receives:

```text
Authorization Code
```

Then exchanges it for:

```text
Access Token
+
ID Token
+
possibly Refresh Token
```

Your application validates the ID Token:

```text
iss
aud
exp
nonce
signature
```

Then:

```text
ID Token
 ↓
sub = Google user ID
 ↓
Find/create local user
 ↓
Create application session
```

The Access Token can be used when the application needs to call a Google API.

This is the real-world pattern behind:

```text
"Sign in with Google"
```

---

# 45. OIDC Login Architecture

A typical architecture looks like:

```text
                    OpenID Provider
                         │
              ┌──────────┴──────────┐
              │                     │
         Authentication        Token Issuance
              │                     │
              ↓                     ↓
             User             Access + ID Token
                                    │
                                    ↓
                              Client Application
                                    │
                         ┌──────────┴──────────┐
                         ↓                     ↓
                    ID Token              Access Token
                         ↓                     ↓
                    Authenticate         Call APIs
```

---

# 46. OIDC vs OAuth — The Interview Version

If someone asks:

> **"What's the difference between OAuth and OIDC?"**

A strong answer is:

> OAuth 2.0 is an authorization framework that allows a client to obtain access to protected resources. It doesn't standardize user authentication. OpenID Connect builds on OAuth 2.0 and adds an identity layer, primarily through the ID Token, allowing applications to verify the user's identity and obtain standardized identity claims.

Short version:

```text
OAuth
 ↓
Authorization

OIDC
 ↓
Authentication + Identity
```

---

# 47. Common OIDC Mistakes

## Mistake 1 — Using OAuth for login

Saying:

```text
"OAuth is a login protocol."
```

is not technically correct.

Better:

```text
OAuth
 ↓
Authorization

OIDC
 ↓
Authentication
```

---

## Mistake 2 — Using the Access Token as the user's identity

Wrong mental model:

```text
Access Token
 ↓
Who is the user?
```

Better:

```text
ID Token
 ↓
Who authenticated?
```

---

## Mistake 3 — Sending ID Token to APIs

Don't use:

```http
Authorization: Bearer ID_TOKEN
```

when the API expects an access token.

Use:

```http
Authorization: Bearer ACCESS_TOKEN
```

---

## Mistake 4 — Trusting decoded ID Token claims

Wrong:

```text
Decode JWT
 ↓
Read email
 ↓
Trust it
```

Correct:

```text
ID Token
 ↓
Verify signature
 ↓
Verify iss
 ↓
Verify aud
 ↓
Verify exp
 ↓
Verify nonce when applicable
 ↓
Trust claims
```

---

## Mistake 5 — Using email as the primary identity

Don't blindly assume:

```text
email = user identity
```

Prefer:

```text
iss + sub
```

as the external identity.

---

# 48. OAuth + OIDC — Complete Mental Model

You can now visualize the whole thing:

```text
                         USER
                           │
                           ↓
                  OpenID Provider
                           │
                    Authenticate
                           │
                           ↓
                   Authorization
                           │
                           ↓
                  Authorization Code
                           │
                           ↓
                     Token Endpoint
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
         Access Token   ID Token   Refresh Token
              │            │            │
              ↓            ↓            ↓
          Resource       Client       Token
           Server      Application    Endpoint
```

And the purpose of each is:

```text
Access Token
     ↓
API authorization


ID Token
     ↓
User authentication / identity


Refresh Token
     ↓
Obtain new access token
```

---

# 49. What You Should Remember

If you remember only these concepts from Phase 4, make sure you know these:

### 1. OIDC is built on OAuth 2.0

```text
OIDC
 ↓
OAuth 2.0
```

---

### 2. OAuth is authorization

```text
"What can this client access?"
```

---

### 3. OIDC adds authentication

```text
"Who is the user?"
```

---

### 4. `openid` is the important scope

```text
scope=openid
```

means:

```text
Use OpenID Connect
```

---

### 5. ID Token is for the client

```text
ID Token
 ↓
Client
 ↓
User identity
```

---

### 6. Access Token is for APIs

```text
Access Token
 ↓
Resource Server
 ↓
Protected Resource
```

---

### 7. `sub` identifies the user

```text
sub
 ↓
Stable user identifier
```

---

### 8. UserInfo provides user claims

```text
Access Token
 ↓
UserInfo endpoint
 ↓
User claims
```

---

### 9. Validate the ID Token

At minimum understand:

```text
Signature
iss
aud
exp
nonce
```

---

### 10. Discovery makes integration easier

```text
/.well-known/openid-configuration
```

provides provider metadata such as:

```text
Authorization endpoint
Token endpoint
UserInfo endpoint
JWKS endpoint
Issuer
```

---

# 50. Final Mental Model

Your entire Phase 3 + Phase 4 knowledge can now be summarized as:

```text
                         OAuth 2.0
                            │
                  ┌─────────┴─────────┐
                  │                   │
                  ↓                   ↓
             Authorization       OAuth Flows
                  │                   │
                  ↓                   ↓
             Access Token    Authorization Code
                             PKCE
                             Client Credentials
                             Device Flow
                             Refresh Token
                  │
                  ↓
             Resource Server
                  │
                  ↓
                APIs
```

Then OIDC adds:

```text
                         OIDC
                          │
                          ↓
                    OAuth 2.0 +
                  Identity Layer
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
         ID Token      UserInfo      Claims
             │            │
             ↓            ↓
          Client       Access Token
             │
             ↓
       "Who is the user?"
```

So the simplest way to remember the entire concept is:

```text
OAuth 2.0
    ↓
"Can this client access the resource?"

OIDC
    ↓
"Who is the user?"
```

And the three most important artifacts are:

```text
Access Token
    ↓
API authorization

ID Token
    ↓
User authentication / identity

Refresh Token
    ↓
Get a new Access Token
```

Once these distinctions are crystal clear, **"Login with Google", "Login with Microsoft", SSO, Auth0, Okta, Keycloak, and enterprise identity systems** become much easier to understand because they are largely implementations and combinations of these concepts.
