# Phase 9 — Authorization Models

**Goal:** Understand how a system decides **what an authenticated user is allowed to do**.

You already know:

```text
Authentication
    ↓
"Who are you?"

Authorization
    ↓
"What are you allowed to do?"
```

RBAC — **Role-Based Access Control** — is one of the simplest and most widely used authorization models.

---

## 1. What is RBAC?

RBAC means:

> **Permissions are assigned to roles, and users are assigned to roles.**

Instead of saying:

```text
Alice can delete users
Alice can create users
Alice can view users
```

you define a role:

```text
Admin
    ↓
view_users
create_users
delete_users
```

and assign Alice to that role:

```text
Alice
  ↓
Admin
  ↓
Permissions
```

The fundamental relationship is:

```text
User
  ↓
Role
  ↓
Permission
  ↓
Action
```

For example:

```text
Alice
  ↓
Admin
  ↓
delete_users
  ↓
DELETE /users/123
```

---

## 2. Why do we need RBAC?

Imagine an application with 100,000 users.

You have permissions such as:

```text
view_users
create_users
update_users
delete_users

view_orders
create_orders
update_orders
delete_orders

view_reports
create_reports
delete_reports
```

If you directly assign permissions to every user, authorization becomes difficult to manage:

```text
Alice → 37 permissions
Bob   → 12 permissions
John  → 24 permissions
Sarah → 31 permissions
...
```

Instead, define roles:

```text
Admin
Manager
Support
Customer
```

Then:

```text
Admin
    ↓
All permissions

Manager
    ↓
Orders + Reports

Support
    ↓
View users + View orders

Customer
    ↓
Own orders
```

Now users simply receive roles.

```text
Alice → Admin

Bob → Manager

John → Support

Sarah → Customer
```

This makes authorization much easier to manage.

---

## 3. The Three Important Concepts

RBAC becomes much easier when you separate these three concepts:

### User

The identity performing an action.

```text
Alice
```

### Role

A named collection of permissions.

```text
Admin
```

### Permission

A specific thing the user is allowed to do.

```text
delete_users
```

So:

```text
Alice
  ↓
Admin
  ↓
delete_users
```

means:

> Alice has the Admin role, and Admin has the `delete_users` permission.

---

## 4. What is a Permission?

A permission represents an **allowed action**.

Common patterns are:

```text
resource + action
```

For example:

```text
users:read
users:create
users:update
users:delete

orders:read
orders:create
orders:update
orders:delete

reports:read
reports:create
```

You can think of a permission as:

```text
Can the user perform X?
```

Examples:

```text
users:read
```

means:

> Can read users.

```text
users:delete
```

means:

> Can delete users.

```text
orders:update
```

means:

> Can modify orders.

---

## 5. Role = Collection of Permissions

A role groups permissions together.

For example:

```text
Admin
```

might have:

```text
users:read
users:create
users:update
users:delete

orders:read
orders:create
orders:update
orders:delete

reports:read
reports:create
```

While:

```text
Support
```

might have:

```text
users:read
orders:read
orders:update
```

And:

```text
Customer
```

might have:

```text
orders:read
orders:create
```

Conceptually:

```text
Admin
 ├── users:read
 ├── users:create
 ├── users:update
 ├── users:delete
 ├── orders:read
 ├── orders:create
 ├── orders:update
 └── orders:delete

Support
 ├── users:read
 ├── orders:read
 └── orders:update

Customer
 ├── orders:read
 └── orders:create
```

---

## 6. User → Role

Users are assigned roles.

For example:

```text
Alice → Admin
Bob   → Support
John  → Customer
```

The authorization system can then determine permissions.

```text
Alice
  ↓
Admin
  ↓
users:delete
```

Therefore:

```text
Alice can delete users
```

But:

```text
John
  ↓
Customer
  ↓
users:delete ❌
```

Therefore:

```text
John cannot delete users
```

---

## 7. The Authorization Check

This is the most important part to understand.

Suppose we have:

```http
DELETE /users/123
```

The request arrives:

```text
Client
   ↓
API
```

First, authentication happens:

```text
JWT
 ↓
Verify signature
 ↓
Identify user
 ↓
Alice
```

Then authorization happens:

```text
Alice
 ↓
What roles does Alice have?
 ↓
Admin
 ↓
What permissions does Admin have?
 ↓
users:delete
 ↓
Does Alice have users:delete?
 ↓
YES
 ↓
Allow request
```

So the complete process is:

```text
HTTP Request
     ↓
Authentication
     ↓
Who is the user?
     ↓
Alice
     ↓
Find roles
     ↓
Admin
     ↓
Find permissions
     ↓
users:delete
     ↓
Check requested action
     ↓
users:delete
     ↓
MATCH
     ↓
ALLOW
```

If there is no matching permission:

```text
HTTP Request
     ↓
Authentication
     ↓
Alice
     ↓
Role
     ↓
Customer
     ↓
Permissions
     ↓
orders:read
     ↓
Requested permission
     ↓
users:delete
     ↓
NO MATCH
     ↓
DENY
```

Usually the API returns:

```http
403 Forbidden
```

---

## 8. Authentication vs RBAC

This distinction is extremely important.

Suppose Alice sends:

```http
GET /admin/users
Authorization: Bearer eyJ...
```

The JWT might prove:

```text
Alice is authenticated
```

That does **not** automatically mean:

```text
Alice is allowed to access /admin/users
```

You need two separate decisions:

```text
Authentication
     ↓
Is this really Alice?
     ↓
YES
     ↓
Authorization
     ↓
Does Alice have permission?
     ↓
YES / NO
```

Therefore:

```text
Authenticated ≠ Authorized
```

A user can be:

```text
Authenticated
     +
Not authorized
```

For example:

```text
Customer
   ↓
Logged in ✅
   ↓
DELETE /users/123
   ↓
Not allowed ❌
```

---

## 9. RBAC in an API

Imagine this API:

```text
GET    /users
POST   /users
PUT    /users/:id
DELETE /users/:id
```

You could define:

```text
users:read
users:create
users:update
users:delete
```

Then:

```text
Admin
 ├── users:read
 ├── users:create
 ├── users:update
 └── users:delete

Manager
 ├── users:read
 ├── users:create
 └── users:update

Support
 └── users:read
```

Now the routes can require permissions:

```text
GET /users
    ↓
users:read

POST /users
    ↓
users:create

PUT /users/:id
    ↓
users:update

DELETE /users/:id
    ↓
users:delete
```

---

## 10. Middleware

In a real application, authorization is commonly implemented using middleware.

Conceptually:

```text
Request
   ↓
Authentication Middleware
   ↓
Identify user
   ↓
Authorization Middleware
   ↓
Check permission
   ↓
Controller
```

For example:

```text
DELETE /users/123
        ↓
authenticate()
        ↓
authorize("users:delete")
        ↓
deleteUser()
```

The important idea is that the controller doesn't need to manually understand every role.

Instead of:

```text
if user.role == "admin":
    deleteUser()
```

you can use:

```text
authorize("users:delete")
```

This separates:

```text
Business logic

from

Authorization logic
```

---

## 11. Don't Confuse Roles With Permissions

This is a common beginner mistake.

Bad design:

```text
if user.role == "admin":
    allow()
```

This works initially, but becomes difficult when the application grows.

Imagine:

```text
Admin
Manager
Support
Auditor
BillingManager
SuperAdmin
Developer
```

Soon you'll have:

```text
if admin
if manager
if support
if auditor
if billing_manager
...
```

Instead, check permissions:

```text
if hasPermission("users:delete"):
    allow()
```

The role is simply one mechanism for obtaining that permission.

Think:

```text
Role
 ↓
Permissions
 ↓
Authorization decision
```

not:

```text
Role
 ↓
Authorization decision
```

---

## 12. Role Hierarchy

RBAC can also support hierarchical roles.

For example:

```text
Admin
  ↑
Manager
  ↑
Employee
```

You might define:

```text
Employee
 ├── users:read
 └── orders:read

Manager
 ├── Employee permissions
 ├── users:update
 └── orders:update

Admin
 ├── Manager permissions
 ├── users:create
 └── users:delete
```

Therefore:

```text
Admin
    ↓
Manager
    ↓
Employee
```

An Admin inherits Manager permissions, and Manager inherits Employee permissions.

However, **role hierarchy is a design choice**, not something you should automatically assume every RBAC implementation has.

---

## 13. Multiple Roles

A user can have multiple roles.

For example:

```text
Alice
 ├── Manager
 └── BillingManager
```

Manager gives:

```text
users:read
users:update
```

BillingManager gives:

```text
invoices:read
invoices:create
invoices:update
```

Alice effectively has:

```text
users:read
users:update

invoices:read
invoices:create
invoices:update
```

Conceptually:

```text
             ┌── Manager ────────┐
Alice ───────┤                    ├── Permissions
             └── BillingManager ─┘
```

Usually, the effective permissions are the union of the permissions granted by her roles.

---

## 14. RBAC With JWT

Now connect this to what you learned earlier.

A JWT might contain:

```json
{
  "sub": "123",
  "roles": ["admin"]
}
```

The API verifies the JWT:

```text
JWT
 ↓
Signature verification
 ↓
Authenticated user
 ↓
sub = 123
 ↓
roles = ["admin"]
```

Then authorization can use the role:

```text
admin
  ↓
Admin role
  ↓
Permissions
  ↓
users:delete
```

However, there is an important distinction:

> A JWT containing `"roles": ["admin"]` does not magically make the user an admin.

The API must **trust the issuer and verify the token correctly** before relying on those claims.

---

## 15. Roles vs Permissions Inside JWT

You can put either roles or permissions into a token.

### Roles

```json
{
  "sub": "123",
  "roles": ["admin"]
}
```

The API resolves:

```text
admin
 ↓
permissions
```

### Permissions

```json
{
  "sub": "123",
  "permissions": [
    "users:read",
    "users:delete"
  ]
}
```

The API can directly check:

```text
users:delete
```

### Both

You could have:

```json
{
  "sub": "123",
  "roles": ["admin"],
  "permissions": [
    "users:read",
    "users:delete"
  ]
}
```

But this introduces an important design question:

> Where is the source of truth?

For example:

```text
Database
    ↓
User roles
    ↓
Permissions
    ↓
JWT
```

If Alice's role changes from:

```text
Admin
```

to:

```text
Customer
```

an already-issued JWT might still say:

```json
{
  "roles": ["admin"]
}
```

until it expires or is otherwise invalidated.

This is one reason authorization and token lifecycle need to be designed together.

---

## 16. RBAC Database Design

A common relational database design is:

```text
users
roles
permissions
user_roles
role_permissions
```

Relationship:

```text
Users
  ↓
user_roles
  ↓
Roles
  ↓
role_permissions
  ↓
Permissions
```

For example:

```text
users
----------------
id
name

1 | Alice
2 | Bob
```

```text
roles
----------------
id
name

1 | admin
2 | support
3 | customer
```

```text
permissions
----------------
id
name

1 | users:read
2 | users:create
3 | users:delete
4 | orders:read
```

Then:

```text
user_roles
----------------
user_id | role_id

1       | 1
2       | 2
```

means:

```text
Alice → Admin
Bob   → Support
```

And:

```text
role_permissions
----------------
role_id | permission_id

1       | 1
1       | 2
1       | 3
2       | 1
2       | 4
```

means:

```text
Admin
 ├── users:read
 ├── users:create
 └── users:delete

Support
 ├── users:read
 └── orders:read
```

---

## 17. RBAC Example: SaaS Application

Imagine you build a SaaS application.

You have:

```text
Organization
 ├── Owner
 ├── Admin
 ├── Editor
 └── Viewer
```

Permissions:

```text
project:read
project:create
project:update
project:delete

member:read
member:invite
member:remove

billing:read
billing:update
```

Roles:

```text
Owner
 ├── project:*
 ├── member:*
 └── billing:*

Admin
 ├── project:*
 ├── member:*
 └── billing:read

Editor
 ├── project:read
 ├── project:create
 └── project:update

Viewer
 └── project:read
```

Now:

```text
Alice → Owner
Bob   → Editor
John  → Viewer
```

Request:

```http
DELETE /projects/123
```

Required permission:

```text
project:delete
```

Alice:

```text
Owner
 ↓
project:delete
 ↓
ALLOW
```

Bob:

```text
Editor
 ↓
project:delete
 ↓
DENY
```

John:

```text
Viewer
 ↓
project:delete
 ↓
DENY
```

---

## 18. RBAC Is Not Always Enough

This is where you need to understand the limitation of RBAC.

Suppose you have:

```text
Manager
```

and the manager can:

```text
edit documents
```

But the rule is actually:

> A manager can edit documents **only in their own department**.

Now simple RBAC isn't enough.

RBAC tells you:

```text
Manager
 ↓
document:edit
```

But the real question is:

```text
Is this document in the manager's department?
```

You need additional information:

```text
User
 ├── role = manager
 └── department = engineering

Document
 └── department = engineering
```

Then:

```text
Manager
      +
Same department
      ↓
ALLOW
```

but:

```text
Manager
      +
Different department
      ↓
DENY
```

This leads into **ABAC — Attribute-Based Access Control**.

---

## 19. RBAC vs ABAC

RBAC asks:

```text
What role does the user have?
```

ABAC can ask:

```text
Who is the user?
What department are they in?
What resource are they accessing?
What department owns that resource?
What time is it?
Where is the request coming from?
What action are they performing?
```

Example RBAC:

```text
Manager
   ↓
edit_document
   ↓
ALLOW
```

Example ABAC:

```text
User.department == Document.department
              +
User.role == Manager
              +
Action == edit
              ↓
ALLOW
```

So:

```text
RBAC
 ↓
Role-based rules
```

while:

```text
ABAC
 ↓
Attribute/context-based rules
```

---

## 20. RBAC vs OAuth Scopes

This is particularly important for your OAuth roadmap.

Don't treat:

```text
Role
```

and:

```text
OAuth Scope
```

as the same thing.

A role is usually an **application authorization concept**:

```text
Admin
Manager
Customer
```

An OAuth scope represents the **access requested/delegated to a client**.

For example:

```text
openid
profile
email
orders:read
orders:write
```

Imagine:

```text
User
 ↓
Admin role
 ↓
Many permissions
```

but the user authorizes an application with:

```text
orders:read
```

The application should not automatically receive every permission the user has.

Conceptually:

```text
User permissions
        ↓
      Admin
        ↓
  many permissions
        ↓
OAuth authorization
        ↓
   granted scopes
        ↓
   orders:read
```

Then the API can enforce both:

```text
Does the token have the required scope?
              +
Is the user allowed to perform the operation?
```

This becomes particularly important in OAuth-protected APIs.

---

## 21. A Useful Mental Model

Keep these layers separate:

```text
                    Authentication
                         ↓
                    Who is the user?
                         ↓
                       User
                         ↓
                       Roles
                         ↓
                    Permissions
                         ↓
                Authorization decision
                         ↓
                    Allow / Deny
```

With OAuth:

```text
OAuth Client
     ↓
Access Token
     ↓
Scopes
     ↓
API
     ↓
User / Client authorization
     ↓
Permission check
     ↓
Allow / Deny
```

The important thing is that **OAuth scopes, roles, and permissions solve related but different problems**.

---

## 22. Common RBAC Mistakes

### Mistake 1 — Checking only authentication

```text
JWT valid
   ↓
ALLOW
```

Wrong.

A valid token proves that the request is authenticated; it does not necessarily mean the requested operation is authorized.

---

### Mistake 2 — Hardcoding roles everywhere

Avoid:

```text
if role == "admin"
```

throughout your application.

Prefer:

```text
hasPermission("users:delete")
```

This makes your authorization system easier to evolve.

---

### Mistake 3 — Creating too many roles

Don't create:

```text
UserCanReadUsers
UserCanCreateUsers
UserCanDeleteUsers
UserCanReadOrders
...
```

Those are permissions, not roles.

Roles should represent meaningful groups:

```text
Admin
Support
Manager
Customer
```

---

### Mistake 4 — Making one giant Admin role

This is common:

```text
Admin
 ↓
EVERYTHING
```

Then everyone who needs one extra permission gets:

```text
Admin
```

This creates privilege creep.

Prefer smaller roles and explicit permissions.

---

### Mistake 5 — Assuming JWT claims are automatically trustworthy

Never do:

```text
Decode JWT
 ↓
Read "role": "admin"
 ↓
Trust it
```

The token must first be properly validated:

```text
JWT
 ↓
Verify signature
 ↓
Validate issuer
 ↓
Validate audience
 ↓
Validate expiration
 ↓
Then trust claims
```

---

## 23. RBAC End-to-End Example

Let's put everything together.

User:

```text
Alice
```

Database:

```text
Alice
  ↓
Admin
```

Role:

```text
Admin
  ↓
users:read
users:create
users:update
users:delete
```

Alice logs in.

Authentication succeeds:

```text
Alice
 ↓
Authenticated
```

Authorization server issues an access token.

Client calls:

```http
DELETE /users/123
Authorization: Bearer ACCESS_TOKEN
```

API:

```text
Receive request
      ↓
Verify access token
      ↓
Identify Alice
      ↓
Determine Alice's roles
      ↓
Admin
      ↓
Determine required permission
      ↓
users:delete
      ↓
Does Admin have users:delete?
      ↓
YES
      ↓
Execute request
```

If Bob is:

```text
Bob
 ↓
Customer
 ↓
orders:read
```

and tries:

```http
DELETE /users/123
```

the result is:

```text
Bob
 ↓
Customer
 ↓
users:delete ❌
 ↓
403 Forbidden
```

---

## 24. What You Should Be Able to Build

At the end of RBAC, you should be comfortable implementing:

```text
User
 ↓
Role
 ↓
Permission
```

For example:

```text
Admin
 ├── users:read
 ├── users:create
 ├── users:update
 └── users:delete

User
 └── users:read
```

Then implement:

```text
authenticate()
       ↓
authorize("users:delete")
       ↓
controller()
```

And understand the complete flow:

```text
                ┌──────────────┐
                │    User      │
                └──────┬───────┘
                       │
                       ↓
                ┌──────────────┐
                │     Role     │
                └──────┬───────┘
                       │
                       ↓
                ┌──────────────┐
                │ Permission   │
                └──────┬───────┘
                       │
                       ↓
                ┌──────────────┐
                │   API Route  │
                └──────┬───────┘
                       │
                 Allow / Deny
```

---

# Phase 9 — What to Learn Next

Once you understand basic RBAC, follow this order:

```text
RBAC Fundamentals
        ↓
Users → Roles → Permissions
        ↓
Permission Checks
        ↓
Authorization Middleware
        ↓
RBAC Database Design
        ↓
Multiple Roles
        ↓
Role Hierarchies
        ↓
RBAC + JWT
        ↓
RBAC + OAuth Scopes
        ↓
RBAC Limitations
        ↓
ABAC
        ↓
Claims-Based Authorization
```

The **most important distinction** to carry forward is:

```text
Authentication
    ↓
Who are you?

Role
    ↓
What category of access do you have?

Permission
    ↓
What action are you allowed to perform?

Authorization
    ↓
Should this specific request be allowed?
```

And in a real OAuth/OIDC application, you should think of the layers roughly as:

```text
OIDC
 ↓
Authentication / Identity

OAuth
 ↓
Delegated API access
 ↓
Scopes

Application Authorization
 ↓
Roles
 ↓
Permissions
 ↓
Resource-level rules
```

That separation will make **ABAC, OAuth scopes, claims-based authorization, and policy engines** much easier to understand in the next part of Phase 9.
