# JSON Web Encryption (JWE)

## 1. What is JWE?

**JWE = JSON Web Encryption**

It is a standard for representing **encrypted data as a JSON-based token**.

The primary goal is **confidentiality**.

For example, suppose an Authorization Server needs to send sensitive information to a client:

```text
Authorization Server
        |
        |  Sensitive data
        ↓
      JWE
        |
        ↓
      Client
```

Only the intended recipient should be able to decrypt the contents.

---

# 2. JWS vs JWE

This distinction is extremely important.

### JWS

JWS provides:

* Integrity
* Authenticity
* Digital signature

It answers:

> "Has this token been modified, and who signed it?"

Example:

```text
Header.Payload.Signature
```

The payload is normally **readable**.

---

### JWE

JWE provides:

* Confidentiality
* Encryption

It answers:

> "Can anyone other than the intended recipient read this data?"

Example:

```text
Encrypted data
```

Without the appropriate decryption key, the contents cannot be read.

---

So:

```text
JWS
 ↓
Sign

JWE
 ↓
Encrypt
```

And they can actually be combined.

```text
Sign
 ↓
Encrypt
 ↓
Send
```

This gives you both authenticity/integrity and confidentiality.

---

# 3. JWT vs JWS vs JWE

This terminology causes a lot of confusion.

Think of it like this:

```text
JWT
 │
 ├── JWS-based JWT
 │      └── Signed
 │
 └── JWE-based JWT
        └── Encrypted
```

A JWT is a **token format/use case**, while JWS and JWE define ways of securing the content.

A very common JWT is:

```text
xxxxx.yyyyy.zzzzz
```

That's a **signed JWT (JWS)**.

A JWE compact token has **five parts**:

```text
xxxxx.yyyyy.zzzzz.aaaaa.bbbbb
```

That's one of the easiest ways to recognize JWE.

---

# 4. JWS has 3 parts

You already learned this:

```text
Header.Payload.Signature
```

For example:

```text
eyJhbGciOiJSUzI1NiJ9
.
eyJzdWIiOiIxMjMifQ
.
SIGNATURE
```

The payload can be Base64URL-decoded.

So:

```text
JWT
 ↓
Base64URL decode
 ↓
Readable payload
```

That's why **a normal signed JWT does NOT provide confidentiality**.

This is an important security point:

> **Never put sensitive information in a normal JWT assuming that signing encrypts it.**

For example:

```json
{
  "sub": "12345",
  "email": "rohan@example.com",
  "salary": 250000
}
```

Anyone who obtains the JWT can decode the payload.

They cannot modify it without invalidating the signature, but they can **read it**.

---

# 5. JWE has 5 parts

A JWE Compact Serialization looks like:

```text
A.B.C.D.E
```

Specifically:

```text
Protected Header
.
Encrypted Key
.
Initialization Vector
.
Ciphertext
.
Authentication Tag
```

So:

```text
JWE

┌──────────────────────┐
│ Protected Header     │
├──────────────────────┤
│ Encrypted Key        │
├──────────────────────┤
│ Initialization Vector│
├──────────────────────┤
│ Ciphertext           │
├──────────────────────┤
│ Authentication Tag   │
└──────────────────────┘
```

This is different from JWS.

---

# 6. Why does JWE have an "Encrypted Key"?

This is probably the most important concept in JWE.

JWE normally uses **two kinds of cryptography**:

### Asymmetric cryptography

Used to protect the encryption key.

### Symmetric cryptography

Used to encrypt the actual data.

Why?

Because symmetric encryption is much faster for encrypting data.

So instead of doing:

```text
Public key
   ↓
Encrypt huge payload
```

we do:

```text
Generate random symmetric key
          ↓
Encrypt payload with symmetric key
          ↓
Encrypt symmetric key with recipient's public key
```

This is called **hybrid encryption**.

---

# 7. JWE encryption flow

Imagine:

```text
Authorization Server
        |
        | wants to send secret data
        ↓
       JWE
        |
        ↓
      Client
```

The Authorization Server has the client's public key.

### Step 1 — Generate Content Encryption Key

A random symmetric key is generated.

Let's call it:

```text
CEK
Content Encryption Key
```

---

### Step 2 — Encrypt the actual payload

Suppose the payload is:

```json
{
  "userId": "123",
  "email": "user@example.com"
}
```

The CEK encrypts this data using an authenticated encryption algorithm such as:

```text
A256GCM
```

Result:

```text
Ciphertext
```

---

### Step 3 — Encrypt the CEK

The CEK itself needs to be delivered to the recipient.

So the sender encrypts the CEK using the recipient's public key.

Conceptually:

```text
Recipient Public Key
        +
       CEK
        ↓
Encrypted CEK
```

---

### Step 4 — Send JWE

The sender sends:

```text
JWE
```

to the recipient.

---

### Step 5 — Recipient decrypts CEK

The recipient has the corresponding private key.

```text
Encrypted CEK
      +
Private Key
      ↓
     CEK
```

---

### Step 6 — Decrypt ciphertext

Now the recipient has the CEK:

```text
CEK
 ↓
Decrypt ciphertext
 ↓
Original payload
```

So the complete flow is:

```text
                 Sender
                   │
             Generate CEK
                   │
          ┌────────┴────────┐
          ↓                 ↓
   Encrypt payload    Encrypt CEK
     with CEK       with public key
          │                 │
          └────────┬────────┘
                   ↓
                  JWE
                   │
                   ↓
                Recipient
                   │
             Private key
                   ↓
             Recover CEK
                   ↓
          Decrypt ciphertext
                   ↓
              Plaintext
```

---

# 8. JWE Header

A JWE has a protected header.

For example:

```json
{
  "alg": "RSA-OAEP-256",
  "enc": "A256GCM"
}
```

There are two particularly important fields.

## `alg`

Defines the **key management algorithm**.

For example:

```text
RSA-OAEP-256
```

This determines how the CEK is protected.

---

## `enc`

Defines the **content encryption algorithm**.

For example:

```text
A256GCM
```

This determines how the actual payload is encrypted.

Therefore:

```text
alg
 ↓
How do I protect the encryption key?

enc
 ↓
How do I encrypt the actual data?
```

This distinction is extremely important.

---

# 9. Example

Imagine we have:

```text
Authorization Server
       |
       | Client's public key
       ↓
```

Payload:

```json
{
  "sub": "12345",
  "email": "user@example.com"
}
```

Header:

```json
{
  "alg": "RSA-OAEP-256",
  "enc": "A256GCM"
}
```

The JWE might look conceptually like:

```text
HEADER
.
ENCRYPTED_KEY
.
IV
.
CIPHERTEXT
.
AUTH_TAG
```

Something like:

```text
eyJhbGciOiJSU0EtT0FFUC0yNTYiLCJlbmMiOiJBMjU2R0NNIn0
.
encrypted-key
.
iv
.
encrypted-payload
.
authentication-tag
```

The important thing is that the actual payload is no longer visible.

---

# 10. What is the Initialization Vector?

The third JWE component is the:

```text
Initialization Vector (IV)
```

It is used by the encryption algorithm to ensure that encryption is randomized properly.

For AES-GCM, for example:

```text
Plaintext
   +
CEK
   +
IV
   ↓
AES-GCM
   ↓
Ciphertext + Authentication Tag
```

The IV itself isn't secret.

It can be transmitted as part of the JWE.

---

# 11. What is the Authentication Tag?

This is another important part.

JWE commonly uses **authenticated encryption** such as:

```text
AES-GCM
```

Encryption produces:

```text
Ciphertext
+
Authentication Tag
```

The authentication tag allows the recipient to detect whether the encrypted content was altered.

So JWE provides:

```text
Confidentiality
      +
Integrity
```

This is better than simply "encrypting bytes."

---

# 12. JWE vs JWS

Here's the comparison I'd recommend remembering:

|                   | JWS                | JWE                                      |
| ----------------- | ------------------ | ---------------------------------------- |
| Purpose           | Signing            | Encryption                               |
| Confidentiality   | ❌                  | ✅                                        |
| Integrity         | ✅                  | ✅                                        |
| Authenticity      | ✅                  | Depends on signing/authentication design |
| Payload readable? | Usually yes        | No                                       |
| Compact format    | 3 parts            | 5 parts                                  |
| Main concern      | "Who signed this?" | "Who can read this?"                     |

And:

```text
JWS:

Header.Payload.Signature


JWE:

Header.EncryptedKey.IV.Ciphertext.AuthTag
```

---

# 13. Can JWE and JWS be combined?

**Yes.**

This is where things get interesting.

Suppose you want:

1. Only the recipient can read the message.
2. Recipient can verify who created the message.

You can do:

```text
Payload
   ↓
JWS
   ↓
Signed data
   ↓
JWE
   ↓
Encrypted signed data
```

Conceptually:

```text
             Original Payload
                    ↓
                  JWS
                    ↓
             Signed Payload
                    ↓
                  JWE
                    ↓
          Encrypted Signed Payload
```

Recipient:

```text
JWE
 ↓
Decrypt
 ↓
JWS
 ↓
Verify signature
 ↓
Trusted payload
```

This gives you:

```text
Encryption
+
Signature
```

---

# 14. Where does JWE fit into OAuth?

This is the part most relevant to your roadmap.

You might think:

> "I've been using JWT access tokens. Why don't we always use JWE?"

Because **most OAuth deployments don't need encrypted JWT access tokens**.

A signed JWT is often sufficient:

```text
Access Token
     ↓
JWS
     ↓
Resource Server verifies signature
```

The Resource Server can inspect claims such as:

```json
{
  "iss": "https://auth.example.com",
  "sub": "123",
  "aud": "orders-api",
  "scope": "orders.read",
  "exp": 1780000000
}
```

That's useful because the Resource Server needs those claims.

---

# 15. When would JWE be useful?

JWE becomes useful when the token contains information that **shouldn't be visible to whoever possesses the token**.

For example:

```json
{
  "sub": "12345",
  "email": "user@example.com",
  "internalId": "ABC-123",
  "sensitiveData": "..."
}
```

A signed JWT exposes these claims.

A JWE doesn't.

So:

```text
JWS JWT
 ↓
Anyone possessing token can read claims

JWE JWT
 ↓
Only authorized recipient can decrypt claims
```

---

# 16. JWE does NOT automatically solve token theft

This is another important security distinction.

Suppose an attacker steals:

```text
JWE access token
```

They may not be able to decrypt it.

But if the API accepts it as a bearer token:

```http
Authorization: Bearer <JWE>
```

the attacker might still be able to **use it**.

So:

```text
JWE
 ↓
Protects confidentiality
```

It does **not automatically provide proof that the presenter is the legitimate owner of the token.

That's why your next advanced topic:

> **Proof-of-Possession (PoP) tokens**

is important.

You can think of it as:

```text
Bearer token

"Whoever has it can potentially use it."


PoP token

"Only the party possessing the required key can use it."
```

---

# 17. JWE and JWK/JWKS

This connects directly to the topic you just studied.

Remember:

```text
JWK
 ↓
Representation of a cryptographic key
```

and:

```text
JWKS
 ↓
Collection of JWKs
```

For JWE, you might have:

```text
Recipient Public JWK
        ↓
Used to encrypt/protect CEK
```

Recipient has:

```text
Private Key
```

which decrypts the CEK.

So your advanced-topic chain starts making sense:

```text
JWK/JWKS
   ↓
Represent/distribute keys
   ↓
JWS
   ↓
Sign/verify data
   ↓
JWE
   ↓
Encrypt/decrypt data
   ↓
PoP
   ↓
Bind token to a key
```

---

# 18. The key idea to remember

Don't try to memorize all the JWE algorithms yet. First lock in this mental model:

### JWS

```text
Private key
     ↓
   Sign
     ↓
Payload
     ↓
Public key verifies
```

### JWE

```text
Generate symmetric CEK
        ↓
Encrypt payload with CEK
        ↓
Encrypt CEK with recipient's public key
        ↓
       JWE
        ↓
Recipient private key
        ↓
Recover CEK
        ↓
Decrypt payload
```

### JWS + JWE

```text
Payload
   ↓
Sign
   ↓
Encrypt
   ↓
Send
   ↓
Decrypt
   ↓
Verify signature
   ↓
Payload
```

---

## One final distinction

If you remember only this from today's topic:

> **JWS protects authenticity/integrity through signatures. JWE protects confidentiality through encryption.**

And don't confuse:

```text
Base64URL ≠ Encryption
```

A normal JWT/JWS payload is Base64URL encoded, **not encrypted**.

```text
JWT/JWS
  ↓
Encoded → readable

JWE
  ↓
Encrypted → not readable without the key
```

For your OAuth roadmap, I'd consider **JWE understood** once you're comfortable with the **5-part structure, `alg` vs `enc`, CEK, hybrid encryption, and JWS-vs-JWE distinction**.
