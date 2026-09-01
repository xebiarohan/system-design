# Phase 6 — Streaming

Streaming is the phase where you move from:

```text
User
 ↓
Spring Boot
 ↓
Spring AI
 ↓
LLM
 ↓
Wait...
 ↓
Complete response
 ↓
User
```

to:

```text
User
 ↓
Spring Boot
 ↓
Spring AI
 ↓
LLM
 ↓
"Hello"
 ↓
" there"
 ↓
"!"
 ↓
" How"
 ↓
" can"
 ↓
" I"
 ↓
" help?"
```

The key idea is:

> **Don't wait for the LLM to finish generating the entire response before sending anything to the user.**

Instead, send the generated content progressively.

---

# 17. Streaming — 3 hours

## 17.1 What problem does streaming solve?

Imagine the user asks:

```text
Explain how Spring Boot dependency injection works.
```

Without streaming:

```text
Request
   ↓
LLM starts generating
   ↓
2 sec
   ↓
3 sec
   ↓
4 sec
   ↓
5 sec
   ↓
Complete response
   ↓
Browser displays everything
```

The user sees nothing for 5 seconds.

With streaming:

```text
Request
   ↓
LLM starts generating
   ↓
"Spring Boot"
   ↓
" uses"
   ↓
" dependency"
   ↓
" injection"
   ↓
" to..."
```

The user starts seeing the answer almost immediately.

This is why ChatGPT-style applications feel responsive.

---

# 17.2 Streaming vs normal request

A normal LLM call conceptually looks like:

```java
String response = chatClient
        .prompt()
        .user("Explain dependency injection")
        .call()
        .content();
```

The important part is:

```text
call()
```

You are essentially saying:

> Give me the completed result.

Streaming changes that:

```java
Flux<String> response = chatClient
        .prompt()
        .user("Explain dependency injection")
        .stream()
        .content();
```

Now you are saying:

> Give me pieces of the response as they become available.

Conceptually:

```text
LLM
 │
 ├── "Spring"
 ├── " Boot"
 ├── " uses"
 ├── " dependency"
 ├── " injection"
 └── ...
```

Spring AI exposes those pieces as a reactive stream.

---

# 17.3 What is actually being streamed?

Be careful with the word **token**.

An LLM generates tokens internally:

```text
The Spring Boot application...
```

might internally be represented by tokens such as:

```text
The
 Spring
 Boot
 application
 ...
```

But your application does **not necessarily receive exactly one token per stream event**.

Depending on the model/provider/API, a streamed chunk may contain:

```text
"Spring"
```

then:

```text
" Boot"
```

then:

```text
" makes"
```

or larger pieces.

So think of it as:

```text
LLM generates tokens
        ↓
Provider creates streamed chunks
        ↓
Spring AI exposes the stream
        ↓
Your application consumes chunks
```

Don't build application logic assuming:

> "One `Flux` element = one LLM token."

That's an implementation detail you shouldn't rely on.

---

# 17.4 Normal `call()` vs `stream()`

The most important distinction for this topic is:

```text
call()
```

versus

```text
stream()
```

### Normal

```java
String answer = chatClient
        .prompt()
        .user("Tell me about Java")
        .call()
        .content();
```

Conceptually:

```text
LLM
 ↓
Generate entire response
 ↓
String
```

### Streaming

```java
Flux<String> answer = chatClient
        .prompt()
        .user("Tell me about Java")
        .stream()
        .content();
```

Conceptually:

```text
LLM
 ↓
Chunk
 ↓
Chunk
 ↓
Chunk
 ↓
Chunk
 ↓
Flux<String>
```

This distinction is fundamental to Spring AI streaming.

---

# 17.5 Understanding `Flux`

You'll encounter:

```java
Flux<String>
```

Don't think of this as:

> "A String."

Think:

> **A sequence of Strings that arrive over time.**

For example:

```text
Flux<String>
```

might produce:

```text
"Spring"
" AI"
" provides"
" abstractions"
" for"
" working"
" with"
" AI"
" models."
```

The important difference is **time**.

A normal collection is:

```java
List<String>
```

which conceptually means:

```text
Here are all the values.
```

A `Flux` means:

```text
Here is a stream of values.
More may arrive later.
```

Conceptually:

```text
Time ───────────────────────>

t1       t2       t3       t4
 │        │        │        │
 ▼        ▼        ▼        ▼
"Spring" "AI"    "is"    "great"
```

---

# 17.6 Streaming with `ChatClient`

A typical Spring AI example looks like:

```java
@GetMapping("/chat")
public Flux<String> chat(String message) {

    return chatClient
            .prompt()
            .user(message)
            .stream()
            .content();
}
```

The flow is:

```text
HTTP request
     ↓
Controller
     ↓
ChatClient
     ↓
LLM
     ↓
stream()
     ↓
Flux<String>
     ↓
HTTP response
```

This is the first important Spring AI streaming architecture you should understand.

---

# 17.7 Why `Flux<String>`?

Because the LLM response isn't available all at once.

Suppose the final answer is:

```text
Spring AI simplifies integrating AI models into Spring applications.
```

Streaming could conceptually produce:

```text
Flux<String>

"Spring"
" AI"
" simplifies"
" integrating"
" AI"
" models"
" into"
" Spring"
" applications."
```

The `Flux` represents the **ongoing production of the response**.

---

# 17.8 Streaming lifecycle

You should understand the lifecycle:

```text
1. Client sends request
          ↓
2. Spring Boot receives request
          ↓
3. ChatClient creates prompt
          ↓
4. Spring AI calls LLM provider
          ↓
5. LLM starts generating
          ↓
6. Provider sends chunks
          ↓
7. Spring AI converts them into stream
          ↓
8. Your application receives Flux elements
          ↓
9. HTTP layer sends them to client
          ↓
10. Stream completes
```

Visually:

```text
                         LLM
                          │
                    Generate response
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
           chunk        chunk        chunk
             │            │            │
             └────────────┼────────────┘
                          ↓
                       Spring AI
                          ↓
                       Flux<String>
                          ↓
                     HTTP response
                          ↓
                        React
```

---

# 17.9 Streaming doesn't make the LLM faster

This is an important distinction.

Streaming does **not necessarily mean**:

```text
LLM generates response faster
```

Instead, it means:

```text
You receive the response earlier.
```

Suppose the model takes 5 seconds to generate a complete response.

### Without streaming

```text
0 sec ───────────────────── 5 sec
                              ↓
                         Entire answer
```

### With streaming

```text
0 sec
 ↓
First chunk
 ↓
Second chunk
 ↓
Third chunk
 ↓
...
5 sec
 ↓
Final chunk
```

The total generation time could still be around 5 seconds.

But the **time to first visible content** can be much lower.

That's extremely important for user experience.

---

# 17.10 Time to First Token

You'll eventually encounter the concept:

**TTFT — Time To First Token**

It measures approximately:

```text
Request sent
     ↓
Model starts responding
     ↓
First streamed content
```

For example:

```text
Request
  │
  │  600 ms
  ▼
First chunk
```

TTFT:

```text
600 ms
```

Then the remaining response streams afterward.

For AI applications, two different performance characteristics matter:

```text
TTFT
+
Generation throughput
```

A model might have:

```text
TTFT = 500 ms
Generation = 50 tokens/sec
```

The user sees something quickly and then watches the answer build.

---

# 17.11 Streaming response vs complete response

Think of it like downloading a large file.

### Normal response

```text
Server
  │
  │ downloading...
  │
  │ downloading...
  │
  ▼
Complete file
  ↓
Browser
```

### Streaming

```text
Server
  │
  ├── chunk 1 ──→ Browser
  ├── chunk 2 ──→ Browser
  ├── chunk 3 ──→ Browser
  ├── chunk 4 ──→ Browser
  └── chunk 5 ──→ Browser
```

LLM streaming follows the same general idea.

---

# 17.12 `ChatResponse` streaming

You can also stream richer objects rather than only content.

Conceptually:

```java
Flux<ChatResponse> responses =
        chatClient
                .prompt()
                .user("Explain Spring AI")
                .stream()
                .chatResponse();
```

Now you're dealing with:

```text
Flux<ChatResponse>
```

instead of:

```text
Flux<String>
```

This matters when you need information beyond the text itself.

For example:

```text
ChatResponse
 ├── generated content
 ├── metadata
 ├── usage information
 └── other response information
```

So remember:

```text
.stream().content()
        ↓
Flux<String>
```

while:

```text
.stream().chatResponse()
        ↓
Flux<ChatResponse>
```

---

# 17.13 The mental model you should have

At this point, you should be able to draw:

```text
             ChatClient
                  │
                  │ stream()
                  ▼
             LLM Provider
                  │
          ┌───────┼────────┐
          ↓       ↓        ↓
        chunk   chunk    chunk
          │       │        │
          └───────┼────────┘
                  ↓
             Flux<T>
                  ↓
             HTTP layer
                  ↓
               Client
```

That is the core of Topic 17.

---

# 17.14 What you should code

Create a simple endpoint:

```java
@RestController
public class ChatController {

    private final ChatClient chatClient;

    public ChatController(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    @GetMapping("/chat")
    public Flux<String> chat(@RequestParam String message) {

        return chatClient
                .prompt()
                .user(message)
                .stream()
                .content();
    }
}
```

Then test it with a request such as:

```text
GET /chat?message=Explain%20dependency%20injection
```

Your goal is to observe:

```text
Request
   ↓
ChatClient
   ↓
LLM
   ↓
Flux<String>
   ↓
Multiple chunks
```

Don't worry about React yet.

That's Topic **19**.

First understand the backend stream.

---

# 17.15 What to experiment with

Spend most of your 3 hours actually coding.

### Experiment 1 — Normal call

```java
.call()
.content()
```

Observe:

```text
complete response
```

### Experiment 2 — Streaming

```java
.stream()
.content()
```

Observe:

```text
chunk
chunk
chunk
chunk
...
```

### Experiment 3 — Slow response

Ask the model for a long answer.

Observe:

```text
First content
      ↓
more content
      ↓
more content
      ↓
more content
```

### Experiment 4 — `Flux<ChatResponse>`

Try:

```java
.stream()
.chatResponse();
```

Inspect the response objects.

---

# 17.16 One important backend distinction

There are actually **two different problems** here:

### Problem A — LLM streaming

```text
LLM
 ↓
chunks
 ↓
Spring AI
```

### Problem B — HTTP streaming

```text
Spring Boot
 ↓
chunks
 ↓
Browser
```

Spring AI solves/abstracts much of **Problem A**.

Your web application still needs to choose an appropriate mechanism for **Problem B**.

That's why Topics 18 and 19 exist.

---

# Topic 17 vs Topic 18 vs Topic 19

Keep these separate in your head:

```text
17. Streaming
       │
       ▼
How does an LLM produce/send
partial responses?
```

Then:

```text
18. Reactive streaming
       │
       ▼
How does Java/Spring represent
and process an asynchronous stream?
```

Then:

```text
19. Spring Boot → React streaming
       │
       ▼
How do we transport those chunks
to the browser?
```

So the complete architecture becomes:

```text
                 LLM
                  │
             streamed chunks
                  │
                  ▼
             Spring AI
                  │
              Flux<String>
                  │
                  ▼
             Spring Boot
                  │
             HTTP streaming
                  │
                  ▼
                React
                  │
                  ▼
        ChatGPT-style UI
```

---

# What you should know after Topic 17

You **do not** need to master WebFlux or SSE yet.

After these 3 hours, you should be able to answer:

### 1. Why do we stream LLM responses?

```text
Lower perceived latency
+
Better user experience
```

### 2. What does `stream()` do?

It requests/handles the model response progressively rather than waiting for the complete response.

### 3. What does this mean?

```java
Flux<String>
```

Answer:

> A reactive stream that can emit multiple `String` values asynchronously over time.

### 4. What's the difference?

```java
.call().content()
```

vs

```java
.stream().content()
```

```text
call()
   ↓
complete result

stream()
   ↓
Flux of partial results
```

### 5. Is each `Flux` element necessarily one token?

**No.**

A streamed element is a chunk; its boundaries depend on the provider/API.

### 6. Does streaming make generation faster?

Not necessarily.

It primarily improves **time to first visible content** and perceived responsiveness.

### 7. What comes next?

```text
Streaming
   ↓
Reactive Streams / Flux
   ↓
HTTP streaming
   ↓
SSE
   ↓
React streaming UI
```

---

# 🛠️ Mini exercise for Topic 17

Build only this:

```text
GET /chat
     │
     ▼
Spring Boot
     │
     ▼
ChatClient
     │
     ▼
.stream()
     │
     ▼
Flux<String>
     │
     ▼
Console
```

For example:

```java
@GetMapping("/chat")
public Flux<String> chat(@RequestParam String message) {

    return chatClient
            .prompt()
            .user(message)
            .stream()
            .content();
}
```

Then compare it with:

```java
@GetMapping("/chat-normal")
public String chatNormal(@RequestParam String message) {

    return chatClient
            .prompt()
            .user(message)
            .call()
            .content();
}
```

Your experiment should demonstrate:

```text
/chat-normal

Request ──────────────────→ Complete response


/chat

Request ──→ chunk ─→ chunk ─→ chunk ─→ chunk ─→ ...
```

Once this makes intuitive sense, **Topic 17 is done**.

The next topic, **18 — Reactive Streaming**, is where you should slow down a little: that's where `Flux`, subscriptions, asynchronous processing, and backpressure become important to understanding why the Spring AI streaming API works the way it does.
