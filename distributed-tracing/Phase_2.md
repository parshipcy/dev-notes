# Core Tracing Concepts

## First, forget about Jaeger.

Think about this user action:

> Click **"Create Todo"**

One click triggers multiple backend operations.

```
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

Distributed tracing records **everything that happened** during this request.

---

# 1. What is a Trace?

A **Trace** represents the **entire journey of one request** through your system.

Imagine a user clicks "Create Todo".

Everything related to that one click belongs to **one trace**.

```
Trace

Create Todo Request
│
├── API Gateway
├── Auth Service
├── Todo Service
└── Database
```

Think of a trace as a **folder**.

Inside the folder are all the operations that happened.

One request = One trace.

---

## Real Example

User opens Amazon.

```
GET /product/123
```

This request may involve

* API Gateway
* Product Service
* Inventory Service
* Recommendation Service
* Pricing Service
* Database

All of these together form **one trace**.

---

# 2. What is a Span?

A **Span** is **one individual operation** inside a trace.

If a trace is a movie,

a span is one scene.

Example:

```
Trace

├── API Gateway
├── Auth Service
├── Todo Service
└── Database
```

Actually looks like

```
Trace

├── Span
│   API Gateway
│
├── Span
│   Auth Service
│
├── Span
│   Todo Service
│
└── Span
    Database Query
```

Every box is a span.

---

# 3. Every Span Has Information

A span isn't just a name.

It stores metadata.

For example

```
Todo Service Span

Name:
POST /todos

Duration:
42 ms

Status:
Success

Service:
todo-service
```

Every span has details like these.

---

# Typical Span Information

```
Span

Name

Start Time

End Time

Duration

Status

Attributes

Events
```

Let's understand each one.

---

# 4. Span Name

The operation being performed.

Examples

```
GET /users

POST /todos

Validate JWT

SELECT * FROM todos

Send Email
```

Think of it as the title of the operation.

---

# 5. Duration

How long the operation took.

Example

```
API Gateway

Duration

10 ms
```

```
Database Query

Duration

500 ms
```

If something is slow,

duration helps identify it.

---

# 6. Start Time

When the operation began.

Example

```
10:00:01.125
```

---

# 7. End Time

When the operation finished.

Example

```
10:00:01.165
```

Duration is simply

```
End Time - Start Time
```

```
40 ms
```

---

# 8. Status

Whether the operation succeeded.

Examples

```
OK
```

or

```
ERROR
```

If your database fails,

that span usually becomes ERROR.

---

# 9. Attributes

Attributes are extra information attached to a span.

Think of them as **key-value pairs**.

Example

```
HTTP Method

POST
```

```
Route

/todos
```

```
User ID

123
```

```
Database

PostgreSQL
```

```
Rows Returned

20
```

These help you filter and understand traces.

---

# 10. Events

Sometimes something important happens **during** a span.

Example

```
Todo Service

↓

Started validation

↓

Validation completed

↓

Started DB query

↓

DB timeout

↓

Retry
```

These are events inside the span.

Events are useful for debugging.

---

# 11. Parent and Child Spans

This is one of the most important ideas.

Suppose

```
Browser

↓

API Gateway

↓

Todo Service

↓

Database
```

The API Gateway calls the Todo Service.

The Todo Service calls the Database.

Relationship:

```
API Gateway

↓

Todo Service

↓

Database
```

becomes

```
API Gateway Span
       │
       ▼
Todo Service Span
       │
       ▼
Database Span
```

API Gateway is the **parent**.

Todo Service is the **child**.

Database is the **child** of Todo Service.

This creates a hierarchy that Jaeger displays as a tree.

---

# 12. Root Span

Every trace starts somewhere.

Usually,

the first service receiving the user's request.

Example

```
Browser

↓

API Gateway
```

The API Gateway creates the first span.

That span is called the **Root Span**.

Everything else comes from it.

```
Root Span

↓

Child

↓

Child

↓

Child
```

One trace has exactly one root span.

---

# Putting It All Together

Imagine a user creates a todo.

```
Trace

Create Todo
```

Inside it

```
Root Span

API Gateway

15 ms
```

↓

```
Child Span

Auth Service

10 ms
```

↓

```
Child Span

Todo Service

45 ms
```

↓

```
Child Span

Insert Database

8 ms
```

This entire tree is what Jaeger visualizes.

When you open Jaeger UI, you're essentially exploring:

* One trace (the whole request)
* Made up of many spans (individual operations)
* Connected through parent-child relationships
* Each span containing timing, status, attributes etc.
