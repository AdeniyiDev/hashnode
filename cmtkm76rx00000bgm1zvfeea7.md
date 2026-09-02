---
title: "Retries Are Not a Strategy: How They Can Take Down Your System"
datePublished: 2026-09-02T21:35:08.330Z
cuid: cmtkm76rx00000bgm1zvfeea7
slug: retries-are-not-a-strategy-how-they-can-take-down-your-system
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/fc06a8d2-ffc6-4e0e-a341-525e63d1d7e8.jpg

---

One request fails.

The application retries it.

It fails again.

So the application retries again.

At first, this sounds like a good thing.

After all, networks fail. APIs timeout. Databases occasionally become slow. A temporary failure should not immediately become a user-facing error.

That is exactly why retries exist.

But there is a problem.

**A retry is not free.**

Every retry creates another request, another connection, another piece of work for the system to process.

And when a system is already struggling, sending it more work can be the exact thing that pushes it over the edge.

This is where retries stop being a resilience mechanism and start becoming an outage amplifier.

* * *

# The Problem With “Just Retry It”

Imagine you have a service called `Order Service`.

A user places an order:

```text
User
  |
  v
Order Service
  |
  v
Payment Service
```

The Order Service sends a request to the Payment Service.

Normally:

```text
Order Service ---> Payment Service
                     |
                     v
                  Success
```

Everything works.

Now imagine the Payment Service becomes slow.

Maybe the database behind it is overloaded.

Maybe the network is experiencing problems.

Maybe the service is running out of connections.

The request takes longer than expected.

The Order Service waits.

Eventually:

```text
Request ---> Timeout
```

The application thinks:

> "Maybe that was just a temporary network problem."

So it tries again.

```text
Request 1 ---> Timeout
Request 2 ---> Timeout
Request 3 ---> Timeout
```

That still sounds reasonable.

But now imagine **1,000 users** are doing the same thing.

Instead of receiving 1,000 requests, the Payment Service could suddenly receive several thousand attempts.

The original problem was:

> Payment Service is slow.

The new problem becomes:

> Payment Service is slow because everyone keeps asking it to do the same work again.

That's a retry storm.

* * *

# What Is a Retry Storm?

A retry storm happens when many clients repeatedly retry failed requests at the same time, creating additional traffic against an already unhealthy dependency.

It creates a feedback loop:

```text
Service becomes slow
       ↓
Requests timeout
       ↓
Clients retry
       ↓
More requests arrive
       ↓
Service becomes even slower
       ↓
More requests timeout
       ↓
More retries
       ↓
System collapses
```

This is one of the most dangerous things about retries.

The system can essentially **attack itself**.

Nobody is intentionally sending malicious traffic.

Your own services are generating it.

* * *

# Let's Look at a Realistic Example

Suppose you have:

```text
Frontend
   |
   v
API Service
   |
   v
Payment Service
   |
   v
Database
```

Everything is working normally.

Then the Payment Service's database starts responding slowly.

Requests that normally take:

```text
100ms
```

now take:

```text
2 seconds
```

Your API has a timeout of 1 second.

So requests begin failing.

The API retries twice.

Now one user request can generate:

```text
Attempt 1
Attempt 2
Attempt 3
```

That means one user request has potentially become three requests to the Payment Service.

Now imagine 5,000 users.

Instead of approximately:

```text
5,000 requests
```

you could generate:

```text
15,000 requests
```

And that's assuming every request reaches all three attempts.

The database hasn't recovered.

The Payment Service hasn't recovered.

You've simply increased the amount of work the system is trying to perform.

* * *

# The Retry Multiplier Gets Worse With Microservices

This is where things become particularly dangerous.

Imagine a request travels through several services:

```text
Service A
   ↓
Service B
   ↓
Service C
   ↓
Database
```

Now imagine every service has its own retry mechanism.

Service A retries Service B.

Service B retries Service C.

Service C retries the database.

A single failure can now produce a surprising amount of additional work.

For example:

```text
A → B
    B → C
        C → DB
```

If each layer independently retries three times, the amount of work can multiply rapidly.

This is why retries in distributed systems need to be designed as a **system-level behavior**, not simply added independently to every service.

* * *

# The Hidden Problem: Timeouts

Retries and timeouts are closely connected.

You usually don't retry immediately.

You wait for the first request to fail.

That means you need a timeout.

For example:

```text
Timeout = 2 seconds
Retries = 3
```

A request could potentially spend several seconds waiting for responses.

Now imagine the user is waiting for the API.

They don't care that your service is retrying intelligently.

They just see:

> "Something is taking too long."

And if hundreds or thousands of requests are sitting around waiting, you can also exhaust:

*   Worker threads
    
*   Connection pools
    
*   CPU
    
*   Memory
    
*   Network connections
    

So retries don't only create more traffic.

They can also cause **resource exhaustion**.

* * *

# Why Exponential Backoff Exists

If you are going to retry, you generally don't want to do this:

```text
Fail
Retry immediately
Fail
Retry immediately
Fail
Retry immediately
```

That's effectively hammering the failing service.

Instead, you can introduce **backoff**.

For example:

```text
Attempt 1 → Fail

Wait 100ms

Attempt 2 → Fail

Wait 200ms

Attempt 3 → Fail
```

The delay increases between attempts.

This gives the dependency some breathing room.

A common approach is exponential backoff:

```text
100ms
200ms
400ms
800ms
```

The exact values depend on the system.

The important idea is:

> **Don't immediately send another request to a service that just told you it can't handle the previous one.**

* * *

# But Backoff Alone Isn't Enough

This is another common mistake.

Teams hear:

> "Use exponential backoff."

So they add it and assume the retry problem is solved.

Not necessarily.

Imagine 10,000 clients experience the same failure at approximately the same time.

If all of them follow the exact same retry schedule:

```text
100ms
200ms
400ms
800ms
```

they can still wake up and retry together.

You have simply delayed the storm.

This is where **jitter** becomes useful.

Jitter introduces randomness into the retry delay.

Instead of every client retrying at exactly:

```text
400ms
```

you might have requests retrying around different points within that window.

That helps prevent synchronized retry waves.

* * *

# The Bigger Question: Should You Retry At All?

This is probably the most important question.

Not every failure should be retried.

Consider a request that failed because of:

```text
HTTP 500
```

A retry might make sense if the failure is temporary.

But consider:

```text
HTTP 400
```

The client sent an invalid request.

Retrying the exact same invalid request probably won't fix anything.

You need to understand **what failed and why** before deciding to retry.

Some failures are temporary.

Some failures are permanent.

Some operations are safe to repeat.

Others are not.

* * *

# The Dangerous Case: Retrying Non-Idempotent Operations

This is where retries can become more than a performance problem.

Imagine a payment request:

```text
POST /payment
```

The client sends:

```text
Charge customer ₦50,000
```

The payment succeeds.

But the response gets lost because of a network problem.

The client doesn't know whether the payment succeeded.

So it retries.

Now you have:

```text
Payment attempt 1 → SUCCESS
Payment attempt 2 → SUCCESS
```

The customer could potentially be charged twice.

The retry itself didn't fail.

The retry **worked**.

That's the problem.

For operations that must not be duplicated, you need mechanisms such as **idempotency keys** and carefully designed request semantics.

The goal is to make repeated attempts produce the intended result rather than duplicate side effects.

* * *

# Retries Can Hide the Real Problem

There is another issue that doesn't get enough attention.

Retries can make your dashboards look deceptively healthy.

Suppose your application normally processes:

```text
10,000 requests
```

But because of retries, it actually sends:

```text
25,000 requests
```

Your users may still receive successful responses.

So your success rate might look acceptable.

But underneath:

```text
CPU ↑
Network traffic ↑
Database connections ↑
Latency ↑
Retries ↑
```

The system is working much harder than it should.

Retries can therefore **mask an unhealthy dependency** instead of fixing it.

* * *

# What Should You Do Instead?

Retries are useful.

The answer isn't:

> "Never retry."

The answer is:

> **Retry deliberately.**

Before implementing retries, ask a few questions.

### 1\. What type of failure are we seeing?

Is it:

*   A timeout?
    
*   A temporary network failure?
    
*   A server error?
    
*   A validation error?
    
*   Rate limiting?
    
*   Dependency overload?
    

Don't treat every failure the same way.

* * *

### 2\. How many times should we retry?

More retries don't automatically mean more reliability.

Sometimes:

```text
1 retry
```

is better than:

```text
10 retries
```

because the system can fail fast instead of consuming resources waiting for a dependency that is already unhealthy.

* * *

### 3\. How long should we wait?

Use appropriate backoff.

Avoid:

```text
Retry → Retry → Retry → Retry
```

with no meaningful delay.

* * *

### 4\. Should we add jitter?

If many clients can fail at the same time, jitter can prevent synchronized retry waves.

* * *

### 5\. Is the operation safe to repeat?

For operations that create side effects, think carefully.

Ask:

> "If this request runs twice, what happens?"

That question can prevent some very expensive incidents.

* * *

### 6\. What happens when retries are exhausted?

Eventually, you need to stop.

The system needs a failure strategy.

Depending on the architecture, that might involve:

*   Returning an error
    
*   Falling back to cached data
    
*   Queueing the work
    
*   Circuit breaking
    
*   Rate limiting
    
*   Load shedding
    
*   Degrading functionality
    

A retry policy without an exit strategy is just delayed failure.

* * *

# Circuit Breakers: Stop Hammering a Failing Service

One useful pattern is a **circuit breaker**.

The basic idea is simple.

If a dependency keeps failing, stop sending requests to it temporarily.

Instead of:

```text
Request
   ↓
Fail
   ↓
Retry
   ↓
Fail
   ↓
Retry
   ↓
Fail
```

the circuit breaker can move into an open state:

```text
Request
   ↓
Circuit OPEN
   ↓
Fail fast
```

The failing dependency gets time to recover.

After a period of time, the system can test whether the dependency has recovered before allowing normal traffic again.

This prevents your application from continuously hammering an unhealthy service.

* * *

# Rate Limiting and Load Shedding

Sometimes the best thing your system can do is **refuse work**.

That sounds strange.

But during severe overload, accepting every request can cause the entire system to fail.

Rate limiting controls how much traffic a service accepts.

Load shedding goes a step further:

> If the system cannot safely handle more work, reject or defer some work so the system can remain available for the work it can handle.

It is often better to return a controlled failure to some users than to bring down the entire platform.

* * *

# Queue-Based Processing Can Change the Game

Not every operation needs to happen synchronously.

Instead of:

```text
User
 ↓
API
 ↓
Payment Service
 ↓
Database
```

you might use:

```text
User
 ↓
API
 ↓
Queue
 ↓
Worker
 ↓
Payment Service
```

Now work can be processed asynchronously.

If the Payment Service temporarily slows down, the queue can absorb some of the pressure.

Workers can process the backlog at a controlled rate.

This doesn't magically solve every problem, but it can prevent sudden traffic spikes from immediately overwhelming downstream services.

* * *

# The Production Scenario

Let's put everything together.

Imagine you're on call.

At 2:15 PM, your monitoring system starts showing:

```text
API latency ↑
5xx errors ↑
Database connections ↑
CPU ↑
```

You check Kubernetes.

Pods are running.

Nothing has crashed.

You check the API.

It appears healthy.

Then you notice something interesting:

```text
Retry rate: 18%
```

You investigate further.

The Payment Service is experiencing high latency.

The API is retrying failed payment requests three times.

Those retries are increasing traffic to the Payment Service.

The Payment Service sends more requests to the database.

The database reaches its connection limit.

The Payment Service gets even slower.

The API retries even more.

You now have a loop:

```text
Payment Service slows down
        ↓
API requests timeout
        ↓
API retries
        ↓
Payment traffic increases
        ↓
Database connections increase
        ↓
Payment Service slows down further
        ↓
More timeouts
        ↓
More retries
```

The original problem may have been a slow database.

But the outage became much worse because the application kept retrying.

* * *

# The Lesson

Retries are not bad.

**Uncontrolled retries are bad.**

A retry should have a reason.

It should have:

*   A timeout
    
*   A retry limit
    
*   Backoff
    
*   Jitter where appropriate
    
*   An understanding of which errors are retryable
    
*   Awareness of whether the operation is safe to repeat
    
*   A clear failure strategy
    

And most importantly, retries should be designed with the **whole system** in mind.

Because your service doesn't exist alone.

It depends on databases, APIs, queues, caches, networks, and other services.

When one of those dependencies becomes unhealthy, your retry logic determines whether your application absorbs the failure—or makes it worse.

* * *

# Final Thought

One of the easiest things to write in an application is:

```text
if request fails:
    try again
```

One of the hardest things is deciding **when not to try again**.

That's the difference between simply adding retries and actually designing for resilience.

Because sometimes the most resilient thing your system can do...

**is stop asking.**