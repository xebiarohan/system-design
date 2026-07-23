Sessions are one of the most fundamental concepts in web authentication. If you truly understand sessions, you'll also understand why JWTs and token-based authentication became popular later.

---

# 4. Sessions

## What is a Session?

HTTP is a **stateless protocol**.

That means every request is independent.

For example:

```
GET /login

↓

Server responds
```

Then:

```
GET /dashboard
```

The server does **not** automatically remember that this request came from the same user.

So after logging in, how does the server remember you?

**Answer: Sessions.**

A session is simply **the server remembering information about a user across multiple HTTP requests.**

---

# Without Sessions

Imagine a website without sessions.

```
Request 1

Username: alice
Password: secret

↓

Server:
Authenticated!
```

Now Alice requests another page.

```
Request 2

/dashboard
```

The server has forgotten everything.

It asks:

> Who are you?

Alice would have to log in **every request**.

That would be terrible.

---

# With Sessions

Instead, after login:

```
Username
Password

↓

Server verifies password

↓

Creates a Session

↓

Stores Session

↓

Returns Session ID
```

Now every request sends the Session ID.

```
Browser

↓

Cookie:
SessionID = abc123

↓

Server

↓

Looks up session

↓

"Oh, this is Alice."
```

The user stays logged in.

---

# What is Stored in a Session?

A session usually stores information like:

```
Session ID:
abc123

Data:
---------
User ID: 25
Username: alice
Role: admin
Login Time: 10:15 AM
Expires: 11:15 AM
```

Notice:

The password is **NOT stored**.

Only information needed to identify the user.

---

# Session ID

The Session ID is just a random unique identifier.

Example:

```
e9c84a13e72ab9f1d09d4c
```

or

```
3f94db0b5f0d1bcb89fd8d7b65
```

It should be:

* Random
* Unpredictable
* Long enough to prevent guessing

Think of it like a **claim ticket** at a coat check:

* You leave your coat (session data) with the attendant (server).
* You receive a ticket (session ID).
* Later, you hand over the ticket to retrieve your coat.

The ticket itself doesn't contain the coat—it only identifies which coat belongs to you.

---

# Where is the Session Stored?

This is the key idea.

**The Session ID is stored in the browser.**

**The actual session data is stored on the server.**

```
Browser

Cookie:
SessionID = abc123
```

Server:

```
abc123
↓

{
 userId:25,
 username:"alice",
 role:"admin"
}
```

The browser never stores this session data.

---

# Why Use a Cookie?

The browser needs a way to send the Session ID automatically with every request.

Cookies solve this.

When login succeeds:

```
HTTP Response

Set-Cookie:

SessionID=abc123
```

The browser stores it.

Later:

```
GET /dashboard

Cookie:
SessionID=abc123
```

The browser sends it automatically for the appropriate website, so your application code usually doesn't need to attach it manually.

---

# Complete Login Flow

### Step 1

User enters credentials.

```
alice
password123
```

↓

### Step 2

Server checks password.

```
Database

↓

Password hash matches
```

↓

### Step 3

Server creates session.

```
Session ID

abc123
```

↓

### Step 4

Stores session.

```
Session Store

abc123

↓

User ID:25
```

↓

### Step 5

Returns cookie.

```
Set-Cookie:

SessionID=abc123
```

↓

### Step 6

Browser stores cookie.

---

# Next Request

Browser visits:

```
GET /profile
```

Automatically sends:

```
Cookie:

SessionID=abc123
```

↓

Server:

```
Find session

↓

User ID = 25

↓

Authenticated
```

---

# Session Store

Where does the server store sessions?

Common options:

### Memory

```
Server RAM

↓

Fast

↓

Lost if server restarts
```

Good for development.

---

### Database

```
MySQL

PostgreSQL

MongoDB
```

Persistent but usually slower than memory-based stores.

---

### Redis

Very popular in production.

```
Browser

↓

Session ID

↓

Redis

↓

User data
```

Why Redis?

* Very fast
* In-memory storage
* Supports expiration
* Works across multiple servers

---

# Session Expiration

Sessions shouldn't last forever.

Example:

```
Expires in

30 minutes
```

If no activity occurs:

```
Session deleted

↓

User logs in again
```

Some systems use **sliding expiration**:

```
User active

↓

Expiration extended
```

Others use **absolute expiration**, where the session ends after a fixed time regardless of activity.

---

# Logout

When the user clicks Logout:

```
Delete session

↓

Delete cookie
```

Now:

```
Cookie

↓

Invalid

↓

Server says:

Please login
```

A secure logout removes the session on the server so the old Session ID can no longer be used.

---

# Session Hijacking

One of the biggest security risks.

Suppose an attacker steals your Session ID.

```
Your Cookie

↓

SessionID=abc123
```

Attacker uses it.

```
Cookie

↓

abc123
```

The server sees a valid Session ID and may think the attacker is you.

This is why protecting the Session ID is critical.

---

# How Can Session IDs Be Stolen?

Some examples:

* Malware on the user's device
* Insecure network traffic if HTTPS isn't used
* Cross-site scripting (XSS) that reads cookies not marked `HttpOnly`
* Session fixation attacks
* Leaked logs or browser extensions

---

# Protecting Sessions

## 1. HTTPS

Always encrypt traffic.

```
HTTPS

↓

Cookie protected during transit
```

Without HTTPS, attackers on the network could potentially intercept session cookies.

---

## 2. HttpOnly Cookie

```
Set-Cookie:

HttpOnly
```

JavaScript cannot read the cookie.

This greatly reduces the risk of cookie theft through XSS.

---

## 3. Secure Cookie

```
Secure
```

Browser sends the cookie **only over HTTPS**.

---

## 4. SameSite

Helps defend against **Cross-Site Request Forgery (CSRF)**.

Common values:

* `Strict`
* `Lax`
* `None` (must also use `Secure`)

You'll study CSRF in Phase 6.

---

## 5. Regenerate the Session ID

After a successful login:

```
Old Session

↓

Destroy

↓

New Session ID
```

This helps prevent **session fixation**, where an attacker tries to force a victim to use a known Session ID.

---

# Why Does the Browser Stay Logged In?

Because it automatically sends:

```
Cookie

↓

Session ID
```

The server finds the matching session and recognizes the user.

If the cookie is removed or the session expires, the server no longer recognizes the user.

---

# Where Is Login Information Stored?

This is a common interview question.

**Browser stores:**

* Session ID (usually in a cookie)

**Server stores:**

* User ID
* Roles
* Permissions (if needed)
* Login timestamp
* Expiration
* Any other session data

The browser does **not** normally know the user's role or permissions from the session cookie itself; it only holds the identifier.

---

# Sessions vs JWT (Preview)

| Sessions                           | JWT                                                              |
| ---------------------------------- | ---------------------------------------------------------------- |
| Server stores session data         | Token stores claims                                              |
| Browser stores Session ID          | Browser/client stores token                                      |
| Requires server-side session store | Can be stateless                                                 |
| Easy to revoke by deleting session | Revocation is more complex unless additional mechanisms are used |
| Common for traditional web apps    | Common for APIs and distributed systems                          |

You'll understand this comparison much more deeply when you reach the JWT section.

---

# End-to-End Request Flow

```text
          Login
             │
             ▼
   Username + Password
             │
             ▼
   Server verifies credentials
             │
             ▼
   Create Session (ID: abc123)
             │
             ▼
 Store session on server
   ┌─────────────────────────────┐
   │ Session ID: abc123          │
   │ User ID: 25                 │
   │ Role: Admin                 │
   │ Expires: 30 min             │
   └─────────────────────────────┘
             │
             ▼
 Send Set-Cookie: SessionID=abc123
             │
             ▼
 Browser stores cookie
             │
             ▼
 Future request to /dashboard
             │
             ▼
 Cookie: SessionID=abc123
             │
             ▼
 Server looks up session
             │
             ▼
 User authenticated
             │
             ▼
 Return protected content
```

## Key takeaways

* HTTP is stateless, so the server needs a way to remember authenticated users.
* A session is server-side state associated with a random Session ID.
* The browser typically stores only the Session ID in a cookie and sends it automatically with future requests.
* The server uses the Session ID to retrieve the user's session data.
* Session security depends on protecting the Session ID with HTTPS, `HttpOnly`, `Secure`, proper `SameSite` settings, session expiration, and Session ID regeneration after login.
* Logging out usually means invalidating the server-side session and removing the browser's cookie.

Once you're comfortable with sessions, the next concept—access tokens and refresh tokens—will make much more sense because you'll see exactly what problems token-based authentication is trying to solve.
