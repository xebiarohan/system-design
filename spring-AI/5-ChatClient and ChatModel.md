
# Topic 5 & 6 — ChatModel and ChatClient

**Roadmap Phase:** Phase 2 — Chat Models  
**Estimated Time:** 5 hours  
**Prerequisite:** Topic 4 — Spring AI Core Abstractions

---

# 1. Learning Goal

By the end of these topics, you should understand:

- What `ChatModel` is
- Why Spring AI provides `ChatModel`
- What `Prompt` represents
- What `Message` represents
- What `ChatResponse` represents
- What `ChatClient` is
- Why `ChatClient` exists when `ChatModel` already exists
- How `ChatClient` communicates with `ChatModel`
- How `prompt()`, `system()`, `user()`, `call()`, `content()`, and `chatResponse()` work
- The difference between `ChatModel` and `ChatClient`
- How these abstractions fit into a Spring Boot application

> **Goal:** You should be able to explain the difference between `ChatModel` and `ChatClient`, and build a basic Spring Boot AI endpoint using `ChatClient`.

---

# 2. Where Do ChatModel and ChatClient Fit?

Before learning the individual APIs, understand the overall architecture.

```text
Spring Boot Application
        |
        v
    ChatClient
        |
        v
    ChatModel
        |
        v
 Provider Implementation
        |
        v
    AI Provider
        |
        v
       LLM
```

For example:

```text
Your Spring Boot Application
        |
        v
    ChatClient
        |
        v
    ChatModel
        |
        v
      OpenAI
        |
        v
   GPT Model
```

Or:

```text
Your Spring Boot Application
        |
        v
    ChatClient
        |
        v
    ChatModel
        |
        v
      Gemini
        |
        v
   Gemini Model
```

Your application can work primarily with Spring AI abstractions.

That means your business code does not need to know every detail of the underlying provider API.

---

# 3. What Problem Does ChatModel Solve?

Imagine writing your application directly against an AI provider SDK.

For example:

```java
OpenAiClient client;
```

Your application is now coupled to a specific provider.

The architecture becomes:

```text
Spring Boot
    |
    v
OpenAI SDK
    |
    v
OpenAI
```

If you later move to another provider:

```text
Spring Boot
    |
    v
Gemini SDK
    |
    v
Google Gemini
```

Your application code may need significant changes.

Spring AI introduces the `ChatModel` abstraction:

```text
Spring Boot
    |
    v
 ChatModel
    |
    v
Provider Implementation
    |
    v
AI Provider
```

Now your application depends on the abstraction rather than directly depending on a specific provider implementation.

This is the same general principle you already know from Spring:

```text
Application
     |
     v
  Interface
     |
     v
Implementation
```

For example:

```text
Service
   |
   v
Repository
   |
   v
JPA Implementation
```

Spring AI applies a similar idea to AI models.

---

# 4. What Is ChatModel?

`ChatModel` is a core Spring AI abstraction representing a chat-oriented AI model.

In simple terms:

> `ChatModel` provides the application-level interface for sending a prompt to a chat model and receiving a response.

The basic flow is:

```text
Prompt
   |
   v
ChatModel
   |
   v
AI Provider
   |
   v
LLM
   |
   v
ChatResponse
```

Conceptually:

```java
ChatResponse response = chatModel.call(prompt);
```

The important point is that your code is interacting with:

```text
ChatModel
```

rather than directly with:

```text
OpenAI SDK
Gemini SDK
Anthropic SDK
etc.
```

---

# 5. ChatModel Is an Abstraction

Do not think of:

```text
ChatModel
```

as meaning:

```text
OpenAI
```

or:

```text
Gemini
```

Instead, think:

```text
ChatModel
    |
    +---- OpenAI implementation
    |
    +---- Gemini implementation
    |
    +---- Anthropic implementation
    |
    +---- Other implementations
```

The actual implementation communicates with the selected provider.

Your application can therefore depend on the common Spring AI abstraction.

---

# 6. Basic ChatModel Flow

A simple request looks like this:

```text
Java Application
       |
       v
     Prompt
       |
       v
    ChatModel
       |
       v
  AI Provider
       |
       v
      LLM
       |
       v
 ChatResponse
       |
       v
Java Application
```

In code, conceptually:

```java
Prompt prompt = new Prompt(
        "Explain dependency injection."
);

ChatResponse response = chatModel.call(prompt);
```

There are two important objects here:

```text
Prompt
   |
   v
ChatModel
   |
   v
ChatResponse
```

---

# 7. What Is a Prompt?

A `Prompt` represents the input/request that is sent to a chat model.

The simplest conceptual example is:

```java
Prompt prompt = new Prompt(
        "Explain dependency injection."
);
```

But a prompt is more than just a plain `String`.

It can contain conversational messages.

For example:

```text
System Message
       +
User Message
       |
       v
     Prompt
       |
       v
    ChatModel
```

A prompt can therefore represent structured conversational input.

---

# 8. What Is a Message?

A `Message` represents an individual message in the conversation.

A typical chat interaction contains different roles.

For example:

```text
SYSTEM:
You are an experienced Java teacher.

USER:
Explain dependency injection.
```

Conceptually:

```text
Message
   |
   +---- System message
   |
   +---- User message
   |
   +---- Assistant message
   |
   +---- Tool-related messages
```

The exact APIs and available message types can vary with the Spring AI version, but the underlying concept remains the same.

---

# 9. System Message

A system message provides instructions about how the model should behave.

Example:

```text
You are an experienced Java teacher.
Explain concepts using simple examples.
```

Then the user asks:

```text
Explain dependency injection.
```

The model receives something conceptually like:

```text
SYSTEM:
You are an experienced Java teacher.
Explain concepts using simple examples.

USER:
Explain dependency injection.
```

The system message establishes behavior or context.

---

# 10. User Message

The user message represents the actual request from the user.

For example:

```text
Explain dependency injection.
```

Conceptually:

```text
System Message
      |
      v
"You are an experienced Java teacher."

User Message
      |
      v
"Explain dependency injection."
```

Both can become part of the prompt sent to the model.

---

# 11. Assistant Message

An assistant message represents a response generated by the AI assistant.

For example:

```text
USER:
Explain dependency injection.

ASSISTANT:
Dependency injection is a design pattern...
```

Assistant messages become particularly important when you start learning:

- Conversation history
- Chat memory
- Multi-turn conversations
- Tool calling
- Agent interactions

For now, understand the basic idea:

```text
User Message
     |
     v
    LLM
     |
     v
Assistant Message
```

---

# 12. What Is ChatResponse?

`ChatResponse` represents the response returned by the chat model.

Conceptually:

```text
ChatModel
    |
    v
ChatResponse
```

A response is not necessarily just a `String`.

It can contain:

```text
ChatResponse
    |
    +---- Generated content
    |
    +---- Generation information
    |
    +---- Response metadata
```

Depending on the model provider and Spring AI version, response information can include things such as:

- Generated text
- Model information
- Token usage
- Finish reason
- Generation metadata

This becomes important later when you study observability and cost tracking.

---

# 13. ChatResponse vs Generated Content

This distinction is important.

Suppose the model generates:

```text
Dependency injection is a design pattern...
```

You might only need:

```java
String answer;
```

But the complete response can contain additional information.

Conceptually:

```text
ChatResponse
    |
    +---- Generation
    |       |
    |       +---- Assistant message
    |       |
    |       +---- Generated content
    |
    +---- Metadata
```

Therefore:

```text
ChatResponse
```

represents the broader response.

The generated text is only one part of that response.

---

# 14. Using ChatModel Directly

A conceptual example:

```java
@Service
public class AiService {

    private final ChatModel chatModel;

    public AiService(ChatModel chatModel) {
        this.chatModel = chatModel;
    }

    public String ask(String question) {

        Prompt prompt = new Prompt(question);

        ChatResponse response = chatModel.call(prompt);

        // Extract generated text from the response.
        return response
                .getResult()
                .getOutput()
                .getText();
    }
}
```

The exact accessor methods can differ between Spring AI versions.

The important architecture is:

```text
Question
    |
    v
Prompt
    |
    v
ChatModel.call()
    |
    v
ChatResponse
    |
    v
Generated Content
```

---

# 15. Why Is ChatModel Considered a Lower-Level Abstraction?

With `ChatModel`, you work closer to the model interaction itself.

You are responsible for concepts such as:

```text
Prompt
Messages
ChatResponse
Model interaction
Response extraction
```

For example:

```java
Prompt prompt = new Prompt(...);

ChatResponse response = chatModel.call(prompt);
```

This is perfectly valid.

But as applications become more complex, you often want a more convenient API.

That's where `ChatClient` becomes important.

---

# 16. What Is ChatClient?

`ChatClient` is a higher-level, fluent API for interacting with a `ChatModel`.

The relationship can be represented as:

```text
Application
     |
     v
ChatClient
     |
     v
ChatModel
     |
     v
AI Provider
     |
     v
LLM
```

In simple terms:

> `ChatModel` provides the model abstraction, while `ChatClient` provides a convenient application-facing API for using that model.

---

# 17. Why Does ChatClient Exist?

Without `ChatClient`, you might write:

```java
Prompt prompt = ...;

ChatResponse response =
        chatModel.call(prompt);
```

That works.

But as your application grows, requests can involve:

```text
System instructions
User messages
Model options
Structured output
Advisors
Memory
RAG
Tools
Streaming
```

A fluent API makes these operations easier to express.

For example:

```java
String answer = chatClient
        .prompt()
        .user("Explain dependency injection.")
        .call()
        .content();
```

This is much easier to read.

---

# 18. Understanding the ChatClient Fluent API

The most important pattern is:

```java
chatClient
        .prompt()
        .user("...")
        .call()
        .content();
```

Do not memorize this as one large statement.

Break it into individual operations:

```text
chatClient
     |
     v
  prompt()
     |
     v
   user()
     |
     v
   call()
     |
     v
  content()
```

Each method has a different responsibility.

---

# 19. What Does `prompt()` Do?

`prompt()` starts constructing a chat request.

For example:

```java
chatClient.prompt()
```

Conceptually:

```text
ChatClient
     |
     v
"Start building a request."
```

You can then configure that request.

For example:

```java
chatClient
        .prompt()
        .user("Explain Spring Boot.");
```

The request has now been configured with a user message.

---

# 20. What Does `user()` Do?

`user()` specifies the user message.

Example:

```java
chatClient
        .prompt()
        .user("Explain dependency injection.");
```

Conceptually:

```text
ChatClient
     |
     v
   Prompt
     |
     v
 User Message
     |
     v
"Explain dependency injection."
```

You can think of:

```java
.user(...)
```

as:

> Add the user's request to this prompt.

---

# 21. What Does `system()` Do?

`system()` specifies system-level instructions.

For example:

```java
chatClient
        .prompt()
        .system("You are an experienced Java teacher.")
        .user("Explain dependency injection.");
```

Conceptually:

```text
SYSTEM:
You are an experienced Java teacher.

USER:
Explain dependency injection.
```

The model can use the system instructions when generating its response.

---

# 22. System + User Together

This is one of the most important examples for Topic 6.

```java
String answer = chatClient
        .prompt()
        .system("""
                You are an experienced Java teacher.
                Explain concepts clearly.
                Use examples where useful.
                """)
        .user("Explain dependency injection.")
        .call()
        .content();
```

The architecture is:

```text
                    ChatClient
                        |
                        v
                     prompt()
                        |
              +---------+---------+
              |                   |
              v                   v
          system()             user()
              |                   |
              v                   v
       System Message        User Message
              |                   |
              +---------+---------+
                        |
                        v
                      call()
                        |
                        v
                       LLM
                        |
                        v
                    Response
                        |
                        v
                    content()
```

---

# 23. What Does `call()` Do?

`call()` executes the request.

Before `call()`:

```java
chatClient
        .prompt()
        .user("Explain Spring Boot.");
```

You are constructing/configuring the request.

When you call:

```java
.call()
```

the request is executed.

Conceptually:

```text
Build request
      |
      v
    call()
      |
      v
Send request to model
      |
      v
Receive response
```

This distinction is important.

```text
prompt()
system()
user()
```

are primarily about constructing/configuring the request.

```text
call()
```

is where you execute the request.

---

# 24. What Does `content()` Do?

After executing the request, you often only want the generated text.

For example:

```java
String answer = chatClient
        .prompt()
        .user("Explain Spring Boot.")
        .call()
        .content();
```

The final:

```java
.content()
```

means:

> Give me the generated textual content from the response.

Think of:

```text
call()
   |
   v
Response
   |
   v
content()
   |
   v
String
```

Therefore:

```java
.call().content()
```

can be mentally read as:

```text
Execute the request
        |
        v
Get the response
        |
        v
Extract the generated text
```

---

# 25. What Does `chatResponse()` Do?

Sometimes you don't want only the generated text.

You want the complete `ChatResponse`.

For example:

```java
ChatResponse response = chatClient
        .prompt()
        .user("Explain Spring Boot.")
        .call()
        .chatResponse();
```

Conceptually:

```text
call()
   |
   v
ChatResponse
   |
   +---- Content
   |
   +---- Metadata
   |
   +---- Generation information
```

Use `chatResponse()` when you need access to the broader response rather than only the generated String.

---

# 26. `content()` vs `chatResponse()`

This is an important distinction.

### `content()`

Use when you mainly want the generated text:

```java
String answer = chatClient
        .prompt()
        .user("What is Spring Boot?")
        .call()
        .content();
```

Result:

```text
String
```

---

### `chatResponse()`

Use when you want the complete response:

```java
ChatResponse response = chatClient
        .prompt()
        .user("What is Spring Boot?")
        .call()
        .chatResponse();
```

Result:

```text
ChatResponse
```

So remember:

```text
content()
    |
    v
String

chatResponse()
    |
    v
ChatResponse
```

---

# 27. ChatModel vs ChatClient

This is the most important comparison in these two topics.

| Concept | `ChatModel` | `ChatClient` |
|---|---|---|
| Abstraction level | Lower-level | Higher-level |
| Main purpose | Model interaction abstraction | Convenient chat API |
| API style | Model-oriented | Fluent |
| Typical usage | `chatModel.call(...)` | `chatClient.prompt()...` |
| Request construction | More explicit | Fluent |
| Response | `ChatResponse` | Can expose content or `ChatResponse` |
| Application convenience | Lower | Higher |
| Important for Spring AI internals | Yes | Yes |
| Typical application code | Less common | Very common |

The key relationship is:

```text
ChatClient
    |
    v
ChatModel
    |
    v
AI Provider
```

Do not think:

```text
ChatClient replaces ChatModel
```

Instead think:

```text
ChatClient uses ChatModel
```

---

# 28. The Most Important Mental Model

Memorize this architecture:

```text
Your Application
       |
       v
  ChatClient
       |
       v
   ChatModel
       |
       v
Provider Implementation
       |
       v
   AI Provider
       |
       v
      LLM
       |
       v
   Response
       |
       v
   ChatModel
       |
       v
  ChatClient
       |
       v
Your Application
```

And inside the request:

```text
Prompt
   |
   +---- System Message
   |
   +---- User Message
   |
   +---- Other messages/context
```

---

# 29. How ChatClient Uses ChatModel

Suppose you write:

```java
String answer = chatClient
        .prompt()
        .system("You are a Java expert.")
        .user("Explain interfaces.")
        .call()
        .content();
```

Conceptually, the process is:

```text
1. ChatClient receives request
        |
        v
2. prompt() creates request builder
        |
        v
3. system() adds system message
        |
        v
4. user() adds user message
        |
        v
5. call() executes request
        |
        v
6. ChatClient delegates model interaction
        |
        v
7. ChatModel communicates with provider
        |
        v
8. Provider calls LLM
        |
        v
9. LLM generates response
        |
        v
10. ChatModel represents the response
        |
        v
11. ChatClient exposes the result
        |
        v
12. content() extracts generated text
```

This is the architecture you should understand rather than memorizing individual method chains.

---

# 30. Creating a ChatClient

A common pattern is to create a `ChatClient` using the configured `ChatModel`.

Conceptually:

```java
@Bean
ChatClient chatClient(ChatModel chatModel) {
    return ChatClient.create(chatModel);
}
```

Then inject it into your service:

```java
@Service
public class AiService {

    private final ChatClient chatClient;

    public AiService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String ask(String question) {

        return chatClient
                .prompt()
                .user(question)
                .call()
                .content();
    }
}
```

The important dependency relationship is:

```text
ChatModel
    |
    v
ChatClient
    |
    v
AiService
```

The exact configuration can depend on your Spring AI version and selected model provider.

---

# 31. A Simple Chat Service

A practical service could look like this:

```java
@Service
public class ChatService {

    private final ChatClient chatClient;

    public ChatService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String ask(String question) {

        return chatClient
                .prompt()
                .user(question)
                .call()
                .content();
    }
}
```

The request flow is:

```text
question
    |
    v
ChatService
    |
    v
ChatClient
    |
    v
ChatModel
    |
    v
LLM
    |
    v
String response
```

This is the basic pattern you will use repeatedly throughout the roadmap.

---

# 32. Adding a System Instruction

Now improve the service:

```java
@Service
public class ChatService {

    private final ChatClient chatClient;

    public ChatService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String ask(String question) {

        return chatClient
                .prompt()
                .system("""
                        You are an experienced Java developer.
                        Explain concepts clearly and provide examples.
                        """)
                .user(question)
                .call()
                .content();
    }
}
```

Now the request contains:

```text
SYSTEM:
You are an experienced Java developer.
Explain concepts clearly and provide examples.

USER:
<question>
```

This is the foundation for the next roadmap topic:

```text
Topic 7 — System Prompts
```

---

# 33. Building a REST API

Now connect `ChatClient` to Spring MVC.

Suppose your API is:

```text
POST /api/chat
```

Request:

```json
{
  "message": "Explain dependency injection"
}
```

Architecture:

```text
Client
   |
   | POST /api/chat
   v
Controller
   |
   v
ChatService
   |
   v
ChatClient
   |
   v
ChatModel
   |
   v
LLM
   |
   v
Response
```

For example:

```java
@RestController
@RequestMapping("/api/chat")
public class ChatController {

    private final ChatClient chatClient;

    public ChatController(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    @PostMapping
    public String chat(@RequestBody ChatRequest request) {

        return chatClient
                .prompt()
                .user(request.message())
                .call()
                .content();
    }
}
```

This is enough to build a basic AI endpoint.

---

# 34. What Happens During a REST Request?

Suppose the client sends:

```json
{
  "message": "What is dependency injection?"
}
```

The complete flow is:

```text
HTTP Request
     |
     v
ChatController
     |
     v
ChatClient
     |
     v
Prompt
     |
     v
ChatModel
     |
     v
AI Provider
     |
     v
LLM
     |
     v
ChatResponse
     |
     v
Generated Content
     |
     v
HTTP Response
```

This is the first real Spring AI application architecture you should be comfortable with.

---

# 35. ChatClient Is More Than a String Wrapper

At first, this may look like:

```java
String input
      |
      v
ChatClient
      |
      v
String output
```

But that is too simplistic.

`ChatClient` becomes important because the request can eventually include:

```text
System instructions
        +
User messages
        +
Model configuration
        +
Advisors
        +
Memory
        +
RAG
        +
Tools
        +
Structured output
        +
Streaming
```

That is why `ChatClient` becomes one of the central APIs in your Spring AI learning path.

---

# 36. How Topic 6 Connects to Later Topics

Your roadmap is structured correctly.

You first learn:

```text
ChatModel
    |
    v
ChatClient
```

Then:

```text
System Prompts
```

Then:

```text
Conversation History
```

Then:

```text
Prompt Engineering
```

Then:

```text
Structured Output
```

Then:

```text
Streaming
```

Then:

```text
RAG
```

Then:

```text
Advisors
```

Then:

```text
Memory
```

Then:

```text
Tools
```

A large part of these later features can be understood as capabilities added around the basic chat request.

---

# 37. ChatClient and Streaming

Your roadmap later contains a complete Streaming phase.

For a normal request:

```java
String response = chatClient
        .prompt()
        .user("Explain Spring Boot.")
        .call()
        .content();
```

The conceptual flow is:

```text
Request
   |
   v
LLM
   |
   v
Complete Response
   |
   v
Application
```

With streaming, the conceptual flow becomes:

```text
Request
   |
   v
LLM
   |
   +---- Chunk
   |
   +---- Chunk
   |
   +---- Chunk
   |
   +---- Chunk
   |
   v
Application
```

For example, conceptually:

```text
"Spring"
" Boot"
" is"
" a"
" framework..."
```

The exact streaming API and return types will be covered in your later Streaming phase.

For now, remember:

```text
call()
    |
    v
Complete response

stream()
    |
    v
Incremental response
```

---

# 38. Why Learn ChatModel Before ChatClient?

Your roadmap has:

```text
5. ChatModel
6. ChatClient
```

This order is useful.

If you only learn:

```java
chatClient
        .prompt()
        .user(...)
        .call()
        .content();
```

you can use Spring AI, but you may not understand what happens underneath.

You might incorrectly think:

```text
ChatClient = AI Model
```

Instead:

```text
ChatClient
     |
     v
ChatModel
     |
     v
Provider
     |
     v
LLM
```

Once you understand this relationship, `ChatClient` becomes much easier to understand.

---

# 39. When Would You Use ChatModel Directly?

Most normal application code will usually prefer the higher-level `ChatClient`.

However, `ChatModel` remains important for:

- Understanding Spring AI internals
- Provider integrations
- Custom implementations
- Framework extensions
- Testing
- Advanced customization

This becomes especially relevant later in your roadmap:

```text
Phase 20 — Advanced Spring AI

70. Spring AI internals
71. Custom ChatModel
```

If you understand `ChatModel` now, those topics will be significantly easier later.

---

# 40. When Would You Use ChatClient?

For normal Spring Boot application development, think:

```text
Controller
     |
     v
Service
     |
     v
ChatClient
     |
     v
AI Model
```

For example:

```java
@Service
public class CustomerAssistantService {

    private final ChatClient chatClient;

    public CustomerAssistantService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String answer(String question) {

        return chatClient
                .prompt()
                .system("""
                        You are a customer support assistant.
                        Answer clearly and concisely.
                        """)
                .user(question)
                .call()
                .content();
    }
}
```

This is much closer to the code you will write in real Spring AI applications.

---

# 41. Important Terminology

Make sure these terms don't get mixed up.

| Term | Meaning |
|---|---|
| `ChatModel` | Core abstraction for interacting with a chat model |
| `ChatClient` | Higher-level fluent API for using a `ChatModel` |
| `Prompt` | Structured request sent to a chat model |
| `Message` | Individual conversational message |
| System message | Instructions/context for model behavior |
| User message | User's request |
| Assistant message | Model-generated conversational message |
| `ChatResponse` | Model response abstraction |
| `content()` | Extracts generated textual content |
| `chatResponse()` | Returns the broader `ChatResponse` |

---

# 42. Common Beginner Mistakes

## Mistake 1 — Thinking ChatClient Is the Model

Incorrect mental model:

```text
ChatClient = GPT
```

Correct:

```text
ChatClient
     |
     v
ChatModel
     |
     v
Provider
     |
     v
LLM
```

---

## Mistake 2 — Thinking ChatModel and ChatClient Are Alternatives

Don't think:

```text
ChatModel OR ChatClient
```

Think:

```text
ChatClient
     |
     v
ChatModel
```

They operate at different abstraction levels.

---

## Mistake 3 — Thinking Prompt Means Only String

A prompt can represent structured conversational input.

Conceptually:

```text
Prompt
   |
   +---- System Message
   |
   +---- User Message
   |
   +---- Other messages
```

---

## Mistake 4 — Treating ChatResponse as Just a String

A `ChatResponse` can contain more than generated text.

Think:

```text
ChatResponse
   |
   +---- Generated content
   |
   +---- Metadata
   |
   +---- Generation information
```

---

## Mistake 5 — Memorizing the Fluent Chain

Don't just memorize:

```java
.prompt()
.user()
.call()
.content()
```

Understand:

```text
prompt()
    ↓
Build request

user()
    ↓
Add user message

system()
    ↓
Add system instruction

call()
    ↓
Execute request

content()
    ↓
Extract generated text
```

---

# 43. Hands-On Exercise 1 — Direct ChatModel

Create a simple service using `ChatModel`.

Your goal is:

```text
Input
  |
  v
Prompt
  |
  v
ChatModel
  |
  v
ChatResponse
  |
  v
Output
```

Practice understanding:

```java
Prompt prompt = ...;

ChatResponse response = chatModel.call(prompt);
```

Do not move on until you understand what each object represents.

---

# 44. Hands-On Exercise 2 — Basic ChatClient

Now implement:

```java
String response = chatClient
        .prompt()
        .user("Explain dependency injection.")
        .call()
        .content();
```

You should be able to explain every part:

```text
chatClient
    ↓
prompt()
    ↓
user()
    ↓
call()
    ↓
content()
```

---

# 45. Hands-On Exercise 3 — System + User

Build:

```java
String response = chatClient
        .prompt()
        .system("""
                You are an expert Java teacher.
                Explain concepts using simple examples.
                """)
        .user("Explain interfaces.")
        .call()
        .content();
```

Then experiment with different system instructions.

For example:

```text
You are a beginner-friendly Java teacher.
```

Then:

```text
You are a senior Java architect.
```

Observe how the model's response changes.

This prepares you for:

```text
Topic 7 — System Prompts
```

---

# 46. Hands-On Exercise 4 — Inspect ChatResponse

Instead of:

```java
.content();
```

use:

```java
.chatResponse();
```

Your goal is to inspect the response and understand what information is available.

Conceptually:

```text
ChatResponse
   |
   +---- Generated text
   |
   +---- Metadata
   |
   +---- Model information
   |
   +---- Usage information
```

This will become useful later when you learn:

```text
Observability
Token usage
Cost
Tracing
Evaluation
```

---

# 47. Hands-On Exercise 5 — Build a Chat REST API

Create:

```text
POST /api/chat
```

Request:

```json
{
  "message": "Explain dependency injection"
}
```

Response:

```text
Dependency injection is...
```

Architecture:

```text
Client
   |
   v
POST /api/chat
   |
   v
ChatController
   |
   v
ChatService
   |
   v
ChatClient
   |
   v
ChatModel
   |
   v
LLM
```

This should be your first small Spring AI project.

---

# 48. Recommended 5-Hour Study Plan

Your roadmap gives:

```text
ChatModel  → 2 hours
ChatClient → 3 hours
```

A good approach is:

## Hour 1 — ChatModel Concepts

Learn:

```text
ChatModel
Prompt
Message
SystemMessage
UserMessage
AssistantMessage
ChatResponse
```

Draw:

```text
Prompt
   |
   v
ChatModel
   |
   v
ChatResponse
```

Then explain the diagram without looking at your notes.

---

## Hour 2 — ChatModel Coding

Build a small service using:

```java
Prompt
ChatModel.call()
ChatResponse
```

Inspect the returned response.

The goal is not to build a production application.

The goal is to understand the abstraction.

---

## Hour 3 — ChatClient Fundamentals

Practice:

```java
chatClient
        .prompt()
        .user(...)
        .call()
        .content();
```

Then:

```java
chatClient
        .prompt()
        .system(...)
        .user(...)
        .call()
        .content();
```

Understand every method.

---

## Hour 4 — ChatResponse and API

Practice:

```java
.call()
.content()
```

versus:

```java
.call()
.chatResponse()
```

Then build a simple REST endpoint.

---

## Hour 5 — Mini Project

Build:

```text
POST /api/chat
```

Architecture:

```text
HTTP
  |
  v
Controller
  |
  v
ChatClient
  |
  v
ChatModel
  |
  v
LLM
  |
  v
Response
```

At the end of the five hours, you should be able to build a basic AI endpoint without copying a tutorial line by line.

---

# 49. Interview-Level Understanding

If someone asks:

> What is `ChatModel` in Spring AI?

A good answer is:

> `ChatModel` is a core Spring AI abstraction for interacting with chat-oriented AI models. It provides a common interface for sending prompts and receiving model responses while hiding provider-specific implementation details.

If someone asks:

> What is `ChatClient`?

A good answer is:

> `ChatClient` is a higher-level fluent API for interacting with a `ChatModel`. It simplifies constructing chat requests and working with features such as system messages, user messages, responses, streaming, and other Spring AI capabilities.

If someone asks:

> What is the difference between ChatModel and ChatClient?

A strong answer is:

> `ChatModel` is the underlying model abstraction, while `ChatClient` is a higher-level application-facing API that uses the `ChatModel` to make chat interactions easier to construct and execute.

---

# 50. The Most Important Diagram

Keep this diagram in your notes:

```text
                    YOUR APPLICATION
                           |
                           v
                      ChatClient
                           |
                           v
                       ChatModel
                           |
                           v
                  Provider Implementation
                           |
                           v
                      AI Provider
                           |
                           v
                           LLM
                           |
                           v
                      ChatResponse
                           |
                           v
                      ChatClient
                           |
                           v
                    YOUR APPLICATION
```

Inside the request:

```text
                    Prompt
                      |
          +-----------+-----------+
          |                       |
          v                       v
   System Message           User Message
          |                       |
          +-----------+-----------+
                      |
                      v
                  ChatModel
```

---

# 51. Final Cheat Sheet

## ChatModel

```text
Core Spring AI abstraction
        |
        v
Interact with chat model
        |
        v
ChatResponse
```

---

## Prompt

```text
Represents the request
        |
        +---- System message
        |
        +---- User message
        |
        +---- Other conversational information
```

---

## Message

```text
Individual conversational message
        |
        +---- System
        +---- User
        +---- Assistant
        +---- Tool-related
```

---

## ChatResponse

```text
Model response
        |
        +---- Generated content
        +---- Metadata
        +---- Generation information
```

---

## ChatClient

```text
Higher-level fluent API
        |
        v
ChatModel
        |
        v
AI Provider
        |
        v
LLM
```

---

## Common ChatClient Flow

```java
chatClient
        .prompt()
        .system("You are a helpful Java teacher.")
        .user("Explain dependency injection.")
        .call()
        .content();
```

Read it as:

```text
Create request
      |
      v
Add system instructions
      |
      v
Add user request
      |
      v
Execute request
      |
      v
Extract generated text
```

---

# 52. Final Mental Model

If you remember only one thing from Topics 5 and 6, remember this:

```text
                 ChatClient
                     |
              builds a request
                     |
                     v
                  Prompt
                     |
          +----------+----------+
          |                     |
          v                     v
   System Message         User Message
          |                     |
          +----------+----------+
                     |
                     v
                 ChatModel
                     |
                     v
              AI Provider
                     |
                     v
                    LLM
                     |
                     v
               ChatResponse
                     |
             +-------+-------+
             |               |
             v               v
         content()      chatResponse()
             |               |
             v               v
           String       ChatResponse
```

The fundamental relationship is:

```text
ChatClient
    |
    | uses
    v
ChatModel
    |
    | communicates with
    v
AI Provider
    |
    v
LLM
```

And the fundamental request is:

```text
System instructions
        +
User request
        |
        v
      Prompt
        |
        v
    ChatModel
        |
        v
   ChatResponse
```

> **If you understand this architecture, you have the foundation for the next topics: System Prompts, Conversation History, Prompt Templates, Structured Output, Streaming, Advisors, Memory, RAG, and Tool Calling.**
