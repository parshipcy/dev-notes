# Why Distributed Tracing Exists

## Step 1: Imagine a Simple Website

Suppose you build a basic TODO app.

```
Browser
    │
    ▼
Node.js Server
    │
    ▼
Database
```

When you click **"Add Todo"**, here's what happens:

1. Browser sends an HTTP request.
2. Node.js receives it.
3. Node.js saves the todo in the database.
4. Node.js sends the response back.

Everything happens in **one backend application**.

This is called a **monolith**.

---

## What is a Monolith?

A **monolith** is an application where all the business logic lives in one codebase and typically runs as one service.

For example:

```
my-app/

├── auth/
├── todos/
├── users/
├── payments/
└── notifications/
```

Everything runs together.

If something goes wrong, debugging is relatively straightforward because you only have one backend.

---

## Step 2: Why Companies Don't Keep Everything in One App

Imagine your TODO app becomes as popular as Google Docs.

Now you have:

* millions of users
* authentication
* notifications
* file uploads
* payments
* AI features
* analytics

Putting everything into one application becomes difficult.

So companies split the application into **multiple services**.

---

## Step 3: Microservices

Instead of one backend, you now have several independent services.

```
Browser
      │
      ▼
API Gateway
      │
 ┌────┼──────────────┐
 ▼    ▼              ▼
Auth  Todo        Notification
Service Service      Service
           │
           ▼
        Database
```

Each service has a specific responsibility.

For example:

**Auth Service**

* Login
* Signup
* JWT validation

**Todo Service**

* Create todo
* Delete todo
* Update todo

**Notification Service**

* Send email
* Send push notification

Each service can even be written in a different language.

* Auth → Go
* Todo → Node.js
* Notification → Java

This architecture is called **microservices**.

---

## Step 4: A Request Travels Through Many Services

Suppose you click **"Create Todo"**.

The request may flow like this:

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
Notification Service
   │
   ▼
Database
```

One user action now involves multiple services.

---

## Step 5: The Problem

One day, a user reports:

> "Creating a todo takes 12 seconds."

You need to find out **where the delay is**.

Which service is slow?

```
Browser
   │
   ▼
API Gateway      15 ms ✅
   │
   ▼
Auth Service     20 ms ✅
   │
   ▼
Todo Service     40 ms ✅
   │
   ▼
Notification     11,900 ms ❌
   │
   ▼
Database         10 ms ✅
```

You don't know yet.

This is the core problem.

---

## Step 6: Can Logs Solve This?

Every service writes its own logs.

### Auth Service

```
Request received
User authenticated
Request completed
```

### Todo Service

```
Creating todo
Todo saved
```

### Notification Service

```
Sending email...
```

Now imagine:

* 100 services
* 1 million requests/day
* thousands of log lines every second

To debug **one request**, you have to search logs across multiple services and manually match timestamps.

This quickly becomes difficult.

---

## Step 7: This Is Why Distributed Tracing Exists

Distributed tracing follows a request **across every service** it touches.

Instead of disconnected logs, you get a complete picture.

```
Request: POST /todos

API Gateway
   │ 15 ms
   ▼
Auth Service
   │ 20 ms
   ▼
Todo Service
   │ 40 ms
   ▼
Notification Service
   │ 11.9 s
   ▼
Database
   │ 10 ms
```

Immediately you can see:

> The Notification Service is causing the delay.

You didn't have to inspect thousands of log entries.

---

## Step 8: What Jaeger Does

Jaeger **doesn't run your application**.

It **collects tracing data** from your services and presents it visually.

Instead of reading raw logs, you see a timeline like:

```
POST /todos

├── API Gateway (15 ms)
│
├── Auth Service (20 ms)
│
├── Todo Service (40 ms)
│
└── Notification Service (11.9 s)
```

You can click on any service to inspect details, which is exactly what the Jaeger UI is designed for.
