## 1. What we're going to create

We'll have:

```text
                    Root CA
                mycompany-ca
                /          \
               /            \
              ▼              ▼
       Service A cert    Service B cert
              │              │
              ▼              ▼
          Service A       Service B
           Client           Server
```

Files:

```text
CA
├── ca-keystore.p12
└── ca.crt

Service A
├── service-a.p12
└── service-a-truststore.p12

Service B
├── service-b.p12
└── service-b-truststore.p12
```

And conceptually:

```text
service-a.p12
    ├── Service A private key
    └── Service A certificate
          └── signed by CA

service-b.p12
    ├── Service B private key
    └── Service B certificate
          └── signed by CA

service-a-truststore.p12
    └── CA certificate

service-b-truststore.p12
    └── CA certificate
```

This maps directly to the keystore/truststore distinction in your roadmap: **keystore = my identity**, **truststore = who I trust**. 

---

# 2. Step 1 — Create the Root CA

First, we create a CA private key and a self-signed CA certificate.

Run:

```bash
keytool -genkeypair \
  -alias mycompany-ca \
  -keyalg RSA \
  -keysize 4096 \
  -validity 3650 \
  -dname "CN=MyCompany Root CA, OU=Security, O=MyCompany, C=IN" \
  -ext bc=ca:true \
  -keystore ca-keystore.p12 \
  -storetype PKCS12
```

You'll be asked for a password.

For this learning exercise, let's say:

```text
changeit
```

### What happened?

`keytool` generated:

```text
ca-keystore.p12
```

Inside it:

```text
mycompany-ca
    │
    ├── CA private key
    │
    └── CA certificate
```

The certificate is **self-signed**.

That's important.

Normally:

```text
Root CA certificate
        ↑
   signed by itself
```

because there is no higher CA in our little lab.

---

# 3. Export the CA certificate

Other services don't need the CA's private key.

They only need the **CA certificate**.

Export it:

```bash
keytool -exportcert \
  -alias mycompany-ca \
  -keystore ca-keystore.p12 \
  -storetype PKCS12 \
  -file ca.crt
```

You'll now have:

```text
ca.crt
```

This is the certificate that we'll distribute to Service A and Service B.

### VERY IMPORTANT

Never distribute:

```text
ca-keystore.p12
```

because it contains:

```text
CA private key
```

Distribute only:

```text
ca.crt
```

Think:

```text
CA private key
       │
       │ SECRET
       ▼
   CA keystore


CA certificate
       │
       │ PUBLIC
       ▼
   Services
```

---

# 4. Step 2 — Create Service A's private key

Now we create Service A's identity.

```bash
keytool -genkeypair \
  -alias service-a \
  -keyalg RSA \
  -keysize 2048 \
  -validity 825 \
  -dname "CN=service-a, OU=Services, O=MyCompany, C=IN" \
  -ext SAN=dns:service-a \
  -keystore service-a.p12 \
  -storetype PKCS12
```

Now:

```text
service-a.p12
    │
    └── service-a
          ├── private key
          └── self-signed certificate
```

At this point, **Service A's certificate is NOT trusted by our CA yet**.

That's because we haven't asked the CA to sign it.

---

# 5. Generate Service A's Certificate Signing Request

This is where PKI starts getting interesting.

Create a CSR:

```bash
keytool -certreq \
  -alias service-a \
  -keystore service-a.p12 \
  -storetype PKCS12 \
  -file service-a.csr
```

You'll get:

```text
service-a.csr
```

CSR means:

> Certificate Signing Request

Conceptually:

```text
Service A

Private Key
    │
    │
    ▼
Public Key
    │
    ▼
CSR
    │
    │
    ▼
CA
```

The CSR contains information such as:

```text
Subject:
    CN=service-a

Public Key:
    Service A public key

Requested extensions:
    SAN=dns:service-a
```

**It does NOT contain Service A's private key.**

That's crucial.

---

# 6. Step 3 — CA signs Service A's certificate

Now our CA acts as the Certificate Authority.

Run:

```bash
keytool -gencert \
  -alias mycompany-ca \
  -keystore ca-keystore.p12 \
  -storetype PKCS12 \
  -infile service-a.csr \
  -outfile service-a.crt \
  -validity 825 \
  -ext KU=digitalSignature,keyEncipherment \
  -ext EKU=clientAuth,serverAuth \
  -ext SAN=dns:service-a
```

Now:

```text
service-a.crt
```

has been signed by:

```text
MyCompany Root CA
```

So the relationship is:

```text
MyCompany Root CA
        │
        │ signs
        ▼
Service A Certificate
```

---

# 7. Import the CA certificate into Service A's keystore

Before importing the signed Service A certificate, import the CA certificate.

```bash
keytool -importcert \
  -alias mycompany-ca \
  -file ca.crt \
  -keystore service-a.p12 \
  -storetype PKCS12
```

You should get a prompt asking whether you trust the certificate.

Answer:

```text
yes
```

Now Service A's keystore contains:

```text
service-a.p12

├── mycompany-ca
│     └── CA certificate
│
└── service-a
      └── private key
```

---

# 8. Import Service A's signed certificate

Now:

```bash
keytool -importcert \
  -alias service-a \
  -file service-a.crt \
  -keystore service-a.p12 \
  -storetype PKCS12
```

This is an important step.

`keytool` recognizes that the alias `service-a` already has a private key and associates the signed certificate with that private key.

The result is effectively:

```text
service-a.p12

service-a
   │
   ├── Private Key
   │
   └── Certificate
          │
          └── signed by MyCompany Root CA
```

---

# 9. Step 4 — Create Service B's identity

Exactly the same process.

Generate the key:

```bash
keytool -genkeypair \
  -alias service-b \
  -keyalg RSA \
  -keysize 2048 \
  -validity 825 \
  -dname "CN=service-b, OU=Services, O=MyCompany, C=IN" \
  -ext SAN=dns:service-b \
  -keystore service-b.p12 \
  -storetype PKCS12
```

Create CSR:

```bash
keytool -certreq \
  -alias service-b \
  -keystore service-b.p12 \
  -storetype PKCS12 \
  -file service-b.csr
```

CA signs it:

```bash
keytool -gencert \
  -alias mycompany-ca \
  -keystore ca-keystore.p12 \
  -storetype PKCS12 \
  -infile service-b.csr \
  -outfile service-b.crt \
  -validity 825 \
  -ext KU=digitalSignature,keyEncipherment \
  -ext EKU=clientAuth,serverAuth \
  -ext SAN=dns:service-b
```

Import CA certificate:

```bash
keytool -importcert \
  -alias mycompany-ca \
  -file ca.crt \
  -keystore service-b.p12 \
  -storetype PKCS12
```

Finally import Service B's signed certificate:

```bash
keytool -importcert \
  -alias service-b \
  -file service-b.crt \
  -keystore service-b.p12 \
  -storetype PKCS12
```

Now:

```text
service-b.p12

service-b
   │
   ├── Private Key
   │
   └── Certificate
          │
          └── signed by MyCompany Root CA
```

---

# 10. Step 5 — Create Service A's truststore

Now we move to the **trust** side.

Service A needs to answer:

> "Do I trust Service B's certificate?"

We don't put Service B's private key anywhere.

Service A only needs the CA certificate.

Create:

```bash
keytool -importcert \
  -alias mycompany-ca \
  -file ca.crt \
  -keystore service-a-truststore.p12 \
  -storetype PKCS12
```

Result:

```text
service-a-truststore.p12

└── mycompany-ca
      └── CA certificate
```

Therefore:

```text
Service A

service-a.p12
    │
    └── My identity
         ├── private key
         └── certificate


service-a-truststore.p12
    │
    └── Who I trust
         └── MyCompany CA
```

---

# 11. Step 6 — Create Service B's truststore

Same thing:

```bash
keytool -importcert \
  -alias mycompany-ca \
  -file ca.crt \
  -keystore service-b-truststore.p12 \
  -storetype PKCS12
```

Now:

```text
service-b-truststore.p12

└── mycompany-ca
      └── CA certificate
```

---

# 12. Final directory

You should now have something like:

```text
mtls/
│
├── ca-keystore.p12
├── ca.crt
│
├── service-a.p12
├── service-a.csr
├── service-a.crt
├── service-a-truststore.p12
│
├── service-b.p12
├── service-b.csr
├── service-b.crt
└── service-b-truststore.p12
```

The `.csr` and `.crt` files are intermediate artifacts. The important runtime files are:

```text
                    CA
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    Service A cert        Service B cert
          │                     │
          ▼                     ▼
    service-a.p12          service-b.p12
          │                     │
          │                     │
          ▼                     ▼
 service-a-truststore   service-b-truststore
          │                     │
          └──────────┬──────────┘
                     │
                     ▼
                  ca.crt
```

---

# 13. The really important part: understand what each file means

This is the part I'd memorize, not the commands.

### CA

```text
ca-keystore.p12
```

Contains:

```text
CA private key
+
CA certificate
```

Used by the **CA** to sign certificates.

---

### Service A

```text
service-a.p12
```

Contains:

```text
Service A private key
+
Service A certificate
+
CA certificate/chain
```

Used by Service A to **prove its identity**.

---

### Service B

```text
service-b.p12
```

Contains:

```text
Service B private key
+
Service B certificate
+
CA certificate/chain
```

Used by Service B to **prove its identity**.

---

### Service A truststore

```text
service-a-truststore.p12
```

Contains:

```text
CA certificate
```

Used by Service A to determine:

```text
"Do I trust Service B?"
```

---

### Service B truststore

```text
service-b-truststore.p12
```

Contains:

```text
CA certificate
```

Used by Service B to determine:

```text
"Do I trust Service A?"
```

---

# 14. Now see the mTLS handshake

This is where all these files suddenly make sense.

Service A calls Service B:

```text
Service A                              Service B
   │                                      │
   │          ClientHello                 │
   │─────────────────────────────────────►│
   │                                      │
   │          Server Certificate          │
   │◄─────────────────────────────────────│
   │                                      │
   │                                      │
   │ Verify using                         │
   │ service-a-truststore                 │
   │                                      │
   │                                      │
   │          Client Certificate          │
   │─────────────────────────────────────►│
   │                                      │
   │                         Verify using │
   │                         service-b-   │
   │                         truststore   │
   │                                      │
   │◄────────── Handshake complete ───────│
   │                                      │
   │══════ Encrypted communication ══════►│
```

And the trust relationships are:

```text
Service A
   │
   │ trusts
   ▼
MyCompany CA
   │
   │ signed
   ▼
Service B certificate
```

and:

```text
Service B
   │
   │ trusts
   ▼
MyCompany CA
   │
   │ signed
   ▼
Service A certificate
```

That's **mutual TLS**.

---

## One subtle but VERY important point

You might wonder:

> "Why don't we simply put Service B's certificate directly into Service A's truststore?"

You *can* do that in some setups.

But with a CA-based PKI, you generally trust the **CA**, not individual service certificates.

So:

```text
Service A
    │
    ▼
Trust CA
    │
    ├── Service B certificate ✓
    ├── Service C certificate ✓
    └── Service D certificate ✓
```

provided those certificates are properly issued and meet the validation requirements.

This is why CA-based PKI scales much better.

---

## 🔥 One command I'd like you to run

After creating everything, inspect the Service A keystore:

```bash
keytool -list -v \
  -keystore service-a.p12 \
  -storetype PKCS12
```

And Service B:

```bash
keytool -list -v \
  -keystore service-b.p12 \
  -storetype PKCS12
```

Look specifically for:

```text
Entry type: PrivateKeyEntry
```

for the service alias.

And for the CA:

```text
Entry type: trustedCertEntry
```

That distinction is **extremely useful** when debugging Java mTLS.

### Your mental model should now be

```text
                 CA
                 │
          signs certificates
             /       \
            /         \
           ▼           ▼
      Service A     Service B
      private key   private key
      certificate   certificate
           │           │
           ▼           ▼
       Keystore     Keystore
           
           ▲           ▲
           │           │
       Trust CA      Trust CA
           │           │
           ▼           ▼
       Truststore   Truststore
```

Once this is clear, the next step is to take **these exact files and configure two Java programs with `SSLContext`**, so you can see how `KeyStore → KeyManager`, `TrustStore → TrustManager`, and finally `SSLContext` fit together. That is the bridge from `keytool` to the Java TLS internals in your roadmap. 
