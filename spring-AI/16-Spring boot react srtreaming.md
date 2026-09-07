

# Spring Boot → React Streaming

## 1. What problem are we solving?

Normally, your application works like this:

```text
React
   |
   | HTTP Request
   ↓
Spring Boot
   |
   | Request to LLM
   ↓
LLM
   |
   | Wait for complete response
   ↓
Spring Boot
   |
   | Complete response
   ↓
React
```

The user sees nothing while the LLM is generating.

For example:

> User: Explain OAuth 2.0

The backend might wait 5–10 seconds and then return:

```text
OAuth 2.0 is an authorization framework that allows...
```

That's not how ChatGPT-like applications behave.

Instead:

```text
React
   |
   | Request
   ↓
Spring Boot
   |
   | stream()
   ↓
LLM
   |
   | "OAuth"
   ↓
Spring Boot → React
   |
   | " 2.0"
   ↓
Spring Boot → React
   |
   | " is"
   ↓
Spring Boot → React
   |
   | " an"
   ↓
...
```

The user sees the answer being generated **incrementally**.

That's the goal of Topic 19.

---

# 2. End-to-end architecture

The architecture we want is:

```text
                    ┌──────────────┐
                    │    React     │
                    │              │
                    │ Chat UI      │
                    └──────┬───────┘
                           │
                     HTTP / SSE
                           │
                           ↓
                    ┌──────────────┐
                    │ Spring Boot  │
                    │              │
                    │ Controller   │
                    └──────┬───────┘
                           │
                           ↓
                    ┌──────────────┐
                    │  Spring AI   │
                    │              │
                    │ ChatClient   │
                    └──────┬───────┘
                           │
                           ↓
                    ┌──────────────┐
                    │     LLM      │
                    └──────────────┘
```

The important part is:

```text
LLM
 ↓
Flux<String>
 ↓
Spring Boot
 ↓
SSE
 ↓
React
 ↓
Update UI
```

Your previous topic on reactive streaming introduced `Flux`, which is exactly what makes this possible. Your roadmap explicitly lists `Flux`, Reactive Streams, and backpressure immediately before this topic. 

---

# 3. What is SSE?

**SSE = Server-Sent Events**

It is a mechanism where:

> The client makes one HTTP request, and the server keeps the HTTP connection open while continuously sending events to the client.

Normal HTTP:

```text
Client ────────────────→ Server
       Request

Client ←──────────────── Server
       Complete response

Connection ends
```

SSE:

```text
Client ────────────────→ Server
       Request

Client ←──────────────── Server
       Event 1

Client ←──────────────── Server
       Event 2

Client ←──────────────── Server
       Event 3

Client ←──────────────── Server
       Event 4

       Connection remains open

Client ←──────────────── Server
       Complete

Connection ends
```

This is particularly useful for LLM applications because the server doesn't need to wait for the entire LLM response.

---

# 4. Why SSE is a good fit for LLM streaming

LLM generation is naturally incremental:

```text
Token 1
   ↓
Token 2
   ↓
Token 3
   ↓
Token 4
   ↓
...
```

Spring AI can expose that as a reactive stream:

```java
Flux<String>
```

Conceptually:

```text
LLM

"Spring"
   ↓
" AI"
   ↓
" provides"
   ↓
" abstractions"
   ↓
" for"
   ↓
...
```

Spring Boot can send those pieces to React as SSE events.

So:

```text
LLM
 ↓
Flux<String>
 ↓
SSE
 ↓
React
```

---

# 5. Spring AI side

Suppose you have:

```java
private final ChatClient chatClient;
```

A normal non-streaming request might look like:

```java
String response = chatClient
        .prompt()
        .user("Explain OAuth 2.0")
        .call()
        .content();
```

Notice:

```java
.call()
```

You wait for the complete response.

For streaming:

```java
Flux<String> response = chatClient
        .prompt()
        .user("Explain OAuth 2.0")
        .stream()
        .content();
```

The important difference is:

```text
call()
  ↓
complete response

stream()
  ↓
Flux
  ↓
many pieces
```

---

# 6. Spring Boot Controller

A simple endpoint can return the `Flux` directly.

For example:

```java
@RestController
@RequestMapping("/api/chat")
public class ChatController {

    private final ChatClient chatClient;

    public ChatController(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    @GetMapping(
        value = "/stream",
        produces = MediaType.TEXT_EVENT_STREAM_VALUE
    )
    public Flux<String> stream(@RequestParam String message) {

        return chatClient
                .prompt()
                .user(message)
                .stream()
                .content();
    }
}
```

The important line is:

```java
produces = MediaType.TEXT_EVENT_STREAM_VALUE
```

which tells Spring:

> This endpoint produces an SSE stream.

And:

```java
Flux<String>
```

means:

> I am going to produce multiple values over time.

---

# 7. What actually happens?

Suppose React calls:

```text
GET /api/chat/stream?message=Explain%20OAuth
```

Spring receives it.

Then:

```java
chatClient
    .prompt()
    .user(message)
    .stream()
    .content();
```

Spring AI starts receiving the generated content from the LLM.

Imagine the LLM produces:

```text
OAuth
```

then:

```text
 2.0
```

then:

```text
 is
```

then:

```text
 an
```

then:

```text
 authorization
```

Your `Flux` might conceptually look like:

```text
Flux

 ┌──────────────┐
 │ "OAuth"      │
 ├──────────────┤
 │ " 2.0"       │
 ├──────────────┤
 │ " is"        │
 ├──────────────┤
 │ " an"        │
 ├──────────────┤
 │ " authorization" │
 └──────────────┘
```

Spring sends these incrementally to the browser.

---

# 8. What does SSE actually look like?

Under the hood, SSE uses a special content type:

```http
Content-Type: text/event-stream
```

The server sends events roughly like:

```text
data: OAuth

data: 2.0

data: is

data: an
```

The browser can process each event as it arrives.

You don't have to wait for:

```text
OAuth 2.0 is an authorization framework...
```

to be completed.

---

# 9. React side — EventSource

For SSE, browsers provide a built-in API:

```javascript
EventSource
```

For example:

```javascript
const eventSource =
    new EventSource(
        "http://localhost:8080/api/chat/stream?message=Explain%20OAuth"
    );

eventSource.onmessage = (event) => {
    console.log(event.data);
};
```

Suppose Spring sends:

```text
OAuth
2.0
is
an
authorization
framework
```

React receives:

```javascript
event.data
```

as each event arrives.

You can then append it to the existing response.

---

# 10. Building a ChatGPT-style UI

Suppose your React state is:

```javascript
const [response, setResponse] = useState("");
```

Then:

```javascript
eventSource.onmessage = (event) => {

    setResponse(previous =>
        previous + event.data
    );
};
```

So initially:

```text
response = ""
```

Event 1:

```text
"OAuth"
```

becomes:

```text
"OAuth"
```

Event 2:

```text
" 2.0"
```

becomes:

```text
"OAuth 2.0"
```

Event 3:

```text
" is"
```

becomes:

```text
"OAuth 2.0 is"
```

And so on.

The UI appears to "type" the answer.

---

# 11. Complete React example

A simple component:

```jsx
import { useState } from "react";

function Chat() {

    const [message, setMessage] = useState("");
    const [response, setResponse] = useState("");

    const sendMessage = () => {

        setResponse("");

        const eventSource = new EventSource(
            `http://localhost:8080/api/chat/stream?message=${encodeURIComponent(message)}`
        );

        eventSource.onmessage = (event) => {

            setResponse(previous =>
                previous + event.data
            );
        };

        eventSource.onerror = () => {
            eventSource.close();
        };
    };

    return (
        <div>

            <input
                value={message}
                onChange={e => setMessage(e.target.value)}
            />

            <button onClick={sendMessage}>
                Send
            </button>

            <div>
                {response}
            </div>

        </div>
    );
}

export default Chat;
```

That's enough to demonstrate the basic concept.

---

# 12. But there's an important problem

You might notice something:

We're using:

```http
GET
```

because `EventSource` traditionally works with GET.

But chat applications normally want to send a request body:

```json
{
    "message": "Explain OAuth 2.0"
}
```

You can't use a normal `EventSource` API with a POST request body.

That's an important architectural consideration.

---

# 13. Two common approaches

### Approach 1 — GET + EventSource

```text
React
  |
  | GET /stream?message=hello
  ↓
Spring Boot
  |
  ↓
Flux
  |
  ↓
SSE
```

Very simple.

Good for:

* learning
* simple demos
* basic streaming

---

### Approach 2 — POST + streaming response

For a real chat API, you may want:

```http
POST /api/chat
Content-Type: application/json

{
    "message": "Explain OAuth 2.0"
}
```

and then:

```text
HTTP response
       ↓
stream
       ↓
chunk
chunk
chunk
chunk
```

React can use:

```javascript
fetch()
```

and read the response stream.

This is often more flexible for chat applications.

---

# 14. Streaming HTTP response with fetch()

Conceptually:

```javascript
const response = await fetch(
    "http://localhost:8080/api/chat/stream",
    {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            message: "Explain OAuth 2.0"
        })
    }
);
```

Instead of:

```javascript
await response.json();
```

you access:

```javascript
response.body
```

which is a `ReadableStream`.

Then:

```javascript
const reader = response.body.getReader();
```

And read chunks:

```javascript
while (true) {

    const { value, done } =
        await reader.read();

    if (done) {
        break;
    }

    const chunk =
        new TextDecoder().decode(value);

    console.log(chunk);
}
```

Now you have:

```text
Spring Boot
     ↓
HTTP stream
     ↓
React fetch()
     ↓
ReadableStream
     ↓
chunk
     ↓
UI
```

---

# 15. SSE vs streaming HTTP

This distinction is worth understanding very well.

### SSE

```text
Server → Client
```

Long-lived connection with events.

```text
Client
  |
  | HTTP GET
  ↓
Server
  |
  | event
  ↓
Client
  |
  | event
  ↓
Client
```

It's designed specifically for server-to-client event streaming.

---

### Streaming HTTP

More general:

```text
HTTP request
      ↓
HTTP response
      ↓
chunk
chunk
chunk
chunk
```

You can use POST and send JSON in the request.

For an AI chat application, this can be very convenient.

---

# 16. Why not WebSocket?

You might ask:

> Why don't we just use WebSockets?

WebSocket provides:

```text
Client ←→ Server
```

Both directions can send messages at any time.

SSE is:

```text
Server → Client
```

So if your requirement is simply:

```text
User sends request
        ↓
LLM generates response
        ↓
Server streams response
        ↓
Browser
```

SSE is often simpler.

Think of it like this:

```text
SSE

Client ──────── Request ───────→ Server
Client ←──────── chunk ───────── Server
Client ←──────── chunk ───────── Server
Client ←──────── chunk ───────── Server
```

Whereas WebSocket:

```text
Client ←──────────────→ Server
       messages both ways
```

You don't necessarily need the complexity of WebSockets just to stream an LLM response.

---

# 17. Important: token ≠ HTTP chunk

This is a subtle but **very important** concept.

You learned earlier:

```text
LLM
 ↓
Token
 ↓
Token
 ↓
Token
```

But don't assume:

```text
1 LLM token = 1 HTTP chunk
```

That's not guaranteed.

For example, the LLM might produce:

```text
Token A
Token B
Token C
Token D
```

but the network could deliver:

```text
HTTP chunk 1:
Token A + Token B

HTTP chunk 2:
Token C

HTTP chunk 3:
Token D
```

Or:

```text
HTTP chunk 1:
Token A

HTTP chunk 2:
Token B + Token C + Token D
```

So the architecture is better understood as:

```text
LLM generation
      ↓
Spring AI stream
      ↓
HTTP serialization
      ↓
Network chunks/events
      ↓
Browser
```

Don't build application logic assuming a one-to-one relationship.

---

# 18. Why streaming improves UX

Suppose the LLM takes 8 seconds.

### Without streaming

```text
0s ───────────────────────── 8s
                              ↓
                        Complete answer
```

The user sees:

```text
[Loading...]
```

for 8 seconds.

---

### With streaming

```text
0s     1s    2s    3s    4s    5s
│      │     │     │     │     │
OAuth  2.0   is    an    auth  ...
```

The user sees progress immediately.

This improves **perceived latency** even if the total generation time hasn't changed.

That's one of the major reasons virtually every ChatGPT-style interface streams responses.

---

# 19. Handling completion

Your frontend needs to know:

> Is the server finished?

With SSE, you can send a custom event such as:

```text
event: done
data: true
```

React can listen for:

```javascript
eventSource.addEventListener("done", () => {
    eventSource.close();
});
```

Conceptually:

```text
data: OAuth

data: 2.0

data: is

data: an

event: done
data: true
```

Then:

```text
React receives "done"
        ↓
close connection
        ↓
stop loading indicator
```

---

# 20. Error handling

This is another important part of production streaming.

Imagine:

```text
React
 ↓
Spring Boot
 ↓
LLM
 ↓
Error
```

You need to communicate that to React.

For example:

```text
event: error
data: LLM request failed
```

React:

```javascript
eventSource.addEventListener("error", (event) => {
    // show error
});
```

You also need to handle:

* network failure
* LLM timeout
* provider error
* authentication failure
* connection termination
* browser refresh
* user cancelling generation

---

# 21. User cancellation

This is particularly interesting for ChatGPT-style applications.

Imagine the user clicks:

```text
Stop generating
```

React needs to terminate the streaming connection.

For `EventSource`:

```javascript
eventSource.close();
```

But there's another question:

> Does closing the browser connection actually stop the LLM request?

Not necessarily.

Your backend needs to propagate cancellation appropriately.

Conceptually:

```text
User clicks STOP
       ↓
React
       ↓
HTTP connection cancelled
       ↓
Spring WebFlux cancellation
       ↓
LLM subscription cancelled
       ↓
Stop consuming generation
```

This becomes important when you're paying for model generation.

---

# 22. Backpressure

You learned this in Topic 18.

Suppose:

```text
LLM produces:
100 chunks/sec

React can process:
20 chunks/sec
```

You have a producer/consumer mismatch.

Reactive Streams gives you mechanisms to handle this.

Conceptually:

```text
LLM
 ↓
Producer
 ↓
Flux
 ↓
Consumer
 ↓
React
```

Backpressure is essentially about:

> What happens when the producer is faster than the consumer?

For most basic LLM applications, you won't manually implement complicated backpressure logic. But as an architect, you should understand that **streaming isn't just "send strings in a loop."**

---

# 23. A better production API

Instead of returning just strings, you may eventually want structured events.

For example:

```json
{
    "type": "token",
    "content": "OAuth"
}
```

then:

```json
{
    "type": "token",
    "content": " 2.0"
}
```

Then:

```json
{
    "type": "done"
}
```

And potentially:

```json
{
    "type": "error",
    "message": "..."
}
```

This gives your frontend a proper protocol.

For example:

```text
StreamEvent

TOKEN
DONE
ERROR
```

That's much easier to extend than sending raw strings.

---

# 24. Production architecture

Eventually your application might look like:

```text
                    React
                      │
                      │ HTTPS
                      ↓
                API Gateway
                      │
                      ↓
              Spring Boot API
                      │
                      ↓
                Chat Service
                      │
                ┌─────┴─────┐
                ↓           ↓
             Memory       Spring AI
                            │
                            ↓
                           LLM
```

And the response:

```text
LLM
 ↓
Flux
 ↓
Chat Service
 ↓
SSE / HTTP streaming
 ↓
Gateway
 ↓
React
 ↓
Chat UI
```

Now you are starting to think about **real system architecture**, rather than simply learning a Spring AI API.

---

# 25. Common mistakes

### Mistake 1 — Calling `.call()` instead of `.stream()`

```java
.call()
```

returns the completed result.

For streaming:

```java
.stream()
```

---

### Mistake 2 — Returning `String`

This defeats streaming:

```java
public String chat() {
    ...
}
```

Instead:

```java
public Flux<String> chat() {
    ...
}
```

---

### Mistake 3 — Buffering the entire response

Don't accidentally do:

```text
LLM
 ↓
collect everything
 ↓
return response
```

That's no longer streaming.

---

### Mistake 4 — Assuming HTTP chunks equal tokens

As mentioned earlier:

```text
Token ≠ HTTP chunk
```

They're different layers.

---

### Mistake 5 — Forgetting connection cleanup

You need to handle:

```text
completion
error
cancellation
disconnect
```

---

# 26. The complete mental model

This is the diagram I'd recommend remembering:

```text
                         USER
                           │
                           ↓
                     ┌──────────┐
                     │  React   │
                     │ Chat UI  │
                     └────┬─────┘
                          │
                   HTTP streaming
                          │
                          ↓
                 ┌────────────────┐
                 │  Spring Boot   │
                 │   Controller   │
                 └───────┬────────┘
                         │
                       Flux
                         │
                         ↓
                 ┌────────────────┐
                 │   Spring AI    │
                 │   ChatClient   │
                 └───────┬────────┘
                         │
                       stream()
                         │
                         ↓
                    ┌─────────┐
                    │   LLM   │
                    └────┬────┘
                         │
                 generated content
                         │
                         ↓
                    Spring AI
                         │
                        Flux
                         │
                         ↓
                   Spring Boot
                         │
                  SSE / HTTP chunks
                         │
                         ↓
                      React
                         │
                  append to state
                         │
                         ↓
                    Chat UI
```

---

# 27. What you should actually practice

For your **1–2 hour Topic 19 slot**, I would do this hands-on exercise:

### Step 1 — Create a Spring Boot endpoint

```text
POST /api/chat/stream
```

Input:

```json
{
    "message": "Explain OAuth 2.0"
}
```

### Step 2 — Use Spring AI

```java
chatClient
    .prompt()
    .user(request.message())
    .stream()
    .content();
```

### Step 3 — Return a reactive stream

```java
Flux<String>
```

### Step 4 — Make the HTTP response streaming

Understand:

```text
text/event-stream
```

### Step 5 — Create React chat UI

```text
User input
     ↓
Send
     ↓
Streaming response
     ↓
Append chunks
     ↓
Display
```

### Step 6 — Add Stop button

```text
STOP
 ↓
cancel stream
```

### Step 7 — Add error handling

```text
LLM error
 ↓
Spring Boot
 ↓
React
 ↓
Display error
```

---

# 28. The most important concepts to take away

If you finish Topic 19 understanding these **8 things**, you've got it:

| Concept             | Understand                               |
| ------------------- | ---------------------------------------- |
| `Flux`              | Multiple values arriving over time       |
| `stream()`          | Spring AI incremental generation         |
| SSE                 | Server → browser event streaming         |
| `text/event-stream` | SSE HTTP content type                    |
| `EventSource`       | Browser SSE client                       |
| `ReadableStream`    | Browser API for streaming HTTP responses |
| Cancellation        | Stop generation when user stops          |
| Backpressure        | Producer vs consumer speed               |

And the big picture:

```text
Spring AI streaming
        ↓
       Flux
        ↓
Spring Boot streaming endpoint
        ↓
SSE / streaming HTTP
        ↓
React
        ↓
Incrementally update UI
```

That is **Topic 19**.

One architectural point I'd especially keep in your notes: **LLM token streaming, reactive `Flux`, HTTP streaming, and SSE are four different layers.** They work together, but they are not interchangeable terms. Understanding that separation will save you a lot of confusion later when you get into gateways, proxies, buffering, cancellation, and production architecture.
