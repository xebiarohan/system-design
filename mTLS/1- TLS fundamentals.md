# TLS Fundamentals

## 1. What problem does TLS solve?

Imagine you have two components:

```text
Service A                         Service B
   │                                 │
   │────── HTTP request ────────────►│
   │                                 │
   │◄───── HTTP response ────────────│
```

If this communication happens over plain HTTP:

```text
http://service-b:8080/api/orders
```

the network traffic can potentially be observed or modified by someone sitting between the two services.

For example:

```text
Service A
    │
    │  "GET /api/orders"
    ▼
┌───────────────┐
│     Attacker  │
│               │
│ can potentially│
│ see / modify  │
│ the traffic   │
└───────────────┘
    │
    ▼
Service B
```

TLS is designed to provide three important security properties:

### 1. Confidentiality

Other parties shouldn't be able to read the communication.

```text
Service A ───── encrypted data ─────► Service B
```

Someone intercepting the traffic sees encrypted bytes rather than:

```text
username=rohan
amount=5000
account=12345
```

---

### 2. Integrity

The communication shouldn't be silently modified.

For example, Service A sends:

```text
amount = 100
```

An attacker shouldn't be able to change it to:

```text
amount = 100000
```

without the receiving side detecting that something went wrong.

---

### 3. Authentication

The client should be able to establish:

> "I'm actually talking to the server I intended to talk to."

This is extremely important.

Imagine you call:

```text
https://payment-service
```

You don't want an attacker to pretend to be the payment service.

TLS uses **certificates and cryptography** to establish this identity.

---

# 2. TLS vs HTTPS

These terms are often mixed together.

### TLS

TLS is the **security protocol**.

### HTTP

HTTP is the **application protocol**.

Put them together:

```text
HTTP
  ↓
TLS
  ↓
TCP
  ↓
IP
```

That's essentially HTTPS.

So:

```text
HTTPS = HTTP over TLS
```

For example:

```text
https://example.com
```

means HTTP communication protected by TLS.

And TLS isn't limited to HTTP.

You can use TLS to protect many protocols:

```text
HTTP ──────► TLS
SMTP ──────► TLS
LDAP ──────► TLS
gRPC ──────► TLS
custom TCP ─► TLS
```

That's an important distinction when you eventually work with service-to-service communication.

---

# 3. Where does TLS sit?

Think about the network stack:

```text
┌─────────────────────────┐
│       Application       │
│       HTTP / gRPC       │
├─────────────────────────┤
│          TLS            │  ← Security layer
├─────────────────────────┤
│          TCP            │
├─────────────────────────┤
│           IP            │
└─────────────────────────┘
```

Your Java application doesn't normally implement TLS itself.

For example:

```java
RestClient client = RestClient.create();

client.get()
      .uri("https://service-b/orders")
      .retrieve();
```

Your application says:

> "Send an HTTPS request."

Underneath, Java's TLS implementation handles the TLS protocol.

In Java, this functionality is primarily provided through **JSSE — Java Secure Socket Extension**.

You'll eventually encounter classes such as:

```text
SSLContext
KeyStore
TrustStore
KeyManager
TrustManager
SSLSocket
```

Don't worry about those yet. We'll build toward them.

---

# 4. The cryptography behind TLS

Before understanding the TLS handshake, you need three cryptographic concepts.

## 4.1 Symmetric encryption

With symmetric encryption, the same secret key is used for encryption and decryption.

```text
              SECRET KEY
                  │
                  ▼
Message ──────► Encrypt ──────► Ciphertext
                                  │
                                  ▼
                              Decrypt
                                  │
                             SECRET KEY
                                  │
                                  ▼
                               Message
```

For example:

```text
Message:
Hello Service B

             ↓ encryption

Encrypted:
8fA7$k29...blah...

             ↓ decryption

Hello Service B
```

The important thing is:

> Both sides need the same secret key.

This makes symmetric encryption very fast.

TLS therefore uses symmetric encryption for the actual application data.

---

# 5. The problem with symmetric encryption

Suppose Service A and Service B want to communicate securely.

They need a shared secret:

```text
Service A                  Service B

   KEY ────────────────────── KEY
```

But how do they safely get the key to each other?

You can't simply send:

```text
Here's our secret key:

ABC123
```

over the network.

An attacker could intercept it.

So we need another mechanism.

That's where **asymmetric cryptography** comes in.

---

# 6. Asymmetric cryptography

Asymmetric cryptography uses a pair of keys:

```text
Public Key
+
Private Key
```

For example:

```text
Service B

Public Key  → can be shared
Private Key → must remain secret
```

Conceptually:

```text
             Service B
           ┌─────────────┐
           │             │
           │ Public Key  │ ───► share with others
           │             │
           │ Private Key │ ───► KEEP SECRET
           │             │
           └─────────────┘
```

The public key can be distributed freely.

The private key must never leave the owner.

---

# 7. What can asymmetric cryptography do?

Two major things matter for TLS.

### A. Establishing secrets

Asymmetric cryptography can participate in establishing a shared secret.

### B. Digital signatures

A private key can be used to create a digital signature, and the corresponding public key can be used to verify it.

Conceptually:

```text
Private Key
     │
     ▼
Sign data
     │
     ▼
Digital Signature
```

Someone with the public key can verify:

```text
Data + Signature + Public Key
             │
             ▼
          Valid?
```

This allows us to establish identity and detect tampering.

---

# 8. Symmetric vs asymmetric encryption

This distinction is extremely important.

|              | Symmetric                | Asymmetric                            |
| ------------ | ------------------------ | ------------------------------------- |
| Keys         | One shared secret        | Public + private                      |
| Performance  | Very fast                | Much slower                           |
| Main TLS use | Encrypt application data | Authentication/key establishment      |
| Example      | AES                      | RSA / ECDSA / ECDH-related mechanisms |

TLS generally uses **both**.

Think:

```text
              TLS
               │
       ┌───────┴────────┐
       │                │
 Asymmetric          Symmetric
 cryptography        encryption
       │                │
       ▼                ▼
 Authentication     Actual data
 Key establishment  encryption
```

This combination is one of the fundamental ideas behind TLS.

---

# 9. What is a digital certificate?

Now we reach one of the most important concepts.

Suppose Service B gives Service A its public key:

```text
Service B:

"Here is my public key."

Public Key:
ABC123...
```

How does Service A know that the public key actually belongs to Service B?

An attacker could say:

```text
Hello Service A!

I'm Service B.

Here's my public key:

ATTACKER_KEY
```

Service A needs a trusted way to associate:

```text
Identity
   +
Public Key
```

That's what a **digital certificate** helps with.

---

# 10. Certificate

A simplified certificate looks conceptually like:

```text
┌──────────────────────────────────┐
│         Digital Certificate      │
├──────────────────────────────────┤
│ Subject: service-b.example.com   │
│                                  │
│ Public Key:                      │
│     ABCDEF123456...              │
│                                  │
│ Issuer:                          │
│     MyCompany Root CA            │
│                                  │
│ Valid From: ...                  │
│ Valid Until: ...                 │
│                                  │
│ Digital Signature: ...           │
└──────────────────────────────────┘
```

The important relationship is:

```text
Certificate
     │
     ├── Identity
     │
     ├── Public Key
     │
     ├── Validity
     │
     └── CA Signature
```

---

# 11. Who signs the certificate?

A **Certificate Authority (CA)**.

Think of a CA as a trusted authority that says:

> "I have verified/accepted that this public key is associated with this identity."

For example:

```text
             Company Root CA
                    │
                    │ signs
                    ▼
          Service B Certificate
```

The certificate contains the CA's signature.

When Service A receives Service B's certificate, it can verify the CA's signature.

---

# 12. Why do we trust a CA?

This is where the concept of **trust** enters TLS.

Your operating system/browser/JVM has a set of trusted certificates.

Conceptually:

```text
Trust Store

┌─────────────────────────┐
│ Trusted Root CA #1      │
│ Trusted Root CA #2      │
│ Trusted Root CA #3      │
│ Company Root CA         │
└─────────────────────────┘
```

If Service B presents a certificate:

```text
Service B Certificate
        │
        │ signed by
        ▼
Company Root CA
        │
        │
        ▼
Is Company Root CA trusted?
        │
      YES
        │
        ▼
Certificate can be trusted
```

This concept becomes **very important when we configure Java mTLS**.

---

# 13. Certificate chain

Real-world certificates aren't always directly signed by a root CA.

You often have:

```text
Root CA
   │
   ▼
Intermediate CA
   │
   ▼
Server Certificate
```

For example:

```text
                    Root CA
                       │
                       │ signs
                       ▼
               Intermediate CA
                       │
                       │ signs
                       ▼
              Service B Certificate
```

This is called a **certificate chain**.

Java will validate this chain when establishing TLS.

---

# 14. The TLS handshake

Now we're ready for the fun part.

Suppose:

```text
Java Service A
       │
       │ HTTPS
       ▼
Java Service B
```

Service A wants to establish a secure TLS connection.

Before sending normal application data, they perform a **TLS handshake**.

The exact handshake differs between TLS versions and cipher suites, but conceptually you can think of it like this.

---

# 15. Step 1 — ClientHello

Service A starts:

```text
Service A                         Service B

ClientHello ────────────────────►
```

The ClientHello contains information such as:

* supported TLS versions
* supported cryptographic algorithms/cipher suites
* random values
* key-exchange information/extensions
* hostname information via SNI in typical HTTPS usage

Essentially:

> "Here are the TLS capabilities I support."

---

# 16. Step 2 — ServerHello

Service B responds:

```text
Service A                         Service B

ClientHello ────────────────────►

             ◄──────────────────── ServerHello
```

The server selects appropriate parameters from the client's offerings.

Conceptually:

```text
Client supports:

TLS 1.2
TLS 1.3

Server chooses:

TLS 1.3
```

Similarly, cryptographic parameters are negotiated.

---

# 17. Step 3 — Server sends its certificate

Service B now proves its identity.

```text
Service A                         Service B

             ◄────────────── Server Certificate
```

The certificate contains:

```text
Service B identity
       +
Service B public key
       +
CA signature
       +
validity information
```

---

# 18. Step 4 — Client validates certificate

This is a **huge** part of TLS.

Service A receives:

```text
Service B Certificate
```

It checks things such as:

### Is it expired?

```text
Not Before < Current Time < Not After
```

### Does the hostname match?

For example:

```text
Requested:

https://payment.example.com

Certificate:

payment.example.com
```

Good.

But:

```text
Certificate:

some-other-service.example.com
```

That's a problem.

### Is the certificate chain trusted?

For example:

```text
Service B Certificate
        │
        ▼
Intermediate CA
        │
        ▼
Root CA
        │
        ▼
Trusted?
```

### Is the certificate cryptographically valid?

The signatures in the certificate chain must validate.

If validation fails, the TLS connection fails.

---

# 19. Step 5 — Key establishment

Now both sides establish cryptographic keying material that will be used for the connection.

In modern TLS, this is typically based on an ephemeral Diffie-Hellman-style key exchange.

The important conceptual result is:

```text
Service A                      Service B

        Shared secret/keying material
        ◄────────────────────────────►
```

An attacker observing the network should not be able to derive the same secret.

From this handshake, TLS derives symmetric keys for the connection.

---

# 20. Step 6 — Encrypted communication

Now the handshake is complete.

The application can communicate:

```text
Service A                         Service B

        ═══ encrypted data ═══════►
        ◄══ encrypted data ════════
        ═══ encrypted data ═══════►
```

HTTP sits above TLS:

```text
Your Java code
      │
      ▼
HTTP request

GET /orders
      │
      ▼
TLS
      │
      ▼
Encrypted bytes
      │
      ▼
TCP
```

---

# 21. Putting the handshake together

Here's the mental model I'd like you to remember:

```text
Service A                                      Service B
   │                                               │
   │────────────── ClientHello ──────────────────►│
   │                                               │
   │◄────────────── ServerHello ─────────────────│
   │                                               │
   │◄──────────── Server Certificate ────────────│
   │                                               │
   │                                               │
   │ Validate server certificate                   │
   │                                               │
   │                                               │
   │◄──────── TLS key establishment ─────────────►│
   │                                               │
   │                                               │
   │══════════ Encrypted communication ══════════►│
   │◄═════════ Encrypted communication ══════════│
```

**Important:** This is a conceptual flow, not a literal TLS 1.3 packet-by-packet transcript. TLS 1.3 compresses and changes several handshake details compared with older TLS versions.

---

# 22. Where authentication happens

Here's a subtle but very important point.

In **normal TLS**, the server authenticates itself to the client.

```text
Client
   │
   │ "Prove you're Service B"
   ▼
Server
   │
   │ Server certificate
   ▼
Client verifies
```

But the server doesn't normally require a certificate from the client.

So:

```text
NORMAL TLS

Client ──────────────────► Server
       Server proves identity
```

This is what HTTPS normally does.

---

# 23. Where mTLS comes in

With mTLS, the server says:

> "I also want YOU to prove your identity."

So:

```text
             NORMAL TLS

Client ──────────────────► Server
        Server authenticates


             mTLS

Client ──────────────────► Server
        Server authenticates

Client ◄────────────────── Server
        Client authenticates
```

The client presents its own certificate.

```text
Client Certificate
       │
       ▼
Server validates it
       │
       ▼
Trusted?
```

We'll study this separately in the next topic.

---

# 24. TLS does NOT necessarily mean "the user is authenticated"

This is another important distinction.

Suppose:

```text
Browser
   │
   │ HTTPS
   ▼
api.example.com
```

TLS proves that the browser has established a secure connection to the server.

It does **not** automatically mean:

> "Rohan is logged into the application."

Application authentication could instead happen using:

```text
OAuth
JWT
Session cookie
Username/password
OIDC
```

So keep these concepts separate:

```text
TLS
 │
 ├── Secure communication
 ├── Server authentication
 └── Optional client authentication (mTLS)


Application Authentication
 │
 ├── OAuth
 ├── OIDC
 ├── JWT
 ├── Session
 └── etc.
```

This distinction will fit nicely with the OAuth/OIDC material you've already been studying.

---

# 25. What does an attacker see?

Suppose Service A sends:

```http
POST /payment

{
    "amount": 5000,
    "account": "123456"
}
```

With plain HTTP:

```text
Attacker
   │
   ├── Can potentially see request
   ├── Can potentially modify request
   └── Can potentially inspect response
```

With TLS:

```text
Service A
    │
    │ encrypted
    ▼
Network
    │
    │ encrypted
    ▼
Service B
```

An attacker may still see some metadata such as IP addresses and traffic characteristics, but the TLS-protected application payload is encrypted and authenticated.

---

# 26. What TLS does NOT protect you from

TLS isn't magic.

For example, if Service B itself is compromised:

```text
Service A
    │
    │ TLS
    ▼
Compromised Service B
```

TLS doesn't help.

Service B legitimately decrypts the traffic.

Similarly, TLS doesn't prevent:

* application-level authorization bugs
* SQL injection
* compromised endpoints
* stolen private keys
* malicious application code
* an authenticated user abusing an API

TLS protects the **communication channel**.

---

# 27. TLS 1.2 vs TLS 1.3

You will see both in Java environments.

Today, you should primarily understand **TLS 1.3**, while being familiar with TLS 1.2 because many enterprise systems still support it.

Very roughly:

```text
TLS 1.2
   │
   ├── Older
   ├── More complex handshake
   └── Still widely deployed


TLS 1.3
   │
   ├── Modern
   ├── Faster handshake
   ├── Removes many obsolete algorithms
   └── Stronger security defaults
```

For your learning, don't get stuck memorizing every TLS 1.2 handshake packet.

Understand the concepts first.

---

# 28. How this maps to Java

Now let's connect everything to your world.

When your Java application makes:

```java
client.get()
      .uri("https://service-b/orders")
      .retrieve();
```

the stack conceptually looks like:

```text
Your Java Application
        │
        ▼
HTTP Client
        │
        ▼
     TLS
        │
        ├── SSLContext
        ├── KeyManager
        ├── TrustManager
        │
        ▼
     TCP Socket
        │
        ▼
     Network
```

Later, when we implement mTLS, you'll have something like:

```text
                 Java Service A
                       │
                       ▼
                  SSLContext
                   /       \
                  /         \
                 ▼           ▼
           KeyManager    TrustManager
               │              │
               ▼              ▼
          Client cert     Trusted CA
          + private key
```

And Service B has the corresponding setup.

This is where **keystore and truststore** will make sense.

---

# 29. The four concepts you should remember

If you remember only four things from today's topic, remember these:

### ① TLS provides secure communication

```text
Confidentiality
+
Integrity
+
Authentication
```

---

### ② Certificates bind identity to a public key

```text
Certificate
    =
Identity + Public Key + CA Signature + metadata
```

---

### ③ The TLS handshake establishes trust and encryption

```text
ClientHello
     ↓
ServerHello
     ↓
Certificate
     ↓
Certificate validation
     ↓
Key establishment
     ↓
Encrypted communication
```

---

### ④ Normal TLS and mTLS differ in client authentication

```text
TLS:

Client ──────────► Server
       Server identity


mTLS:

Client ──────────► Server
       Server identity

Client ◄────────── Server
       Client identity
```

---

# 🧠 Your mental model

I'd keep this diagram in your notes:

```text
                         TLS
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
    Confidentiality    Integrity      Authentication
          │               │                │
          │               │                ▼
          │               │          Certificates
          │               │                │
          │               │                ▼
          │               │               CA
          │               │
          └───────────────┴───────────────┐
                                          ▼
                                  TLS Handshake
                                          │
                                          ▼
                                  Shared keying material
                                          │
                                          ▼
                                Symmetric encryption
                                          │
                                          ▼
                                Secure application data
```

And one final distinction that will save you a **lot** of confusion later:

```text
                    TLS
                     │
        ┌────────────┴────────────┐
        │                         │
      Server                    Client
    identity                   identity
        │                         │
   certificate              certificate
        │                         │
        │                    only with mTLS
        │                         │
        └───────────┬─────────────┘
                    ▼
             Trusted connection
                    │
                    ▼
             Encrypted traffic
```

### Next topic

Now that TLS fundamentals are clear, the natural next step is **PKI + Certificates + CA + certificate chains**. That's the piece I'd recommend learning **before** mTLS itself, because once you understand PKI, mTLS becomes surprisingly straightforward.
