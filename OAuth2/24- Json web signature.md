# JSON Web Signature (JWS)

## 1. What is JWS?

**JWS = JSON Web Signature**

JWS is a standard for **digitally signing data**.

Its purpose is to provide:

* **Integrity** → the data wasn't modified
* **Authenticity** → the data was signed by someone possessing the signing key

In simple terms:

> **JWS lets the receiver verify that the data came from the expected signer and hasn't been tampered with.**

The basic idea is:

```text
Original Data
     ↓
   Sign
     ↓
   JWS
     ↓
Send to receiver
     ↓
Verify signature
```

---

# 2. JWS is NOT encryption

This is the most important thing to understand before moving to JWE.

Suppose we have:

```json
{
  "sub": "12345",
  "role": "ADMIN"
}
```

If we create a JWS, the payload is **not secret**.

Someone can decode the payload and see:

```json
{
  "sub": "12345",
  "role": "ADMIN"
}
```

But they **cannot modify it and create a valid signature** unless they have the signing key.

So:

```text
JWS
 ↓
Protects integrity + authenticity

JWE
 ↓
Protects confidentiality
```

---

# 3. JWS and JWT relationship

This is where developers often get confused.

You have probably seen:

```text
xxxxx.yyyyy.zzzzz
```

That's a **compact JWS representation** containing a JWT payload.

Conceptually:

```text
JWT
 │
 └── Often represented as a JWS
          │
          ├── Header
          ├── Payload
          └── Signature
```

So when you create a typical signed JWT:

```text
JWT
 ↓
JWS
 ↓
Header.Payload.Signature
```

That's why JWT and JWS are often discussed together.

---

# 4. JWS structure

A JWS in Compact Serialization has **three parts**:

```text
Header.Payload.Signature
```

For example:

```text
eyJhbGciOiJSUzI1NiJ9
.
eyJzdWIiOiIxMjM0NSJ9
.
abc123signature
```

The three components are:

```text
┌───────────────┐
│ Protected     │
│ Header        │
└───────────────┘
       .
┌───────────────┐
│ Payload       │
└───────────────┘
       .
┌───────────────┐
│ Signature      │
└───────────────┘
```

All three are Base64URL encoded.

---

# 5. JWS Header

The header contains information about how the signature was created.

For example:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

The most important field is:

```json
"alg": "RS256"
```

It tells the receiver which algorithm was used to create the signature.

For example:

```text
HS256
RS256
RS384
RS512
ES256
ES384
ES512
EdDSA
```

We'll focus mainly on **RS256**, because it is very common in OAuth/OIDC systems.

---

# 6. JWS Payload

The payload contains the actual data being signed.

For a JWT:

```json
{
  "sub": "12345",
  "iss": "https://auth.example.com",
  "aud": "orders-api",
  "exp": 1780000000,
  "scope": "orders.read"
}
```

This is the information the Resource Server will eventually use.

Again:

> **The payload is encoded, not encrypted.**

You can Base64URL-decode it.

---

# 7. JWS Signature

The signature is the security-critical part.

Conceptually:

```text
Header
   +
Payload
   ↓
Signing Algorithm
   +
Private Key
   ↓
Signature
```

For RSA:

```text
Header.Payload
       ↓
     SHA-256
       ↓
  RSA Private Key
       ↓
   Signature
```

The final JWS becomes:

```text
Base64URL(Header)
.
Base64URL(Payload)
.
Base64URL(Signature)
```

---

# 8. What happens when the receiver gets the JWS?

Suppose the Authorization Server creates:

```text
Header.Payload.Signature
```

and sends it to the Resource Server.

The Resource Server has the Authorization Server's **public key**.

It performs roughly this process:

```text
                 JWS
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
    Header.Payload       Signature
        │                   │
        │              Public Key
        │                   │
        └─────────┬─────────┘
                  ↓
            Verify Signature
                  ↓
             Valid / Invalid
```

If valid:

```text
Token is authentic
AND
Token wasn't modified
```

If someone changes:

```json
"role": "USER"
```

to:

```json
"role": "ADMIN"
```

the signature will no longer match.

Verification fails.

---

# 9. Example with RS256

Let's make this concrete.

Authorization Server has:

```text
Private Key
Public Key
```

The private key must remain secret.

The public key can be distributed to Resource Servers.

### Authorization Server

Creates:

```json
{
  "sub": "12345",
  "scope": "orders.read"
}
```

Then:

```text
Payload
   ↓
SHA-256
   ↓
RSA + Private Key
   ↓
Signature
```

It sends:

```text
Header.Payload.Signature
```

---

### Resource Server

Receives the JWS.

It gets the public key:

```text
Public Key
```

Then:

```text
Header.Payload
       ↓
Verification
       ↑
Signature
       ↑
Public Key
```

If verification succeeds:

```text
✅ Signature valid
```

The Resource Server can trust that the token was signed by the expected key.

---

# 10. Why is the private key important?

This is the fundamental security model:

```text
Authorization Server
        │
        │ Private Key
        ↓
      SIGN
        │
        ↓
       JWS
        │
        ↓
Resource Server
        │
        │ Public Key
        ↓
     VERIFY
```

Anyone can have the public key.

Only the Authorization Server should have the private key.

Therefore, an attacker can't simply create:

```json
{
  "sub": "attacker",
  "scope": "admin"
}
```

and sign it with the public key.

**Public keys verify. Private keys sign.**

---

# 11. How JWK/JWKS fits here

This connects directly with the topic you just studied.

The Authorization Server might expose a JWKS endpoint:

```text
Authorization Server
       │
       │ JWKS endpoint
       ↓
   Public JWKs
       │
       ↓
Resource Server
```

The Resource Server retrieves the public key from the JWKS.

Then:

```text
JWT
 ↓
Read kid
 ↓
Find matching JWK
 ↓
Get public key
 ↓
Verify JWS signature
```

For example, a JWT header might contain:

```json
{
  "alg": "RS256",
  "kid": "key-2026-01",
  "typ": "JWT"
}
```

The `kid` tells the Resource Server:

> "Which key from the JWKS should you use?"

So your topics connect like this:

```text
JWK/JWKS
    ↓
Provides public keys
    ↓
JWS
    ↓
Uses public key to verify signature
    ↓
JWT
    ↓
Contains claims
```

---

# 12. JWS signing algorithms

You don't need to memorize every algorithm right now. Understand the families.

### HMAC

```text
HS256
HS384
HS512
```

Uses a **shared secret**.

```text
Signer
  │
  │ Shared Secret
  ↓
Receiver
```

Both sides need the same secret.

---

### RSA

```text
RS256
RS384
RS512
```

Uses:

```text
Private key → Sign
Public key  → Verify
```

Very common in OAuth/OIDC.

---

### ECDSA

```text
ES256
ES384
ES512
```

Uses elliptic-curve cryptography.

Again:

```text
Private key → Sign
Public key  → Verify
```

---

### EdDSA

Modern public-key signature scheme based on Edwards curves.

You'll encounter it in some modern identity systems, although **RS256 is still extremely common** in OAuth/OIDC implementations.

---

# 13. JWS vs JWT

A useful way to think about it:

### JWT describes the token/content

For example:

```json
{
  "sub": "12345",
  "exp": 1780000000,
  "scope": "orders.read"
}
```

### JWS describes how that content is signed

```text
Header
Payload
Signature
```

So:

```text
JWT payload
     ↓
put into
     ↓
JWS
     ↓
signed JWT
```

That's why people casually say:

> "JWT is signed using RS256."

Technically, the JWT is being represented as a **JWS**.

---

# 14. JWS does not prove the user is currently logged in

Another subtle but important OAuth point.

A valid signature means:

> "This token was signed by the holder of the corresponding private key, and the signed content hasn't changed."

It does **not** by itself mean:

> "This user is currently logged in."

The Resource Server must also validate claims such as:

```text
iss   → Correct issuer?
aud   → Intended API?
exp   → Still valid?
nbf   → Not before?
scope → Required permission?
```

So token validation is more like:

```text
                JWT/JWS
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    Signature     exp       issuer
    valid?       valid?     correct?
        │          │          │
        └──────────┼──────────┘
                   ↓
              Accept token
```

---

# 15. What happens if someone changes the payload?

Suppose the original token contains:

```json
{
  "sub": "12345",
  "scope": "orders.read"
}
```

An attacker changes it to:

```json
{
  "sub": "12345",
  "scope": "admin"
}
```

The attacker doesn't have the private signing key.

Therefore:

```text
Modified Payload
      ↓
Signature no longer matches
      ↓
❌ Verification fails
```

This is the core value of JWS.

---

# 16. JWS vs JWE

Now your roadmap will make much more sense.

### JWS

```text
Payload
   ↓
SIGN
   ↓
JWS

Purpose:
Integrity + Authenticity
```

### JWE

```text
Payload
   ↓
ENCRYPT
   ↓
JWE

Purpose:
Confidentiality
```

And they can be combined:

```text
Payload
   ↓
JWS
   ↓
JWE
```

Meaning:

```text
Sign first
   ↓
Encrypt
```

The receiver:

```text
Decrypt
   ↓
Verify signature
   ↓
Payload
```

---

# 17. The mental model I want you to keep

As a Java developer, think about JWS almost like this:

```text
                PRIVATE KEY
                     │
                     ↓
Header + Payload → SIGN
                     │
                     ↓
                 Signature
                     │
                     ↓
        ┌────────────────────────┐
        │ Header.Payload.Signature│
        └────────────────────────┘
                     │
                     ↓
               Resource Server
                     │
                PUBLIC KEY
                     │
                     ↓
                  VERIFY
                     │
              ┌──────┴──────┐
              ↓             ↓
            Valid         Invalid
              ↓             ↓
           Accept         Reject
```

And the single sentence to remember is:

> **JWS is a standard way to digitally sign data so that the receiver can verify its integrity and authenticity; it does not hide the data.**

Once this is clear, **JWE becomes very easy**, because JWE is essentially the next question: *"Okay, what if I also need to hide that payload?"*
