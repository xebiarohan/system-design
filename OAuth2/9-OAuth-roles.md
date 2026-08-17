Absolutely. OAuth Roles are one of the most important concepts to understand before learning the OAuth flow. If you understand these four roles, OAuth diagrams become much easier to read.

11. OAuth 2.0 Roles

OAuth 2.0 defines four main roles:

Resource Owner
Client
Authorization Server
Resource Server

Let's use a concrete example throughout:

You build an application called PhotoApp, and users can connect their Google account so PhotoApp can access their Google Photos.

The architecture looks like this:

                    ┌──────────────────────┐
                    │   Resource Owner     │
                    │       (User)         │
                    └──────────┬───────────┘
                               │
                               │ authorizes
                               ▼
                    ┌──────────────────────┐
                    │ Authorization Server │
                    │      (Google)        │
                    └──────────┬───────────┘
                               │
                               │ access token
                               ▼
┌──────────────────┐     ┌──────────────────────┐
│      Client      │────▶│   Resource Server    │
│    (PhotoApp)    │     │   (Google Photos)    │
└──────────────────┘     └──────────────────────┘


The key thing to understand is:

The user gives the client permission to access resources, but the client does not receive the user's password.

1. Resource Owner

The Resource Owner is the entity that owns the protected resources.

Usually, this is the user.

For example, suppose you have:

Google Account
    │
    ├── Photos
    ├── Contacts
    ├── Calendar
    └── Files


You are the owner of those resources.

Therefore:

Resource Owner = You


The resource owner has the authority to decide:

"I allow PhotoApp to access my Google Photos."

Important distinction

The resource owner isn't necessarily always a human.

OAuth defines the resource owner as an entity capable of granting access to a protected resource.

In most applications:

Resource Owner
       ↓
     Human


But conceptually it could be another entity.

2. Client

The Client is the application that wants access to the resource owner's protected resources.

In our example:

PhotoApp


is the client.

The client asks the authorization server:

"Can I get permission to access this user's Google Photos?"

So:

User
 ↓
owns photos

PhotoApp
 ↓
wants access to photos


PhotoApp is therefore the OAuth Client.

Important

"Client" does not necessarily mean a frontend application.

A client can be:

Web application
Mobile application
Desktop application
Backend service
CLI application
Device/TV application

For example:

React application
       ↓
     Client


or:

Mobile app
       ↓
     Client


or:

Backend server
       ↓
     Client


The word client means:

The application requesting access to protected resources.

3. Authorization Server

The Authorization Server is responsible for obtaining authorization from the resource owner and issuing tokens to the client.

This is where OAuth gets interesting.

Suppose PhotoApp wants your Google Photos.

PhotoApp should not say:

"Give me your Google password."


Instead, PhotoApp redirects you to Google's authorization server:

PhotoApp
   │
   │ "Please authorize Google Photos access"
   ▼
Google Authorization Server
   │
   ▼
You


Google then shows something like:

PhotoApp wants to:

☑ View your photos
☑ View your profile

[ Allow ] [ Deny ]


You choose Allow.

Google then issues an authorization result that allows PhotoApp to obtain an access token.

So the authorization server handles things such as:

Authenticating the user
Obtaining user consent
Issuing authorization codes
Issuing access tokens
Issuing refresh tokens
Managing authorization policies

Conceptually:

Authorization Server
        │
        ├── Authenticate user
        │
        ├── Ask for consent
        │
        ├── Issue authorization code
        │
        ├── Issue access token
        │
        └── Issue refresh token

Very important distinction

The authorization server is not necessarily the same server that stores the actual protected resource.

For example:

Authorization Server
       │
       │ issues token
       ▼
    Client
       │
       │ access token
       ▼
Resource Server
       │
       ▼
Protected data


These can be separate components.

4. Resource Server

The Resource Server hosts the protected resources.

In our example:

Google Photos API


is the resource server.

It receives the access token and asks:

"Is this token valid, and does it allow access to this resource?"

For example:

GET /photos

Authorization: Bearer eyJhbGci...


The resource server validates the token and, if authorized, returns the photos.

Conceptually:

Client
  │
  │ Access Token
  ▼
Resource Server
  │
  │ Validate token
  ▼
Protected Resource


The resource server is responsible for protecting APIs/data.

Examples could include:

Google Photos API
Google Calendar API
GitHub API
Microsoft Graph API

Putting All Four Together

Let's walk through the complete scenario.

You have:

User
 ↓
wants PhotoApp to access Google Photos

Step 1 — User

You are the:

Resource Owner


because you own the photos.

Step 2 — PhotoApp

PhotoApp is the:

Client


because it wants access to your photos.

Step 3 — Google authorization

PhotoApp sends you to Google's:

Authorization Server


You authenticate with Google and approve the requested permissions.

Step 4 — Token

Google's authorization server gives the client an authorization result, which ultimately allows PhotoApp to obtain an:

Access Token

Step 5 — API request

PhotoApp sends:

GET /photos

Authorization: Bearer ACCESS_TOKEN


to the:

Resource Server

Step 6 — Resource server

The resource server validates the access token and checks whether it grants the required permission.

If everything is valid:

Resource Server
      ↓
Your photos

The Four Roles in One Diagram
                    RESOURCE OWNER
                         User
                          │
                          │ grants permission
                          ▼
                AUTHORIZATION SERVER
                       Google
                          │
                          │ issues token
                          ▼
                       CLIENT
                     PhotoApp
                          │
                          │ Access Token
                          ▼
                  RESOURCE SERVER
                   Google Photos API
                          │
                          ▼
                  Protected Resources
                       Your Photos


This is the mental model I recommend memorizing.

A Crucial Distinction: Authorization Server vs Resource Server

This is where many beginners get confused.

Think:

Authorization Server
"Can this client access the resource?"


and:

"Here is a token representing that authorization."

Resource Server
"Here's the actual protected resource."


So:

Authorization Server
        ↓
     TOKEN

Resource Server
        ↓
     DATA


For example:

Google Authorization Server
        ↓
    Access Token
        ↓
Google Photos API
        ↓
      Photos

Another Example: GitHub

Suppose you're building:

DeveloperDashboard


and want to access a user's GitHub repositories.

The roles become:

OAuth Role	Example
Resource Owner	GitHub user
Client	DeveloperDashboard
Authorization Server	GitHub's authorization/token service
Resource Server	GitHub API

The flow is:

User
 │
 │ "Allow DeveloperDashboard to access my repositories"
 ▼
GitHub Authorization Server
 │
 │ Access Token
 ▼
DeveloperDashboard
 │
 │ Authorization: Bearer TOKEN
 ▼
GitHub API
 │
 ▼
Repositories

OAuth Client ≠ User

This is another important concept.

Suppose:

Alice


uses:

PhotoApp


to access:

Google Photos


You have three different entities:

Alice
 ↓
Resource Owner

PhotoApp
 ↓
Client

Google
 ↓
Authorization Server / Resource Server


Don't think:

Client = User


Instead:

User
  ↓
Resource Owner

Application
  ↓
Client

OAuth Roles vs Authentication

Here's an important connection to your roadmap.

OAuth answers:

"Can this application access this resource?"

It does not fundamentally answer:

"Who is this user?"

For example:

Alice
   ↓
Google
   ↓
"Allow PhotoApp to access my photos?"
   ↓
Yes
   ↓
Access Token


The access token primarily represents authorization to access resources.

If your goal is:

"Log this person into my application using Google"


that's where OpenID Connect (OIDC) comes in.

OIDC adds an identity layer on top of OAuth 2.0.

You'll learn:

OAuth 2.0
    ↓
Authorization

OIDC
    ↓
Authentication + Identity


This distinction will become extremely important when you get to Topic 12 and Topic 14.

The Mental Model to Memorize

If you remember only one thing from Topic 11, remember this:

RESOURCE OWNER
    │
    │ grants permission
    ▼
AUTHORIZATION SERVER
    │
    │ gives client token
    ▼
CLIENT
    │
    │ presents token
    ▼
RESOURCE SERVER
    │
    │
    ▼
PROTECTED RESOURCE


Or in plain English:

The Resource Owner owns the data. The Client wants access to it. The Authorization Server issues permission/tokens. The Resource Server protects and serves the data.

Once this is clear, the Authorization Code Flow becomes much easier to understand because you'll know exactly who each participant in the flow is.
