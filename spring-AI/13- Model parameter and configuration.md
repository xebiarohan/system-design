
# Model Parameters & Configuration


This phase is about understanding **how to control LLM behavior** and how to configure Spring AI applications properly.

At this point you already know:

```text
Prompt
   ↓
ChatClient
   ↓
LLM
   ↓
Response
```

Now you need to understand what happens when you start changing the **model configuration**.

For example:

```text
Same Prompt
     ↓
Different Parameters
     ↓
Different Output
```

The goal is not to memorize every available parameter.

The goal is to understand:

> **Which parameter should I change, why should I change it, and at what configuration level should I change it?**

---

### 15. Model configuration

**⏱️ 2–2.5 hours**

Learn:

* Temperature
* Max output tokens
* Top-p
* Stop sequences
* Model selection
* Request-level configuration
* Application-level configuration

---

### 15.1 Temperature

Temperature controls the **randomness / variability** of the model's output.

Think of it roughly as:

```text
Low temperature
      ↓
More predictable output
      ↓
More consistent responses
```

versus:

```text
High temperature
      ↓
More variation
      ↓
More creative / unpredictable responses
```

For example:

```text
temperature = 0.0
```

might produce:

```text
Java is an object-oriented programming language.
```

while a higher temperature could produce a more varied explanation:

```text
Java is a versatile object-oriented language widely used
for building backend services, enterprise applications,
and large-scale systems.
```

The exact behavior depends on the model.

### Important

Do **not** think:

```text
temperature = 0 → deterministic
temperature = 1 → random
```

Temperature influences probability distribution; it does not guarantee deterministic behavior.

Different models may also expose different supported ranges or behavior.

### When to use lower temperature

Good for:

* Structured output
* Data extraction
* Classification
* Summarization
* Code generation where consistency matters
* Business workflows

Example:

```text
Invoice
   ↓
LLM
   ↓
Structured JSON
```

You generally don't want the model being unnecessarily creative.

### When to use higher temperature

Useful when you want:

* Creative writing
* Brainstorming
* Multiple ideas
* Story generation
* Marketing copy

Think:

```text
Low temperature → consistency
High temperature → variation
```

---

### 15.2 Max output tokens

This controls the maximum amount of output the model can generate.

For example:

```text
max output tokens = 100
```

means the model cannot generate an arbitrarily long response.

Think:

```text
Prompt
  ↓
LLM
  ↓
Maximum output limit
  ↓
Response
```

This is useful for controlling:

* Response size
* Cost
* Latency
* Unexpectedly long responses

### Important distinction

Don't confuse:

```text
Context window
```

with:

```text
Maximum output tokens
```

The context window represents the total amount of information the model can work with.

Conceptually:

```text
Input tokens
+
Output tokens
+
Other model context
-------------------
Context window
```

While max output tokens primarily limits:

```text
How much the model can generate
```

The exact accounting depends on the model/provider.

---

### 15.3 Top-p

Top-p is another parameter that controls how the model chooses tokens.

It is commonly called:

> **Nucleus sampling**

Instead of considering every possible next token, the model considers tokens whose cumulative probability reaches a specified threshold.

Conceptually:

```text
Possible tokens

A → 0.50
B → 0.25
C → 0.15
D → 0.05
E → 0.05
```

If:

```text
top-p = 0.75
```

the model may consider:

```text
A + B = 0.75
```

and sample from that probability mass.

### Temperature vs Top-p

Both influence generation behavior, but they work differently.

Think:

```text
Temperature
    ↓
Changes probability distribution

Top-p
    ↓
Limits the probability mass considered
```

In practice:

> **Usually tune temperature or top-p rather than aggressively changing both at the same time.**

Different model providers may recommend different defaults.

---

### 15.4 Stop sequences

A stop sequence tells the model:

> Stop generating when this sequence is reached.

For example:

```text
stop = "END"
```

If the model generates:

```text
Here is the answer.

END

Additional information...
```

generation can stop when the stop sequence is encountered.

This can be useful for:

* Structured text
* Delimited output
* Special application protocols
* Controlling generated content

However, stop-sequence support and behavior can vary by model/provider.

Don't assume every model supports every parameter identically.

---

### 15.5 Model selection

Spring AI provides abstractions so your application can work with different model providers.

Conceptually:

```text
Spring AI
    ↓
ChatModel
    ↓
┌───────────────┐
│ OpenAI        │
│ Anthropic     │
│ Google        │
│ Azure         │
│ Other models  │
└───────────────┘
```

The important concept is:

> Your application should depend on Spring AI abstractions rather than tightly coupling business logic to one model provider.

For example:

```java
ChatClient
    ↓
ChatModel
    ↓
LLM Provider
```

This gives you more flexibility when changing models.

---

### 15.6 Model vs Provider

Understand the difference between:

```text
Provider
```

and:

```text
Model
```

For example:

```text
Provider
   ↓
OpenAI
   ↓
Model
   ↓
Specific model name
```

A provider may expose multiple models with different characteristics.

Different models can vary in:

* Cost
* Latency
* Context window
* Reasoning capability
* Multimodal capabilities
* Output quality
* Tool-calling support
* Structured-output support

Therefore:

```text
Choosing an LLM
```

is an architectural decision, not simply a configuration detail.

---

### 15.7 Request-level configuration

Sometimes you want different parameters for different requests.

For example:

```text
Normal application request
        ↓
Low temperature
```

but:

```text
Creative writing request
        ↓
Higher temperature
```

Conceptually:

```text
Application defaults
        ↓
Request
        ↓
Request-specific configuration
        ↓
LLM
```

For example, you may configure a default model at the application level and customize certain properties when building a particular request.

The exact API depends on the Spring AI version and model implementation you're using.

The important concept is:

> **Defaults should live in configuration; special behavior can be configured at request time.**

---

### 15.8 Application-level configuration

Application-level configuration is useful when you want consistent defaults across your application.

For example:

```text
Spring Boot configuration
        ↓
Chat model configuration
        ↓
ChatClient
        ↓
Requests
```

Typical configuration can include:

* Model
* API credentials
* Default temperature
* Token limits
* Provider-specific options

For example, conceptually:

```yaml
spring:
  ai:
    <provider>:
      chat:
        options:
          temperature: 0.2
```

The exact property names depend on the Spring AI provider and version.

### Why application-level configuration matters

Imagine you have:

```text
100 ChatClient calls
```

You don't want:

```java
temperature(0.2)
temperature(0.2)
temperature(0.2)
temperature(0.2)
...
```

repeated throughout the codebase.

Instead:

```text
Central configuration
        ↓
Application defaults
        ↓
Individual requests
```

This makes your application easier to:

* Maintain
* Change
* Test
* Deploy
* Tune

---

### 15.9 Configuration hierarchy

One of the most important concepts in this phase is understanding **where configuration is applied**.

Think about it as:

```text
Application-level defaults
          ↓
Model-level configuration
          ↓
Request-level configuration
          ↓
Provider / model behavior
```

For example:

```text
Default temperature = 0.2

Request A
    ↓
uses 0.2

Request B
    ↓
overrides with 0.7
```

This allows you to have:

```text
Safe application defaults
+
Request-specific customization
```

You should understand this concept before moving into production architecture.

---

### 🛠️ Hands-on exercise

Build a small endpoint:

```text
POST /ai/generate
```

Request:

```json
{
  "prompt": "Explain dependency injection",
  "temperature": 0.2
}
```

Experiment with:

```text
temperature = 0.0
temperature = 0.2
temperature = 0.7
temperature = 1.0
```

Ask the same question multiple times.

Observe:

```text
Temperature
     ↓
Output variation
```

Then experiment with:

```text
max output tokens
top-p
stop sequences
```

The goal is to **observe the behavior**, not memorize definitions.

---

### 🛠️ Configuration exercise

Create different application configurations.

For example:

```text
Development
     ↓
More verbose / experimental

Production
     ↓
More controlled / predictable
```

Understand how Spring Boot configuration can be separated from Java business logic.

For example:

```text
application.yml
application-dev.yml
application-prod.yml
```

The exact structure depends on your Spring Boot setup.

---

### 16. Reliability configuration

**⏱️ 1–1.5 hours**

Learn:

* Timeouts
* Retries
* Error handling

This section is extremely important because an LLM call is an **external network dependency**.

Think of:

```text
Spring Boot
    ↓
Spring AI
    ↓
Internet
    ↓
LLM Provider
```

That means the request can fail because of:

* Network problems
* Provider outages
* Rate limits
* Authentication problems
* Invalid requests
* Server errors
* Timeouts
* Model availability

Therefore:

> **Never treat an LLM call like a normal in-memory Java method.**

---

### 16.1 Timeouts

A timeout defines how long your application is willing to wait for a response.

Without appropriate timeouts:

```text
Request
   ↓
LLM
   ↓
Waiting...
   ↓
Waiting...
   ↓
Waiting...
   ↓
Application resources consumed
```

With a timeout:

```text
Request
   ↓
LLM
   ↓
Timeout
   ↓
Failure handled
```

Timeouts are important for:

* Preventing requests from hanging
* Protecting application threads/resources
* Improving user experience
* Preventing cascading failures

Think about at least two different concerns:

```text
Connection timeout
```

and:

```text
Response/read timeout
```

The exact configuration depends on the HTTP client and Spring AI/provider integration being used.

---

### 16.2 Retries

Sometimes an LLM request fails temporarily.

For example:

```text
Request
   ↓
LLM Provider
   ↓
Temporary failure
```

A retry can help:

```text
Request
   ↓
Failure
   ↓
Retry
   ↓
Success
```

Retries are useful for transient failures such as:

* Temporary network problems
* Some server-side failures
* Temporary rate limiting, when appropriate

But:

> **Don't retry every error.**

For example:

```text
Invalid API key
```

should not normally be retried.

Neither should:

```text
Invalid request
```

because the same request will likely fail again.

---

### 16.3 Exponential backoff

Don't do this:

```text
Retry immediately
Retry immediately
Retry immediately
Retry immediately
```

Instead use:

```text
Attempt 1
   ↓
wait

Attempt 2
   ↓
wait longer

Attempt 3
   ↓
wait even longer
```

This is called:

> **Exponential backoff**

Conceptually:

```text
1s
 ↓
2s
 ↓
4s
 ↓
8s
```

Usually you also add **jitter** so many clients don't retry at exactly the same time.

---

### 16.4 Retry limits

Never create:

```text
Infinite retry
```

Use:

```text
Maximum attempts
```

For example:

```text
Request
 ↓
Attempt 1
 ↓
Failure
 ↓
Attempt 2
 ↓
Failure
 ↓
Attempt 3
 ↓
Failure
 ↓
Return error
```

The exact number should be based on the operation and business requirements.

---

### 16.5 Error handling

Your application should distinguish between different types of failures.

Conceptually:

```text
LLM Request
     ↓
   Error
     ↓
┌────┼─────────────┐
↓    ↓             ↓
4xx  429           5xx
↓    ↓             ↓
Bad  Rate limit    Server
request            error
```

You should understand how your application responds to each category.

For example:

```text
Invalid request
     ↓
Don't retry
     ↓
Return useful error
```

while:

```text
Temporary server error
     ↓
Retry
     ↓
If still failing
     ↓
Return fallback/error
```

---

### 16.6 Production mindset

Think of your LLM provider as another external dependency:

```text
Database
     ↓
External dependency

Redis
     ↓
External dependency

LLM
     ↓
External dependency
```

Therefore, the same backend engineering principles apply:

* Timeouts
* Retries
* Backoff
* Rate limiting
* Error handling
* Logging
* Monitoring
* Circuit breakers
* Fallbacks

You will revisit these concepts in **Phase 17 — Observability** and **Phase 19 — Production Architecture**.

---

### 🛠️ Mini exercise

Modify your AI application so that you can simulate:

```text
Successful request
```

```text
Timeout
```

```text
Temporary failure
```

```text
Rate limit
```

Then design:

```text
LLM Request
     ↓
   Error?
     ↓
 ┌───┴────┐
 No       Yes
 ↓         ↓
Return   Classify
answer    error
            ↓
       Retryable?
        /      \
      Yes       No
       ↓         ↓
    Retry      Return
    + backoff   error
```

---

# What you should know after Phase 5

You should be able to explain:

### Temperature

```text
Controls output variability
```

### Max output tokens

```text
Limits generated output
```

### Top-p

```text
Controls the probability mass considered during sampling
```

### Stop sequences

```text
Tell the model when generation should stop
```

### Model selection

```text
Choose the appropriate model for the workload
```

### Request-level configuration

```text
Customize behavior for a specific request
```

### Application-level configuration

```text
Define reusable application defaults
```

### Timeouts

```text
Prevent requests from waiting indefinitely
```

### Retries

```text
Recover from appropriate transient failures
```

### Backoff

```text
Avoid retry storms
```

### Error handling

```text
Classify failures and respond appropriately
```

---

# Phase 5 mental model

The most important thing to take away is this:

```text
                    Spring Boot
                        ↓
                  Spring AI
                        ↓
                 ChatClient
                        ↓
              ┌─────────────────┐
              │ Model Config    │
              │                 │
              │ Temperature     │
              │ Max tokens      │
              │ Top-p            │
              │ Stop sequences  │
              │ Model           │
              └─────────────────┘
                        ↓
                      LLM
                        ↓
                    Response
```

And for reliability:

```text
                    ChatClient
                        ↓
                    LLM Request
                        ↓
              ┌──────────────────┐
              │ Timeout          │
              │ Retry            │
              │ Backoff          │
              │ Error handling   │
              └──────────────────┘
                        ↓
                    LLM Provider
```

The key mindset is:

> **Model parameters control how the LLM behaves. Configuration controls how your application manages those parameters. Reliability configuration controls what happens when the LLM doesn't behave as expected.**

Once you understand this, you're ready for **Phase 6 — Streaming**, where you'll learn how the response can be delivered incrementally instead of waiting for the complete answer.

This keeps the roadmap's original hierarchy and numbering, while adding enough hands-on work to make the **3–4 hour allocation realistic**.
