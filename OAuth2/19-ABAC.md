# Phase 9 — Authorization Models : ABAC

**Goal:** Understand how authorization decisions can be made using **attributes about the user, resource, action, and context**.

You already learned RBAC:

```text
User
 ↓
Role
 ↓
Permission
 ↓
Allow / Deny
```

ABAC extends this idea.

Instead of asking only:

```text
"What role does this user have?"
```

ABAC can ask:

```text
"Who is the user?"
"What are they trying to access?"
"What are they trying to do?"
"What attributes does the user have?"
"What attributes does the resource have?"
"What is the current context?"
```

---

# 1. What is ABAC?

**ABAC = Attribute-Based Access Control.**

Authorization decisions are made using **attributes** and **policies**.

The basic model is:

```text
Subject
   +
Resource
   +
Action
   +
Environment
   ↓
Policy
   ↓
Allow / Deny
```

Where:

```text
Subject
    ↓
The user/client making the request

Resource
    ↓
The thing being accessed

Action
    ↓
What the subject wants to do

Environment
    ↓
Context surrounding the request
```

For example:

```text
Alice
    +
Document #123
    +
edit
    +
Alice.department = engineering
Document.department = engineering
    ↓
Policy
    ↓
ALLOW
```

---

# 2. Why Do We Need ABAC?

RBAC works very well for many applications.

For example:

```text
Admin
 ↓
users:delete
```

But real-world authorization rules often become more complicated.

Suppose your requirement is:

> Managers can edit documents belonging to their own department.

RBAC alone can say:

```text
Manager
 ↓
document:edit
```

But that isn't enough.

Consider:

```text
Alice
Role: Manager
Department: Engineering
```

and:

```text
Document A
Department: Engineering
```

Alice should be allowed:

```text
Alice
 +
Manager
 +
Engineering document
 ↓
ALLOW
```

But:

```text
Document B
Department: Finance
```

should be denied:

```text
Alice
 +
Manager
 +
Finance document
 ↓
DENY
```

The role is the same.

The difference is an **attribute**:

```text
Alice.department
```

and:

```text
Document.department
```

This is where ABAC becomes useful.

---

# 3. The Four Main ABAC Components

A useful way to understand ABAC is:

```text
Subject
Resource
Action
Environment
```

Let's examine each.

---

## 3.1 Subject

The **subject** is the entity requesting access.

Usually this is a user.

For example:

```text
Alice
```

The subject can have attributes:

```text
User
 ├── id = 123
 ├── name = Alice
 ├── role = manager
 ├── department = engineering
 ├── country = UAE
 └── employmentType = full_time
```

These attributes can be used by authorization policies.

For example:

```text
subject.role == "manager"
```

or:

```text
subject.department == resource.department
```

---

## 3.2 Resource

The **resource** is what the user wants to access.

Examples:

```text
Document
File
Invoice
Order
Database record
API endpoint
Project
Customer account
```

A resource also has attributes.

For example:

```text
Document
 ├── id = 123
 ├── owner = Alice
 ├── department = engineering
 ├── classification = confidential
 └── createdAt = ...
```

The policy can use those attributes.

For example:

```text
resource.department == subject.department
```

---

## 3.3 Action

The **action** describes what the user wants to do.

Examples:

```text
read
create
update
delete
approve
download
share
execute
```

For example:

```text
action == "read"
```

or:

```text
action == "delete"
```

---

## 3.4 Environment

The **environment** contains contextual information about the request.

Examples:

```text
currentTime
IP address
device
network
location
risk level
authentication method
```

For example:

```text
environment.time
environment.ipAddress
environment.device
environment.network
```

A policy might say:

> Employees can access sensitive documents only during business hours.

That becomes something like:

```text
subject.employee == true
AND
resource.classification == "confidential"
AND
environment.time between 09:00 and 18:00
```

---

# 4. The ABAC Authorization Decision

The authorization system evaluates:

```text
Subject
    +
Resource
    +
Action
    +
Environment
    ↓
Policy
    ↓
Decision
```

For example:

```text
Subject:
Alice
department = engineering

Resource:
Document #123
department = engineering

Action:
edit

Environment:
09:30
```

Policy:

```text
Managers can edit documents
if the document belongs to their department.
```

Evaluation:

```text
Alice is Manager?
        ↓
       YES

Alice.department == Document.department?
        ↓
       YES

Action == edit?
        ↓
       YES

       ↓

     ALLOW
```

---

# 5. RBAC vs ABAC

This is one of the most important comparisons in Phase 9.

### RBAC

```text
User
 ↓
Role
 ↓
Permission
 ↓
Allow / Deny
```

Example:

```text
Alice
 ↓
Manager
 ↓
document:edit
 ↓
ALLOW
```

### ABAC

```text
User attributes
       +
Resource attributes
       +
Action
       +
Environment
       ↓
Policy
       ↓
Allow / Deny
```

Example:

```text
Alice.department == document.department
       +
Alice.role == manager
       +
action == edit
       ↓
ALLOW
```

The key difference:

> **RBAC primarily answers authorization questions through roles, while ABAC evaluates attributes against policies.**

---

# 6. A Simple ABAC Example

Suppose we have:

```text
User:
Alice

Role:
Manager

Department:
Engineering
```

And:

```text
Document:
Project Plan

Department:
Engineering
```

Request:

```text
Alice wants to edit Project Plan
```

The policy:

```text
Managers can edit documents
belonging to their department.
```

ABAC evaluates:

```text
subject.role == "manager"
        AND
subject.department == resource.department
        AND
action == "edit"
```

Values:

```text
"manager" == "manager"
        AND
"engineering" == "engineering"
        AND
"edit" == "edit"
```

Result:

```text
true AND true AND true
        ↓
      ALLOW
```

---

# 7. Same User, Different Resource

Now Alice tries to edit:

```text
Document:
Financial Report

Department:
Finance
```

The policy is still:

```text
subject.role == "manager"
AND
subject.department == resource.department
AND
action == "edit"
```

Evaluation:

```text
"manager" == "manager"
        ↓
       true

"engineering" == "finance"
        ↓
       false
```

Therefore:

```text
true AND false
        ↓
      DENY
```

Notice something important:

**Alice's role did not change.**

The resource changed.

That's one of the major strengths of ABAC.

---

# 8. ABAC Policies

A policy is a rule describing when access should be allowed or denied.

Simple policy:

```text
IF
    subject.role == "admin"
THEN
    allow
```

Another:

```text
IF
    subject.department == resource.department
    AND
    action == "read"
THEN
    allow
```

Another:

```text
IF
    subject.role == "manager"
    AND
    subject.department == resource.department
    AND
    action == "edit"
THEN
    allow
```

Policies can become more expressive than simple role checks.

---

# 9. Policy Example: Document Access

Imagine a company has:

```text
Departments:

Engineering
Finance
HR
Legal
```

Documents have:

```text
classification:
public
internal
confidential
restricted
```

Users have:

```text
department
role
clearanceLevel
```

A policy might say:

```text
Users can read a document when:

user.clearanceLevel >= document.requiredClearance
```

For example:

```text
Alice
clearanceLevel = 3
```

Document:

```text
requiredClearance = 2
```

Evaluation:

```text
3 >= 2
 ↓
ALLOW
```

But:

```text
Alice
clearanceLevel = 1
```

Document:

```text
requiredClearance = 3
```

Then:

```text
1 >= 3
 ↓
DENY
```

This kind of comparison is difficult to express cleanly using simple RBAC.

---

# 10. Attribute Types

ABAC can use many kinds of attributes.

### User attributes

```text
user.id
user.role
user.department
user.country
user.clearanceLevel
user.employmentType
```

### Resource attributes

```text
resource.owner
resource.department
resource.type
resource.classification
resource.requiredClearance
```

### Action attributes

```text
action = read
action = update
action = delete
action = approve
```

### Environment attributes

```text
environment.time
environment.ip
environment.device
environment.location
environment.network
```

So an authorization decision might look like:

```text
User
 ↓
department = engineering

Resource
 ↓
department = engineering

Action
 ↓
edit

Environment
 ↓
business hours
```

Then:

```text
Policy
 ↓
ALLOW
```

---

# 11. Attribute-Based Example: Business Hours

Suppose company policy says:

> Employees can access the internal admin system only between 9 AM and 6 PM.

The policy might be conceptually:

```text
subject.employee == true
AND
environment.time >= 09:00
AND
environment.time <= 18:00
```

At:

```text
10:00
```

we get:

```text
employee == true
AND
10:00 within business hours
        ↓
      ALLOW
```

At:

```text
23:00
```

we get:

```text
employee == true
AND
23:00 within business hours
        ↓
      DENY
```

The user's role didn't change.

The resource didn't change.

The **environment changed**.

---

# 12. Attribute-Based Example: Device

Suppose a company has:

```text
Admin Dashboard
```

Policy:

> Access is allowed only from company-managed devices.

The subject:

```text
Alice
```

Environment:

```text
device.managed = true
```

Then:

```text
device.managed == true
 ↓
ALLOW
```

If Alice uses an unmanaged device:

```text
device.managed == false
 ↓
DENY
```

Again, the user's identity hasn't changed.

The context has changed.

---

# 13. Combining RBAC and ABAC

In real applications, you don't necessarily have to choose between RBAC and ABAC.

You can combine them.

For example:

```text
User
 ↓
Role = Manager
```

Then apply additional attributes:

```text
Manager
 +
Same department
 +
Business hours
 +
Managed device
 ↓
ALLOW
```

Policy:

```text
subject.role == "manager"
AND
subject.department == resource.department
AND
environment.time within businessHours
AND
environment.device.managed == true
```

This is extremely powerful.

Conceptually:

```text
                User
                 ↓
              Role
                 ↓
             Manager
                 ↓
       ┌─────────┴─────────┐
       ↓                   ↓
 Department            Environment
       ↓                   ↓
Same as resource      Business hours
       │                   │
       └─────────┬─────────┘
                 ↓
              Policy
                 ↓
            ALLOW / DENY
```

---

# 14. RBAC + ABAC Example

Imagine a banking application.

Roles:

```text
Customer
Employee
Manager
Auditor
```

A Manager can approve transactions.

RBAC gives:

```text
Manager
 ↓
transaction:approve
```

But the company has an additional rule:

> A manager cannot approve a transaction from their own account.

Now we need resource/user attributes.

```text
subject.id != resource.accountOwner
```

Final policy:

```text
subject.role == "manager"
AND
action == "approve"
AND
subject.id != resource.accountOwner
```

Now:

```text
Manager
+
Approve transaction
+
Transaction belongs to someone else
        ↓
ALLOW
```

But:

```text
Manager
+
Approve transaction
+
Transaction belongs to manager
        ↓
DENY
```

This is a classic example of why RBAC alone can be insufficient.

---

# 15. ABAC and Ownership

Ownership is another common use case.

Suppose users can edit their own posts.

You could express this as:

```text
subject.id == resource.ownerId
AND
action == "edit"
```

For example:

```text
Alice.id = 123

Post.ownerId = 123
```

Then:

```text
123 == 123
 ↓
ALLOW
```

But:

```text
Alice.id = 123

Post.ownerId = 456
```

Then:

```text
123 == 456
 ↓
DENY
```

This is often called an **ownership rule** or **object-level authorization**.

---

# 16. Object-Level Authorization

This is extremely important for APIs.

Suppose your API has:

```http
GET /orders/123
```

Authentication tells you:

```text
This is Alice.
```

RBAC might tell you:

```text
Alice has orders:read.
```

But that still doesn't answer:

> Can Alice read **order 123**?

Order 123 might belong to Bob.

So you need:

```text
subject.id == resource.ownerId
```

Conceptually:

```text
JWT
 ↓
Alice
 ↓
orders:read
 ↓
Order #123
 ↓
Who owns Order #123?
 ↓
Bob
 ↓
Alice != Bob
 ↓
DENY
```

This is a very important distinction:

```text
Permission-level authorization
        ≠
Resource-level authorization
```

ABAC is particularly useful for resource-level authorization.

---

# 17. RBAC Alone vs ABAC

Consider:

```http
GET /users/123/orders/456
```

RBAC might say:

```text
User
 ↓
Customer
 ↓
orders:read
 ↓
ALLOW
```

But ABAC can say:

```text
User.id == Order.customerId
```

Therefore:

```text
Alice
 ↓
Customer
 ↓
orders:read
 ↓
Order belongs to Alice?
 ↓
YES
 ↓
ALLOW
```

Or:

```text
Alice
 ↓
Customer
 ↓
orders:read
 ↓
Order belongs to Bob?
 ↓
NO
 ↓
DENY
```

This pattern is extremely common in production APIs.

---

# 18. ABAC in an API

A typical API request:

```http
PUT /documents/123
Authorization: Bearer ACCESS_TOKEN
```

The request goes through several stages:

```text
HTTP Request
      ↓
Authentication
      ↓
Verify Access Token
      ↓
Identify User
      ↓
Load Resource
      ↓
Collect Attributes
      ↓
Evaluate Policy
      ↓
ALLOW / DENY
      ↓
Controller
```

For example:

```text
User:
Alice
role = manager
department = engineering

Resource:
Document #123
department = engineering

Action:
update

Environment:
business hours
```

Policy:

```text
manager
AND
same department
AND
business hours
```

Result:

```text
ALLOW
```

---

# 19. Where Do Attributes Come From?

This is an important practical question.

ABAC needs attributes.

Those attributes can come from different places.

### User database

```text
users
 ├── id
 ├── department
 ├── role
 └── clearanceLevel
```

### Resource database

```text
documents
 ├── id
 ├── ownerId
 ├── department
 └── classification
```

### JWT claims

For example:

```json
{
  "sub": "123",
  "department": "engineering",
  "role": "manager"
}
```

### Request context

```text
IP address
Device
Time
Location
```

So:

```text
Database
   +
JWT
   +
HTTP request
   +
Resource
   +
Environment
      ↓
   Attributes
      ↓
    Policy
      ↓
 Allow / Deny
```

---

# 20. JWT + ABAC

This connects directly to the earlier JWT material.

Suppose your access token contains:

```json
{
  "sub": "123",
  "department": "engineering",
  "role": "manager"
}
```

The API validates the token.

Then it obtains:

```text
subject.id = 123
subject.department = engineering
subject.role = manager
```

The API loads:

```text
Document #456
department = engineering
ownerId = 789
```

The request:

```text
action = update
```

The policy evaluates:

```text
subject.role == "manager"
AND
subject.department == resource.department
AND
action == "update"
```

Result:

```text
ALLOW
```

But remember:

> **JWT claims should not automatically be treated as the complete source of truth for authorization.**

Claims can become stale.

For security-sensitive decisions, you may need current data from your authorization/database system.

---

# 21. ABAC and OAuth Scopes

ABAC also works alongside OAuth.

Suppose an OAuth access token has:

```text
scope = documents:write
```

That answers one question:

```text
Does this token have permission to perform document writes?
```

But it doesn't necessarily answer:

```text
Can this user modify THIS document?
```

You can therefore have:

```text
OAuth Scope
     ↓
documents:write
     ↓
API
     ↓
ABAC Policy
     ↓
Is this user allowed to modify this document?
     ↓
ALLOW / DENY
```

For example:

```text
Scope:
documents:write

AND

User.department == Document.department

AND

Document.owner == User.id
```

The request must satisfy all relevant authorization requirements.

---

# 22. ABAC Policy Example

Let's build a complete policy.

Requirement:

> An employee can download a document if:
>
> 1. They have document access.
> 2. Their security clearance is high enough.
> 3. The document belongs to their department.
> 4. They are using a managed device.

Conceptually:

```text
subject.hasDocumentAccess == true
AND
subject.clearanceLevel >= resource.requiredClearance
AND
subject.department == resource.department
AND
environment.device.managed == true
AND
action == "download"
```

The request:

```text
Subject
 ├── hasDocumentAccess = true
 ├── clearanceLevel = 4
 └── department = engineering

Resource
 ├── requiredClearance = 3
 └── department = engineering

Action
 └── download

Environment
 └── device.managed = true
```

Evaluation:

```text
true
AND
4 >= 3
AND
engineering == engineering
AND
true
AND
download == download
```

Result:

```text
ALLOW
```

---

# 23. Policy Decision Point

As systems become more advanced, authorization is often separated from application code.

A useful architecture is:

```text
Application
    ↓
Authorization Request
    ↓
Policy Decision Point
    ↓
Evaluate Policies
    ↓
ALLOW / DENY
```

The component that makes the authorization decision is commonly called the:

**Policy Decision Point (PDP).**

The application asks:

```text
"Can Alice update document 123?"
```

The policy engine evaluates:

```text
Subject
+
Resource
+
Action
+
Environment
```

and returns:

```text
ALLOW
```

or:

```text
DENY
```

---

# 24. Policy Enforcement Point

Another useful concept is the:

**Policy Enforcement Point (PEP).**

The PEP is where the application actually enforces the decision.

For example:

```text
HTTP Request
     ↓
API Gateway / Middleware
     ↓
PEP
     ↓
Ask authorization service
     ↓
PDP
     ↓
ALLOW
     ↓
Continue request
```

If PDP returns:

```text
DENY
```

the PEP stops the request.

Conceptually:

```text
             ┌──────────────┐
Request ───→ │     PEP      │
             └──────┬───────┘
                    │
                    ↓
             ┌──────────────┐
             │     PDP      │
             └──────┬───────┘
                    │
              Policy evaluation
                    │
                    ↓
              ALLOW / DENY
                    │
                    ↓
                   PEP
                    │
                    ↓
                Application
```

You don't necessarily need separate services for these components in a small application. They are useful architectural concepts.

---

# 25. Policy Administration

In larger systems, you may also hear about a **Policy Administration Point (PAP)**.

Its job is essentially managing policies.

Conceptually:

```text
Policy Administration
        ↓
Policies
        ↓
Policy Decision Point
        ↓
Authorization Decision
```

For example, administrators might define:

```text
Managers can edit documents
in their department.
```

The policy is then evaluated whenever an authorization request occurs.

So a conceptual authorization architecture can look like:

```text
                Policies
                   ↓
                  PAP
                   ↓
                  PDP
                   ↑
                   │
Request → PEP ─────┘
           ↓
       ALLOW / DENY
```

These concepts are useful when you start studying centralized authorization systems and policy engines.

---

# 26. ABAC vs Claims-Based Authorization

These concepts are related but shouldn't be treated as identical.

Claims-based authorization uses claims such as:

```text
sub
role
department
clearance
tenant
```

For example:

```json
{
  "sub": "123",
  "department": "engineering",
  "role": "manager"
}
```

ABAC can **use those claims as attributes**.

For example:

```text
claim.department == resource.department
```

So:

```text
Claims
 ↓
Attributes
 ↓
ABAC Policy
 ↓
Decision
```

Claims are data.

ABAC is an **authorization model/policy approach** that can use that data.

---

# 27. ABAC vs RBAC vs Scopes

You should now be able to distinguish these three:

### RBAC

```text
Role
 ↓
Permissions
```

Example:

```text
Manager
 ↓
orders:update
```

### OAuth Scopes

```text
Access Token
 ↓
Granted Scope
```

Example:

```text
orders:read
```

This generally represents what the client/token has been granted to access.

### ABAC

```text
Attributes
 +
Policy
 ↓
Decision
```

Example:

```text
user.department == order.department
AND
user.role == manager
AND
action == update
```

A real system may use all three:

```text
OAuth
 ↓
Scope check
 ↓
RBAC
 ↓
Role/permission check
 ↓
ABAC
 ↓
Resource/context check
 ↓
ALLOW / DENY
```

---

# 28. ABAC and Multi-Tenant Applications

ABAC is especially useful for SaaS applications.

Suppose you have:

```text
Tenant A
 ├── Alice
 ├── Bob
 └── Projects

Tenant B
 ├── John
 ├── Sarah
 └── Projects
```

You must ensure:

```text
Alice cannot access Tenant B's data.
```

An attribute might be:

```text
user.tenantId
```

and the resource:

```text
resource.tenantId
```

Policy:

```text
subject.tenantId == resource.tenantId
```

Therefore:

```text
Alice
tenantId = A

Project
tenantId = A

A == A
 ↓
ALLOW
```

But:

```text
Alice
tenantId = A

Project
tenantId = B

A == B
 ↓
DENY
```

This is one of the most important real-world authorization rules in multi-tenant systems.

---

# 29. ABAC and Hierarchical Resources

Consider:

```text
Organization
    ↓
Project
    ↓
Folder
    ↓
Document
```

A user's access might depend on relationships between these objects.

For example:

```text
User.organizationId
    ==
Document.organizationId
```

and:

```text
User.projectId
    ==
Document.projectId
```

The policy can evaluate those relationships.

This is much more expressive than simply:

```text
role == admin
```

---

# 30. ABAC and "Deny by Default"

A strong authorization principle is:

```text
No matching policy
       ↓
      DENY
```

Don't design authorization as:

```text
No rule says no
       ↓
     ALLOW
```

Instead:

```text
Explicitly authorized
       ↓
      ALLOW

Otherwise
       ↓
      DENY
```

Conceptually:

```text
Request
  ↓
Policy evaluation
  ↓
Matching allow policy?
  ├── YES → ALLOW
  └── NO  → DENY
```

This is called **default deny**.

---

# 31. Common ABAC Mistakes

## Mistake 1 — Putting everything into JWTs

You might be tempted to put:

```json
{
  "role": "manager",
  "department": "engineering",
  "clearance": 5,
  "tenant": "123",
  "permissions": [...]
}
```

into the access token.

But authorization data can change.

For example:

```text
Alice
 ↓
Manager
```

changes to:

```text
Alice
 ↓
Employee
```

An existing token may still contain:

```text
role = manager
```

until it expires.

So carefully consider which attributes belong in tokens and which should be retrieved from current authoritative data.

---

## Mistake 2 — Trusting user-controlled attributes

Never blindly trust:

```http
X-Department: engineering
```

or:

```http
X-Role: admin
```

from the client.

A malicious client could send:

```http
X-Role: admin
```

Authorization attributes must come from a trusted source.

For example:

```text
Verified JWT
      +
Database
      +
Trusted authorization service
      ↓
Attributes
```

---

## Mistake 3 — Checking only the route

This:

```text
PUT /documents/123
```

does not mean:

```text
User can modify document 123
```

You need object-level authorization.

For example:

```text
user.department == document.department
```

or:

```text
user.id == document.ownerId
```

---

## Mistake 4 — Mixing authorization with business logic

Avoid spreading authorization conditions everywhere:

```text
if manager:
    ...
if engineering:
    ...
if owner:
    ...
```

Centralize policy decisions where practical.

Conceptually:

```text
Request
 ↓
Authorization Policy
 ↓
ALLOW / DENY
 ↓
Business Logic
```

---

# 32. A Complete ABAC Example

Let's build a realistic example.

You have a company document management system.

### User

```text
Alice

id = 123
role = manager
department = engineering
clearanceLevel = 4
```

### Resource

```text
Document #999

ownerId = 456
department = engineering
requiredClearance = 3
classification = confidential
```

### Action

```text
edit
```

### Environment

```text
time = 14:00
device.managed = true
```

Policy:

```text
Managers may edit confidential documents
when:

1. They are in the same department.
2. Their clearance is sufficient.
3. They use a managed device.
4. It is business hours.
```

The policy becomes:

```text
subject.role == "manager"

AND

subject.department == resource.department

AND

subject.clearanceLevel >= resource.requiredClearance

AND

environment.device.managed == true

AND

environment.time within businessHours

AND

action == "edit"
```

Evaluation:

```text
manager == manager
        ↓
true

engineering == engineering
        ↓
true

4 >= 3
        ↓
true

managed == true
        ↓
true

14:00 within business hours
        ↓
true

edit == edit
        ↓
true
```

Therefore:

```text
ALLOW
```

Now suppose Alice accesses the same document from an unmanaged device:

```text
environment.device.managed = false
```

Everything else is still true.

But:

```text
false
```

causes:

```text
DENY
```

This demonstrates the power of ABAC:

```text
User
+
Resource
+
Action
+
Environment
+
Policy
↓
Authorization decision
```

---

# 33. The Mental Model You Should Remember

The simplest ABAC mental model is:

```text
WHO?
 ↓
Subject

WHAT?
 ↓
Resource

DOING WHAT?
 ↓
Action

UNDER WHAT CONDITIONS?
 ↓
Environment

        ↓

     Policy

        ↓

   ALLOW / DENY
```

For example:

```text
WHO?
Alice

WHAT?
Document #123

DOING WHAT?
Edit

CONDITIONS?
Same department
Business hours
Managed device

        ↓

      Policy

        ↓

     ALLOW
```

---

# 34. RBAC → ABAC Progression

Your learning progression should now look like:

```text
RBAC
 ↓
User
 ↓
Role
 ↓
Permission
 ↓
Allow / Deny
```

Then:

```text
ABAC
 ↓
Subject
Resource
Action
Environment
 ↓
Attributes
 ↓
Policy
 ↓
Allow / Deny
```

And finally:

```text
RBAC + ABAC
       ↓
Role-based permissions
       +
Resource attributes
       +
User attributes
       +
Context
       ↓
Fine-grained authorization
```

---

# 35. What You Should Be Able to Build

After learning ABAC, you should be comfortable implementing rules such as:

### Ownership

```text
user.id == resource.ownerId
```

### Tenant isolation

```text
user.tenantId == resource.tenantId
```

### Department isolation

```text
user.department == resource.department
```

### Clearance

```text
user.clearanceLevel >= resource.requiredClearance
```

### Role + resource

```text
user.role == "manager"
AND
user.department == resource.department
```

### Context

```text
user.role == "employee"
AND
environment.device.managed == true
AND
environment.time within businessHours
```

And understand the overall architecture:

```text
                    ┌──────────────┐
                    │    User      │
                    └──────┬───────┘
                           │
                           ↓
                     Attributes
                           │
                           │
┌──────────────┐           │
│   Resource   │───────────┤
└──────┬───────┘           │
       │                   ↓
       │              ┌──────────┐
       └────────────→ │  Policy  │ ← Environment
                      └────┬─────┘
                           │
                           ↓
                    ALLOW / DENY
```

---

# Phase 9 — Authorization Models

You can now think about the three major concepts like this:

```text
RBAC
 ↓
"Who are you?"
"Which role do you have?"
 ↓
Role
 ↓
Permissions
```

```text
ABAC
 ↓
"What attributes apply to this request?"
 ↓
Subject
Resource
Action
Environment
 ↓
Policy
 ↓
ALLOW / DENY
```

```text
OAuth Scopes
 ↓
"What access has been delegated to this client/token?"
 ↓
Scope
 ↓
API access
```

And in a production system, they can work together:

```text
                 Request
                    ↓
             Authentication
                    ↓
              Identify User
                    ↓
             OAuth Scope Check
                    ↓
              RBAC Check
                    ↓
              ABAC / Policy
                    ↓
             Resource Check
                    ↓
              ALLOW / DENY
```

**The key idea to remember:**

```text
RBAC asks:

"What role do you have?"

ABAC asks:

"What attributes and conditions apply to this request?"

OAuth Scope asks:

"What access has been delegated to this token/client?"
```

Once this distinction is clear, the next topic in your roadmap — **Scopes, Permissions, and Claims-Based Authorization** — becomes much easier, because you'll see exactly where each one fits rather than treating them all as different names for "permission."
