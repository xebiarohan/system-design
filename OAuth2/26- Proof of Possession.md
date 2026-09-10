# Proof-of-Possession (PoP) Tokens

## 1. First: the problem with normal OAuth access tokens

A normal OAuth access token is usually a **Bearer token**.

That means:

> **Whoever possesses the token can use it.**

For example:

```http
GET /api/accounts
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

The API doesn't care *who* is presenting the token.

If an attacker steals it:

```text
Legitimate client
      |
      | Access Token
      v
    API
```

and the attacker gets a copy:

```text
Attacker
   |
   | stolen Access Token
   v
  API
```

The API may accept it.

That's the weakness PoP tokens address.

---

# 2. What is Proof-of-Possession?

A **Proof-of-Possession token** binds the access token to a cryptographic key.

Instead of simply saying:

> "I have the access token."

the client must prove:

> "I have the private key associated with this access token."

Conceptually:

```text
Access Token
     |
     +---- bound to ----> Public Key
                           |
                           |
                    Private Key
                       held by
                        Client
```

The client never sends the private key to the server.

---

# 3. Bearer vs PoP

### Bearer token

```text
Access Token
     |
     v
   Client
     |
     | Bearer token
     v
    API
```

Possession = authorization.

### PoP token

```text
Access Token
     |
     v
 Public Key
     ^
     |
Private Key
     |
   Client
```

The client must demonstrate possession of the private key.

So:

|                   | Bearer              | PoP                  |
| ----------------- | ------------------- | -------------------- |
| Token stolen      | Attacker can use it | Usually insufficient |
| Cryptographic key | Not required        | Required             |
| Request proof     | No                  | Yes                  |
| Security          | Lower               | Higher               |
| Complexity        | Low                 | Higher               |

---

# 4. How does it actually work?

Let's use an example.

Suppose:

```text
Client = Mobile application
Authorization Server = OAuth server
Resource Server = API
```

The client generates a key pair:

```text
             Client
               |
        +------+------+
        |             |
   Private Key    Public Key
        |             |
     SECRET        Shareable
```

For example:

```text
Private Key:
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----

Public Key:
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
```

The **private key stays on the client**.

---

# 5. Binding the token to the key

During OAuth authorization, the Authorization Server learns about the client's public key.

The access token can contain information identifying that key.

For example, conceptually:

```json
{
  "sub": "user123",
  "scope": "read write",
  "exp": 1799999999,
  "cnf": {
    "jkt": "abc123..."
  }
}
```

The important part is:

```json
"cnf"
```

`cnf` means **confirmation**.

It tells the Resource Server:

> "This token is associated with this key."

---

# 6. But how does the API know the client owns the private key?

This is the clever part.

The client creates a **cryptographic proof** for the HTTP request.

Conceptually:

```text
HTTP Request
     +
Access Token
     +
Private Key
     |
     v
Cryptographic Proof
```

The client sends:

```http
GET /api/accounts
Authorization: Bearer <access-token>
DPoP: <proof>
```

The `DPoP` header is one standardized way to implement PoP.

DPoP stands for:

**Demonstrating Proof of Possession**

---

# 7. What is inside the proof?

A DPoP proof is typically a signed JWT.

Conceptually:

```json
{
  "typ": "dpop+jwt",
  "alg": "ES256",
  "jti": "unique-request-id",
  "htm": "GET",
  "htu": "https://api.example.com/accounts",
  "iat": 1799999999
}
```

The important fields are:

### `htm`

HTTP method:

```text
GET
POST
PUT
DELETE
```

### `htu`

HTTP URI:

```text
https://api.example.com/accounts
```

### `iat`

Issued-at timestamp.

### `jti`

Unique identifier for the proof.

It helps prevent replay attacks.

And importantly, the JWT is **signed with the client's private key**.

---

# 8. Request flow

The overall flow looks like this:

```text
                  OAuth
               Authorization
                  Server
                     |
                     | Access Token
                     | bound to Public Key
                     v
                   Client
                     |
             Private Key
                stored here
                     |
                     | HTTP request
                     | Access Token
                     | DPoP Proof
                     v
                Resource Server
                     |
                     | Verify:
                     | 1. Access token
                     | 2. DPoP signature
                     | 3. Public key matches token
                     | 4. Request details match
                     v
                   API
```

---

# 9. What happens if the token is stolen?

Suppose an attacker steals:

```text
Access Token
```

but **doesn't have the private key**.

They try:

```http
GET /api/accounts
Authorization: Bearer <stolen-token>
```

The Resource Server says:

```text
Token is PoP-bound.

Where is the cryptographic proof?

❌ No valid proof
```

Request rejected.

Even if the attacker creates a new proof, they can't sign it with the legitimate client's private key.

So:

```text
Stolen token
     +
No private key
     =
❌ Not enough
```

That's the primary security benefit.

---

# 10. DPoP vs mTLS

You'll encounter two major PoP approaches.

### DPoP

```text
Client
  |
  | Access Token
  | DPoP Proof
  v
API
```

Uses application-level cryptographic proof.

Useful for:

* Browser/mobile applications
* Public clients
* Environments where client certificates are inconvenient

### mTLS

```text
Client
  |
  | TLS connection
  | Client Certificate
  v
API
```

The TLS connection itself proves possession of the client's private key.

So:

```text
PoP
├── DPoP
└── mTLS
```

Both provide proof-of-possession, but at different layers.

---

# 11. Why not simply use HTTPS?

Great question.

HTTPS protects the connection:

```text
Client <======== TLS ========> API
```

It prevents someone from simply sniffing the token over the network.

But consider:

```text
Client
   |
   | HTTPS
   v
API
```

If malware, a compromised application, a proxy, logs, browser storage, etc. causes the **access token itself to leak**, HTTPS doesn't magically make that stolen token useless.

PoP adds another protection:

```text
Access Token
     +
Private Key possession
     |
     v
Authorization
```

So HTTPS and PoP solve different problems.

---

# 12. The key idea to remember

For your OAuth roadmap, remember this distinction:

### Bearer token

> **"If you have the token, you can use it."**

### PoP token

> **"Having the token isn't enough; you must prove that you possess the associated private key."**

And:

```text
PoP
 |
 +-- DPoP → application-level proof
 |
 +-- mTLS → TLS-level proof
```

### Where this fits in your roadmap

Since you're already at the **advanced OAuth topics** stage and have covered things like JWK/JWKS, JWS and JWE, PoP is a good next connection:

```text
JWK
 ↓
JWS
 ↓
DPoP / PoP
 ↓
mTLS-bound access tokens
```

The most important next topic to study here is **DPoP**, because it shows exactly how the client creates the proof and how the Resource Server validates it.
