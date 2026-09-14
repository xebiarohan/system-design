# Passkeys and WebAuthn

## 1. First: What problem are they solving?

Traditional authentication:

```text
User
  ↓
Username + Password
  ↓
Server
  ↓
Authenticated
```

The problem is that passwords can be:

* stolen
* reused
* phished
* leaked in database breaches
* guessed
* intercepted through fake login pages

MFA improves this:

```text
Password
   +
OTP / Authenticator
```

But OTP can still be phished.

WebAuthn/passkeys solve this using **public-key cryptography**.

The basic idea:

```text
Private Key  → stays on user's device
Public Key   → stored by server
```

The private key is never sent to the server.

---

# 2. What is WebAuthn?

**WebAuthn = Web Authentication API**

It is a web standard that allows a website to authenticate a user using a cryptographic authenticator instead of a password.

The architecture looks like:

```text
Browser
   |
   | WebAuthn API
   ↓
Authenticator
   |
   | Public-key cryptography
   ↓
Authentication
```

The authenticator could be:

* phone
* laptop
* hardware security key
* platform authenticator

Examples of platform authentication include:

* Face ID
* Touch ID
* Windows Hello
* Android device authentication

---

# 3. What is a Passkey?

A **passkey is a credential based on public-key cryptography that can be used for passwordless sign-in**.

You can think of:

> **WebAuthn = the technology/API**

> **Passkey = the user-facing credential/experience built using that technology**

So they're closely related, but not exactly the same thing.

A simplified picture:

```text
                 Passkey
                    |
             Public-key credential
                    |
                 WebAuthn
                    |
              Browser API
                    |
               Authenticator
```

---

# 4. Registration

Let's say you create an account on:

```text
example.com
```

You click:

```text
Create a passkey
```

The browser invokes WebAuthn.

Conceptually:

```text
Website
   ↓
Browser
   ↓
WebAuthn API
   ↓
Authenticator
```

The authenticator generates a key pair:

```text
              Key Pair
                 |
        ┌────────┴────────┐
        ↓                 ↓
   Private Key        Public Key
        |                 |
        |                 ↓
        |             Server
        |
     Device
```

The critical rule:

**The private key stays with the authenticator.**

The server receives the public key and credential information.

---

# 5. Why is this better than passwords?

Compare the two.

### Password

```text
User
  |
  | password
  ↓
Server
```

The server needs some representation of the password.

If the password is stolen/phished, the attacker can potentially log in.

---

### Passkey

```text
User
  |
  ↓
Authenticator
  |
  | signs challenge
  ↓
Server
```

The private key never leaves the authenticator.

The server only has:

```text
Public Key
```

Therefore, stealing the server's credential database doesn't give the attacker the private key.

---

# 6. Authentication with a passkey

This is the really important part.

Suppose you've already registered.

The server has:

```text
User: Rohan

Credential:
    Public Key
```

You return to the website.

The server generates a random **challenge**:

```text
Challenge = random123...
```

Then:

```text
Server
  ↓
Challenge
  ↓
Browser
  ↓
Authenticator
```

The authenticator uses the private key to sign the authentication data:

```text
Private Key
     +
Challenge
     ↓
Digital Signature
```

Then:

```text
Authenticator
      ↓
Signature
      ↓
Browser
      ↓
Server
```

The server verifies:

```text
Signature
     +
Public Key
     ↓
Valid?
```

If valid:

```text
Authentication successful
```

---

# 7. The key idea

This is very similar to the JWS concept you recently learned.

With JWS:

```text
Data
 ↓
Private Key
 ↓
Signature
```

Then:

```text
Signature
 +
Public Key
 ↓
Verification
```

WebAuthn uses the same fundamental asymmetric cryptography idea.

But WebAuthn authentication is **not simply "a JWS token."**

It has its own protocol and signed data structures.

---

# 8. Where does Face ID / fingerprint come in?

This often confuses people.

Suppose you're logging in with a passkey:

```text
Website
   ↓
"Use your passkey"
   ↓
Phone
   ↓
Face ID
   ↓
Authenticated
```

It might look like:

```text
Face
 ↓
Private Key
 ↓
Signature
```

But that's not quite what's happening.

The biometric typically **unlocks/authorizes use of the credential on the device**.

Conceptually:

```text
             Face ID
                ↓
       Verify local user
                ↓
      Unlock/authorize
       passkey usage
                ↓
          Private Key
                ↓
             Sign
                ↓
           Signature
```

The website doesn't receive your fingerprint or face data.

---

# 9. Why are passkeys phishing-resistant?

This is one of the biggest advantages.

Imagine a phishing website:

```text
fake-google.com
```

You think you're logging into Google.

With a password:

```text
You
 ↓
Password
 ↓
fake-google.com
 ↓
Attacker gets password
```

Bad.

With WebAuthn/passkeys, credentials are associated with the website's **origin**.

Conceptually:

```text
Credential
   ↓
Associated with
   ↓
https://example.com
```

If an attacker creates:

```text
https://fake-example.com
```

the browser/authenticator won't simply use the legitimate site's credential there.

So:

```text
Legitimate site
      ↓
Correct origin
      ↓
Passkey works
```

but:

```text
Phishing site
      ↓
Different origin
      ↓
Passkey doesn't authenticate there
```

This origin binding is a major reason WebAuthn is considered **phishing-resistant**.

---

# 10. Passkeys and MFA

Remember your previous topic?

MFA:

```text
Password
   +
OTP
```

Passkey authentication can provide a strong authentication factor without requiring a password.

For example:

```text
Passkey
   +
Biometric/device PIN
```

The biometric/device unlock is local to the authenticator.

Depending on the authenticator and policy, this can provide strong user verification.

So instead of:

```text
Password
   +
SMS OTP
```

you can have:

```text
Passkey
   ↓
User verification
   ↓
Cryptographic authentication
```

---

# 11. Passkey vs traditional security key

You may have seen physical FIDO security keys.

For example:

```text
Computer
   ↓ USB
Security Key
```

The security key stores the credential.

A **platform passkey** might instead live on your:

```text
Phone
Laptop
Tablet
```

So there are broadly:

### Platform authenticator

```text
Phone / Laptop
     ↓
Built-in authenticator
     ↓
Passkey
```

### Roaming authenticator

```text
USB / NFC / Bluetooth security key
             ↓
          Passkey
```

Both use the same underlying WebAuthn/FIDO ecosystem.

---

# 12. What does the server store?

This is another major difference from passwords.

Password authentication:

```text
User
 ↓
Password
 ↓
Server stores password hash
```

Passkey:

```text
User
 ↓
Authenticator generates key pair

Private Key
   ↓
Authenticator

Public Key
   ↓
Server
```

The server stores things such as:

```text
User ID
Credential ID
Public Key
Credential metadata
```

But **not the private key**.

---

# 13. Passkeys and public/private keys

Let's connect this to your mTLS learning.

You already learned:

```text
Service A
    |
    | Private Key
    | Certificate containing Public Key
    ↓
Service B
```

Passkeys use the same fundamental asymmetric cryptography concept:

```text
User Device
     |
     | Private Key
     ↓
Authenticator

Server
     |
     | Public Key
     ↓
Credential Database
```

But there is an important difference:

### mTLS

The certificate binds a public key to an identity and TLS uses it as part of mutual authentication.

### WebAuthn

The credential is registered to a website/app, and the authenticator signs WebAuthn authentication data/challenges.

So don't think:

> "WebAuthn is basically mTLS for users."

The cryptographic concept is similar, but the protocols and purposes are different.

---

# 14. WebAuthn in an OIDC system

This is especially important for **your OAuth roadmap**.

Suppose you have:

```text
React Application
       |
       ↓
Identity Provider
       |
       ↓
WebAuthn / Passkey
```

The user clicks:

```text
Login with Passkey
```

The IdP performs WebAuthn authentication.

```text
React App
    ↓
Authorization Request
    ↓
Identity Provider
    ↓
WebAuthn authentication
    ↓
User authenticated
    ↓
Authorization decision
    ↓
Authorization Code
    ↓
React App
    ↓
Token Endpoint
    ↓
ID Token + Access Token
```

So WebAuthn can be used **inside the authentication process of the Identity Provider**.

OAuth/OIDC doesn't get replaced.

---

# 15. Complete OIDC + Passkey flow

Here's the architecture I'd remember:

```text
                    React App
                       |
                       | Authorization Request
                       ↓
               ┌──────────────────┐
               │ Identity Provider│
               └────────┬─────────┘
                        |
                        ↓
                 "Login with
                   Passkey"
                        |
                        ↓
                  WebAuthn API
                        |
                        ↓
                  Authenticator
                        |
                   User verifies
                  (Face/PIN/etc.)
                        |
                        ↓
                  Private Key
                        |
                        ↓
                    Signature
                        |
                        ↓
               Identity Provider
                        |
                  Verify signature
                        |
                        ↓
                 User authenticated
                        |
                        ↓
                 Authorization
                        |
                        ↓
                Authorization Code
                        |
                        ↓
                   React App
                        |
                        ↓
                  Token Endpoint
                        |
              ┌─────────┴─────────┐
              ↓                   ↓
         Access Token          ID Token
```

This is how passkeys fit into your OAuth/OIDC knowledge.

---

# 16. Discoverable credentials

One useful WebAuthn concept is **discoverable credentials**.

Traditional login:

```text
Username
   ↓
Find credential
   ↓
Authenticate
```

With a discoverable passkey:

```text
Website
   ↓
"Sign in with passkey"
   ↓
Authenticator
   ↓
Choose account
   ↓
Authenticate
```

You may not even need to type a username.

This is why passkeys can provide a very smooth:

```text
Passwordless + username-less
```

experience.

---

# 17. Passkey synchronization

This is another important modern concept.

Suppose you create a passkey on your phone.

You don't necessarily want:

```text
Phone only
```

You want to use it on your other devices too.

Modern passkey ecosystems can synchronize passkeys across a user's devices through platform credential managers.

Conceptually:

```text
             Passkey
                |
       ┌────────┼────────┐
       ↓        ↓        ↓
     Phone    Laptop    Tablet
```

The synchronization mechanism is provided by the platform/password manager ecosystem, not by WebAuthn itself.

This is one reason the term **passkey** is broader from a user-experience perspective than simply saying "WebAuthn credential."

---

# 18. Passkey vs password + OTP

Here's the comparison worth remembering:

|                                 | Password + OTP | Passkey         |
| ------------------------------- | -------------- | --------------- |
| Password required               | Yes            | No              |
| Server stores password verifier | Yes            | No              |
| Public-key cryptography         | No             | Yes             |
| Private key leaves device       | N/A            | No              |
| Phishing resistance             | Limited        | Strong          |
| OTP required                    | Usually        | No              |
| Biometric possible              | Sometimes      | Yes             |
| Replay-resistant                | Depends        | Yes             |
| User experience                 | More steps     | Usually simpler |

---

# 19. Three concepts you should keep separate

These are often mixed together:

### WebAuthn

The **web API/protocol** that allows websites to interact with authenticators.

### FIDO2

The broader authentication technology/ecosystem consisting of WebAuthn plus the authenticator/client protocols.

### Passkey

A user-friendly term for credentials based on public-key authentication, commonly implemented using WebAuthn/FIDO technology and often designed to be synchronized across devices.

Simplified:

```text
FIDO
 |
 +-- WebAuthn
 |
 +-- Authenticator protocols
 |
 +-- Passkey ecosystem
```

---

# 20. The most important mental model

You've now learned several authentication mechanisms. Put them together:

```text
                 Authentication
                       |
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
    Password          OTP            Passkey
       |               |                |
    Shared            Shared        Public-key
    secret            secret        cryptography
       |               |                |
       ↓               ↓                ↓
   Password          TOTP          WebAuthn
```

And in an OIDC architecture:

```text
                         OIDC
                          |
                 Authentication
                          |
             ┌────────────┴────────────┐
             ↓                         ↓
         Password                  Passkey
             |                         |
            MFA                   WebAuthn
             |                         |
             └────────────┬────────────┘
                          ↓
                   User authenticated
                          ↓
                    OAuth flow
                          ↓
                  Authorization Code
                          ↓
                       Tokens
```

### In one sentence:

> **WebAuthn is the protocol/API that lets a website authenticate using public-key cryptography, while passkeys are the modern, user-friendly credentials built around that technology—giving you passwordless and strongly phishing-resistant authentication.**

For your roadmap, the next logical thing to understand after this would be **SAML**, because you've now covered the modern OAuth/OIDC world and SAML will show you how enterprise SSO works in the older, XML/assertion-based ecosystem.
