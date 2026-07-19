# Trace ID, Span ID & Context Propagation

Let's start with a simple example.

A user opens your app and clicks **"Create Todo"**.

The request flows like this:

```text
Browser
   │
   ▼
API Gateway
   │
   ▼
Auth Service
   │
   ▼
Todo Service
   │
   ▼
Database
```

Imagine each service is running on a different machine.

None of them share memory.

So how does the Todo Service know:

> "I'm handling the same request that started at the API Gateway."

The answer is **context propagation**.

---

# Step 1: Trace ID

Every new request gets a unique identifier called a **Trace ID**.

For example:

```text
Trace ID

7f3b2d91a84f6c12
```

Think of it like a parcel tracking number.

When Amazon ships your package, it receives a tracking ID.

Every warehouse handling that package uses the same tracking ID.

Similarly,

When a request enters your system:

```text
Browser
   │
   ▼
API Gateway
```

the API Gateway creates

```text
Trace ID

7f3b2d91a84f6c12
```

Every service involved in handling that request uses this same Trace ID.

---

## Why is this important?

Suppose your system processes 10,000 requests every second.

Without a Trace ID, all spans from all users would be mixed together.

With a Trace ID:

```text
Request A

Trace ID

AAA111
```

```text
Request B

Trace ID

BBB222
```

```text
Request C

Trace ID

CCC333
```

Jaeger can group spans correctly because they all share the same Trace ID.

---

# Step 2: Span ID

Every span also has its own identifier.

Example:

```text
Trace ID

AAA111
```

Inside it

```text
API Gateway

Span ID

001
```

```text
Auth Service

Span ID

002
```

```text
Todo Service

Span ID

003
```

Notice:

The Trace ID stays the same.

The Span ID changes for every span.

---

Think of it like this:

**Trace ID** → identifies the entire movie.

**Span ID** → identifies each scene in the movie.

---

# Step 3: Parent Span ID

Now suppose:

```text
API Gateway

↓

Todo Service

↓

Database
```

The Todo Service span was created because the API Gateway called it.

So the Todo Service stores:

```text
Parent Span ID

001
```

Meaning:

"My parent is Span 001."

Then the Database span stores

```text
Parent Span ID

002
```

Now Jaeger can reconstruct the hierarchy.

```text
Span 001
API Gateway

│

├── Span 002
│    Todo Service
│
│     └── Span 003
│          Database
```

This is why the UI displays traces as a tree.

---

# Step 4: Context

Now comes the most important idea.

A **context** is simply the tracing information that travels with the request.

It usually contains things like:

* Trace ID
* Span ID
* Trace flags (used for decisions like sampling)

Whenever one service calls another, it sends this context along with the request.

---

# Step 5: Context Propagation

Let's walk through it.

The browser sends

```text
POST /todos
```

to the API Gateway.

The API Gateway creates

```text
Trace ID

AAA111
```

Span ID

```text
001
```

Now it calls the Auth Service.

Instead of sending only

```text
POST /validate
```

it also sends the tracing context.

Conceptually:

```text
POST /validate

Trace ID

AAA111

Parent Span

001
```

The Auth Service receives this information and knows:

"I should continue this trace, not start a new one."

It creates:

```text
Trace ID

AAA111

Span ID

002
```

Notice:

The Trace ID stays the same.

Only the Span ID changes.

---

The Auth Service then calls the Todo Service.

Again, it forwards the context.

```text
Trace ID

AAA111

Parent Span

002
```

The Todo Service creates

```text
Span ID

003
```

Again,

Same Trace ID.

New Span ID.

---

# Without Context Propagation

Imagine every service created its own Trace ID.

```text
API Gateway

Trace A
```

```text
Auth Service

Trace B
```

```text
Todo Service

Trace C
```

Jaeger would think these are three unrelated requests.

You would completely lose the request flow.

---

# With Context Propagation

Everything belongs to one trace.

```text
Trace AAA111

├── API Gateway
├── Auth Service
├── Todo Service
└── Database
```

Now Jaeger can show the entire journey.

---

# How is the Context Sent?

Usually through HTTP headers.

A common standard is **W3C Trace Context**.

One important header is:

```text
traceparent
```

A service receiving an HTTP request reads this header to continue the existing trace instead of starting a new one.

You don't need to memorize the header format yet. Just remember:

> The tracing context is carried in request headers between services.

---

# What if One Service Doesn't Propagate the Context?

Suppose:

```text
API Gateway

↓

Auth Service

↓

Todo Service
```

The Auth Service forgets to forward the tracing context.

Then:

```text
API Gateway

Trace AAA111
```

```text
Auth Service

Trace AAA111
```

```text
Todo Service

Trace XYZ999
```

The Todo Service starts a brand-new trace.

Jaeger now shows:

**Trace 1**

```text
API Gateway

↓

Auth Service
```

**Trace 2**

```text
Todo Service
```

From the UI, it looks like the Todo Service was called independently, even though it was actually part of the original request. The end-to-end picture is broken.
