Absolutely. **Topic 18 — Reactive Streaming** is where you learn what is happening underneath the `Flux<String>` you saw in Topic 17.

# Phase 6 — Streaming

## 18. Reactive Streaming — 1–2 hours

The goal of this topic is **not** to become a Reactor expert.

You need to understand enough of **Reactive Streams + Project Reactor + `Flux`** to confidently build Spring AI streaming applications.

The mental model is:

```text
LLM
 ↓
streamed chunks
 ↓
Spring AI
 ↓
Flux<String>
 ↓
Reactive pipeline
 ↓
HTTP streaming
 ↓
React
```

---

# 18.1 First: What does "reactive" mean?

In traditional programming, you often think:

```text
Do something
 ↓
Wait
 ↓
Get result
 ↓
Continue
```

For example:

```java
String answer = getAnswer();

System.out.println(answer);
```

Your code asks for the answer and waits for it.

Reactive programming changes the mental model.

Instead:

```text
Start operation
      ↓
Receive values whenever they arrive
      ↓
Process each value
      ↓
Continue doing other work
```

For streaming AI:

```text
LLM
 │
 ├── chunk 1 ──→
 ├── chunk 2 ──→
 ├── chunk 3 ──→
 ├── chunk 4 ──→
 └── complete
```

You don't need to wait for the complete answer.

---

# 18.2 The most important concept: `Flux`

If you remember only one thing from this topic, remember:

> **`Flux<T>` represents 0..N values that can arrive over time.**

For example:

```java
Flux<String>
```

means:

```text
0 or more Strings
        +
they may arrive asynchronously
        +
the stream eventually completes or errors
```

Imagine:

```text
Flux<String>

"Hello"
   ↓
" there"
   ↓
"!"
   ↓
" How"
   ↓
" are"
   ↓
" you?"
   ↓
COMPLETE
```

This is perfect for LLM streaming.

---

# 18.3 `Mono` vs `Flux`

You'll see both in Spring applications.

### `Mono<T>`

Represents:

```text
0 or 1 value
```

Example:

```java
Mono<String>
```

Conceptually:

```text
Request
   ↓
One result
```

---

### `Flux<T>`

Represents:

```text
0..N values
```

Example:

```java
Flux<String>
```

Conceptually:

```text
Request
   ↓
value
   ↓
value
   ↓
value
   ↓
value
   ↓
complete
```

For Spring AI streaming:

```text
Single response
      ↓
Mono / String

Streaming response
      ↓
Flux
```

A useful mental shortcut:

```text
Mono = one
Flux = many
```

---

# 18.4 Why is `Flux` perfect for LLMs?

Suppose the model produces:

```text
Spring AI is a framework for building AI applications.
```

A streaming API might produce:

```text
"Spring"
" AI"
" is"
" a"
" framework"
" for"
" building"
" AI"
" applications."
```

Instead of:

```java
String response
```

you have:

```java
Flux<String> response
```

So:

```java
Flux<String> response =
        chatClient
                .prompt()
                .user("What is Spring AI?")
                .stream()
                .content();
```

Now you have a **stream of data**, not one completed value.

---

# 18.5 A `Flux` is not the data itself

This distinction is extremely important.

When you write:

```java
Flux<String> response;
```

you don't have:

```text
"Spring AI is..."
```

You have something closer to:

```text
A description of a future stream of values.
```

Think of it as a pipeline:

```text
             Flux
              │
              ▼
       ┌───────────────┐
       │ Future stream │
       └───────────────┘
              │
       values arrive
              ↓
        "Spring"
        " AI"
        " is"
        ...
```

This is why reactive programming initially feels strange to developers accustomed to imperative Java.

---

# 18.6 Creating a `Flux`

You can create a simple `Flux` yourself:

```java
Flux<String> words =
        Flux.just(
                "Spring",
                "AI",
                "is",
                "awesome"
        );
```

Conceptually:

```text
Flux
 │
 ├── Spring
 ├── AI
 ├── is
 └── awesome
```

You can consume it:

```java
words.subscribe(System.out::println);
```

Output:

```text
Spring
AI
is
awesome
```

---

# 18.7 `subscribe()` — a very important concept

This is one of the most important things to understand about Reactor.

Consider:

```java
Flux<String> words =
        Flux.just("A", "B", "C");
```

Creating the `Flux` does not necessarily mean:

```text
A
B
C
```

have already been consumed.

You need a subscriber.

```java
words.subscribe(value -> {
    System.out.println(value);
});
```

Conceptually:

```text
Flux
 │
 │ subscribe()
 ▼
Subscriber
 │
 ├── A
 ├── B
 └── C
```

The subscriber says:

> I'm interested in the values produced by this stream.

---

# 18.8 Reactive pipeline

Here's where Reactor becomes powerful.

You can create a pipeline:

```java
Flux<String> words =
        Flux.just("spring", "ai", "reactive");
```

Then:

```java
Flux<String> result =
        words
            .map(String::toUpperCase);
```

Now:

```text
spring
   ↓
SPRING

ai
   ↓
AI

reactive
   ↓
REACTIVE
```

The pipeline is:

```text
Flux
 ↓
map()
 ↓
Flux
```

You can chain operations:

```java
Flux<String> result =
        words
            .map(String::toUpperCase)
            .filter(word -> word.length() > 2);
```

Conceptually:

```text
spring
  ↓
SPRING
  ↓
passes filter
  ↓
SPRING

ai
  ↓
AI
  ↓
filtered out

reactive
  ↓
REACTIVE
  ↓
passes filter
  ↓
REACTIVE
```

---

# 18.9 Why is this useful for Spring AI?

Imagine:

```java
Flux<String> response =
        chatClient
                .prompt()
                .user("Explain Spring AI")
                .stream()
                .content();
```

You can process the stream:

```java
Flux<String> processed =
        response.map(chunk -> chunk.toUpperCase());
```

Or:

```java
Flux<String> processed =
        response.filter(chunk -> !chunk.isBlank());
```

Or:

```java
Flux<String> processed =
        response.map(this::processChunk);
```

So:

```text
LLM chunks
    ↓
Flux
    ↓
map/filter/etc.
    ↓
HTTP response
```

That's the real power of reactive streaming.

---

# 18.10 Reactive operators

You don't need to memorize dozens of operators.

For your Spring AI roadmap, understand these first:

### `map()`

Transform every element.

```java
flux.map(value -> value.toUpperCase());
```

```text
"a"
 ↓ map
"A"

"b"
 ↓ map
"B"
```

---

### `filter()`

Keep only certain elements.

```java
flux.filter(value -> value.length() > 3);
```

```text
"Hi"      → removed
"Spring"  → kept
"AI"      → removed
```

---

### `flatMap()`

Useful when each value triggers another asynchronous operation.

For example:

```text
chunk
 ↓
async operation
 ↓
another result
```

You don't need to master this yet, but you should recognize it.

---

### `doOnNext()`

Useful for observing values:

```java
flux.doOnNext(chunk -> {
    log.info("Received: {}", chunk);
});
```

It is very useful for debugging streaming applications.

---

### `doOnComplete()`

Runs when the stream completes:

```java
flux.doOnComplete(() -> {
    log.info("Streaming finished");
});
```

Useful for:

```text
Stream started
      ↓
chunks
      ↓
chunks
      ↓
Stream completed
```

---

### `doOnError()`

Useful for observing failures:

```java
flux.doOnError(error -> {
    log.error("Streaming failed", error);
});
```

This matters a lot with LLM APIs because network/model/provider failures can occur **after some chunks have already been delivered**.

---

# 18.11 Reactive streams have three important outcomes

A stream isn't simply:

```text
values
```

It has a lifecycle:

```text
       ┌── values ──┐
       │            │
       ▼            ▼
START → DATA → DATA → DATA → COMPLETE
```

Or:

```text
START → DATA → DATA → ERROR
```

So conceptually:

```text
Flux
 │
 ├── onNext(value)
 ├── onNext(value)
 ├── onNext(value)
 └── onComplete()
```

or:

```text
Flux
 │
 ├── onNext(value)
 ├── onNext(value)
 └── onError(error)
```

There are three concepts you should know:

```text
onNext()
onComplete()
onError()
```

---

# 18.12 `onNext`

Every emitted value is conceptually an:

```text
onNext(value)
```

For example:

```text
onNext("Spring")
onNext(" AI")
onNext(" is")
onNext(" great")
```

Your application receives these values one at a time.

---

# 18.13 `onComplete`

When there are no more values:

```text
onComplete()
```

For an LLM:

```text
"Spring"
" AI"
" is"
" great"
     ↓
onComplete()
```

This tells your application:

> The response is finished.

This is important for things such as:

```text
Enable "Send" button
Save completed conversation
Close streaming connection
Update UI state
Record metrics
```

---

# 18.14 `onError`

Something can go wrong:

```text
LLM
 ↓
chunk
 ↓
chunk
 ↓
NETWORK FAILURE
```

Then:

```text
onError(exception)
```

For example:

```text
"Spring"
" AI"
" provides"
ERROR
```

The client may receive only a partial answer.

This is very different from a traditional request where you might simply receive:

```text
HTTP 500
```

after waiting for the entire operation.

Streaming applications therefore need to think about **partial responses**.

---

# 18.15 The big concept: Backpressure

Now we get to the most important reactive concept after `Flux`.

**Backpressure** means:

> A consumer can communicate how much data it can handle, rather than the producer blindly overwhelming it.

Imagine:

```text
Producer
   │
   │ 1000 values/sec
   ▼
Consumer
   │
   │ can handle 100/sec
   ▼
💥
```

Without some mechanism to control the flow:

```text
Producer >>>>>>>>>>> Consumer
```

The producer can overwhelm the consumer.

With backpressure:

```text
Producer
   │
   │ "How much can you handle?"
   ▼
Consumer
   │
   │ "Give me what I can process."
   ▼
Controlled stream
```

---

# 18.16 Backpressure in an AI application

Imagine an extreme scenario:

```text
LLM
 ↓
very fast chunks
 ↓
Spring Boot
 ↓
slow network
 ↓
Browser
```

The producer and consumer operate at different speeds.

You might have:

```text
LLM generation
      ↓
     fast
      ↓
Spring application
      ↓
     fast
      ↓
Network
      ↓
     slow
      ↓
Browser
```

Reactive Streams provides mechanisms for handling this mismatch.

---

# 18.17 Do you need to implement backpressure yourself?

Usually:

**No.**

This is important.

As a Spring developer, don't interpret learning backpressure as:

> "I need to manually implement a queue and control every token."

Reactor and the Reactive Streams ecosystem provide the infrastructure.

Your job at this stage is to understand:

```text
Producer
    ↓
Reactive stream
    ↓
Consumer
```

and recognize that the consumer may not be able to process data as quickly as the producer.

---

# 18.18 Why Spring WebFlux appears here

You will probably encounter:

```text
Spring MVC
```

and:

```text
Spring WebFlux
```

Reactive streaming is closely associated with **Spring WebFlux**.

Traditional MVC commonly uses:

```text
Thread
 ↓
Request
 ↓
Wait
 ↓
Response
```

Reactive applications are designed around:

```text
Request
 ↓
Reactive pipeline
 ↓
Flux
 ↓
asynchronous processing
 ↓
streaming response
```

Don't interpret this as:

> "Spring MVC cannot stream."

That's too simplistic.

The important point for your roadmap is:

> **WebFlux and Reactor provide a natural programming model for asynchronous, non-blocking streams.**

---

# 18.19 Blocking vs non-blocking

This distinction is important.

### Blocking

Imagine:

```java
String result = callLLM();
```

Your code waits for:

```text
callLLM()
   ↓
waiting...
   ↓
result
```

The calling thread is blocked during that operation.

---

### Reactive/non-blocking

Conceptually:

```java
Flux<String> result = streamFromLLM();
```

You're working with a stream that emits values asynchronously.

```text
Start
 ↓
register interest
 ↓
continue
 ↓
chunks arrive
 ↓
process chunks
```

This allows the application to handle many concurrent I/O operations efficiently.

---

# 18.20 Don't confuse asynchronous with multithreading

This is a common mistake.

People sometimes think:

```text
Reactive
=
new thread for every chunk
```

No.

Reactive programming is primarily about:

```text
asynchronous data flow
+
non-blocking processing
+
streams
+
backpressure
```

It does **not** simply mean:

> Create more threads.

In fact, one of the benefits of non-blocking I/O is that you don't need a dedicated blocked thread sitting around waiting for every network operation.

---

# 18.21 A useful analogy

Imagine a restaurant.

### Traditional approach

You order:

```text
"Give me my entire meal."
```

Then:

```text
wait
wait
wait
wait
```

Finally:

```text
🍽️ Entire meal
```

### Streaming approach

The restaurant brings:

```text
🥗 appetizer
 ↓
🍲 soup
 ↓
🍛 main course
 ↓
🍰 dessert
```

You can start consuming as things arrive.

### Reactive system

There is also coordination:

```text
Kitchen
  ↓
Food
  ↓
Waiter
  ↓
Customer
```

If the customer is slow:

```text
Kitchen → Waiter → Customer
```

the system needs a way to avoid endlessly accumulating work.

That's where the idea of **backpressure** becomes useful.

---

# 18.22 Spring AI + Reactor

Now connect everything you've learned.

Spring AI:

```java
Flux<String> response =
        chatClient
                .prompt()
                .user("Explain Spring AI")
                .stream()
                .content();
```

This gives you:

```text
             Spring AI
                 │
                 ▼
             LLM stream
                 │
                 ▼
             Flux<String>
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
     chunk     chunk     chunk
       │         │         │
       └─────────┼─────────┘
                 ▼
           Reactive pipeline
                 │
                 ▼
             HTTP stream
```

---

# 18.23 Add processing to the stream

For example:

```java
Flux<String> response =
        chatClient
                .prompt()
                .user("Explain Spring AI")
                .stream()
                .content()
                .doOnNext(chunk ->
                        log.info("Chunk: {}", chunk)
                )
                .doOnComplete(() ->
                        log.info("Completed")
                )
                .doOnError(error ->
                        log.error("Streaming failed", error)
                );
```

Now you have:

```text
LLM
 ↓
chunk
 ↓
doOnNext()
 ↓
chunk
 ↓
doOnNext()
 ↓
chunk
 ↓
doOnNext()
 ↓
complete
 ↓
doOnComplete()
```

And if something fails:

```text
chunk
 ↓
chunk
 ↓
ERROR
 ↓
doOnError()
```

---

# 18.24 Important: Don't subscribe unnecessarily in Spring WebFlux

This is a subtle but very important Spring concept.

You might be tempted to do:

```java
@GetMapping("/chat")
public Flux<String> chat(String message) {

    Flux<String> response = chatClient
            .prompt()
            .user(message)
            .stream()
            .content();

    response.subscribe();

    return response;
}
```

**Don't do this.**

In a reactive Spring application, normally you return the `Flux`:

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

Spring WebFlux can subscribe to and manage the reactive pipeline.

Think:

```text
Your controller
      ↓
returns Flux
      ↓
Spring WebFlux
      ↓
subscribes/manages stream
      ↓
client
```

Rather than:

```text
Your controller
      ↓
subscribe()
      ↓
Spring WebFlux
      ↓
subscribe again
```

Manual subscription inside application code can lead to duplicated work, lifecycle problems, and broken reactive composition.

---

# 18.25 The "lazy" concept

Reactive pipelines are often described as **lazy**.

For example:

```java
Flux<String> result =
        Flux.just("A", "B", "C")
            .map(String::toLowerCase);
```

You're defining:

```text
A → lowercase
B → lowercase
C → lowercase
```

The pipeline isn't necessarily doing all the work simply because you constructed it.

Subscription activates consumption.

Conceptually:

```text
Define pipeline
      ↓
Flux
      ↓
subscribe
      ↓
pipeline executes
```

This is another reason `Flux` feels different from a `List`.

---

# 18.26 `List` vs `Flux`

This comparison is worth memorizing.

| `List<T>`                    | `Flux<T>`                              |
| ---------------------------- | -------------------------------------- |
| Values are already available | Values can arrive later                |
| Usually finite collection    | 0..N values                            |
| Synchronous access           | Asynchronous/reactive                  |
| No completion signal         | Has completion/error lifecycle         |
| No reactive backpressure     | Reactive Streams supports backpressure |
| `for` loop                   | Reactive operators                     |

Example:

```java
List<String> names =
        List.of("John", "Jane", "Bob");
```

Everything is already there.

With:

```java
Flux<String> names;
```

you are dealing with a stream whose values may arrive over time.

---

# 18.27 `Flux` vs Java `Stream`

Don't confuse:

```java
Stream<T>
```

with:

```java
Flux<T>
```

They look similar because both let you do things like:

```java
.map()
.filter()
```

But they solve different problems.

### Java Stream

```text
Collection
 ↓
Stream
 ↓
process values
 ↓
result
```

Generally synchronous.

### Reactor `Flux`

```text
asynchronous producer
 ↓
Flux
 ↓
values over time
 ↓
consumer
```

Reactive and asynchronous.

For example:

```java
list.stream()
```

is not equivalent to:

```java
flux
```

The syntax can look similar, but the execution model is very different.

---

# 18.28 The complete Spring AI reactive flow

This is the diagram I'd recommend remembering:

```text
                   USER
                     │
                     ▼
              Spring Controller
                     │
                     ▼
                 ChatClient
                     │
                     ▼
                    LLM
                     │
            streamed response
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
       chunk       chunk       chunk
         │           │           │
         └───────────┼───────────┘
                     ▼
                 Flux<String>
                     │
              Reactive pipeline
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        map()      filter()   logging
          │          │          │
          └──────────┼──────────┘
                     ▼
                Spring WebFlux
                     │
                     ▼
              HTTP streaming
                     │
                     ▼
                   React
```

This is the connection between Topics **17, 18, and 19**.

---

# 18.29 What you should actually learn in your 1–2 hours

Don't go down a huge Reactor rabbit hole.

For your Spring AI roadmap, focus on these:

### Must understand

```text
Flux<T>
Mono<T>
subscribe()
onNext
onComplete
onError
map()
filter()
doOnNext()
doOnComplete()
doOnError()
```

And conceptually:

```text
Reactive stream
Non-blocking
Asynchronous data
Backpressure
```

### Nice to know

```text
flatMap()
concatMap()
buffer()
take()
timeout()
retry()
```

### Don't study deeply yet

You don't need to spend hours on:

```text
Schedulers
ParallelFlux
Advanced Reactor internals
Custom Publishers
Reactive Streams implementation details
```

Those can come later if your application requires them.

---

# 18.30 Hands-on exercise

Create a simple artificial stream first.

```java
@GetMapping("/numbers")
public Flux<Integer> numbers() {

    return Flux.just(1, 2, 3, 4, 5)
            .map(number -> number * 10)
            .doOnNext(number ->
                    System.out.println("Received: " + number)
            )
            .doOnComplete(() ->
                    System.out.println("Completed")
            );
}
```

Understand:

```text
1
 ↓ map
10
 ↓ doOnNext

2
 ↓ map
20
 ↓ doOnNext

...

5
 ↓ map
50
 ↓ doOnNext

 ↓
COMPLETE
```

Then build a delayed stream so you can actually **see** the reactive behavior.

For example, conceptually:

```text
1
   ↓ 1 second
2
   ↓ 1 second
3
   ↓ 1 second
4
   ↓ 1 second
5
```

Now you can observe that the values are arriving over time.

---

# 18.31 Then connect it to Spring AI

Once you understand the artificial example, replace:

```java
Flux.just(...)
```

with:

```java
chatClient
        .prompt()
        .user(message)
        .stream()
        .content()
```

Now the architecture becomes:

```text
Flux.just()
       │
       │ learn Reactor
       ▼
Flux<String>
       │
       │ Spring AI
       ▼
LLM streaming
```

This is the best way to learn this topic.

---

# 18.32 One subtle but important production issue

Suppose the LLM generates:

```text
The capital of France is Paris.
```

You might receive:

```text
"The capital"
" of"
" France"
" is"
" Paris"
"."
```

Don't assume every chunk is a complete sentence.

Therefore, avoid code like:

```java
response.map(chunk -> {
    // Assume chunk is a complete sentence
});
```

Instead, treat each chunk as:

> **an arbitrary piece of the generated response.**

This becomes particularly important when you reach:

* Markdown rendering
* JSON/structured output
* tool calls
* citations
* UI rendering

Streaming isn't simply:

```text
sentence → sentence → sentence
```

It is:

```text
partial data → partial data → partial data
```

---

# 18.33 Topic 17 vs Topic 18

This distinction should now be clear.

### Topic 17 — Streaming

You learned:

```text
LLM
 ↓
chunks
 ↓
Flux<String>
```

The question was:

> **Why and how do we receive LLM responses incrementally?**

### Topic 18 — Reactive Streaming

Now you're learning:

```text
Flux
 ↓
operators
 ↓
subscription
 ↓
lifecycle
 ↓
backpressure
 ↓
non-blocking processing
```

The question is:

> **How do we work with those continuously arriving chunks in a robust Java/Spring application?**

### Topic 19 — Spring Boot → React

Next you'll learn:

```text
Flux<String>
    ↓
Spring Boot
    ↓
SSE / HTTP streaming
    ↓
Browser
    ↓
React
    ↓
ChatGPT-style UI
```

The question becomes:

> **How do I get those reactive chunks all the way to the user's browser?**

---

# 🎯 What you should be able to explain after Topic 18

If someone asks you:

### "What is `Flux<String>`?"

You should say:

> `Flux<String>` is a Reactor publisher representing a potentially asynchronous sequence of zero or more `String` values that can be emitted over time.

### "Why does Spring AI use `Flux` for streaming?"

> Because an LLM's response can arrive incrementally. `Flux` provides a reactive abstraction for consuming and processing those chunks as they arrive.

### "What are the three signals?"

```text
onNext()
onComplete()
onError()
```

### "What is backpressure?"

> A mechanism in Reactive Streams that allows the consumer to control or signal demand so that a fast producer doesn't overwhelm a slower consumer.

### "Does reactive mean multiple threads?"

> No. Reactive programming is primarily about asynchronous, non-blocking data processing and flow control. It doesn't inherently mean one thread per operation.

### "Should I manually call `subscribe()` in my Spring controller?"

> Usually no. Return the `Flux` and let the reactive web framework manage subscription and the request lifecycle.

---

# 🧠 The one diagram to remember

```text
             LLM
              │
       streamed chunks
              │
              ▼
         Spring AI
              │
              ▼
         Flux<String>
              │
      ┌───────┴────────┐
      │ Reactive       │
      │ Pipeline       │
      │                │
      │ map()          │
      │ filter()       │
      │ doOnNext()     │
      │ doOnError()    │
      │ doOnComplete() │
      └───────┬────────┘
              │
              ▼
       Spring WebFlux
              │
       HTTP streaming
              │
              ▼
            React
```

If **Topic 17 taught you what streaming is**, Topic 18 teaches you **how Java/Spring handles that stream**.

And that sets you up perfectly for **Topic 19 — Spring Boot → React streaming**, where we'll take the `Flux<String>` and actually push those AI chunks into a browser in real time.
