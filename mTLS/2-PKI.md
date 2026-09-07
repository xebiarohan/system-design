# PKI + Certificates + CA + Certificate Chains

---

# 1. First: What problem is PKI solving?

Suppose you have:

```text
Service A                         Service B
   │                                 │
   │─────── TLS connection ─────────►│
```

Service A wants to know:

> **"How do I know that this really is Service B?"**

Service B sends its public key.

```text
Service B

Public Key
    │
    └──────────────► Service A
```

But there's a problem.

An attacker could intercept the connection:

```text
Service A
    │
    │ "Give me Service B's public key"
    ▼
 Attacker
    │
    │ "Sure! Here is my public key."
    ▼
Service A
```

Service A now has a public key—but **whose public key is it?**

This is the fundamental problem PKI solves.

---

# 2. What is PKI?

**PKI = Public Key Infrastructure**

Don't think of PKI as one particular piece of software.

It's an ecosystem of:

```text
PKI
│
├── Public/private keys
├── Digital certificates
├── Certificate Authorities (CA)
├── Certificate signing
├── Certificate validation
├── Certificate chains
├── Trust stores
├── Certificate revocation
└── Policies/processes
```

The purpose is essentially:

> **Establish and manage trust between identities and public keys.**

A simplified picture:

```text
                    PKI
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
    Identity       Keys       Certificates
        │                         │
        └───────────┬─────────────┘
                    ▼
              Trust relationship
```

---

# 3. Public and private keys

Before certificates, let's quickly establish this.

An entity has a key pair:

```text
                Service B
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
     Private Key        Public Key
          │                 │
          │                 │
       SECRET            SHAREABLE
```

The private key must remain secret.

The public key can be distributed.

For example:

```text
server-private.key
server-public.key
```

The private key is used for cryptographic operations such as creating signatures.

The public key is used to verify those signatures.

---

# 4. But a public key doesn't identify anyone

This distinction is **very important**.

Suppose Service B gives you:

```text
Public Key:

ABC123XYZ...
```

You know:

> "This is a valid public key."

But you don't know:

> "This public key belongs to Service B."

That's where a **certificate** comes in.

---

# 5. What is a digital certificate?

A digital certificate is essentially a **signed statement binding an identity to a public key**.

Conceptually:

```text
Certificate

┌─────────────────────────────────┐
│ Subject: service-b.example.com  │
│                                 │
│ Public Key: ABC123...            │
│                                 │
│ Issuer: MyCompany Root CA        │
│                                 │
│ Valid From: 2026-01-01           │
│ Valid Until: 2027-01-01          │
│                                 │
│ Digital Signature: XYZ...        │
└─────────────────────────────────┘
```

The important part is:

```text
Identity
    +
Public Key
    +
CA Signature
    =
Certificate
```

Technically, certificates contain more fields than this, but this is the mental model you want.

---

# 6. Who creates the certificate?

Usually a **Certificate Authority (CA)** signs it.

Imagine your company has its own internal CA:

```text
                 MyCompany CA
                      │
              signs certificates
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
   Service A certificate    Service B certificate
```

The CA effectively says:

> "I certify that this public key is associated with this identity."

The CA signs the certificate using the CA's private key.

---

# 7. What exactly does "CA signs the certificate" mean?

This is worth understanding.

Suppose Service B generates:

```text
Service B

Private Key B
Public Key B
```

It wants a certificate.

It sends a **Certificate Signing Request (CSR)** to the CA.

Conceptually:

```text
Service B
    │
    │ Public key + identity information
    │
    ▼
   CSR
    │
    ▼
Company CA
```

The CA verifies whatever identity/policy requirements apply.

Then the CA signs the certificate.

```text
Company CA Private Key
          │
          ▼
     Sign certificate
          │
          ▼
Service B Certificate
```

Now Service B has:

```text
Service B
│
├── Private Key B
│
└── Certificate B
```

---

# 8. The CSR

You'll encounter this constantly when working with real certificates.

**CSR = Certificate Signing Request**

A simplified CSR contains information such as:

```text
Subject
Public Key
Requested extensions
Digital signature
```

For example:

```text
CSR

Subject:
    CN=service-b.internal

Public Key:
    <Service B public key>

Requested SAN:
    DNS:service-b.internal

Signature:
    <signed using Service B private key>
```

Notice something important:

### The private key isn't sent to the CA.

You don't give the CA your private key.

Instead:

```text
Service B
│
├── Private Key
│       │
│       └── stays here
│
└── Public Key
        │
        ▼
       CSR
        │
        ▼
       CA
```

This is an important security principle.

---

# 9. The CA signs the certificate

The CA receives the CSR.

Conceptually:

```text
              CSR
               │
               ▼
        ┌──────────────┐
        │      CA      │
        │              │
        │ Verify CSR   │
        │ Verify       │
        │ identity     │
        └──────┬───────┘
               │
               ▼
      Signed Certificate
```

The resulting certificate says, in effect:

```text
"This public key belongs to
service-b.internal."

Signed by:
MyCompany CA
```

---

# 10. Why should Service A trust the CA?

Now we reach the **trust anchor**.

Service A needs to know:

> "Which CAs do I trust?"

For Java, this information is typically represented through a **truststore**.

For example:

```text
Service A Truststore

┌──────────────────────────┐
│ MyCompany Root CA        │
│ Other trusted CA         │
└──────────────────────────┘
```

So when Service B presents:

```text
Service B Certificate
        │
        │ issued by
        ▼
MyCompany Root CA
```

Service A can say:

```text
Do I trust MyCompany Root CA?

             YES
              │
              ▼
     Continue validation
```

This is why truststores matter so much in mTLS.

---

# 11. Root CA

A **Root CA** is at the top of a certificate hierarchy.

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
          Service B Cert
```

The root CA is generally **self-signed**.

Meaning:

```text
Issuer = Root CA
Subject = Root CA
```

But don't interpret "self-signed" as "automatically trusted."

A root certificate becomes trusted because it is configured as a **trust anchor** in the relevant trust store/system trust configuration.

This is a subtle but very important point.

---

# 12. Why have Intermediate CAs?

You might wonder:

> Why not just have one Root CA sign every service certificate?

You could, but it's generally better to protect the root CA and use intermediate CAs.

Think of the root CA as the company's **master key**.

You don't want it being used every day.

Instead:

```text
                     Root CA
                       │
                       │ signs
                       ▼
                Intermediate CA
                       │
            ┌──────────┼──────────┐
            │          │          │
            ▼          ▼          ▼
         Service A  Service B  Service C
         Certificate Certificate Certificate
```

The root CA can remain highly protected/offline, while intermediates handle operational certificate issuance.

---

# 13. Certificate chain

Now we can define a **certificate chain**.

Suppose Service B has:

```text
Service B Certificate
```

It was signed by:

```text
Intermediate CA
```

which was signed by:

```text
Root CA
```

Then:

```text
Service B Certificate
        │
        ▼
Intermediate CA Certificate
        │
        ▼
Root CA Certificate
```

This is the certificate chain.

Or visually:

```text
┌───────────────────────────┐
│         Root CA           │
│                           │
│ Trusted directly          │
└─────────────┬─────────────┘
              │
              │ signed
              ▼
┌───────────────────────────┐
│     Intermediate CA       │
│                           │
│ Signed by Root CA         │
└─────────────┬─────────────┘
              │
              │ signed
              ▼
┌───────────────────────────┐
│     Service B Cert        │
│                           │
│ Signed by Intermediate CA │
└───────────────────────────┘
```

---

# 14. How does certificate validation work?

This is probably the **single most important process to understand**.

Service A connects to Service B.

Service B sends:

```text
Service B certificate
+
Intermediate CA certificate(s)
```

Service A already has:

```text
Root CA
```

in its truststore.

So Java can construct:

```text
Service B
   │
   │ certificate signed by
   ▼
Intermediate CA
   │
   │ certificate signed by
   ▼
Root CA
   │
   │
   ▼
Trusted by Service A
```

Java then validates the chain.

---

# 15. Chain validation step-by-step

Suppose:

```text
Service B
Certificate B
```

is signed by:

```text
Intermediate CA
```

Java verifies:

```text
Certificate B
       │
       │ signature verification
       ▼
Intermediate CA public key
```

If valid:

```text
Certificate B
      ✓
```

Then Java checks the intermediate:

```text
Intermediate CA
       │
       │ signed by
       ▼
Root CA
```

If valid:

```text
Intermediate CA
      ✓
```

Then Java reaches:

```text
Root CA
```

and asks:

> Is this Root CA trusted?

If the root is in the truststore:

```text
YES
```

Then the chain can be trusted.

---

# 16. The trust path

You may hear another term:

**Trust path**

It's essentially the path from the presented certificate to a trusted root.

```text
Service B certificate
        │
        ▼
Intermediate CA
        │
        ▼
Root CA
        │
        ▼
Trusted anchor
```

So:

```text
Certificate chain
≈
Path from end-entity certificate to trusted root
```

---

# 17. What happens if the root CA isn't trusted?

Suppose Service B sends:

```text
Service B Certificate
        │
        ▼
Intermediate CA
        │
        ▼
Company Root CA
```

But Service A's truststore contains:

```text
Public CA 1
Public CA 2
Public CA 3
```

and **not**:

```text
Company Root CA
```

Then:

```text
Certificate
    │
    ▼
Intermediate
    │
    ▼
Company Root CA
    │
    X
    │
Not trusted
```

The TLS handshake fails.

This is one of the most common causes of Java TLS errors.

For example:

```text
PKIX path building failed
```

We'll later dissect exactly what that means.

---

# 18. Trust is directional

This becomes **very important for mTLS**.

Imagine:

```text
Service A                    Service B
   │                            │
   │                            │
   │────── TLS connection ─────►
```

Service A trusts some CA:

```text
Service A Truststore
        │
        └── Company CA
```

Service B trusts some CA:

```text
Service B Truststore
        │
        └── Company CA
```

If both certificates were issued by the same CA:

```text
                 Company CA
                /          \
               /            \
              ▼              ▼
       Service A Cert    Service B Cert
```

then both can potentially authenticate each other.

But trust does **not** automatically flow both ways.

Each side makes its own trust decision.

---

# 19. This is the foundation of mTLS

Now you can already see how mTLS will work.

Service A has:

```text
A private key
A certificate
A truststore containing trusted CA(s)
```

Service B has:

```text
B private key
B certificate
B truststore containing trusted CA(s)
```

During mTLS:

```text
Service A                              Service B
   │                                       │
   │        B Certificate                 │
   │◄──────────────────────────────────────│
   │                                       │
   │ Validate B certificate                │
   │                                       │
   │        A Certificate                 │
   │──────────────────────────────────────►│
   │                                       │
   │                       Validate A cert │
   │                                       │
   │◄════════ encrypted traffic ═════════►│
```

The CA infrastructure allows each side to decide whether the other's certificate is trusted.

---

# 20. One CA vs separate CAs

You could have:

```text
                 Company CA
                /          \
               ▼            ▼
          Service A      Service B
```

Or:

```text
       CA A                     CA B
        │                        │
        ▼                        ▼
   Service A                Service B
```

Then you might configure:

```text
Service A trusts CA B
Service B trusts CA A
```

This can be useful when different organizations/domains of trust are involved.

For an internal service mesh or enterprise environment, you might instead have a common internal PKI.

---

# 21. Certificate vs private key

This causes **a lot** of confusion, so make this distinction crystal clear.

### Certificate

Generally safe to send over the network.

```text
service-b.crt
```

contains the public key and identity information.

### Private key

Must remain secret.

```text
service-b.key
```

So:

```text
Certificate
    │
    ├── Public key
    ├── Identity
    ├── Issuer
    ├── Validity
    └── Signature
```

versus:

```text
Private key
    │
    └── SECRET
```

Never think:

> "My certificate is secret."

Usually it isn't.

Think:

> **"My private key is secret."**

---

# 22. Certificate vs CA certificate

Another common confusion.

Suppose:

```text
Company Root CA
        │
        ▼
Service B Certificate
```

You have two certificates:

```text
Root CA certificate
Service B certificate
```

They're both X.509 certificates, but they represent different roles.

### Service certificate

Identifies an entity:

```text
CN=service-b
```

### CA certificate

Identifies a CA and can be used to validate certificates it is authorized to sign.

```text
CN=Company Root CA
CA=true
```

---

# 23. What is X.509?

You'll encounter this term everywhere.

**X.509** is the standard format/specification used for public-key certificates.

So when someone says:

> "X.509 certificate"

they're talking about the standardized certificate structure used extensively by TLS.

For example:

```text
server.crt
```

might contain an X.509 certificate.

Java works heavily with X.509 certificates.

You'll encounter:

```java
X509Certificate
```

later.

---

# 24. What's inside an X.509 certificate?

A simplified view:

```text
X.509 Certificate
│
├── Version
├── Serial Number
├── Signature Algorithm
├── Issuer
├── Validity
│   ├── Not Before
│   └── Not After
├── Subject
├── Subject Public Key Info
├── Extensions
│   ├── Subject Alternative Name
│   ├── Key Usage
│   ├── Extended Key Usage
│   └── Basic Constraints
└── CA Signature
```

You don't need to memorize every field yet.

But **Subject Alternative Name (SAN)** is worth knowing now.

---

# 25. SAN — Subject Alternative Name

Suppose your service is accessed as:

```text
https://orders.mycompany.com
```

The certificate needs to identify that hostname.

Modern TLS hostname verification relies on the **Subject Alternative Name (SAN)** extension.

For example:

```text
Subject Alternative Name:

DNS: orders.mycompany.com
```

It can also contain things like IP addresses.

Example:

```text
SAN:

DNS:service-b
DNS:service-b.internal
IP:10.20.30.40
```

This is why you may encounter errors like:

```text
No subject alternative DNS name matching ...
```

when configuring Java TLS.

---

# 26. Certificate validity

Certificates have a validity period:

```text
Not Before
      │
      ▼
2026-01-01
      │
      │   valid
      │
      ▼
2027-01-01
      │
      ▼
Not After
```

If the current date is outside this period:

```text
Certificate expired
        ↓
TLS handshake fails
```

This is why certificate expiry is such a big operational problem.

---

# 27. Certificate revocation

What if a private key gets compromised?

Imagine:

```text
Service B private key
       │
       X
   COMPROMISED
```

Waiting until the certificate naturally expires isn't good enough.

PKI therefore has mechanisms for certificate revocation.

You'll encounter:

### CRL

**Certificate Revocation List**

A list of revoked certificates.

### OCSP

**Online Certificate Status Protocol**

Allows checking the status of a certificate.

Conceptually:

```text
Certificate
     │
     ▼
Is it revoked?
     │
 ┌───┴───┐
 │       │
 NO     YES
 │       │
 ▼       ▼
Continue Reject
```

For your first mTLS implementation, don't get too deep into revocation yet. But know that it exists.

---

# 28. Root CA vs Intermediate CA vs Server certificate

This table is worth keeping in your notes:

| Component          | Purpose                                                              |
| ------------------ | -------------------------------------------------------------------- |
| Root CA            | Top-level trust anchor                                               |
| Intermediate CA    | Issues/signs certificates under the root                             |
| Server certificate | Identifies server                                                    |
| Client certificate | Identifies client                                                    |
| Private key        | Secret cryptographic key corresponding to a certificate's public key |

And:

```text
                    Root CA
                       │
                       │ signs
                       ▼
                Intermediate CA
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        Server Certificate  Client Certificate
              │                 │
              ▼                 ▼
        Server Private Key  Client Private Key
```

---

# 29. A complete example

Let's build a realistic internal architecture.

You have:

```text
Payment Service
Order Service
```

You want:

```text
Order Service ───── mTLS ─────► Payment Service
```

Your company creates:

```text
Company Root CA
```

Then:

```text
Company Root CA
       │
       ▼
Company Intermediate CA
       │
       ├───────────────┐
       │               │
       ▼               ▼
Order Certificate   Payment Certificate
```

Order Service has:

```text
order-service.p12

    ├── Order private key
    ├── Order certificate
    └── Certificate chain
```

Payment Service has:

```text
payment-service.p12

    ├── Payment private key
    ├── Payment certificate
    └── Certificate chain
```

And each trusts the company's CA:

```text
Order Truststore
    └── Company Root CA


Payment Truststore
    └── Company Root CA
```

Now:

```text
                    Company Root CA
                           │
                           ▼
                  Intermediate CA
                    /           \
                   /             \
                  ▼               ▼
          Order Certificate   Payment Certificate
                  │               │
                  ▼               ▼
             Order Service   Payment Service
```

During mTLS:

```text
Order Service                         Payment Service
     │                                       │
     │──── ClientHello ─────────────────────►│
     │                                       │
     │◄── Payment Certificate ───────────────│
     │                                       │
     │ Validate Payment cert                │
     │ against trusted Company CA            │
     │                                       │
     │◄── Request Client Certificate ───────│
     │                                       │
     │──── Order Certificate ───────────────►│
     │                                       │
     │                  Validate Order cert  │
     │                  against Company CA   │
     │                                       │
     │══════ Secure encrypted channel ══════►│
```

Now you have **mutual authentication**.

---

# 30. Where Java comes into this

Now let's connect this to the Java terminology you'll encounter.

Suppose your Java application has:

```text
order-service.p12
```

containing:

```text
Private Key
Order Certificate
Certificate Chain
```

This is your **identity material**.

Then:

```text
order-truststore.p12
```

contains:

```text
Company Root CA
```

This is your **trust material**.

Conceptually:

```text
             Java Service
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
    KeyStore            TrustStore
        │                   │
        │                   │
        ▼                   ▼
 Private Key            Trusted CA
 Certificate
 Chain
```

Later Java's TLS stack uses:

```text
KeyStore
   ↓
KeyManager
   ↓
SSLContext


TrustStore
   ↓
TrustManager
   ↓
SSLContext
```

And:

```text
SSLContext
    ↓
TLS connection
```

This is the bridge between **PKI theory** and the Java code you'll write.

---

# 31. One thing that confuses almost everyone

You'll often hear:

> "Put the certificate in the truststore."

That's technically incomplete.

What you're really saying is:

> "Put the certificate(s) representing the CA(s) that I trust into my trust configuration."

For example:

```text
Payment Service trusts Order Service

Payment Truststore
        │
        └── Company CA certificate
```

It doesn't necessarily need to contain:

```text
order-service.crt
```

if the trust model is based on a CA and the presented certificate chains to that CA.

You can configure trust more narrowly or broadly depending on the architecture and PKI design.

---

# 32. The entire picture

Let's combine everything.

```text
                         PKI
                          │
                          ▼
                     Root CA
                          │
                          │ signs
                          ▼
                  Intermediate CA
                    /           \
                   /             \
                  ▼               ▼
        Service A Certificate   Service B Certificate
                  │               │
                  │               │
           A Private Key    B Private Key
                  │               │
                  ▼               ▼
             Service A       Service B
                  │               │
                  │               │
                  └───── TLS ─────┘
                         │
                         ▼
                  Certificate
                   validation
                         │
                         ▼
                    Trust CA
                         │
                         ▼
                Secure connection
```

And with mTLS:

```text
                  Service A
                  /       \
                 /         \
          A Identity       Trust
              │               │
          A Certificate    Company CA
              │               │
              └──────┐ ┌──────┘
                     │ │
                     ▼ ▼
                   mTLS
                     ▲ ▲
              ┌──────┘ └──────┐
              │               │
          B Certificate    Company CA
              │               │
          B Identity       Trust
                 \           /
                  \         /
                   Service B
```

---

# 33. The most important mental model

If you remember nothing else, remember this:

```text
PRIVATE KEY
    │
    │ proves possession / creates signatures
    ▼
CERTIFICATE
    │
    │ says "this public key belongs to this identity"
    ▼
CA SIGNATURE
    │
    │ establishes who vouched for the certificate
    ▼
CERTIFICATE CHAIN
    │
    │ leads toward
    ▼
ROOT CA
    │
    │ must be trusted
    ▼
TRUSTSTORE
```

Or, even shorter:

> **Certificate = "Who am I?"**
> **Private key = "I can prove I'm the owner of this certificate."**
> **CA = "I vouch for this certificate."**
> **Certificate chain = "Here's the path from this certificate to a trusted CA."**
> **Truststore = "Here are the CAs I trust."**

---

# 34. What you should understand before moving on

You don't need to memorize commands yet. You should be comfortable explaining this scenario:

```text
Service A
   │
   │ connects to
   ▼
Service B
```

Service B sends:

```text
B Certificate
+
Intermediate CA Certificate
```

Service A already trusts:

```text
Root CA
```

Therefore Java can validate:

```text
B Certificate
      ↓ signed by
Intermediate CA
      ↓ signed by
Root CA
      ↓
Trusted by Service A
```

And then TLS can establish the secure connection.

For **mTLS**, the exact same idea happens in the opposite direction:

```text
Service A Certificate
      ↓ signed by
Intermediate CA
      ↓
Root CA trusted by Service B
```

That's the core of mTLS.

---

## 🧩 One practical exercise I'd recommend

Before moving to the next topic, we should make this **real on your machine**.

Use Java's `keytool` to create:

```text
Root CA
   ↓
Server certificate
   ↓
Client certificate
```

Then inspect the certificates with:

```bash
keytool -list -v ...
```

and use OpenSSL to inspect the actual X.509 contents.

That exercise will make **CA → certificate → chain → truststore → mTLS** much more concrete than another hour of theory.

After that, the next logical topic in your roadmap is **Java Keystore vs Truststore + KeyManager + TrustManager + SSLContext**, which is where all of this finally connects directly to your Java code.
