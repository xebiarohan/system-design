# Topic 7: JWT (JSON Web Token) — Complete Guide

JWT is one of the most widely used authentication mechanisms in modern web applications and APIs. If you truly understand JWT, OAuth 2.0 and OpenID Connect become much easier to learn.

---

# 1. What is a JWT?

**JWT (JSON Web Token)** is a compact, URL-safe string used to securely transmit information between two parties as a JSON object.

A JWT is **not encrypted by default**. It is **signed**, which means:

* Anyone can read its contents.
* Nobody can modify it without invalidating the signature.

Think of it like a signed passport.

* Everyone can read your name and nationality.
* But they can't change your passport because the government's signature would no longer be valid.

---

# 2. Why JWT Exists

Before JWT, many applications used sessions.

### Session-based authentication

```
Browser
   |
Session ID
   |
Server
   |
Database / Session Store
```

For every request:

1. Browser sends Session ID.
2. Server looks it up.
3. Server finds the user.

The server must store session data.

---

JWT works differently.

```
Browser
   |
JWT
   |
Server
```

The JWT itself contains the user's identity and other information.

The server verifies the signature and does **not** need to look up session state (in many designs).

This is why JWT is often called **stateless authentication**.

---

# 3. JWT Structure

A JWT has three parts.

```
HEADER.PAYLOAD.SIGNATURE
```

Example

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

.

eyJzdWIiOiIxMjM0NTYiLCJuYW1lIjoiQWxpY2UiLCJhZG1pbiI6dHJ1ZX0

.

SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

It always follows this format:

```
xxxxx.yyyyy.zzzzz
```

---

# 4. Header

The header tells us:

* What type of token it is
* Which algorithm signed it

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning:

```
alg = HS256

Signature algorithm
```

```
typ = JWT

Token type
```

---

# 5. Payload

The payload contains **claims**.

Claims are pieces of information about the user or token.

Example:

```json
{
  "sub": "12345",
  "name": "Alice",
  "role": "admin"
}
```

This says:

```
User ID = 12345

Name = Alice

Role = admin
```

The payload is **Base64URL encoded**, not encrypted.

Anyone with the token can decode it.

Never store:

* passwords
* API secrets
* credit card numbers
* private information

inside a JWT.

---

# 6. Signature

The signature proves the token hasn't been modified.

For HS256:

```
Signature =
HMACSHA256(

Base64(Header)
+
"."
+
Base64(Payload),

Secret Key
)
```

The secret key is known only by the server.

If someone changes:

```
role = admin
```

to

```
role = superadmin
```

the signature becomes invalid.

The server rejects it.

---

# 7. Visual Overview

```
+-------------------+
| Header            |
| alg = HS256       |
| typ = JWT         |
+-------------------+

        |

Base64URL Encode

        |

+-------------------+
| Payload           |
| sub = 123         |
| role = admin      |
| exp = ...         |
+-------------------+

        |

Base64URL Encode

        |

Header.Payload

        |

Sign using Secret Key

        |

+-------------------+
| Signature         |
+-------------------+

Final JWT

xxxxx.yyyyy.zzzzz
```

---

# 8. What are Claims?

Claims are statements about the user or token.

Examples:

```
User ID

Username

Role

Email

Permissions

Expiration
```

There are three categories.

---

## A. Registered Claims

These are standardized names.

### sub

Subject

Usually the user ID.

Example

```json
{
  "sub": "98765"
}
```

---

### exp

Expiration time

```json
{
  "exp": 1750000000
}
```

The token becomes invalid after this Unix timestamp.

---

### iat

Issued At

When the token was created.

```json
{
  "iat": 1740000000
}
```

---

### iss

Issuer

Who created the token.

Example

```json
{
  "iss": "auth.mycompany.com"
}
```

---

### aud

Audience

Who should accept this token.

```json
{
  "aud": "payments-api"
}
```

If another API receives it:

```
aud = payments-api

Current API = inventory-api
```

It should reject the token.

---

## B. Public Claims

Custom names that avoid collisions by using agreed conventions or namespaces.

Example:

```json
{
  "department": "engineering"
}
```

---

## C. Private Claims

Application-specific claims.

Example

```json
{
  "theme": "dark",
  "favoriteColor": "blue"
}
```

Only your application understands them.

---

# 9. Standard Claims Summary

| Claim | Meaning           |
| ----- | ----------------- |
| sub   | User ID           |
| exp   | Expiration        |
| iat   | Issued At         |
| iss   | Issuer            |
| aud   | Intended audience |

These five are the most commonly used.

---

# 10. Example JWT Payload

```json
{
  "sub": "123456",

  "name": "Alice",

  "email": "alice@example.com",

  "role": "admin",

  "iss": "Auth Server",

  "aud": "Inventory API",

  "iat": 1700000000,

  "exp": 1700003600
}
```

---

# 11. Login Flow Using JWT

Imagine a user logs in.

```
POST /login
```

Request

```json
{
  "email": "alice@test.com",
  "password": "password123"
}
```

---

Server verifies the password.

If correct:

```
Generate JWT

↓

Sign JWT

↓

Return JWT
```

Response

```json
{
  "access_token": "xxxxx.yyyyy.zzzzz"
}
```

---

# 12. Using JWT

Every API request includes the token.

```
GET /profile
```

Header

```
Authorization: Bearer xxxxx.yyyyy.zzzzz
```

Server:

```
Receive JWT

↓

Verify Signature

↓

Check Expiration

↓

Read Claims

↓

Authorize Request
```

If valid:

```
200 OK
```

Otherwise:

```
401 Unauthorized
```

---

# 13. JWT Verification Process

Every protected request follows roughly this sequence:

```
Receive JWT

↓

Split into 3 parts

↓

Decode Header

↓

Decode Payload

↓

Recompute Signature

↓

Compare Signatures

↓

Check exp

↓

Check iss

↓

Check aud

↓

Accept or Reject
```

---

# 14. Why Expiration Exists

Suppose an attacker steals a token.

Without expiration:

```
Token works forever.
```

Bad.

With expiration:

```
Expires after 15 minutes.
```

Much safer.

Typical access token lifetime:

```
5–15 minutes
```

Refresh tokens are used to obtain new access tokens without forcing the user to log in again.

---

# 15. Common Mistakes

### Mistake 1

Thinking JWT is encrypted.

Wrong.

```
JWT

≠

Encryption
```

Anyone can decode it.

---

### Mistake 2

Putting passwords inside JWT.

Never do this.

---

### Mistake 3

Trusting payload without verifying signature.

Never do:

```
Decode JWT

↓

Trust User
```

Always:

```
Verify Signature

↓

Check exp

↓

Check iss

↓

Check aud

↓

Trust User
```

---

### Mistake 4

Using extremely long expiration times.

Bad:

```
exp = 1 year
```

Better:

```
15 minutes
```

---

# 16. Real Example

Suppose Alice logs in.

Server creates:

```json
{
  "sub": "1001",
  "name": "Alice",
  "role": "admin",
  "exp": 1755000000
}
```

The server signs it and returns:

```
xxxxx.yyyyy.zzzzz
```

Alice requests:

```
GET /orders
```

Header:

```
Authorization: Bearer xxxxx.yyyyy.zzzzz
```

The API:

1. Extracts the token.
2. Verifies the signature.
3. Checks `exp`.
4. Reads `sub` and `role`.
5. Allows or denies access based on the user's permissions.

---

# 17. JWT vs Session

| Feature                | Session                     | JWT                             |
| ---------------------- | --------------------------- | ------------------------------- |
| Stored on server       | Yes                         | No (typically)                  |
| Stored in browser      | Session ID                  | JWT                             |
| Server lookup          | Required                    | Often not required              |
| Scales across services | More complex                | Easier                          |
| Can carry user claims  | No (claims are server-side) | Yes                             |
| Revocation             | Easier (delete session)     | Harder without extra mechanisms |

---

# 18. Interview Questions

Make sure you can confidently answer questions like:

1. What is a JWT?
2. Why is JWT called stateless?
3. What are the three parts of a JWT?
4. What is the purpose of the header?
5. What is a claim?
6. What is the difference between registered and custom claims?
7. What do `sub`, `exp`, `iat`, `iss`, and `aud` mean?
8. Why is a signature necessary?
9. Can anyone read a JWT payload?
10. Why shouldn't sensitive data be stored in a JWT?
11. How does the server verify a JWT?
12. What happens if someone modifies the payload?
13. Why should access tokens expire?
14. What is the difference between JWT signing and encryption?
15. When would you choose JWT over sessions?

## Key takeaways

* A JWT is a **signed, compact token** used to carry identity and authorization-related information.
* It has three parts: **Header**, **Payload**, and **Signature**.
* The **payload is readable** by anyone who has the token; the **signature** protects against tampering.
* Standard claims like `sub`, `exp`, `iat`, `iss`, and `aud` help identify the user, control token lifetime, and ensure the token is used by the intended recipient.
* **Always verify** the signature and validate claims (especially `exp`, `iss`, and `aud`) before trusting a JWT.
* JWTs are excellent for stateless APIs, but they require careful handling of expiration, storage, and revocation to be secure.

Once you're comfortable with JWTs, the next logical topic is **JWT Signing (HS256 vs. RS256)**, where you'll learn exactly how signatures are created and verified using symmetric and asymmetric cryptography.
