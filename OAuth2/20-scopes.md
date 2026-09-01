> **“What is this access token allowed to do?”**

They are not the same thing as roles or permissions, although they are closely related.

---

# 1. What is a Scope?

A **scope** is a named boundary of access that a client requests and an authorization server grants.

For example:

```text
read:profile
read:orders
write:orders
delete:orders
```

Suppose an application asks:

```text
scope=read:profile read:orders
```

The authorization server may issue an access token containing:

```text
scope = "read:profile read:orders"
```

The client can then use that token to access APIs covered by those scopes.

Think:

```text
Scope
  ↓
What access was delegated to this token?
```

---

# 2. Real-World Example

Imagine you have a banking API.

It has:

```text
GET  /accounts
GET  /transactions
POST /transfers
```

You could define scopes:

```text
accounts:read
transactions:read
transfers:write
```

A mobile banking application might request:

```text
accounts:read transactions:read
```

The user approves it.

The access token might effectively represent:

```text
User: Alice

Scopes:
    accounts:read
    transactions:read
```

Now:

```text
GET /accounts
```

✅ Allowed

```text
GET /transactions
```

✅ Allowed

But:

```text
POST /transfers
```

❌ Not allowed

because the token doesn't have:

```text
transfers:write
```

---

# 3. Scope vs Permission

This distinction is important for your roadmap.

A **permission** is generally an authorization capability:

```text
read_invoice
delete_invoice
create_invoice
```

A **scope** is an OAuth mechanism for expressing the access being delegated to a client/token.

For example:

```text
Scope:

invoices:read
```

could correspond to an API permission:

```text
permission:

invoice.read
```

So conceptually:

```text
OAuth Scope
     ↓
Delegated API access
     ↓
API permission
     ↓
Can perform operation
```

They can be named similarly, but **scope and permission are not inherently the same concept**.

---

# 4. Scope vs Role

This is even more important.

Suppose your application has:

```text
Admin
Manager
User
```

These are **roles**.

A role represents a collection of capabilities for a user.

For example:

```text
Admin
 ├── users:read
 ├── users:write
 ├── reports:read
 └── reports:delete
```

Now imagine an OAuth client:

```text
Reporting App
```

It only needs:

```text
reports:read
```

Even if Alice is an Admin, you don't necessarily want to give the Reporting App all of Alice's administrative capabilities.

You can have:

```text
User role:

Admin

        ↓

Potential permissions:

users:read
users:write
reports:read
reports:delete

        ↓

OAuth delegated scopes:

reports:read
```

This is one of the key ideas behind OAuth.

**The user's authority and the client's delegated authority are separate concepts.**

---

# 5. The Important Mental Model

When you see:

```text
scope=read:orders
```

don't think:

> "The user has the read:orders role."

Think:

> **"The client is requesting permission to access the user's orders in this way."**

A simplified model is:

```text
                  USER
                   │
                   │ has
                   ↓
                 ROLES
                   │
                   ↓
              PERMISSIONS
                   │
                   │
                   ↓
            ┌──────────────┐
            │ Authorization│
            │    Server    │
            └──────┬───────┘
                   │
            grants scopes
                   │
                   ↓
              ACCESS TOKEN
                   │
             scope = ...
                   │
                   ↓
            RESOURCE SERVER
                   │
                   ↓
             Allow / Deny
```

---

# 6. Where Does the Scope Come From?

In OAuth Authorization Code Flow, the client sends something like:

```http
GET /authorize?
    client_id=123
    &response_type=code
    &scope=orders:read orders:write
```

The important part is:

```text
scope=orders:read orders:write
```

The client is saying:

> "I want these capabilities."

The authorization server decides whether those scopes can be granted.

The user may see a consent screen such as:

```text
Acme App wants permission to:

☑ View your orders
☑ Create orders

Cancel       Allow
```

After authorization, the access token is issued with the granted scopes.

---

# 7. Scope Is Usually Associated With the Access Token

For example, imagine the access token has claims like:

```json
{
  "sub": "user123",
  "scope": "orders:read orders:write",
  "exp": 1790000000
}
```

The resource server receives:

```http
Authorization: Bearer <access_token>
```

It verifies the token.

Then it checks:

```text
Does token have orders:read?
```

If yes:

```text
GET /orders
```

can proceed.

For:

```text
DELETE /orders/123
```

the API might require:

```text
orders:delete
```

If the token doesn't have it:

```text
403 Forbidden
```

---

# 8. Scopes Are Not Just "CRUD"

You will often see:

```text
read
write
delete
```

But scopes can be designed around business capabilities.

For example:

```text
calendar:read
calendar:write

contacts:read

email:send

payments:create

payments:refund
```

A GitHub-like API might have scopes representing things such as:

```text
repository:read
repository:write
organization:read
```

The exact scope vocabulary is defined by the authorization server/API.

---

# 9. OIDC Scopes Are Slightly Different

This connects directly to your **Phase 4 — OpenID Connect** section.

OIDC defines standard scopes such as:

```text
openid
profile
email
address
phone
```

For example:

```text
scope=openid profile email
```

means the client is asking for OIDC-related identity information.

The important one is:

```text
openid
```

It tells the authorization server:

> "I am making an OpenID Connect authentication request."

So:

```text
OAuth API scopes

orders:read
payments:write
files:read
```

are typically about **access to resources**.

Whereas:

```text
OIDC scopes

openid
profile
email
```

are about **identity information**.

---

# 10. Scope vs Claims

You'll encounter another important distinction:

```text
Scopes
Claims
```

A scope is a requested/granted access boundary.

Claims are pieces of information about the subject/token.

For example:

```json
{
  "sub": "123",
  "iss": "https://auth.example.com",
  "aud": "orders-api",
  "scope": "orders:read",
  "email": "alice@example.com"
}
```

Here:

```text
sub
iss
aud
email
```

are claims.

And:

```text
orders:read
```

is a scope value represented in the token.

Don't fall into the trap of thinking:

> "Everything inside a JWT is a scope."

It isn't.

---

# 11. Scopes and Least Privilege

One of the biggest reasons scopes exist is **least privilege**.

Imagine an application only needs to read your calendar.

Bad design:

```text
Give application everything

users:read
users:write
calendar:read
calendar:write
payments:read
payments:write
files:read
files:write
```

Better:

```text
calendar:read
```

The application gets only the access it needs.

This reduces the damage if:

```text
Application gets hacked
        ↓
Access token stolen
        ↓
Attacker gets only limited API access
```

instead of:

```text
Attacker
   ↓
Full account access
```

---

# 12. Scopes Don't Automatically Mean the User Can Do Something

This is a subtle but **very important** point.

Suppose:

```text
Alice
```

has no permission to access:

```text
Bob's private orders
```

The OAuth client somehow has:

```text
orders:read
```

That doesn't automatically mean:

```text
GET /users/bob/orders
```

is allowed.

The resource server may need to evaluate:

```text
Does token have orders:read?

AND

Is Alice allowed to access this particular resource?
```

So authorization can become:

```text
Token scope
      +
User permissions
      +
Resource ownership
      +
Other policies
      ↓
Allow / Deny
```

This is where scopes start interacting with **RBAC and ABAC**.

---

# 13. Scopes + RBAC

Imagine:

```text
Alice
Role = Admin
```

Admin has:

```text
reports:read
reports:delete
```

But the OAuth application requests:

```text
reports:read
```

The resulting token might only have:

```text
scope = reports:read
```

Therefore:

```text
Role authority
        +
OAuth delegation
        ↓
Effective API access
```

The exact implementation varies, but conceptually the OAuth grant should not magically expand beyond what the authorization system permits.

---

# 14. Scopes + ABAC

ABAC can make this even more interesting.

Suppose:

```text
scope = documents:read
```

But your policy says:

```text
User can read documents
ONLY IF

document.department == user.department
```

Then:

```text
Scope says:
"You may perform document-read operations."

ABAC says:
"You may read THIS document under THESE conditions."
```

So:

```text
Scope
  ↓
Coarse-grained delegated capability

ABAC
  ↓
Fine-grained policy decision
```

This is a useful mental model.

---

# 15. A Complete Example

Suppose you build:

```text
Photo App
```

The app wants to access a user's photos.

It requests:

```text
photos:read
```

OAuth flow:

```text
User
  ↓
Photo App
  ↓
Authorization Server
  ↓
"Photo App wants photos:read"
  ↓
User approves
  ↓
Authorization Code
  ↓
Token Endpoint
  ↓
Access Token
```

Access token:

```json
{
  "sub": "alice",
  "scope": "photos:read",
  "aud": "photos-api",
  "exp": 1790000000
}
```

Photo App calls:

```http
GET /photos
Authorization: Bearer ACCESS_TOKEN
```

Photos API checks:

```text
1. Is token valid?
2. Is token intended for this API?
3. Is token expired?
4. Does token contain photos:read?
5. Is Alice allowed to access these photos?
```

If all relevant checks pass:

```text
200 OK
```

If scope is missing:

```text
403 Forbidden
```

---

# 16. The Best Mental Model for Your Roadmap

I'd put this in your notes:

```text
Authentication
    ↓
Who is this?
    ↓
User identity


Authorization
    ↓
What can they do?
    ↓
Roles / Permissions / Policies


OAuth
    ↓
What access is being delegated
to a client?
    ↓
Scopes


Access Token
    ↓
Represents delegated API access
    ↓
Scopes


Resource Server
    ↓
Validates token
    ↓
Checks scopes
    ↓
Applies additional authorization rules
    ↓
Allow / Deny
```

And remember this distinction:

| Concept            | Main question                                               |
| ------------------ | ----------------------------------------------------------- |
| **Authentication** | Who are you?                                                |
| **Role**           | What category of access do you have?                        |
| **Permission**     | What operation can you perform?                             |
| **Scope**          | What access has been delegated to this OAuth client/token?  |
| **Claim**          | What information is being asserted about the token/subject? |
| **Policy**         | Under what conditions is access allowed?                    |

### The one sentence to remember

> **OAuth scopes limit what an OAuth client/access token is authorized to access; they are not a replacement for your application's roles, permissions, or authorization policies.**

For your roadmap, I'd learn **Scopes → Permissions → RBAC → ABAC → Claims-based authorization** in that order, because then you'll see exactly where scopes fit rather than treating them as another name for permissions.
