Topic **3. Password Authentication** is one of the most important concepts in authentication. Every modern authentication system starts here. If you understand this topic deeply, sessions, JWTs, OAuth, and identity providers become much easier.

---

# 3. Password Authentication

## What is Password Authentication?

Password authentication is the process of verifying that the password entered by a user matches the password associated with their account.

Simple flow:

```
User enters password
        ↓
Server verifies password
        ↓
If correct → Login successful
Else → Login failed
```

A common misconception is that the server stores the user's password directly.

**It should never do that.**

---

# Why Not Store Plain Text Passwords?

Imagine a database table like this:

| Username | Password    |
| -------- | ----------- |
| alice    | password123 |
| bob      | qwerty      |
| admin    | admin123    |

If a hacker steals the database:

* Every password is immediately visible.
* Users often reuse passwords across multiple websites.
* Attackers can try the same passwords on email, banking, and social media accounts.

This is why storing passwords in plain text is considered a critical security flaw.

Instead, we store a **hash** of the password.

---

# What is Hashing?

A hash function converts data into a fixed-length string.

Example:

```
Password:
hello123

Hash:
8d969eef6ecad3c29a3a629280e686cf...
```

Characteristics:

* Same input → same output
* One-way function
* Fast to verify
* Cannot realistically be reversed to recover the original password

Example:

```
"cat"
↓

Hash

↓

A8D43E7A912...
```

The server stores only the hash:

```
Username

alice

Password Hash

A8D43E7A912...
```

---

# Login Process Using Hashes

### Registration

User chooses:

```
Password = mySecret123
```

Server computes:

```
Hash(mySecret123)

↓

93ABF....
```

Database:

| Username | Password Hash |
| -------- | ------------- |
| alice    | 93ABF...      |

Notice:

The original password is **discarded**.

---

### Login

User enters:

```
mySecret123
```

Server computes:

```
Hash(mySecret123)

↓

93ABF...
```

Compare:

```
Stored Hash

93ABF...

Entered Hash

93ABF...
```

Match:

```
Login Successful
```

---

# Why Can't We Reverse a Hash?

A cryptographic hash function is designed to be **one-way**.

```
Password

↓

Hash

↓

Impossible (in practice) to recover password
```

Unlike encryption:

Encryption:

```
Plain Text

↓

Encrypted

↓

Decrypt back
```

Hashing:

```
Plain Text

↓

Hash

↓

No decryption
```

That's why passwords are hashed, not encrypted.

---

# Problem: Identical Passwords Produce Identical Hashes

Suppose:

Alice:

```
password123
```

Bob:

```
password123
```

Without extra protection:

```
Hash(password123)

↓

ABC123
```

Database:

| User  | Hash   |
| ----- | ------ |
| Alice | ABC123 |
| Bob   | ABC123 |

An attacker immediately knows they use the same password.

This also enables attacks using precomputed lookup tables.

To solve this, we use **salts**.

---

# Salt

A salt is a **random value generated for each password** and combined with the password before hashing.

Example:

```
Password

password123

Salt

xY89K@2!

↓

Hash(password + salt)
```

Even if two users choose the same password, different salts produce different hashes.

Example:

Alice:

```
Password

password123

Salt

ABC

↓

Hash

F92A...
```

Bob:

```
Password

password123

Salt

XYZ

↓

Hash

8DE2...
```

Database:

| User  | Salt | Hash |
| ----- | ---- | ---- |
| Alice | ABC  | F92A |
| Bob   | XYZ  | 8DE2 |

Now identical passwords no longer result in identical hashes.

---

# Why Store the Salt?

People often think the salt must be secret.

It doesn't.

The salt is stored alongside the hash:

```
User

alice

Salt

ABC123

Hash

98AF...
```

During login:

```
Entered Password

↓

Read Salt

↓

Password + Salt

↓

Hash

↓

Compare
```

The security comes from the randomness and uniqueness of the salt, not from hiding it.

---

# Pepper

A **pepper** is another secret value added to the password before hashing.

Unlike a salt:

* The same pepper is typically used for all users.
* It is **not stored in the database**.
* It is kept in a secure location, such as an environment variable or secret manager.

Example:

```
Password

password123

Salt

ABC

Pepper

SECRET_KEY

↓

Hash(password + salt + pepper)
```

If an attacker steals only the database, they still don't have the pepper.

---

## Salt vs Pepper

| Salt                      | Pepper                                             |
| ------------------------- | -------------------------------------------------- |
| Random for each user      | Usually one secret for the application             |
| Stored in the database    | Not stored in the database                         |
| Prevents identical hashes | Adds an extra layer if the database is compromised |
| Public                    | Secret                                             |

---

# Why Not Use SHA-256?

A common question is:

> "SHA-256 is secure. Why not hash passwords with SHA-256?"

Because SHA-256 is **designed to be fast**.

Modern hardware can compute billions of SHA-256 hashes per second.

If a database is leaked, attackers can rapidly try huge numbers of guessed passwords.

Password hashing algorithms are intentionally **slow**, making brute-force attacks much more expensive.

---

# Password Hashing Algorithms

These algorithms are specifically designed for storing passwords securely.

## 1. bcrypt

Very widely used.

Features:

* Built-in salt
* Adjustable work factor (cost)
* Intentionally slow

Example:

```
Password

↓

bcrypt

↓

Hash
```

A bcrypt hash contains information such as the algorithm version, cost factor, salt, and resulting hash.

---

## 2. scrypt

Adds another layer of defense by requiring significant memory as well as CPU time.

Benefits:

* Slow
* Memory-intensive
* More resistant to attackers using specialized hardware like GPUs and ASICs

---

## 3. Argon2

Winner of the Password Hashing Competition and widely recommended today.

Advantages:

* Configurable memory usage
* Configurable CPU cost
* Configurable parallelism
* Excellent resistance to GPU attacks

Variants include:

* **Argon2d** – optimized against GPU cracking but can leak information through side channels.
* **Argon2i** – designed to resist side-channel attacks.
* **Argon2id** – combines the strengths of both and is the recommended choice for most password storage.

---

# bcrypt vs scrypt vs Argon2

| Feature               | bcrypt | scrypt | Argon2id             |
| --------------------- | ------ | ------ | -------------------- |
| Slow                  | ✅      | ✅      | ✅                    |
| Built-in salt         | ✅      | ✅      | ✅                    |
| Memory-hard           | ❌      | ✅      | ✅                    |
| GPU resistance        | Good   | Better | Excellent            |
| Modern recommendation | Good   | Good   | Best for new systems |

---

# Complete Registration Flow

```
User

↓

Enter Password

↓

Generate Salt

↓

(Optional) Add Pepper

↓

Argon2id / bcrypt / scrypt

↓

Password Hash

↓

Store:

User
Salt (if applicable)
Hash
```

---

# Complete Login Flow

```
User enters password

↓

Read user's stored salt and hash

↓

(Optional) Add pepper

↓

Hash entered password using the same algorithm and parameters

↓

Compare the new hash with the stored hash

↓

Match?

↓

Yes → Login
No → Reject
```

Many libraries provide a `verify` function that handles the hashing and comparison safely.

---

# Common Attacks

### Brute Force

Trying many possible passwords until one works.

Example:

```
123456
password
password1
admin
...
```

Defense:

* Strong passwords
* Slow password hashing
* Rate limiting
* Account lockout after repeated failures

---

### Dictionary Attack

Trying passwords from a list of common or leaked passwords instead of every possible combination.

Defense:

* Reject commonly used passwords
* Encourage passphrases
* Slow password hashing

---

### Rainbow Table Attack

A rainbow table is a precomputed database mapping common passwords to their hashes.

Without salts:

```
password123

↓

Hash

↓

Attacker looks it up instantly
```

With unique salts, precomputed tables become impractical because the attacker would need a separate table for every possible salt.

---

# Best Practices

* Never store passwords in plain text.
* Never use reversible encryption for password storage.
* Use a dedicated password hashing algorithm such as **Argon2id**, **bcrypt**, or **scrypt**.
* Use a unique random salt for every password (these algorithms handle this automatically).
* Consider adding a server-side pepper for extra protection.
* Compare hashes using constant-time comparison (or the library's verify function) to reduce timing attack risks.
* Enforce strong password policies and rate-limit login attempts.
* Consider multi-factor authentication (MFA) for additional security.

---

# Real-World Example

Imagine you sign up for a website.

```
Password:
MyDog@2026
```

The server might do something like:

```
Salt:
u7K9!xL2

Pepper:
(server secret)

↓

Argon2id(
    password="MyDog@2026",
    salt="u7K9!xL2",
    pepper=server_secret
)

↓

Hash:
$argon2id$v=19$m=65536,t=3,p=4$...
```

The database stores the hash (and the embedded salt, depending on the algorithm's format), but **never the original password**. On login, the server repeats the hashing process with the entered password and verifies that it matches the stored hash.

---

## Key Takeaways

* **Hashing** transforms a password into a one-way value for secure storage.
* **Salts** ensure identical passwords produce different hashes and defeat rainbow tables.
* **Peppers** add an application-wide secret that's stored outside the database.
* **Argon2id** is generally the preferred choice for new applications, while **bcrypt** and **scrypt** remain secure, widely used options.
* Password verification works by hashing the entered password again with the same parameters and comparing the result to the stored hash—**the original password is never retrieved**.
