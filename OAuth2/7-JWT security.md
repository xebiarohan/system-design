
# 9. JWT Security

First, keep this mental model:

```text
JWT = data + cryptographic proof

              JWT
               │
       ┌───────┴────────┐
       │                │
    Payload          Signature
       │                │
   "Who/what?"      "Was this
                     signed by
                     a trusted key?"
```

A JWT is **not automatically secure just because it is a JWT**.

The security comes from things like:

* choosing the correct signing algorithm
* securely storing signing keys
* verifying the signature
* validating claims
* limiting token lifetime
* preventing theft
* handling refresh/revocation correctly

---

# 1. Never Trust the Payload Without Verification

This is the most important rule.

A JWT might look like:

```text
eyJhbGciOiJIUzI1NiJ9.
eyJzdWIiOiIxMjMiLCJyb2xlIjoiYWRtaW4ifQ.
SIGNATURE
```

The payload might decode to:

```json
{
  "sub": "123",
  "role": "admin"
}
```

A beginner might think:

> "The JWT says the user is an admin, therefore they're an admin."

Wrong.

The payload is only trustworthy **after the server verifies the JWT's signature and validates the relevant claims**.

Conceptually:

```text
Client
  │
  │ JWT
  ▼
Server
  │
  ├── Decode
  │
  ├── Verify signature
  │
  ├── Validate claims
  │
  └── Accept identity/authorization information
```

Important distinction:

### Decoding ≠ verification

Decoding means:

```text
JWT
 ↓
Read JSON
```

Verification means:

```text
JWT
 ↓
Cryptographic verification
 ↓
Signature is valid
```

Anyone can decode a JWT.

That is not a security operation.

---

# 2. JWTs Are Usually Signed, Not Encrypted

This distinction is extremely important.

A normal signed JWT allows someone who possesses the token to read its payload.

For example:

```json
{
  "sub": "123",
  "email": "alice@example.com",
  "role": "admin"
}
```

The payload is **encoded**, not encrypted.

Therefore:

```text
JWT
 ↓
Base64URL decode
 ↓
Readable JSON
```

Do **not** put secrets into a normal JWT payload.

Bad:

```json
{
  "password": "secret123",
  "creditCard": "..."
}
```

Even with a valid signature, those values are not confidential.

If confidentiality is actually required, encryption mechanisms such as JWE may be relevant.

But for normal OAuth access tokens, the important property is generally **integrity/authenticity**, not hiding the claims.

---

# 3. The Signature Provides Integrity

Suppose the server issues:

```json
{
  "sub": "123",
  "role": "user"
}
```

and signs it.

An attacker might decode the token and change:

```json
{
  "sub": "123",
  "role": "admin"
}
```

But they don't have the signing key.

Therefore:

```text
Original payload
      ↓
   Signature A


Modified payload
      ↓
   Signature A
```

The signature no longer matches.

The server rejects it.

That's what the signature protects:

```text
"Issued by someone possessing the trusted signing key
and hasn't been modified."
```

---

# 4. Algorithm Confusion

This is an important class of JWT implementation vulnerabilities.

JWTs contain a header such as:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

The `alg` field tells the implementation which cryptographic algorithm is being used.

A dangerous implementation might blindly trust whatever algorithm the attacker specifies.

The principle you should remember is:

> **The server should explicitly configure which algorithms it accepts.**

For example:

```text
Expected algorithm:
RS256

Received:
RS256 → continue

Received:
HS256 → reject
```

Don't build authentication logic around:

```text
"Whatever algorithm the JWT says, I'll use."
```

Instead:

```text
Application configuration
        ↓
Allowed algorithms
        ↓
JWT verification
```

This is one reason mature JWT libraries should be configured carefully rather than simply calling a generic "verify token" function without understanding its options.

---

# 5. HS256 vs RS256 Security

You learned this in Topic 8, but it becomes particularly important here.

## HS256

```text
             Shared secret
             ┌───────────┐
             │           │
             ▼           ▼
          Service A   Service B

          sign       verify
```

The same secret is used to sign and verify.

Therefore, anyone who can verify tokens using that secret effectively possesses a key capable of signing tokens too.

This creates a trust boundary problem.

---

## RS256

```text
             Private key
                 │
                 ▼
               Sign


             Public key
                 │
                 ▼
              Verify
```

Only the private-key holder can create valid signatures.

Other services can receive the public key and verify tokens.

That's extremely useful in distributed systems.

For example:

```text
Authorization Server
       │
       │ private key
       ▼
    signs JWT
       │
       ▼
 ┌───────────────┐
 │ API Gateway   │
 │ Service A     │
 │ Service B     │
 │ Service C     │
 └───────────────┘
       │
       │ public key
       ▼
     verify
```

This is one reason asymmetric signing is common in OAuth/OIDC ecosystems.

---

# 6. Token Expiration

JWTs commonly contain:

```json
{
  "sub": "123",
  "iat": 1786320000,
  "exp": 1786323600
}
```

The important claim here is:

```text
exp
```

meaning **expiration time**.

The server should reject an expired token.

Conceptually:

```text
Current time < exp
        │
        ├── yes → potentially valid
        │
        └── no  → reject
```

Why does this matter?

Imagine an attacker steals:

```text
Access token
```

If that token never expires:

```text
Stolen token
    ↓
Attacker
    ↓
API
    ↓
Access forever
```

With a short lifetime:

```text
Stolen token
    ↓
Attacker
    ↓
API
    ↓
Eventually expires
```

This doesn't eliminate token theft, but it reduces the attack window.

---

# 7. Short-Lived Access Tokens

A common architecture is:

```text
Access Token
    ↓
short lifetime

Refresh Token
    ↓
longer lifetime
```

For example, conceptually:

```text
Access Token
   5–15 minutes

Refresh Token
   days/weeks
```

The exact lifetime depends on the system's risk model.

The reason for short-lived access tokens is:

> If an access token is stolen, minimize how long it can be used.

So:

```text
Login
  ↓
Access Token ────────────────┐
                             │
                             ▼
                         API calls
                             │
                             ▼
                         expires
                             │
                             ▼
                       Refresh Token
                             │
                             ▼
                     New Access Token
```

This leads directly into your later topic of **token lifecycle and refresh-token security**.

---

# 8. `iat` Is Not an Expiration Mechanism

You will often see:

```json
{
  "iat": 1786320000
}
```

`iat` means:

```text
Issued At
```

It tells you when the token was issued.

But this:

```json
{
  "iat": 1786320000
}
```

does **not** mean:

> "This token is valid for one hour."

You need an expiration policy, commonly represented by:

```json
{
  "iat": 1786320000,
  "exp": 1786323600
}
```

The application must actually validate `exp`.

---

# 9. Validate `iss`

`iss` means:

```text
Issuer
```

Example:

```json
{
  "iss": "https://auth.example.com"
}
```

Suppose your API trusts tokens issued by:

```text
https://auth.example.com
```

You don't want your API accepting a token issued by some unrelated authorization server.

Conceptually:

```text
Token says:

iss = auth.example.com

        ↓

Does this match our trusted issuer?

        ↓

Yes → continue
No  → reject
```

This is particularly important when your application interacts with multiple identity providers or authorization servers.

---

# 10. Validate `aud`

`aud` means:

```text
Audience
```

It answers:

> Who is this token intended for?

For example:

```json
{
  "aud": "payments-api"
}
```

Your API might be:

```text
payments-api
```

So:

```text
aud = payments-api
        ↓
Correct audience
        ↓
Potentially acceptable
```

But if:

```json
{
  "aud": "some-other-api"
}
```

your API should reject it.

This prevents a token intended for one service from being casually reused against another service.

---

# 11. `sub` — Who Is This Token About?

`sub` means:

```text
Subject
```

Example:

```json
{
  "sub": "user-123"
}
```

This identifies the principal the token represents.

Your application might do:

```text
sub = user-123
       ↓
Find user 123
       ↓
Apply permissions
```

A critical security principle:

> Don't let the client simply tell you which user they are.

For example, don't design an API like:

```http
GET /users/123/account
Authorization: Bearer ...
```

and then assume the token belongs to user 123.

Instead, the server should establish the authenticated identity from the validated token and then enforce authorization.

---

# 12. Token Theft

This is one of the biggest real-world JWT threats.

Suppose Alice has:

```text
Access Token = ABC123
```

An attacker obtains:

```text
ABC123
```

If the token is a bearer token, possession is generally enough to use it:

```text
Attacker
   │
   │ Authorization: Bearer ABC123
   ▼
API
   │
   ▼
Request accepted
```

The attacker doesn't necessarily need Alice's password.

That's why it's called a **bearer** token:

> Whoever possesses it can present it.

This means JWT security isn't only about cryptography.

You also have to protect the token itself.

---

# 13. Token Storage

This connects directly to browser security.

Suppose a web application stores an access token in:

```text
localStorage
```

If an attacker manages to execute malicious JavaScript through an XSS vulnerability, that script may be able to access the token.

Conceptually:

```text
XSS
 ↓
JavaScript
 ↓
localStorage
 ↓
Access Token
 ↓
Attacker
```

That's dangerous.

---

# 14. HttpOnly Cookies

A cookie can be configured:

```http
HttpOnly
Secure
SameSite=...
```

`HttpOnly` means browser JavaScript cannot directly read the cookie.

So:

```text
JavaScript
    │
    X
    │
HttpOnly Cookie
```

But the browser can still send the cookie with appropriate requests.

This can reduce token exposure to certain XSS-based token-theft scenarios.

However, cookies introduce another important security consideration:

```text
CSRF
```

So cookie-based authentication isn't simply:

> "Cookies are secure."

It's:

> Cookies have a different security model and need appropriate CSRF and SameSite protections.

---

# 15. HTTPS Is Mandatory

Never send bearer tokens over plaintext HTTP in production.

Bad:

```text
HTTP
 ↓
Authorization: Bearer TOKEN
```

An attacker able to observe the connection could potentially obtain the token.

Use:

```text
HTTPS/TLS
 ↓
Encrypted connection
 ↓
Bearer TOKEN
```

TLS protects the token **while it is being transported**.

But remember:

```text
HTTPS protects transmission.

It does NOT protect a token
that has already been stolen
from the browser, server, logs, etc.
```

---

# 16. Don't Put Access Tokens in URLs

Avoid:

```text
https://example.com/api?access_token=ABC123
```

URLs can end up in:

* browser history
* logs
* monitoring systems
* proxies
* analytics
* referrer-related contexts

Prefer:

```http
Authorization: Bearer ABC123
```

over HTTPS.

---

# 17. Don't Log Tokens

This is a surprisingly common production mistake.

Bad:

```text
Request:
Authorization: Bearer eyJhbGciOi...
```

and then your logging system stores it.

Now you've created:

```text
JWT
 ↓
Application logs
 ↓
Log aggregation system
 ↓
Many employees/services
```

If someone gets access to those logs, they may obtain usable bearer tokens.

Therefore, sensitive authentication headers should be redacted.

For example:

```text
Authorization: [REDACTED]
```

---

# 18. Replay Attacks

A replay attack occurs when an attacker captures a valid token/request and sends it again.

Example:

```text
Alice
 ↓
API
 ↓
Bearer Token ABC
```

Attacker captures:

```text
ABC
```

Then:

```text
Attacker
 ↓
API
 ↓
Bearer Token ABC
```

The server sees a valid token.

It may have no idea that the request is coming from the attacker.

This is one of the fundamental weaknesses of bearer tokens.

---

# 19. JWT Does Not Automatically Prevent Replay

This is a very important concept.

You might think:

> "The JWT has a signature, so replay is impossible."

No.

The signature proves:

```text
"This token is authentic and hasn't been modified."
```

It does not prove:

```text
"This request is coming from the original user/device."
```

Therefore:

```text
Valid token
      │
      ├── Alice → API ✓
      │
      └── Attacker → API ✓
```

if the attacker has stolen the token and the API accepts bearer tokens.

Advanced mechanisms such as sender-constrained/Proof-of-Possession tokens can address some of this, but that's much later in your roadmap.

---

# 20. JWT Revocation Is Complicated

Here's an important difference between sessions and JWTs.

With traditional server-side sessions:

```text
Session ID
    ↓
Server session store
```

You can delete:

```text
session_id = ABC
```

and the session immediately stops working.

JWTs are often designed to be stateless:

```text
JWT
 ↓
Signature valid?
 ↓
Claims valid?
 ↓
Accept
```

The server may not have a database record for every access token.

So suppose:

```text
JWT expires in 30 minutes
```

and the user clicks:

```text
Logout
```

If you simply delete some client-side copy, a stolen JWT might still technically be valid until its expiration.

This is why JWTs and revocation require careful architecture.

---

# 21. JWT Revocation Strategies

There are several approaches.

### Short-lived access tokens

Make access tokens expire quickly.

```text
Access Token
     ↓
short lifetime
```

This limits the damage window.

---

### Refresh-token revocation

Use short-lived access tokens and maintain stronger control over refresh tokens.

```text
Access Token
   ↓
expires quickly

Refresh Token
   ↓
server can revoke
```

This is a common architecture.

---

### Token denylist

You can maintain a list of revoked token identifiers.

For example:

```json
{
  "jti": "abc123"
}
```

Then:

```text
JWT
 ↓
Check signature
 ↓
Check expiration
 ↓
Check revocation list
 ↓
Accept/reject
```

But now you're introducing server-side state, which reduces some of the simplicity of completely stateless JWT validation.

---

# 22. `jti`

`jti` means:

```text
JWT ID
```

Example:

```json
{
  "sub": "123",
  "jti": "550e8400-e29b-41d4-a716-446655440000"
}
```

It gives the token a unique identifier.

It can be useful for:

* revocation
* tracking
* replay detection strategies
* audit systems

But simply having a `jti` claim does **not** automatically prevent replay.

Your application has to actually use it.

---

# 23. Don't Put Too Much Authorization Data in JWTs

You might have:

```json
{
  "sub": "123",
  "role": "admin",
  "permissions": [
    "delete_users",
    "create_users",
    "view_reports"
  ]
}
```

This can be convenient.

But imagine the user's permissions change:

```text
10:00
Alice = admin

10:05
Admin role removed
```

The old JWT may still say:

```json
{
  "role": "admin"
}
```

until it expires.

This is an important consequence of putting authorization information into self-contained tokens.

You need to decide how much authorization state can safely be cached inside a token and how quickly changes need to take effect.

---

# 24. Don't Confuse Authentication With Authorization

Suppose a JWT says:

```json
{
  "sub": "123"
}
```

After verifying it:

```text
Authenticated user = 123
```

That's authentication.

Now suppose the user requests:

```http
DELETE /users/456
```

You still need authorization:

```text
Is user 123 allowed to delete user 456?
```

The JWT doesn't magically answer that.

Your application needs authorization logic.

For example:

```text
Verify JWT
    ↓
Identify user
    ↓
Check permission
    ↓
Perform action
```

---

# 25. Don't Trust Client-Supplied Roles

This is a classic mistake.

Suppose the client sends:

```json
{
  "role": "admin"
}
```

and the server trusts it.

That's obviously dangerous.

Likewise, if a JWT contains:

```json
{
  "role": "admin"
}
```

the server must establish that this claim came from a **trusted issuer**, using a valid signature and the appropriate issuer/audience validation.

The general principle is:

```text
Client-controlled data
        ↓
Untrusted
```

A properly verified token from a trusted issuer:

```text
Trusted issuer
        ↓
Valid signature
        ↓
Valid claims
        ↓
Potentially trusted identity information
```

---

# 26. Key Management Is JWT Security

Imagine your server uses:

```text
SECRET = "super-secret-key"
```

If that key leaks:

```text
Attacker
   ↓
Obtains signing key
   ↓
Creates JWT
   ↓
Signs JWT
   ↓
Your API verifies it successfully
```

That's catastrophic.

Therefore signing keys need protection.

Don't:

```text
hard-code signing keys
commit them to Git
put them in source code
```

Instead use appropriate secret/key-management infrastructure.

For asymmetric signing:

```text
Private key
    ↓
Protect extremely carefully

Public key
    ↓
Can be distributed to verifiers
```

This is one of the major advantages of asymmetric cryptography.

---

# 27. Key Rotation

Suppose your current signing key is:

```text
Key A
```

Eventually you want to replace it:

```text
Key B
```

You don't necessarily want every existing token to suddenly become invalid.

A key rotation architecture can look like:

```text
                Authorization Server

                     Private Key B
                          │
                          ▼
                       Sign JWT


APIs
 │
 ├── Public Key A
 └── Public Key B
```

During a transition period, APIs can know about both keys.

This becomes particularly important when you learn:

```text
JWK
JWKS
kid
```

later in your roadmap.

---

# 28. `kid`

JWT headers can contain:

```json
{
  "alg": "RS256",
  "kid": "key-2026-01"
}
```

`kid` means:

```text
Key ID
```

It tells the verifier which public key should be used.

Conceptually:

```text
JWT
 │
 └── kid = key-2026-01
             │
             ▼
          JWKS
             │
             ▼
        Find public key
             │
             ▼
          Verify
```

This becomes extremely useful when you have multiple signing keys during key rotation.

---

# 29. Validate Time Carefully

JWTs often contain:

```text
exp
nbf
iat
```

Where:

```text
exp = expiration time
nbf = not valid before
iat = issued at
```

Your server should validate time-based claims according to its security policy.

There can also be small differences between server clocks.

For example:

```text
Authorization Server
       │
       │ token
       ▼
API Server
```

If their clocks differ by a few seconds, a token could appear slightly early or late.

Implementations commonly allow carefully bounded clock skew where appropriate.

Don't compensate with huge windows, though.

---

# 30. Don't Accept Tokens From Arbitrary Issuers

Imagine your API accepts:

```text
Issuer A
```

An attacker creates or obtains a token from:

```text
Issuer B
```

If your application doesn't properly validate the issuer and key/trust configuration, you can end up accepting an identity that your system never intended to trust.

So think of JWT validation as:

```text
JWT
 │
 ├── Signature valid?
 │
 ├── Algorithm allowed?
 │
 ├── Issuer trusted?
 │
 ├── Audience correct?
 │
 ├── Not expired?
 │
 ├── Not before valid?
 │
 └── Other application requirements?
       │
       ▼
     Accept
```

---

# 31. JWT Security Is a Validation Pipeline

This is probably the best mental model for your entire Topic 9.

Don't think:

```text
JWT → verify signature → done
```

Think:

```text
                JWT
                 │
                 ▼
          Parse safely
                 │
                 ▼
       Check allowed algorithm
                 │
                 ▼
          Verify signature
                 │
                 ▼
          Validate issuer
                 │
                 ▼
          Validate audience
                 │
                 ▼
         Validate expiration
                 │
                 ▼
        Validate other claims
                 │
                 ▼
       Authentication result
                 │
                 ▼
        Authorization checks
                 │
                 ▼
          Allow / Reject
```

That's much closer to how you should think about production JWT validation.

---

# 32. A Complete Example

Suppose you have:

```text
Frontend
   ↓
Authorization Server
   ↓
Access Token
   ↓
Payments API
```

The JWT might contain:

```json
{
  "iss": "https://auth.example.com",
  "sub": "user-123",
  "aud": "payments-api",
  "iat": 1786320000,
  "exp": 1786320900,
  "scope": "payments:read payments:write"
}
```

Header:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "2026-key-01"
}
```

The API receives:

```http
GET /payments
Authorization: Bearer eyJ...
```

The API performs something conceptually like:

```text
1. Extract bearer token
        ↓
2. Parse JWT
        ↓
3. Read kid
        ↓
4. Find trusted public key
        ↓
5. Ensure RS256 is allowed
        ↓
6. Verify signature
        ↓
7. Verify iss
        ↓
8. Verify aud
        ↓
9. Verify exp
        ↓
10. Verify any required claims
        ↓
11. Read sub
        ↓
12. Check scope
        ↓
13. Allow request
```

Notice something important:

### Authentication

```text
Who is this?

sub = user-123
```

### Authorization

```text
Can user-123 perform this operation?

scope = payments:read
```

Those are separate decisions.

---

# 33. What Happens If the Token Is Stolen?

Suppose the legitimate user gets:

```text
Access Token
    ↓
Attacker steals it
```

The attacker sends:

```http
Authorization: Bearer STOLEN_TOKEN
```

The API checks:

```text
Signature ✓
Issuer ✓
Audience ✓
Expiration ✓
```

Everything passes.

The API may have no way of knowing the attacker is not the legitimate user.

This is why JWT security has **two separate problems**:

### Problem 1 — Token integrity

```text
Can someone modify/create tokens?
```

Solved primarily through:

```text
Cryptographic signatures
```

### Problem 2 — Token confidentiality/theft

```text
Can someone steal a valid token?
```

Addressed through things such as:

```text
HTTPS
Secure storage
HttpOnly cookies where appropriate
XSS defenses
CSRF defenses where cookies are used
Short token lifetimes
Refresh-token protections
Careful logging
```

This distinction is extremely important.

---

# 34. JWT Security Checklist

When you eventually implement JWT authentication, mentally check:

### Cryptography

```text
□ Strong algorithm
□ Explicitly allow expected algorithms
□ Protect signing keys
□ Rotate keys appropriately
```

### Token validation

```text
□ Verify signature
□ Validate iss
□ Validate aud
□ Validate exp
□ Validate nbf where relevant
□ Validate required claims
```

### Token handling

```text
□ HTTPS
□ Don't put tokens in URLs
□ Don't log tokens
□ Protect browser storage
□ Short-lived access tokens
```

### Authorization

```text
□ Don't confuse authentication with authorization
□ Validate scopes/permissions
□ Don't blindly trust client input
□ Don't assume a valid JWT means every operation is allowed
```

### Lifecycle

```text
□ Access token expiration
□ Refresh-token security
□ Logout/revocation strategy
□ Key rotation
```

---

# 35. The 5 Things I Want You to Remember

If you forget everything else from Topic 9, remember these:

### 1. A JWT payload is not secret

```text
Signed ≠ encrypted
```

Never put sensitive secrets in an ordinary JWT.

### 2. Decoding isn't verification

```text
decode()
```

doesn't establish trust.

You need:

```text
signature verification
+
claim validation
```

### 3. A valid JWT isn't automatically authorized

```text
Valid JWT
    ↓
Who are you?
    ↓
Now ask:
What are you allowed to do?
```

### 4. A stolen bearer token is dangerous

```text
Token stolen
     ↓
Attacker possesses token
     ↓
Attacker can potentially use it
```

JWT cryptography doesn't automatically solve token theft.

### 5. JWT security is about the entire lifecycle

```text
       Issue
         ↓
       Store
         ↓
      Transport
         ↓
       Verify
         ↓
     Authorize
         ↓
      Expire
         ↓
      Refresh
         ↓
      Revoke
```

**That is JWT security.**

And this is exactly why your roadmap puts JWT security **before OAuth 2.0**: when you reach OAuth, you'll see that OAuth isn't just "get a token." You need to understand what that token represents, who issued it, who should accept it, how it's validated, how long it lives, and what happens if it gets stolen.
