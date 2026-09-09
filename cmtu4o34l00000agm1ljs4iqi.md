---
title: "What Good DevOps Engineers Actually Think About"
datePublished: 2026-09-09T13:22:05.429Z
cuid: cmtu4o34l00000agm1ljs4iqi
slug: what-good-devops-engineers-actually-think-about
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/3f282852-30d2-46ed-b558-9bfb7d58bab0.jpg

---

There are a lot of ways to describe a DevOps engineer.

You might say:

```text
AWS
Docker
Kubernetes
Terraform
Jenkins
Prometheus
Grafana
Ansible
Helm
Git
Linux
```

And yes, these skills matter.

But I've noticed something over time.

Two engineers can know the exact same tools and still perform very differently when they're put in front of a production problem.

One starts typing commands immediately.

The other starts asking questions.

That's the difference I want to talk about.

Because good DevOps engineering isn't just about knowing what command to run.

It's about **knowing what question to ask before you run it.**

* * *

# 1\. Good DevOps Engineers Don't Start With Tools

Imagine someone tells you:

> **"Production is slow."**

A tool-focused response might be:

> "Let's check Kubernetes."

Or:

> "Let's increase the EC2 instance."

Or:

> "Let's enable autoscaling."

But none of those are answers yet.

A good engineer starts with:

```text
What is actually slow?

Which users are affected?

When did it start?

What changed?

Is the entire application slow?

Or only one endpoint?

Is the application slow?

Or is the database slow?

Is the network involved?

Is an external dependency slow?
```

Notice something.

There hasn't been a single command yet.

That's intentional.

The first job isn't to fix the system.

The first job is to **understand the problem.**

* * *

# 2\. They Ask: What Problem Are We Actually Solving?

This sounds simple.

But it's surprisingly easy to skip.

Imagine your team says:

> "We need Kubernetes."

A good DevOps engineer doesn't immediately start creating clusters.

They ask:

```text
Why?

What problem are we trying to solve?

What isn't working today?

How does Kubernetes solve that problem?

What complexity does it introduce?

Do we actually need it?
```

Maybe the real problem is:

```text
Manual deployments
```

Maybe you need:

```text
CI/CD
```

Maybe the problem is:

```text
Application scaling
```

Maybe you need:

```text
Horizontal scaling
```

Maybe the problem is:

```text
Poor availability
```

Maybe you need:

```text
Redundancy
```

The tool comes after the problem.

Not before it.

* * *

# 3\. They Think About Failure Before It Happens

A beginner often asks:

> "How do I make this work?"

A stronger engineer also asks:

> **"How can this fail?"**

For example:

```text
Load Balancer
      ↓
EC2
      ↓
RDS
```

Looks fine.

But what happens if:

```text
EC2 crashes?
```

What happens if:

```text
RDS becomes unavailable?
```

What happens if:

```text
The application starts returning 500s?
```

What happens if:

```text
The load balancer can't reach the target?
```

What happens if:

```text
Traffic increases 10x?
```

What happens if:

```text
The deployment introduces a bad version?
```

Good engineers don't assume the happy path is the only path.

They think about:

```text
Failure
Recovery
Fallback
Rollback
Redundancy
```

before production forces them to.

* * *

# 4\. They Think About Bottlenecks, Not Just Resources

A common troubleshooting mistake is:

> "CPU is high. Add more CPU."

Sometimes that's correct.

Sometimes it's completely wrong.

Imagine:

```text
Users
  ↓
Application
  ↓
Database
```

Your application CPU is:

```text
85%
```

You add more servers.

Now:

```text
10 application servers
       ↓
      RDS
```

But RDS is now:

```text
CPU → 95%
Connections → 100%
```

You didn't solve the problem.

You moved the bottleneck.

Good DevOps engineers think about the **whole system**.

They ask:

```text
Where is the constraint?

What component is limiting throughput?

What happens if I increase capacity here?

Will that create pressure somewhere else?
```

Because systems are connected.

Changing one component can change the behavior of another.

* * *

# 5\. They Think About What Happens at 10x Traffic

A system handling:

```text
100 requests/sec
```

doesn't necessarily behave the same way at:

```text
1,000 requests/sec
```

Maybe:

```text
CPU increases
```

But maybe something else happens.

For example:

```text
Requests
   ↓
Application
   ↓
Database
   ↓
Connection pool exhausted
```

Or:

```text
Requests
   ↓
Application
   ↓
External API
   ↓
Rate limit
```

Or:

```text
Pods
   ↓
NAT Gateway
   ↓
Huge outbound traffic
   ↓
Unexpected AWS cost
```

Scaling changes systems.

That's why good engineers ask:

> **"What happens when the workload grows?"**

Not:

> "Does it work right now?"

* * *

# 6\. They Think About Security Before the Incident

Security shouldn't begin after someone gets compromised.

Imagine an engineer needs access to:

```text
ECR
EKS
CloudWatch
```

And someone gives them:

```text
AdministratorAccess
```

because it's easier.

A good engineer asks:

```text
What exactly does this person need?

Why do they need it?

Can we use a role?

Can we use temporary credentials?

What happens if their account is compromised?

What's the blast radius?

Do they really need production access?
```

The goal isn't to distrust engineers.

It's to reduce the consequences when something goes wrong.

Good DevOps engineers think:

```text
If this credential leaks...
```

before:

```text
This credential leaked.
```

* * *

# 7\. They Think About Cost as Part of Architecture

Cost isn't something you check only when the AWS bill arrives.

It should influence architecture decisions.

Imagine:

```text
Private workloads
       ↓
NAT Gateway
       ↓
Internet
```

It works.

But then your workload grows.

Suddenly:

```text
More traffic
     ↓
More NAT traffic
     ↓
Higher bill
```

A good engineer asks:

```text
How much does this architecture cost?

What happens as traffic grows?

Are we paying for idle resources?

Can we use a different architecture?

Is the extra cost justified by reliability or performance?
```

This doesn't mean:

> "Always choose the cheapest option."

That's dangerous too.

Sometimes the more expensive architecture is worth it.

The point is to understand the trade-off.

* * *

# 8\. They Think About Blast Radius

Here's a question I really like:

> **"If this goes wrong, how much of the system does it take down?"**

Imagine you have:

```text
One AWS account
        ↓
Development
Staging
Production
```

Now someone accidentally changes something dangerous.

The blast radius can be large.

Compare that with:

```text
AWS Organization
       |
 ┌─────┼─────┐
 ↓     ↓     ↓
Dev  Staging Prod
```

Now you have another boundary.

The same principle applies to:

```text
IAM permissions
Databases
Networking
Deployments
Secrets
CI/CD
Infrastructure
```

Good engineers don't only ask:

> "Can we do this?"

They ask:

> **"What happens if this goes wrong?"**

* * *

# 9\. They Think About Recovery, Not Just Prevention

You can spend months trying to prevent every possible failure.

You will still experience failures.

Servers crash.

Deployments fail.

Databases become unhealthy.

Dependencies go down.

People make mistakes.

So good engineers ask:

```text
If this fails, how quickly can we recover?

Can we rollback?

Do we have backups?

Have we tested the backups?

Can another instance take over?

Can we restore the service?

Do we know who is responsible?
```

This is the difference between:

```text
"Nothing should ever fail."
```

and:

```text
"Things will fail. Let's make recovery fast."
```

The second mindset is much more realistic.

* * *

# 10\. They Think About the Developer Experience

DevOps isn't only about infrastructure.

You also have developers who need to use the platform.

Imagine a developer needs to deploy an application.

The process is:

```text
Open ticket
   ↓
Wait for approval
   ↓
Send files
   ↓
Someone manually deploys
   ↓
Wait
   ↓
Check logs manually
```

The infrastructure might be technically secure.

But the developer experience is terrible.

A good DevOps engineer asks:

```text
Can developers deploy safely?

Can they understand failures?

Can they see useful logs?

Can they rollback?

Can they do common tasks without waiting for another team?
```

The goal isn't to give developers unlimited access.

It's to build **safe self-service**.

* * *

# 11\. They Think About Observability Before Production

One of the worst times to discover that you don't have enough monitoring is during an incident.

Production is down.

Someone asks:

> "What is failing?"

And the response is:

> "I don't know."

Then someone starts running random commands.

Good engineers think about this before the incident.

They ask:

```text
What should we monitor?

What does healthy look like?

What metrics matter?

What logs do we need?

Can we trace requests?

What should trigger an alert?

Who receives the alert?
```

The goal isn't to collect every possible metric.

It's to collect enough useful information to understand the system.

* * *

# 12\. They Question Assumptions

This is one of the biggest differences between experienced and inexperienced engineers.

Someone says:

> "The database is fine."

Good engineer:

> "How do we know?"

Someone says:

> "The deployment succeeded."

Good engineer:

> "Did the application actually become healthy?"

Someone says:

> "The network is working."

Good engineer:

> "From where to where?"

Someone says:

> "AWS is having issues."

Good engineer:

> "What evidence do we have?"

This isn't about arguing.

It's about replacing assumptions with evidence.

Instead of:

```text
I think...
```

you want:

```text
The metrics show...
The logs show...
The traces show...
The deployment history shows...
```

That's a much stronger way to operate production systems.

* * *

# 13\. They Understand Trade-offs

There is rarely a perfect architecture.

You usually choose between competing priorities.

For example:

```text
Cost
Reliability
Performance
Security
Complexity
Speed
```

Sometimes increasing one affects another.

For example:

```text
More redundancy
      ↓
Higher cost
```

Or:

```text
More security controls
      ↓
Potentially more operational friction
```

Or:

```text
More microservices
      ↓
More independent deployments
      ↓
More networking and operational complexity
```

A good engineer doesn't say:

> "This architecture is perfect."

They say:

> **"This architecture is appropriate for these requirements, and here are the trade-offs."**

That's engineering judgment.

* * *

# 14\. They Don't Overengineer Everything

This is important.

Good DevOps engineers understand that **more infrastructure doesn't automatically mean better infrastructure.**

You don't need:

```text
Kubernetes
Service Mesh
Kafka
Redis
Multi-region
20 microservices
```

for an application with:

```text
100 users
```

Sometimes:

```text
ALB
  ↓
EC2
  ↓
RDS
```

is perfectly reasonable.

The question isn't:

> "Does this architecture look impressive?"

The question is:

> **"Does this architecture solve the actual requirements?"**

Complexity has a cost.

If you don't need it, don't introduce it.

* * *

# 15\. They Automate Repeated Problems

Good engineers don't automate everything just because automation sounds good.

They look for repetition.

For example:

```text
Every deployment
Every backup
Every infrastructure change
Every security scan
Every environment setup
```

If engineers repeatedly perform the same manual process, that's a candidate for automation.

Instead of:

```text
Engineer
   ↓
Manual command
   ↓
Manual verification
   ↓
Manual deployment
```

you might create:

```text
Git
 ↓
Pipeline
 ↓
Tests
 ↓
Security checks
 ↓
Build
 ↓
Deploy
 ↓
Verify
```

Automation should remove unnecessary human effort and reduce the chance of human error.

* * *

# 16\. They Know When NOT to Change Anything

This is an underrated skill.

Production is broken.

Everyone is under pressure.

Someone says:

> "Let's change the configuration."

Another says:

> "Let's restart everything."

Another says:

> "Let's scale it."

Another says:

> "Let's deploy the previous version."

Suddenly five changes are happening at once.

Now you don't know what fixed the problem.

Or what made it worse.

A good engineer knows when to stop.

They ask:

```text
What evidence do we have?

What is the safest action?

What is reversible?

What happens if this doesn't work?

Can we test it without increasing the blast radius?
```

Sometimes the best action is:

> **Do nothing yet. Get more information.**

That's not weakness.

That's control.

* * *

# 17\. They Think in Systems, Not Individual Servers

This is probably one of the biggest mindset shifts in DevOps.

A beginner might see:

```text
EC2
```

An experienced engineer sees:

```text
Users
   ↓
DNS
   ↓
Load Balancer
   ↓
Network
   ↓
Application
   ↓
Database
   ↓
External dependencies
```

And then:

```text
Monitoring
Security
Identity
Backups
CI/CD
Cost
```

Everything is connected.

For example:

```text
More traffic
     ↓
More application requests
     ↓
More database connections
     ↓
More database load
     ↓
More latency
     ↓
More retries
     ↓
Even more traffic
```

One problem can create another.

That's why system thinking matters.

* * *

# 18\. They Learn From Incidents Instead of Just Fixing Them

Imagine production goes down.

You fix it.

Everything works again.

The easy thing to do is:

```text
Incident closed.
```

A better approach is:

```text
What happened?

Why did it happen?

Why didn't we detect it earlier?

Why did our safeguards fail?

What made recovery difficult?

How do we prevent the same class of failure?
```

Maybe the actual issue was:

```text
Bad deployment
```

But the deeper problem was:

```text
No automated rollback
```

Or:

```text
No health check
```

Or:

```text
No alert
```

Or:

```text
No runbook
```

The incident is telling you something about your system.

Good engineers listen.

* * *

# 19\. They Understand the Business Impact

This is something technical teams sometimes forget.

A DevOps engineer might see:

```text
API latency = 4 seconds
```

But the business sees:

```text
Customers abandoning checkout.
```

The engineer sees:

```text
RDS unavailable
```

The business sees:

```text
Orders cannot be processed.
```

The engineer sees:

```text
Deployment failed.
```

The business sees:

```text
New feature cannot reach customers.
```

Good DevOps engineers understand that infrastructure exists to support a business.

That doesn't mean blindly prioritizing speed over reliability.

It means understanding **why the system matters.**

* * *

# 20\. They Think About What Happens Next

A strong engineer doesn't only solve today's problem.

They also ask:

> **"What happens if this keeps growing?"**

For example:

```text
Today
10,000 requests/day
```

What about:

```text
100,000?
```

What about:

```text
1 million?
```

But this doesn't mean overengineering today.

It means understanding the next bottleneck.

Maybe today:

```text
EC2
```

is enough.

Tomorrow:

```text
Autoscaling
```

might be needed.

Later:

```text
Caching
```

might become important.

Later:

```text
Database scaling
```

might become necessary.

The goal is not to build everything today.

The goal is to know **where you're going next.**

* * *

# 21\. The 5 Questions I Would Ask Before Touching Production

When something breaks, slow down.

Ask:

### 1\. What is broken?

Don't assume.

Identify the actual symptom.

### 2\. What are my limits?

Can you restart?

Can you scale?

Can you deploy?

Are there SLA restrictions?

### 3\. Where is it happening?

Application?

Database?

Network?

Load balancer?

External dependency?

### 4\. What evidence do I have?

Metrics.

Logs.

Traces.

Recent changes.

Events.

### 5\. What is the safest action?

Choose something:

```text
Reversible
Controlled
Evidence-based
Low blast radius
```

This mindset is often more valuable than knowing another 20 commands.

* * *

# 22\. Good DevOps Engineers Are Curious

Production systems are complicated.

Sometimes the answer isn't obvious.

You might see:

```text
CPU = 40%
Memory = 50%
```

and still have:

```text
Latency = 5 seconds
```

That's when curiosity matters.

Instead of saying:

> "Everything looks normal."

you ask:

> "What am I not seeing?"

Maybe:

```text
Database connections
```

are exhausted.

Maybe:

```text
Network latency
```

increased.

Maybe:

```text
External API
```

is slow.

Maybe:

```text
Thread pool
```

is exhausted.

Good engineers keep investigating until the evidence explains the behavior.

* * *

# 23\. They Understand That Every Decision Has a Cost

Not just financial cost.

A decision can create:

```text
Operational cost
Maintenance cost
Security risk
Complexity
Learning curve
Failure modes
Vendor dependency
```

For example:

```text
Microservices
```

can give you independent deployments.

But also:

```text
More services
More networking
More observability
More deployments
More failure points
```

Kubernetes can provide powerful orchestration.

But also:

```text
More operational complexity
```

Terraform can give you infrastructure as code.

But now:

```text
State management
Drift
Modules
Provider versions
```

must be understood.

Good engineers don't just ask:

> **"What do we gain?"**

They ask:

> **"What are we taking on?"**

* * *

# 24\. They Don't Try to Look Like Engineers

This might sound strange.

But there's a difference between:

```text
Looking experienced
```

and:

```text
Being useful.
```

Someone might use:

```text
Kubernetes
Terraform
ArgoCD
Istio
Prometheus
Grafana
```

and sound extremely technical.

But when production breaks:

```text
"Let's restart everything."
```

Another engineer might use a much simpler stack.

But they can calmly say:

```text
"Traffic increased 4x at 13:10.

The load balancer is healthy.

Application CPU is normal.

Database connections increased sharply.

The database is now the bottleneck.

Let's investigate the query workload before changing the application tier."
```

That's engineering.

Not the number of tools in the architecture.

* * *

# 25\. The Difference Between Knowing DevOps and Thinking Like a DevOps Engineer

Knowing DevOps tools looks like:

```text
I know Kubernetes.
I know Terraform.
I know AWS.
I know Jenkins.
I know Docker.
```

Thinking like a DevOps engineer looks like:

```text
I understand how the system works.

I understand where it can fail.

I understand what can change its behavior.

I understand the trade-offs.

I know how to investigate it.

I know how to recover it.

I know how to improve it.
```

The first is a list of technologies.

The second is engineering capability.

And that's what you should be building.

* * *

# Final Thought

Good DevOps engineers don't walk into production thinking:

> **"Which command should I run?"**

They walk in thinking:

> **"What is the system telling me?"**

They think about:

```text
Reliability
Security
Performance
Cost
Failure
Recovery
Scalability
Observability
Automation
Complexity
Business impact
```

They question assumptions.

They look for evidence.

They understand trade-offs.

They know that every architecture has limits.

And most importantly, they understand that **tools are only useful when you know why you're using them.**

You can learn another tool tomorrow.

You can learn another command next week.

But learning how to think about systems?

That's what stays with you.

**The best DevOps engineer isn't the one who knows the most tools.**

**It's the one who can walk into a complicated system, understand what is happening, make a safe decision, and explain why they made it.**