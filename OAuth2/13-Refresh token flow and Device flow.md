# 12. OAuth Flows — Refresh Token Flow

The **Refresh Token Flow** is used when an application has an **expired or soon-to-expire access token** and wants to obtain a new access token **without making the user log in again**.

You have already learned:

* Authorization Code Flow
* PKCE Flow
* Client Credentials Flow

The important thing to understand is that the Refresh Token Flow is a little different from those.

> **A refresh token is not primarily a way to authenticate a user. It is a credential used to obtain a new access token after the access token expires.**

The basic idea is:

```text
Access Token
     ↓
Expires
     ↓
Refresh Token
     ↓
Authorization Server
     ↓
New Access Token
```

---

# 12.1 Why Do We Need Refresh Tokens?

Access tokens should generally be **short-lived**.

For example:

```text
Access Token
     ↓
Valid for 15 minutes
```

Why not make it valid for 30 days?

Because access tokens are usually sent with API requests:

```http
Authorization: Bearer ACCESS_TOKEN
```

If an attacker steals the access token, they may be able to use it until it expires.

Short-lived token:

```text
Token stolen
     ↓
Attacker can use it
     ↓
Only for a limited period
```

So we want:

```text
Short-lived Access Token
```

But we also don't want the user to log in every 15 minutes.

That's where the refresh token comes in.

---

# 12.2 Access Token vs Refresh Token

You should clearly understand the difference.

### Access Token

Used to access APIs.

```text
Client
   ↓
Access Token
   ↓
Resource Server
```

Example:

```http
GET /api/orders
Authorization: Bearer ACCESS_TOKEN
```

---

### Refresh Token

Used to obtain a new access token.

```text
Client
   ↓
Refresh Token
   ↓
Authorization Server
   ↓
New Access Token
```

A refresh token is **not normally sent to your API**.

It is sent to the **Authorization Server**.

---

# 12.3 The Most Important Difference

Remember this:

```text
Access Token
     ↓
Used with
     ↓
Resource Server / API
```

Whereas:

```text
Refresh Token
     ↓
Used with
     ↓
Authorization Server
```

So:

```text
                Authorization Server
                       ↑
                       │
                Refresh Token
                       │
                       │
Client ────────────────┘


Client ───── Access Token ─────→ Resource Server
```

---

# 12.4 Where Does the Refresh Token Come From?

The refresh token is typically issued during an OAuth flow such as Authorization Code.

For example:

```text
User
  ↓
Client
  ↓
Authorization Server
  ↓
Authorization Code
  ↓
Token Endpoint
  ↓
Access Token + Refresh Token
```

The response might look like:

```json
{
  "access_token": "ACCESS_TOKEN",
  "refresh_token": "REFRESH_TOKEN",
  "token_type": "Bearer",
  "expires_in": 900
}
```

Now the client has two credentials:

```text
Access Token
    ↓
Short-lived
    ↓
Call APIs


Refresh Token
    ↓
Longer-lived
    ↓
Get new Access Tokens
```

---

# 12.5 Complete Lifecycle

The overall lifecycle looks like this:

```text
User Login
     ↓
Authorization Code Flow
     ↓
Access Token + Refresh Token
     ↓
API Requests
     ↓
Access Token Expires
     ↓
Refresh Token
     ↓
Authorization Server
     ↓
New Access Token
     ↓
More API Requests
     ↓
Access Token Expires
     ↓
Refresh Token
     ↓
New Access Token
```

Notice that the user does **not** need to log in again each time.

---

# 12.6 Step 1 — User Authenticates

Suppose you have:

```text
React Application
       ↓
Authorization Server
```

The user logs in:

```text
User
  ↓
Login
  ↓
Authorization Server
```

After successful authentication and authorization:

```text
Authorization Code
```

is returned to the client.

---

# 12.7 Step 2 — Client Exchanges the Code

The client sends:

```text
Authorization Code
```

to the token endpoint.

For example:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=AUTHORIZATION_CODE
&redirect_uri=https://example.com/callback
```

After validating the code, the Authorization Server returns:

```json
{
  "access_token": "abc123",
  "refresh_token": "xyz789",
  "token_type": "Bearer",
  "expires_in": 900
}
```

Now:

```text
Access Token
     ↓
15 minutes


Refresh Token
     ↓
Longer lifetime
```

---

# 12.8 Step 3 — Use the Access Token

The client calls the API:

```http
GET /api/orders
Authorization: Bearer abc123
```

The API validates the access token.

If valid:

```text
200 OK
```

---

# 12.9 Step 4 — Access Token Expires

Eventually:

```text
Access Token
     ↓
Expires
```

The API may return:

```http
401 Unauthorized
```

The client should not immediately ask the user to log in again.

Instead:

```text
Access Token expired
       ↓
Use Refresh Token
```

---

# 12.10 Step 5 — Request a New Access Token

The client sends the refresh token to the Authorization Server.

For example:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=REFRESH_TOKEN
```

Notice:

```text
grant_type=refresh_token
```

This tells the Authorization Server:

> "I already have a refresh token. Please issue me a new access token."

---

# 12.11 Step 6 — Authorization Server Validates the Refresh Token

The Authorization Server checks things such as:

```text
Is the refresh token valid?
        ↓
Has it expired?
        ↓
Has it been revoked?
        ↓
Was it issued to this client?
        ↓
Are the requested permissions still valid?
```

If everything is valid:

```text
Issue new Access Token
```

---

# 12.12 Step 7 — New Access Token

The response might be:

```json
{
  "access_token": "NEW_ACCESS_TOKEN",
  "refresh_token": "NEW_REFRESH_TOKEN",
  "token_type": "Bearer",
  "expires_in": 900
}
```

Whether a new refresh token is returned depends on the Authorization Server and its configuration.

If **refresh token rotation** is used:

```text
Old Refresh Token
       ↓
Consumed
       ↓
New Refresh Token
```

This is an important security technique that we'll cover shortly.

---

# 12.13 Refresh Token Rotation

Refresh token rotation means the Authorization Server issues a new refresh token whenever the old one is used.

For example:

```text
Refresh Token A
       ↓
Used
       ↓
Access Token B
+
Refresh Token C
```

The old token:

```text
Refresh Token A
```

is no longer valid.

Next time:

```text
Refresh Token C
       ↓
Used
       ↓
Access Token D
+
Refresh Token E
```

So the chain becomes:

```text
RT-A
 ↓
RT-C
 ↓
RT-E
 ↓
RT-G
```

This limits the usefulness of a stolen refresh token.

---

# 12.14 Why Are Refresh Tokens Sensitive?

A refresh token can potentially be used to obtain **many access tokens**.

Imagine an attacker steals:

```text
Refresh Token
```

They may be able to repeatedly obtain:

```text
Access Token
Access Token
Access Token
...
```

until the refresh token expires or is revoked.

Therefore:

> **Refresh tokens should be protected more carefully than ordinary access tokens.**

This is particularly important for browser and mobile applications.

---

# 12.15 Refresh Token vs Access Token

A useful comparison:

|                              | Access Token     | Refresh Token            |
| ---------------------------- | ---------------- | ------------------------ |
| Used for                     | API access       | Get new access token     |
| Sent to Resource Server      | Yes              | No                       |
| Sent to Authorization Server | Not normally     | Yes                      |
| Lifetime                     | Usually short    | Usually longer           |
| Used frequently              | Yes              | Only when refreshing     |
| If stolen                    | Limited lifetime | Potentially more serious |
| Can be revoked               | Yes              | Yes                      |

---

# 12.16 Why Not Just Make the Access Token Long-Lived?

You might ask:

> "Why not just make the access token valid for 30 days and forget about refresh tokens?"

Because of token theft.

Suppose:

```text
Access Token lifetime = 30 days
```

If stolen:

```text
Attacker
    ↓
Stolen Access Token
    ↓
Potentially 30 days of API access
```

Instead:

```text
Access Token = 15 minutes
Refresh Token = longer-lived
```

If the access token is stolen:

```text
Attacker
    ↓
Access Token
    ↓
Expires relatively quickly
```

This reduces the attack window.

---

# 12.17 Refresh Token Does Not Mean "Refresh the Session"

Don't confuse:

```text
Session
```

with:

```text
Refresh Token
```

A session is typically a server-side authentication mechanism.

A refresh token is an OAuth credential used to obtain a new access token.

They solve related but different problems.

---

# 12.18 Refresh Tokens and PKCE

PKCE and refresh tokens can be used together.

For example:

```text
SPA / Mobile App
       ↓
Authorization Code + PKCE
       ↓
Access Token
+
Refresh Token
```

Later:

```text
Access Token expires
       ↓
Refresh Token
       ↓
New Access Token
```

PKCE protects the **authorization code exchange**.

Refresh tokens solve the **access-token renewal** problem.

They are not alternatives.

---

# 12.19 What Happens If the Refresh Token Is Invalid?

The Authorization Server may return an error such as:

```json
{
  "error": "invalid_grant"
}
```

For example:

```text
Refresh Token
      ↓
Expired / Revoked
      ↓
Authorization Server
      ↓
invalid_grant
```

At that point, the application may need the user to authenticate again.

So:

```text
Access Token expired
        ↓
Refresh Token valid?
   ↙              ↘
 Yes               No
 ↓                  ↓
New token       Login again
```

---

# 12.20 Refresh Token Flow — Mental Model

Remember this diagram:

```text
             Access Token
                  │
                  ↓
             API Requests
                  │
                  ↓
                Expired
                  │
                  ↓
            Refresh Token
                  │
                  ↓
        Authorization Server
                  │
                  ↓
           New Access Token
                  │
                  ↓
             API Requests
```

The user is not necessarily involved in each refresh.

---

# 12.21 Refresh Token Flow in One Sentence

If you can explain this sentence, you've understood the flow:

> **A refresh token allows a client to obtain a new access token from the Authorization Server without requiring the user to authenticate again.**

---

# 13. OAuth Flows — Device Authorization Flow

The **Device Authorization Flow**, also called the **Device Code Flow**, is designed for devices where normal browser-based OAuth login is difficult or impossible.

Typical examples:

* Smart TVs
* Game consoles
* Streaming devices
* IoT devices
* CLI applications
* Devices with limited input capabilities

The classic problem is:

> "How can a device that doesn't have a convenient browser and keyboard let the user authenticate?"

The solution is:

```text
Device
   ↓
Show code
   ↓
User opens browser on another device
   ↓
User enters code
   ↓
User authenticates
   ↓
Device gets Access Token
```

---

# 13.1 The Problem Device Flow Solves

Imagine a Smart TV.

You want to sign in to:

```text
Netflix
```

Typing this:

```text
john@example.com
MyVeryLongPassword123!
```

using a TV remote is awful.

Instead, the TV can display:

```text
Go to:

https://example.com/device

Enter code:

ABCD-1234
```

The user takes their phone:

```text
Phone
  ↓
Open browser
  ↓
example.com/device
  ↓
Enter ABCD-1234
  ↓
Login
```

The TV then becomes authenticated.

---

# 13.2 The Main Idea

There are two devices involved:

```text
Device A
────────────
TV / Console / CLI
```

and:

```text
Device B
────────────
Phone / Laptop
```

The user authenticates using Device B.

But the access token is ultimately delivered to Device A.

---

# 13.3 The Flow

The high-level flow:

```text
                 Authorization Server
                       ↑       ↓
                       │       │
                  Device Code
                       │
                       │
TV ────────────────────┘
 │
 │ Display user code
 ↓
User
 │
 │ Phone / Laptop
 ↓
Browser
 │
 ↓
Authorization Server
 │
 │ User Login + Consent
 ↓
Authorization Complete
 │
 ↓
TV polls Authorization Server
 │
 ↓
Access Token
```

---

# 13.4 Step 1 — Device Requests Device Code

The device first contacts the Authorization Server.

For example:

```http
POST /oauth2/device_authorization
```

The device identifies itself as an OAuth client.

The Authorization Server returns information such as:

```json
{
  "device_code": "DEVICE_CODE",
  "user_code": "ABCD-1234",
  "verification_uri": "https://example.com/device",
  "expires_in": 600,
  "interval": 5
}
```

The important values are:

### `device_code`

A secret value used by the device when polling the token endpoint.

### `user_code`

A short code that the user enters in a browser.

### `verification_uri`

The URL the user needs to visit.

### `expires_in`

How long the device authorization request is valid.

### `interval`

How often the device should poll.

---

# 13.5 Step 2 — Device Shows the User Code

The TV might display:

```text
To sign in:

Visit:

https://example.com/device

Enter:

ABCD-1234
```

The device is essentially saying:

> "I need you to authenticate using another device."

---

# 13.6 Step 3 — User Opens the Verification URL

The user uses their phone:

```text
Phone
   ↓
Browser
   ↓
https://example.com/device
```

They enter:

```text
ABCD-1234
```

Now the Authorization Server can associate:

```text
User
   ↓
ABCD-1234
   ↓
Device Authorization Request
```

---

# 13.7 Step 4 — User Logs In

The user authenticates normally.

For example:

```text
Username
Password
MFA
```

Depending on the Identity Provider.

Then:

```text
User
   ↓
Authenticated
```

The user may also be asked to authorize the requested permissions.

For example:

```text
Allow Smart TV to access your account?
```

The user selects:

```text
Allow
```

---

# 13.8 Step 5 — Device Polls the Token Endpoint

The TV cannot simply receive a browser redirect.

Instead, it periodically checks the Authorization Server.

For example:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:device_code
&device_code=DEVICE_CODE
&client_id=CLIENT_ID
```

The device might poll every few seconds.

---

# 13.9 Why Does the Device Poll?

Because the user's browser and the TV are separate devices.

The Authorization Server needs a way to tell the TV:

```text
User has completed authentication.
```

So:

```text
TV
 ↓
"Is the user done?"
 ↓
Authorization Server

TV
 ↓
"Is the user done?"
 ↓
Authorization Server

TV
 ↓
"Is the user done?"
 ↓
Authorization Server
```

Eventually:

```text
User completes login
       ↓
Authorization Server
       ↓
TV asks again
       ↓
Access Token
```

---

# 13.10 Before the User Finishes

The Authorization Server might respond:

```json
{
  "error": "authorization_pending"
}
```

This means:

> "The user hasn't completed the authorization yet. Keep polling."

So the device waits:

```text
Wait
 ↓
Poll
 ↓
authorization_pending
 ↓
Wait
 ↓
Poll
 ↓
authorization_pending
```

---

# 13.11 After the User Finishes

Once the user completes authentication and authorization:

```text
TV
 ↓
Poll
 ↓
Authorization Server
 ↓
Access Token
```

The response might be:

```json
{
  "access_token": "ACCESS_TOKEN",
  "refresh_token": "REFRESH_TOKEN",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

The device can now call the API.

---

# 13.12 Device Authorization Flow Does Not Require a Browser on the Device

This is the key advantage.

Normal Authorization Code Flow:

```text
Device
  ↓
Browser
  ↓
Authorization Server
```

Device Flow:

```text
Device
  ↓
Display Code

User's Phone
  ↓
Browser
  ↓
Authorization Server
```

The device itself doesn't need a capable browser.

---

# 13.13 Device Flow vs Authorization Code

|                    | Authorization Code       | Device Authorization   |
| ------------------ | ------------------------ | ---------------------- |
| User involved      | Yes                      | Yes                    |
| Browser            | Usually on client/device | Usually another device |
| Authorization Code | Yes                      | No                     |
| Device Code        | No                       | Yes                    |
| User Code          | No                       | Yes                    |
| Polling            | No                       | Yes                    |
| Typical use        | Web/mobile apps          | TVs/consoles/CLI       |
| PKCE               | Common                   | Not the main mechanism |
| Access Token       | Yes                      | Yes                    |

---

# 13.14 Device Flow vs Client Credentials

This distinction is particularly important because you've just learned Client Credentials.

### Client Credentials

```text
Service
   ↓
Authenticate itself
   ↓
Access Token
   ↓
API
```

No user.

---

### Device Authorization

```text
Device
   ↓
User
   ↓
User authenticates
   ↓
Access Token
   ↓
Device accesses API
```

A user **is involved**.

So:

```text
Client Credentials
        ↓
Machine → Machine
        ↓
No user
```

while:

```text
Device Authorization
        ↓
User → Device
        ↓
User involved
```

---

# 13.15 Example — Smart TV

Imagine you have a streaming application on a TV.

The TV starts:

```text
Streaming App
     ↓
Need authentication
```

It requests a device code:

```text
Device Code = xyz123
User Code = ABCD-1234
```

TV displays:

```text
Go to:

https://stream.example.com/device

Enter:

ABCD-1234
```

User:

```text
Phone
  ↓
https://stream.example.com/device
  ↓
ABCD-1234
  ↓
Login
  ↓
Approve
```

Meanwhile:

```text
TV
 ↓
Poll
 ↓
Authorization Server
```

Eventually:

```text
Authorization Server
 ↓
Access Token
 ↓
TV
```

Now:

```text
TV
 ↓
Authorization: Bearer ACCESS_TOKEN
 ↓
Streaming API
```

The user is now logged in on the TV.

---

# 13.16 Example — CLI Application

Device Authorization Flow isn't only for TVs.

Imagine a CLI:

```bash
my-cli login
```

The CLI displays:

```text
Open:

https://example.com/device

Enter code:

ABCD-1234
```

You open the URL on your laptop:

```text
Browser
 ↓
Login
 ↓
MFA
 ↓
Approve
```

The CLI keeps polling:

```text
CLI
 ↓
Authorization Server
 ↓
Pending

CLI
 ↓
Authorization Server
 ↓
Pending

CLI
 ↓
Authorization Server
 ↓
Access Token
```

Now the CLI can make authenticated API requests.

This is a very practical example of Device Authorization Flow.

---

# 13.17 Important Device Flow Errors

The device needs to handle several possible responses.

### `authorization_pending`

The user hasn't finished yet.

```text
Keep polling
```

---

### `slow_down`

The client is polling too frequently.

```text
Increase polling interval
```

---

### `expired_token`

The device code has expired.

```text
Start a new authorization request
```

---

### `access_denied`

The user denied the request.

```text
Stop authorization
```

---

# 13.18 Why Is Polling Interval Important?

Imagine a million TVs doing this:

```text
Every 1 second
 ↓
"Is the user done?"
```

That's a lot of requests.

So the Authorization Server can tell the device:

```text
interval = 5
```

Meaning:

```text
Poll approximately every 5 seconds
```

If the device ignores this and polls too aggressively, it may receive:

```text
slow_down
```

---

# 13.19 Device Code vs User Code

Don't confuse these two.

### Device Code

Used by the device.

```text
DEVICE_CODE
```

It is typically longer and should be treated as a sensitive value.

### User Code

Typed by the human.

```text
ABCD-1234
```

It's intentionally short and easy to type.

Think:

```text
Device Code
    ↓
Device ↔ Authorization Server


User Code
    ↓
Human ↔ Browser
```

---

# 13.20 Security Considerations

The user code should have:

* Limited lifetime
* Limited number of attempts
* Sufficient entropy
* Clear expiration

The device should:

* Use HTTPS
* Protect the device code
* Respect polling intervals
* Never expose sensitive credentials unnecessarily
* Stop polling when authorization fails or expires

---

# 13.21 Device Authorization Flow — Mental Model

Remember this:

```text
                 Authorization Server
                    ↑          ↓
                    │          │
              Device Code     │
                    │          │
                    │          │
                  TV           │
                   │           │
                   ↓           │
              User Code        │
                   │           │
                   ↓           │
                User           │
                   │           │
                   ↓           │
              Phone Browser ───┘
                   │
                   ↓
              User Login
                   │
                   ↓
               Approve
                   │
                   ↓
            Authorization Server
                   │
                   ↓
               TV Polls
                   │
                   ↓
              Access Token
```

---

# 13.22 Device Authorization Flow in One Sentence

If you can explain this sentence, you've understood it:

> **Device Authorization Flow allows a device with limited input or browser capabilities to obtain an OAuth access token by having the user authenticate and authorize the device through another device.**

---

# 13.23 Final Comparison — All OAuth Flows You've Learned

At this point, your OAuth flow map should look like this:

```text
AUTHORIZATION CODE

User
 ↓
Browser
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Access Token
 ↓
API
```

```text
PKCE

User
 ↓
Browser
 ↓
Authorization Server
 ↓
Authorization Code + PKCE
 ↓
Access Token
 ↓
API
```

```text
CLIENT CREDENTIALS

Service
 ↓
Client Authentication
 ↓
Authorization Server
 ↓
Access Token
 ↓
API
```

```text
REFRESH TOKEN

Access Token
 ↓
Expires
 ↓
Refresh Token
 ↓
Authorization Server
 ↓
New Access Token
```

```text
DEVICE AUTHORIZATION

TV / Console / CLI
 ↓
Device Code
 ↓
User Code
 ↓
Phone / Laptop
 ↓
User Login + Consent
 ↓
Authorization Server
 ↓
Device Polls
 ↓
Access Token
```

---

# 12.24 The Big Picture

You can now categorize the OAuth flows based on **who is involved and what problem you're solving**:

| Flow                 |              User? | Main Purpose                                               |
| -------------------- | -----------------: | ---------------------------------------------------------- |
| Authorization Code   |                Yes | User authorizes an application                             |
| PKCE                 |                Yes | Secure Authorization Code for public clients               |
| Client Credentials   |                 No | Service-to-service authentication                          |
| Refresh Token        | Not during refresh | Obtain a new access token                                  |
| Device Authorization |                Yes | Authenticate devices with limited input/browser capability |

The easiest mental model is:

```text
                    OAuth 2.0
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ↓               ↓                ↓
     User            Service          Device
       │               │                │
       ↓               ↓                ↓
Authorization       Client          Device
   Code            Credentials     Authorization
       │
       ↓
    Access
    Token
       │
       ↓
   Refresh
    Token
```

At this stage, you should be comfortable answering:

> **"Which OAuth flow should I use?"**

```text
User accessing an application?
        ↓
Authorization Code
        ↓
Public client?
        ↓
PKCE


Service calling another service?
        ↓
Client Credentials


Access token expired?
        ↓
Refresh Token


TV / Console / CLI with limited browser/input?
        ↓
Device Authorization
```

That gives you the core OAuth 2.0 flow knowledge. The next major conceptual step in your roadmap is **OpenID Connect (OIDC)**, where you'll connect OAuth to the question you started with back in Phase 1:

```text
"Who is this user?"
```

OAuth answers primarily:

```text
"What is this client allowed to access?"
```

OIDC adds the standardized **authentication/identity layer** on top of OAuth.
