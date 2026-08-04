JWT signing is the **core security mechanism** of JWTs. Without signing, anyone could modify a token and pretend to be another user. JWT signing ensures **integrity** (the token hasn't been altered) and **authenticity** (it was issued by a trusted party).

---

# What is JWT Signing?

When a server creates a JWT, it doesn't just put data into the token.

It also creates a **digital signature**.

```
Header
+
Payload
      ↓
Sign with Secret/Private Key
      ↓
Signature
```

The final JWT becomes:

```
header.payload.signature
```

Example:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiIxMjM0NSIsIm5hbWUiOiJBbGljZSIsImFkbWluIjp0cnVlfQ
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The signature is what protects the token.

---

# Why is Signing Needed?

Imagine a JWT without a signature.

Payload:

```json
{
  "userId": 25,
  "role": "user"
}
```

A malicious user could change it to:

```json
{
  "userId": 25,
  "role": "admin"
}
```

If there were no signature, the server would have no reliable way to detect the change.

JWT signing solves this.

---

# JWT Structure Review

```
Header
↓

{
  "alg": "HS256",
  "typ": "JWT"
}

Payload
↓

{
  "sub": "12345",
  "role": "admin",
  "exp": 1750000000
}

Signature
↓

HMACSHA256(
    base64Url(header) + "." + base64Url(payload),
    secret
)
```

Only the **signature** depends on the secret or private key.

---

# How Signing Works

Suppose the server has:

```
Secret Key

mySuperSecretKey123
```

Header:

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```

Payload:

```json
{
  "userId":42,
  "role":"admin"
}
```

The server:

### Step 1

Base64URL encodes the header.

```
xxxxx
```

### Step 2

Base64URL encodes the payload.

```
yyyyy
```

### Step 3

Combines them.

```
xxxxx.yyyyy
```

### Step 4

Signs them.

```
Signature =
HMAC(secret, "xxxxx.yyyyy")
```

Final token:

```
xxxxx.yyyyy.zzzzz
```

---

# Verification

When the client sends:

```
Authorization: Bearer JWT
```

The server does **not** trust the payload immediately.

Instead it:

```
Receive JWT

↓

Split into

Header

Payload

Signature

↓

Read algorithm

↓

Recalculate signature

↓

Compare signatures

↓

If identical

Token is authentic

Else

Reject
```

This is why changing even **one character** in the payload invalidates the token.

---

# Example of Tampering

Original payload:

```json
{
  "role":"user"
}
```

Signature:

```
abc123
```

Attacker changes payload:

```json
{
  "role":"admin"
}
```

The old signature is still:

```
abc123
```

The server recalculates the signature:

```
Expected signature

xyz789
```

Comparison:

```
Expected: xyz789

Received: abc123

Not equal

Reject
```

---

# HMAC (HS256)

HS256 stands for:

```
HMAC

+

SHA-256
```

It uses **one shared secret**.

```
Server A

Secret
↓

mySecret123
```

Signing:

```
JWT

↓

HMAC(secret)

↓

Signature
```

Verification:

```
JWT

↓

Same Secret

↓

HMAC(secret)

↓

Compare
```

Both signing and verification require the **same secret**.

```
Shared Secret

──────────────

Signs

Verifies
```

---

# HS256 Workflow

```
Server

↓

Header

↓

Payload

↓

Secret

↓

HMAC

↓

JWT
```

Later:

```
Client

↓

JWT

↓

Server

↓

Secret

↓

Verify
```

---

# Advantages of HS256

* Fast
* Simple
* Small tokens
* Great when one service issues and verifies tokens

Example:

```
Frontend

↓

Backend

↓

JWT

↓

Backend verifies
```

---

# Disadvantages of HS256

Every service that verifies the token must know the secret.

```
API 1

Secret

API 2

Secret

API 3

Secret
```

If one service leaks the secret, an attacker can create valid tokens.

---

# RSA (RS256)

RS256 uses **asymmetric cryptography**.

It has two different keys:

```
Private Key

Public Key
```

Unlike HS256, these keys have different roles.

```
Private Key

Signs

────────────

Public Key

Verifies
```

---

# RSA Workflow

Server:

```
Private Key

↓

Sign JWT

↓

JWT
```

API:

```
JWT

↓

Public Key

↓

Verify
```

The public key **cannot** create valid signatures.

Only the private key can.

---

# Visual Comparison

## HS256

```
            Secret
          ┌────────┐
Sign  <───┤ Secret ├───> Verify
          └────────┘
```

## RS256

```
Private Key

↓

Signs

↓

JWT

↓

Public Key

↓

Verifies
```

---

# Why RS256 Is Better for Large Systems

Imagine:

```
Authentication Server

↓

Issues JWT

↓

20 APIs verify JWT
```

With HS256:

```
20 APIs

↓

Need Secret
```

If any API is compromised, the attacker may be able to forge tokens.

With RS256:

```
Authentication Server

↓

Private Key

↓

Signs
```

APIs only have:

```
Public Key
```

Even if an API is compromised, it still cannot create valid JWTs.

---

# Real-World Example

```
User logs in

↓

Auth Server

↓

Signs JWT using Private Key

↓

JWT sent to browser

↓

Browser calls Orders API

↓

Orders API verifies with Public Key

↓

Access granted
```

The Orders API never needs the private key.

---

# When to Use HS256 vs RS256

| Feature      | HS256                                             | RS256                                                              |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------------ |
| Keys         | One shared secret                                 | Public/private key pair                                            |
| Signing      | Shared secret                                     | Private key                                                        |
| Verification | Shared secret                                     | Public key                                                         |
| Speed        | Faster                                            | Slightly slower                                                    |
| Simplicity   | Easier                                            | More setup                                                         |
| Best for     | Single application or tightly controlled services | Distributed systems, APIs, microservices, third-party verification |

---

# Common Misconceptions

### 1. JWT payload is encrypted ❌

No. A JWT is typically only **signed**, not encrypted.

Anyone who has the token can Base64URL-decode the header and payload.

Example:

```json
{
  "email":"alice@example.com",
  "role":"admin"
}
```

The signature prevents modification, but it does **not** hide the contents.

---

### 2. The signature hides the payload ❌

The signature proves:

* the payload wasn't changed
* the issuer had the signing key

It does not make the payload secret.

---

### 3. Clients verify JWTs ❌

In most architectures:

* The server (or API gateway) verifies access tokens.
* Clients typically store and send the token but should not be relied on for security decisions.

---

# Best Practices

* Use strong, randomly generated secrets for HS256.
* Keep private keys secret; never embed them in client-side code.
* Verify the signature **before** trusting any claim in the payload.
* Validate claims such as `exp` (expiration), `iss` (issuer), and `aud` (audience) after signature verification.
* Use short-lived access tokens.
* Prefer RS256 (or other asymmetric algorithms) when multiple independent services need to verify tokens.

---

## Practical Exercise

To make this topic stick, build a small authentication API:

1. Create a login endpoint.
2. Sign a JWT using **HS256** with a secret key.
3. Add middleware that verifies the JWT before allowing access to a protected route.
4. Try editing the payload (for example, change `"role": "user"` to `"role": "admin"`) without regenerating the signature, and observe that verification fails.
5. Repeat the project using **RS256**, generating a private/public key pair, signing with the private key, and verifying with the public key.

By implementing both HS256 and RS256 yourself, you'll gain an intuitive understanding of why JWT signing exists and when each approach is appropriate.
