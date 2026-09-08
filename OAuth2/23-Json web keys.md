The key question we're solving is:

> **"The Authorization Server signed this JWT with a private key. How does my Resource Server get the correct public key needed to verify it?"**

That is where **JWK and JWKS** come in.

---

# 1. First: Why Do We Need JWK/JWKS?

Let's start from what you already know.

Suppose your Authorization Server issues a JWT:

```text
Authorization Server
        |
        | signs JWT
        ↓
     JWT Access Token
        |
        ↓
 Resource Server
```

The JWT might look like:

```text
eyJhbGciOiJSUzI1NiIsImtpZCI6IjEyMyJ9
.
eyJzdWIiOiIxMjM0NSIsInNjb3BlIjoicmVhZCJ9
.
signature...
```

You learned earlier that with **RS256**:

```text
Private Key
    ↓
Sign JWT
```

and:

```text
Public Key
    ↓
Verify JWT
```

So the Resource Server needs the Authorization Server's **public key**.

The naive solution would be:

```text
Resource Server
    |
    | "Give me your public key"
    ↓
Authorization Server
```

But how should the Authorization Server expose the key?

And what happens when the key changes?

That's where **JWK/JWKS** were designed to help.

---

# 2. What Is a JWK?

**JWK = JSON Web Key**

It is simply a **JSON representation of a cryptographic key**.

For example, conceptually:

```json
{
  "kty": "RSA",
  "kid": "key-123",
  "use": "sig",
  "alg": "RS256",
  "n": "...",
  "e": "AQAB"
}
```

This describes an RSA public key.

Think:

```text
RSA Public Key
       ↓
   represented as
       ↓
      JWK
       ↓
     JSON
```

So:

> **JWK is a standardized JSON representation of one cryptographic key.**

---

# 3. What Is JWKS?

**JWKS = JSON Web Key Set**

A JWKS is simply a **collection of JWKs**.

Think:

```text
JWK
 ↓
one key


JWKS
 ↓
multiple JWKs
```

For example:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "key-001",
      "use": "sig",
      "alg": "RS256",
      "n": "...",
      "e": "AQAB"
    },
    {
      "kty": "RSA",
      "kid": "key-002",
      "use": "sig",
      "alg": "RS256",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

The important structure is:

```text
JWKS
 └── keys
      ├── JWK
      ├── JWK
      └── JWK
```

---

# 4. Why Does JWKS Usually Contain Multiple Keys?

This is where things get interesting.

Imagine your Authorization Server currently uses:

```text
Private Key A
```

to sign JWTs.

Resource Servers verify them using:

```text
Public Key A
```

Everything is fine.

But eventually you need to rotate the key.

```text
Old Key A
     ↓
New Key B
```

You can't simply remove Key A immediately.

Why?

Because tokens signed with Key A may still be valid.

So you might temporarily publish:

```text
JWKS

Key A → old
Key B → new
```

Then:

```text
New tokens
     ↓
Private Key B
     ↓
kid = key-B
```

while old tokens:

```text
Old tokens
     ↓
Private Key A
     ↓
kid = key-A
```

The Resource Server can verify both.

This is one of the **most important reasons JWKS exists**.

---

# 5. The `kid` — Key ID

You've probably noticed this field:

```json
"kid": "key-123"
```

`kid` means:

> **Key ID**

It identifies which key was used.

Suppose your JWT header is:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key-2026-01"
}
```

The Resource Server reads:

```text
kid = key-2026-01
```

Then it looks at the JWKS:

```json
{
  "keys": [
    {
      "kid": "key-2025-12",
      ...
    },
    {
      "kid": "key-2026-01",
      ...
    }
  ]
}
```

It finds:

```text
key-2026-01
```

and uses that public key to verify the signature.

So:

```text
JWT
 |
 | read kid
 ↓
"key-2026-01"
 |
 ↓
JWKS
 |
 | find matching kid
 ↓
JWK
 |
 ↓
Public Key
 |
 ↓
Verify JWT
```

That's the core mechanism.

---

# 6. Complete JWT + JWKS Flow

Let's put everything together.

Suppose you have:

```text
                 Authorization Server
                         |
                  Private Key B
                         |
                         ↓
                    Sign JWT
                         |
                         ↓
                       JWT
                         |
                         ↓
                       Client
                         |
                         ↓
                  Resource Server
```

The JWT header contains:

```json
{
  "alg": "RS256",
  "kid": "key-B"
}
```

The Resource Server knows:

```text
Issuer:
https://auth.example.com
```

The Authorization Server exposes a JWKS endpoint such as:

```text
https://auth.example.com/.well-known/jwks.json
```

The Resource Server obtains:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "key-A",
      ...
    },
    {
      "kty": "RSA",
      "kid": "key-B",
      ...
    }
  ]
}
```

It finds:

```text
kid = key-B
```

Then:

```text
JWT
 ↓
kid = key-B
 ↓
JWKS
 ↓
JWK key-B
 ↓
RSA Public Key
 ↓
Verify JWT signature
```

---

# 7. What Is Inside a JWK?

A JWK contains metadata describing a cryptographic key.

For an RSA key, you'll commonly see:

```json
{
  "kty": "RSA",
  "kid": "key-123",
  "use": "sig",
  "alg": "RS256",
  "n": "...",
  "e": "AQAB"
}
```

Let's understand each.

---

## `kty`

Key type.

Example:

```json
"kty": "RSA"
```

Means:

```text
RSA key
```

Other possibilities include:

```text
RSA
EC
OKP
oct
```

We'll get into these later.

---

# 8. `kid`

Key identifier.

```json
"kid": "key-123"
```

Used to identify the key.

Think:

```text
kid = database primary key for cryptographic keys
```

Not literally a database ID, but conceptually that's a useful mental model.

---

# 9. `use`

Specifies the intended use of the key.

For example:

```json
"use": "sig"
```

means:

```text
signature
```

Another possible value is:

```text
"use": "enc"
```

meaning:

```text
encryption
```

So:

```text
sig → signing / signature verification

enc → encryption / decryption
```

---

# 10. `alg`

Indicates the algorithm associated with the key.

Example:

```json
"alg": "RS256"
```

Meaning:

```text
RSA
+
SHA-256
+
signature
```

You already learned about RS256 when studying JWT signing.

---

# 11. RSA `n` and `e`

This is where JWK starts connecting to actual cryptography.

For RSA, a public key consists essentially of:

```text
(n, e)
```

The JWK represents these values as:

```json
{
  "kty": "RSA",
  "n": "...",
  "e": "AQAB"
}
```

### `n`

RSA modulus.

### `e`

RSA public exponent.

Typically:

```text
e = 65537
```

which is represented in Base64URL form as:

```text
AQAB
```

You don't normally need to manipulate these manually in application code.

Libraries convert:

```text
JWK
 ↓
RSA parameters
 ↓
Java PublicKey
```

---

# 12. JWK Is Not the Private Key

This distinction is **very important**.

A JWKS endpoint normally exposes **public keys**.

For example:

```text
JWKS
 ↓
Public Key
```

The private key remains inside the Authorization Server.

Architecture:

```text
                 Authorization Server
                 ┌─────────────────────┐
                 │                     │
                 │  Private Key        │
                 │       🔐            │
                 │                     │
                 └─────────┬───────────┘
                           |
                           | sign
                           ↓
                          JWT
                           
                 Public Key
                     ↓
                   JWKS
                     ↓
              Resource Server
```

The Resource Server gets:

```text
PUBLIC KEY
```

not:

```text
PRIVATE KEY
```

---

# 13. Why Is It Safe to Publish the Public Key?

Because the public key is supposed to be public.

With asymmetric cryptography:

```text
Private Key
    ↓
must remain secret
```

while:

```text
Public Key
    ↓
can be distributed
```

Anyone can have the public key.

They still cannot create a valid signature without the private key.

So:

```text
Attacker gets public key
        ↓
Can verify signatures
        ↓
Cannot sign a valid JWT
```

assuming the cryptographic system and key handling are sound.

---

# 14. JWKS Endpoint

An Authorization Server usually publishes its keys through a JWKS endpoint.

Conceptually:

```text
GET /.well-known/jwks.json
```

For example:

```text
https://auth.example.com/.well-known/jwks.json
```

Response:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "kid": "key-1",
      "use": "sig",
      "alg": "RS256",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

The exact URL is provider-specific, but OpenID Connect discovery normally tells clients where the `jwks_uri` is.

---

# 15. OIDC Discovery + JWKS

Since you've already studied OIDC, this connection is worth understanding.

An OIDC provider publishes discovery metadata.

Conceptually:

```text
https://auth.example.com/.well-known/openid-configuration
```

It might contain:

```json
{
  "issuer": "https://auth.example.com",
  "authorization_endpoint": "...",
  "token_endpoint": "...",
  "userinfo_endpoint": "...",
  "jwks_uri": "https://auth.example.com/.well-known/jwks.json"
}
```

Notice:

```text
jwks_uri
```

That's the URL where the public signing keys can be obtained.

So the flow becomes:

```text
OIDC Discovery
      |
      | jwks_uri
      ↓
JWKS Endpoint
      |
      ↓
Public Keys
      |
      ↓
JWT Verification
```

This is one of the reasons modern OAuth/OIDC systems can be configured with just an **issuer URL**.

Your application can discover the rest.

---

# 16. Spring Security Example

Since you're a Java/Spring developer, this is particularly useful.

Suppose your Spring Boot Resource Server configuration says:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.example.com
```

You don't necessarily need to manually configure:

```text
public-key.pem
```

Spring Security can use the issuer metadata to discover the Authorization Server configuration and obtain its JWKS URI.

Conceptually:

```text
issuer-uri
    ↓
OIDC/OAuth metadata
    ↓
jwks_uri
    ↓
JWKS
    ↓
Public keys
    ↓
JWT validation
```

Then a request:

```http
GET /api/orders
Authorization: Bearer eyJ...
```

results in the Resource Server validating the JWT.

You don't normally write:

```java
PublicKey publicKey = ...
```

yourself.

Spring Security handles the key retrieval and JWT verification machinery.

---

# 17. Key Rotation

Now let's look at the real reason this becomes important in production.

Suppose today:

```text
Private Key A
```

is active.

JWKS:

```json
{
  "keys": [
    {
      "kid": "A",
      ...
    }
  ]
}
```

JWT:

```json
{
  "alg": "RS256",
  "kid": "A"
}
```

Everything works.

---

## Day of Rotation

You generate:

```text
Private Key B
Public Key B
```

Now you publish:

```json
{
  "keys": [
    {
      "kid": "A",
      ...
    },
    {
      "kid": "B",
      ...
    }
  ]
}
```

The Authorization Server starts signing new tokens using:

```text
Private Key B
```

New JWT:

```json
{
  "alg": "RS256",
  "kid": "B"
}
```

Resource Servers download the updated JWKS.

They can now validate:

```text
Old JWT → Key A

New JWT → Key B
```

---

# 18. Why Can't You Just Replace A With B?

Because of existing tokens.

Imagine:

```text
Access Token A
expires at 12:00
```

At:

```text
11:00
```

you rotate the key.

If you immediately remove:

```text
Public Key A
```

then:

```text
Old Access Token
        ↓
kid = A
        ↓
JWKS
        ↓
Key A doesn't exist
        ↓
Validation fails
```

even though the token hasn't expired.

That's bad.

Therefore you typically need an overlap period:

```text
             Rotation
                ↓
Key A ──────────┐
                │
                ├── both available
                │
Key B ──────────┘
                │
                ↓
          Key A retired
```

The exact retention strategy depends on your token lifetime and operational setup.

---

# 19. `kid` Is Extremely Important During Rotation

Imagine JWKS contains:

```text
Key A
Key B
Key C
```

and the JWT says:

```json
{
  "kid": "B"
}
```

The Resource Server knows:

```text
Use B.
```

Without `kid`, selecting the correct key becomes much harder when multiple keys are published.

So the relationship is:

```text
JWT Header
   |
   | kid
   ↓
JWKS
   |
   | matching kid
   ↓
JWK
   |
   ↓
Public Key
```

---

# 20. What If the Resource Server Doesn't Have the Key Yet?

Interesting scenario.

Suppose:

```text
Authorization Server
     |
     | starts using Key B
     ↓
JWT
     |
     ↓
Resource Server
```

But Resource Server's cached JWKS only contains:

```text
Key A
```

It sees:

```text
kid = B
```

but doesn't have B.

A well-designed JWT/JWK implementation can refresh its key set when it encounters an unknown key ID, subject to implementation/security controls.

Eventually:

```text
Refresh JWKS
     ↓
Find Key B
     ↓
Verify JWT
```

This is another reason proper JWKS handling is better than hardcoding a public key.

---

# 21. JWKS Caching

You generally **don't want to call the JWKS endpoint for every API request**.

Imagine:

```text
100,000 requests/second
```

and every request does:

```text
Resource Server
      ↓
GET JWKS
      ↓
Authorization Server
```

That's terrible architecture.

Instead, Resource Servers typically cache the JWKS.

Conceptually:

```text
First request
     ↓
Fetch JWKS
     ↓
Cache keys
```

Then:

```text
Request 1 → cached key
Request 2 → cached key
Request 3 → cached key
Request 4 → cached key
...
```

Only occasionally:

```text
Cache refresh
     ↓
Authorization Server
```

Spring Security and other mature OAuth libraries handle much of this behavior for you.

---

# 22. JWKS vs Introspection

This connects directly to your previous topic.

You just learned:

**Token introspection**

```text
Resource Server
      ↓
Authorization Server
      ↓
"Is this token active?"
```

JWKS works differently.

```text
Resource Server
      ↓
JWKS
      ↓
Get public keys
      ↓
Verify JWT locally
```

So:

|                      | Introspection                               | JWKS                               |
| -------------------- | ------------------------------------------- | ---------------------------------- |
| Purpose              | Determine token state                       | Obtain public keys                 |
| Network request      | Usually per validation/cache                | Usually cached                     |
| Token type           | Especially useful for opaque tokens         | JWT                                |
| Revocation awareness | Yes                                         | Not inherently                     |
| Signature validation | Authorization Server can determine validity | Resource Server does it            |
| Main benefit         | Centralized token state                     | Distributed/local JWT verification |

---

# 23. JWKS Does NOT Solve Revocation

This is an important connection with your previous topic.

Suppose:

```text
JWT
kid = A
exp = 12:00
```

Resource Server has:

```text
Public Key A
```

At:

```text
11:00
```

you revoke the token.

The JWKS still says:

```text
Key A is valid.
```

Why?

Because JWKS answers:

> "Here are the public keys."

It doesn't answer:

> "Is this specific JWT still authorized?"

So:

```text
JWKS
 ↓
Can I cryptographically verify this JWT?
```

whereas:

```text
Introspection
 ↓
Is this token currently active?
```

These are different questions.

---

# 24. JWKS Does Not Mean "Token Validation"

Another subtle distinction:

```text
JWKS
```

contains keys.

It doesn't validate your JWT for you.

The Resource Server still needs to check:

```text
Signature
   ✓

exp
   ✓

nbf
   ✓

iss
   ✓

aud
   ✓

scope/authorities
   ✓
```

So:

```text
JWKS
 ↓
Get public key
 ↓
JWT signature verification
 ↓
Claims validation
 ↓
Authorization
```

---

# 25. JWK vs PEM

You may wonder:

> "Why not just give the Resource Server a `.pem` public key?"

You absolutely can.

For example:

```text
public-key.pem
```

contains a representation of a public key.

But this approach creates operational problems.

Imagine 50 microservices:

```text
service-A
service-B
service-C
...
service-Z
```

All have:

```text
public-key.pem
```

Then you rotate the key.

Now you need to distribute the new public key to all services.

With JWKS:

```text
Authorization Server
        |
        ↓
      JWKS
        |
        ↓
Resource Servers discover keys
```

Much easier.

---

# 26. The Big Production Architecture

Here's the architecture I want you to visualize.

```text
                     Authorization Server
                     ┌──────────────────────┐
                     │                      │
                     │ Private Key A 🔐     │
                     │ Private Key B 🔐     │
                     │                      │
                     └──────────┬───────────┘
                                │
                         signs JWTs
                                │
                                ↓
                             Client
                                │
                         Access Token
                                │
                                ↓
                         API Gateway
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
                 ↓                             ↓
          Resource Server A             Resource Server B
                 │                             │
                 │                             │
                 └─────────────┬───────────────┘
                               │
                         JWKS (cached)
                               │
                               ↓
                     Public Key A / B
```

The private keys **never leave the Authorization Server**.

The public keys can be distributed through JWKS.

---

# 27. The Complete Lifecycle

Let's put everything you've learned so far together.

### Step 1 — Authorization Server has keys

```text
Private Key A
Public Key A
```

---

### Step 2 — Public key published

```text
JWKS
 ↓
Public Key A
```

---

### Step 3 — User authenticates

```text
User
 ↓
Authorization Server
```

---

### Step 4 — JWT issued

```text
Private Key A
     ↓
Sign JWT
     ↓
JWT
```

JWT:

```json
{
  "alg": "RS256",
  "kid": "A"
}
```

---

### Step 5 — Client calls API

```http
Authorization: Bearer JWT
```

---

### Step 6 — Resource Server sees

```text
kid = A
```

---

### Step 7 — Resource Server gets JWK

```text
JWKS
 ↓
kid = A
 ↓
Public Key A
```

---

### Step 8 — JWT signature verified

```text
JWT
 ↓
Public Key A
 ↓
Signature valid
```

---

### Step 9 — Claims validated

```text
exp ✓
iss ✓
aud ✓
nbf ✓
```

---

### Step 10 — Authorization

```text
scope = orders.read
       ↓
GET /orders
       ↓
ALLOW
```

---

# 28. Key Rotation in This Architecture

Later:

```text
Private Key A
      ↓
retiring

Private Key B
      ↓
new signing key
```

JWKS temporarily becomes:

```text
JWKS
 ├── Key A
 └── Key B
```

New JWT:

```json
{
  "kid": "B"
}
```

Resource Server:

```text
kid B
 ↓
JWKS
 ↓
Key B
 ↓
verify
```

Eventually, when tokens signed with A can no longer be valid:

```text
JWKS
 └── Key B
```

Key A can be removed.

---

# 29. Different Types of JWKs

You don't need to master all of these immediately, but know the landscape.

### RSA

```json
"kty": "RSA"
```

Common with:

```text
RS256
RS384
RS512
```

---

### Elliptic Curve

```json
"kty": "EC"
```

Common algorithms include:

```text
ES256
ES384
ES512
```

---

### Octet Key Pair

```json
"kty": "OKP"
```

Used for modern algorithms such as EdDSA.

---

### Symmetric

```json
"kty": "oct"
```

Represents symmetric keys.

For example, HMAC-based algorithms:

```text
HS256
```

But there's an important architectural difference:

```text
RSA / EC

Private key → Authorization Server
Public key  → Resource Server
```

versus:

```text
HMAC

Same secret → both sides
```

For distributed JWT verification, asymmetric signing is often much easier to operate safely because Resource Servers only need public keys.

---

# 30. JWK, JWKS, JWT — Don't Mix Them Up

These three names are annoyingly similar 😄.

Remember:

```text
JWT
 ↓
The TOKEN
```

```text
JWK
 ↓
ONE KEY
```

```text
JWKS
 ↓
SET OF KEYS
```

Or:

```text
JWT = "Here's my identity/authorization data + signature"

JWK = "Here's a cryptographic key represented as JSON"

JWKS = "Here are multiple JWKs"
```

---

# 31. The Most Important Relationship

You can memorize this diagram:

```text
                   Authorization Server
                          |
                    Private Key
                          |
                          ↓
                       Sign JWT
                          |
                          ↓
                         JWT
                          |
                     kid = B
                          |
                          ↓
                   Resource Server
                          |
                          ↓
                        JWKS
                          |
                    find kid = B
                          |
                          ↓
                     JWK B
                          |
                          ↓
                    Public Key B
                          |
                          ↓
                  Verify JWT signature
                          |
                          ↓
                   Validate claims
                          |
                          ↓
                       ALLOW
```

That's **JWK/JWKS in one picture**.

---

# 32. How This Relates to Your OAuth Roadmap

You've now covered:

```text
OAuth
 │
 ├── Access Tokens
 │
 ├── Refresh Tokens
 │
 ├── JWT
 │
 ├── JWT Signing
 │
 ├── Token Security
 │
 ├── Token Introspection
 │
 ├── Token Revocation
 │
 └── JWK / JWKS  ← YOU ARE HERE
```

And the next concepts make even more sense now:

```text
JWK / JWKS
    ↓
JWS
    ↓
JWE
    ↓
Proof-of-Possession
    ↓
mTLS
```

One especially useful distinction for your next topic:

> **JWS is about signing/protecting the integrity and authenticity of content. JWE is about encrypting content so its contents are confidential.**

And **JWK/JWKS are the key representations/distribution mechanism that can support those operations**.

### Final mental model

If you remember only four things from this topic, remember these:

```text
JWK  = one JSON-formatted cryptographic key

JWKS = collection of JWKs

kid  = identifies which key was used

jwks_uri = where Resource Servers can obtain the public keys
```

And the production pattern:

```text
Authorization Server
       |
       | private key
       ↓
    Sign JWT
       |
       ↓
     Client
       |
       ↓
Resource Server
       |
       | kid
       ↓
     JWKS
       |
       ↓
Public Key
       |
       ↓
Verify JWT
```

That's the foundation you'll need when you start looking at **real Spring Security OAuth2 Resource Server configurations**, because Spring essentially automates much of this JWK discovery, caching, key selection, and JWT verification process.
