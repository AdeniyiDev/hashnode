---
title: "Your AWS Bill Is High. But Where Is the Money Actually Going?
"
datePublished: 2026-09-04T09:30:00.000Z
cuid: cmtr9nmlx00000agmfbig68a0
slug: your-aws-bill-is-high-but-where-is-the-money-actually-going
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/ab37e645-ac52-4514-b8f5-b67b944907fa.jpg

---

You open AWS Cost Explorer.

You look at the number.

And your first thought is:

**“Why is this bill so high?”**

Maybe last month you were paying $300.

Now you're looking at $700.

Or maybe the difference is even bigger.

The natural reaction is to start looking for something expensive.

Maybe it's EC2.

Maybe it's RDS.

Maybe somebody created a huge instance.

Maybe someone forgot to shut something down.

So you start clicking around AWS trying to find the problem.

And this is where many engineers make their first mistake.

They look at the **total bill** instead of looking at **what created the bill**.

Because knowing that AWS cost you $700 doesn't tell you much.

The important question is:

> **What exactly generated that $700?**

That is where FinOps starts becoming an engineering problem.

* * *

# 1\. The First Mistake: Looking Only at the Total

Imagine you receive this bill:

```text
AWS Total

$742.18
```

That number looks bad.

But it doesn't tell you where the problem is.

Now break it down:

```text
EC2                  $180
RDS                  $145
NAT Gateway          $210
EBS                   $65
Data Transfer         $82
S3                    $20
CloudWatch             $25
Other                  $15
--------------------------------
Total                $742
```

Now the situation looks completely different.

You don't have **an AWS cost problem**.

You have several different cost categories.

And one immediately stands out:

```text
NAT Gateway
$210
```

If you had simply assumed EC2 was responsible for most of the bill, you could spend hours optimizing EC2 instances while completely missing the actual problem.

This is why the first step in cost optimization isn't:

> **“How do I reduce the bill?”**

It's:

> **“Where is the money going?”**

* * *

# 2\. AWS Billing Is a Map

AWS gives you several ways to understand your spending.

One of the most useful starting points is **Cost Explorer**.

Instead of looking at:

```text
Total = $742
```

you can break spending down by things such as:

```text
Service
Region
Linked account
Usage type
Operation
Tags
```

That changes the question from:

> “Why is AWS expensive?”

to:

> “Which AWS service increased our spending?”

Then you can go one level deeper.

For example:

```text
AWS
 │
 ├── EC2
 ├── RDS
 ├── S3
 ├── NAT Gateway
 ├── CloudWatch
 └── Data Transfer
```

Then:

```text
EC2
 │
 ├── Production
 ├── Staging
 └── Development
```

Then:

```text
Production
 │
 ├── Web servers
 ├── API servers
 └── Workers
```

This is how you should think about cloud costs.

**Don't look at the number. Follow the number.**

* * *

# 3\. The Usual Suspects

There are several AWS services that commonly contribute to unexpected bills.

Not because they are inherently bad or overpriced.

But because they are easy to create, easy to forget, or easy to misunderstand.

Let's look at some of them.

* * *

# 4\. EC2: The Instance That Nobody Shut Down

EC2 is probably one of the first places engineers look.

And sometimes they're right.

Imagine someone launches:

```text
c5.xlarge
```

for testing.

They use it for two hours.

Then they finish testing.

But nobody terminates it.

The instance continues running.

For days.

Then weeks.

And the bill quietly grows.

The dangerous part is that nothing is technically broken.

The server is running exactly as instructed.

AWS isn't making a mistake.

**Your infrastructure is doing exactly what you told it to do.**

This is why idle resources are such a common source of waste.

Look for things like:

```text
Stopped? 
Running?
Actually being used?
Production?
Development?
Temporary?
Forgotten?
```

And don't only check EC2 instances.

The same thinking applies to other resources.

* * *

# 5\. EBS: “I Deleted the Server”

Here's a common surprise.

Someone terminates an EC2 instance.

They think:

> “Good. That resource is gone.”

But depending on how the storage was configured, EBS volumes can remain.

Now you have storage that is still being charged even though the instance that originally used it no longer exists.

You might eventually end up with:

```text
EC2 instances
     ↓
Deleted

EBS volumes
     ↓
Still there
     ↓
Still costing money
```

The same principle applies to snapshots.

You need to understand the lifecycle of the resources you're creating.

Deleting the compute resource doesn't necessarily mean every related resource disappears.

* * *

# 6\. NAT Gateway: The Bill Nobody Expected

This is one of my favorite examples because it catches people by surprise.

You create a private subnet.

Your application needs outbound internet access.

So you configure a NAT Gateway.

Everything works.

Perfect.

Then the bill arrives.

And suddenly:

```text
NAT Gateway
$200+
```

You're thinking:

**“How did a gateway cost this much?”**

The answer is that NAT Gateway costs aren't simply about having a gateway sitting there.

Data processing and traffic volume matter.

Imagine your architecture looks like:

```text
Private Subnet
      ↓
NAT Gateway
      ↓
Internet
```

Now imagine your Kubernetes cluster has dozens of pods pulling images, calling external APIs, downloading packages, sending requests, and transferring data through that gateway.

You can end up processing a significant amount of traffic.

And if the architecture wasn't designed with cost in mind, the bill can become surprisingly large.

This is a perfect example of why:

> **Architecture decisions are also cost decisions.**

* * *

# 7\. Data Transfer: The Cost You Don't See Coming

Another area engineers often overlook is data transfer.

Your application might not be running particularly expensive EC2 instances.

Your RDS database might be reasonably priced.

Your S3 usage might be tiny.

But your architecture could be moving a lot of data around.

For example:

```text
Service A
    ↓
Service B
    ↓
Service C
    ↓
Database
    ↓
External API
```

If large amounts of data are moving between services, availability zones, regions, or out to external networks, those transfers can contribute to your bill.

This becomes especially interesting with microservices.

More services can mean:

```text
More network calls
More traffic
More data transfer
More infrastructure
More observability data
```

Microservices can improve scalability and organizational boundaries.

But they can also introduce additional infrastructure costs.

Again:

**Architecture and cost are connected.**

* * *

# 8\. RDS: The Database Doesn't Care About Your Budget

Databases are another area where costs can quietly grow.

Maybe you provisioned:

```text
db.m5.large
```

because you expected the application to grow.

But months later:

```text
CPU usage: 8%
Connections: Low
Traffic: Low
```

You're still paying for the provisioned capacity.

This doesn't mean:

> “Always choose the smallest database.”

That's dangerous.

The database is often one of the most critical components of the system.

The lesson is:

> **Provision based on actual requirements, then continuously validate those requirements.**

You should understand:

```text
CPU
Memory
Connections
Storage
IOPS
Read/write workload
Backup usage
```

before deciding whether the database is overprovisioned.

* * *

# 9\. Kubernetes Can Quietly Increase Your AWS Bill

This is particularly important if you're running EKS.

You might think:

> “My applications only need a few pods.”

But Kubernetes doesn't only cost money through pods.

Think about the infrastructure around the cluster:

```text
EKS
 │
 ├── EC2 nodes
 ├── EBS volumes
 ├── Load balancers
 ├── NAT gateways
 ├── Data transfer
 ├── CloudWatch
 └── Other AWS services
```

Now imagine your cluster has:

```text
10 nodes
```

but workloads only require:

```text
4 nodes
```

You're paying for unused capacity.

Then perhaps someone deploys several services with resource requests like:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "2Gi"
```

even though the applications rarely need that much.

Kubernetes now has to schedule workloads based on those requests.

You can end up adding more nodes.

More nodes mean:

**More EC2 cost.**

This is why Kubernetes cost optimization isn't simply:

> “Use fewer pods.”

You need to understand:

```text
Pod requests
Pod limits
Node utilization
Cluster autoscaling
Workload patterns
Instance types
Storage
Networking
Load balancers
```

* * *

# 10\. Idle Resources Are Quietly Eating Your Money

Here's the scary thing about cloud waste.

It doesn't usually announce itself.

You don't get an alert saying:

> “Congratulations, you are currently wasting $18 per day.”

Instead, resources simply continue running.

You might have:

```text
Old EC2 instance
Unused EBS volume
Unused Elastic IP
Old snapshots
Unused load balancer
Development database
Test environment
Unused NAT Gateway
```

Each one might look insignificant.

Together?

They become expensive.

This is why a monthly cost review can be extremely useful.

Ask:

> **What are we paying for that we don't actually use?**

* * *

# 11\. Tagging: The Secret to Finding Who Owns the Cost

Imagine your AWS account contains:

```text
Production
Staging
Development
Testing
Security
Monitoring
```

And you only look at the total bill.

Good luck figuring out who spent what.

This is where tagging becomes extremely useful.

For example:

```text
Environment = production
Team = platform
Application = payments
Owner = backend
Project = ecommerce
```

Now your cost data can become much more meaningful.

Instead of:

```text
EC2 = $400
```

you can start asking:

```text
Production = $300
Development = $70
Testing = $30
```

Or:

```text
Payments = $220
Checkout = $110
Internal tools = $40
```

Suddenly, the bill becomes understandable.

And once spending is attributable, teams can actually take responsibility for it.

* * *

# 12\. Don't Start Optimizing Before You Understand the Problem

This is one of the most important lessons.

You notice the AWS bill is high.

You immediately decide:

> “Let's downgrade the EC2 instances.”

That's not necessarily optimization.

It might create a performance problem.

Imagine:

```text
Current instance
8 vCPU
32 GB RAM
```

You downgrade it to:

```text
2 vCPU
8 GB RAM
```

The bill goes down.

Congratulations.

But now:

```text
CPU → 95%
Latency → Higher
Requests → Slower
Users → Complaining
```

You reduced cost by creating a reliability problem.

That's not good FinOps.

Good cost optimization asks:

> **Can I reduce unnecessary spending without damaging reliability, performance, security, or availability?**

That's the balance.

* * *

# 13\. A Better Way to Investigate a High AWS Bill

Let's say your bill suddenly increases.

Don't panic.

Don't start randomly deleting resources.

Use a process.

### Step 1 — Find the increase

Compare the current period with the previous period.

Ask:

```text
What changed?
```

* * *

### Step 2 — Find the service

Break the cost down by AWS service.

For example:

```text
EC2        +$50
RDS        +$20
NAT        +$180
S3         +$5
```

Now you know where to investigate.

* * *

### Step 3 — Find the resource or usage type

Don't stop at:

```text
NAT Gateway
```

Ask:

> What traffic is causing this?

Don't stop at:

```text
EC2
```

Ask:

> Which instances?

Don't stop at:

```text
RDS
```

Ask:

> Which database and what changed?

* * *

### Step 4 — Find the reason

Now investigate the infrastructure.

Maybe someone:

```text
Scaled the cluster
Added nodes
Enabled a new service
Changed traffic patterns
Created a test environment
Increased storage
Changed instance type
Introduced a new data pipeline
```

This is where CloudWatch metrics, AWS resource configuration, deployment history, and application architecture become useful.

* * *

### Step 5 — Decide whether the cost is necessary

This question is important.

A $500 increase isn't automatically waste.

Maybe your business doubled its traffic.

Maybe you launched a new product.

Maybe the database legitimately needs more capacity.

The question isn't:

> **“Why did the bill increase?”**

It's:

> **“Was the increase expected and justified?”**

* * *

# 14\. Cost Optimization Isn't Just About Turning Things Off

There are many ways to optimize AWS spending.

Depending on the workload, you might consider:

```text
Right-sizing
Autoscaling
Scheduling non-production resources
Removing unused resources
Choosing appropriate instance types
Using appropriate storage classes
Optimizing data transfer
Improving caching
Using savings plans or reserved capacity where appropriate
Optimizing Kubernetes requests
Improving architecture
```

But each optimization should come with a question:

> **What trade-off am I making?**

For example:

Turning off development servers overnight may save money.

Reducing production capacity too aggressively might hurt reliability.

Moving data to cheaper storage may reduce cost but increase retrieval time.

Reducing logging may save money but make incidents harder to investigate.

Cost optimization is therefore not:

> **“Make the number smaller.”**

It's:

> **“Get the right amount of value for the money we're spending.”**

* * *

# 15\. The Bigger Lesson

Cloud makes infrastructure incredibly easy to create.

That's one of its greatest strengths.

But it can also become one of its biggest weaknesses.

A few clicks can create:

```text
EC2
RDS
S3
Load Balancers
NAT Gateways
EBS
EKS
CloudWatch
```

And because you're billed based on usage and resources, infrastructure that nobody is paying attention to can continue generating costs.

This is why DevOps engineers need to understand more than:

```text
How to deploy
How to scale
How to automate
How to monitor
```

We also need to understand:

> **How much does this architecture cost to operate?**

Because the infrastructure you design today becomes the AWS bill you receive tomorrow.

* * *

# Final Thought

The next time someone says:

> **“AWS is too expensive.”**

Don't immediately start deleting things.

Ask a better question:

> **“Show me where the money is going.”**

Break the bill down.

Find the service.

Find the resource.

Find the usage.

Find the reason.

Then decide what needs to change.

Because **cost optimization without visibility is just guessing.**

And in cloud engineering, guessing can be just as expensive as the infrastructure itself.