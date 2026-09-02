---
title: "Why “It Works on My Machine” Still Exists in 2026"
datePublished: 2026-09-01T09:30:00.000Z
cuid: cmtkmedrh00000akpg1yi9r5j
slug: why-it-works-on-my-machine-still-exists-in-2026
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/dc5c2d2c-e361-4607-ba7d-cd8c8f77f1b1.jpg

---

There is a sentence almost every engineer has heard at some point:

> **“But it works on my machine.”**

It sounds funny until you're the person trying to figure out why an application works perfectly on a developer's laptop, passes CI, works in staging, and then starts throwing errors five minutes after reaching production.

And in 2026, we have Docker.

We have Kubernetes.

We have Infrastructure as Code.

We have CI/CD pipelines.

We have automated testing.

We have configuration management.

So why does **“it works on my machine”** still exist?

Because most of the time, the problem was never simply *the machine*.

The real problem is that **our environments are different in ways we don't always notice.**

And those small differences can become production failures.

* * *

# Part 1 — What Does “It Works on My Machine” Actually Mean?

Let's start with a simple example.

A developer builds an application locally.

They run:

```bash
npm install
npm run build
npm start
```

Everything works.

The developer pushes the code.

The CI pipeline runs.

Tests pass.

The Docker image builds successfully.

The application gets deployed to staging.

Still working.

Then it reaches production.

Suddenly:

```text
500 Internal Server Error
Connection refused
Module not found
Permission denied
Database connection failed
```

Now everyone starts asking:

**“What changed?”**

Sometimes, nothing changed in the application code.

What changed was the **environment around the application**.

Maybe the developer was using:

```text
Node.js 22
```

while production was using:

```text
Node.js 20
```

Maybe the developer had an environment variable defined locally:

```text
DATABASE_URL=localhost:5432/app
```

but production expected something completely different.

Maybe the developer's machine had a package installed globally.

Maybe the application depended on a file that existed locally but wasn't included in the container.

Maybe the developer's filesystem is case-insensitive while the Linux production environment is case-sensitive.

The application didn't necessarily break.

**The assumption that all environments behaved the same was wrong.**

* * *

# Part 2 — Your Environments Are Not Actually the Same

One of the biggest mistakes teams make is thinking about environments like this:

```text
Development
     ↓
Staging
     ↓
Production
```

It looks like the same application simply moves from one place to another.

In reality, it can look more like this:

```text
Developer Laptop
    ↓
Different OS
Different runtime
Different dependencies
Different configuration
Different network
Different credentials
Different resources

        ↓

CI
    ↓
Different runner
Different environment variables
Different filesystem
Different tools

        ↓

Staging
    ↓
Different infrastructure
Different database
Different traffic
Different secrets

        ↓

Production
    ↓
Different infrastructure
Different scale
Different permissions
Different dependencies
Different traffic
```

That's where the problems start.

## Runtime Versions

Consider a developer using:

```text
Python 3.12
```

while production runs:

```text
Python 3.10
```

The code may work locally because a library supports Python 3.12 differently.

Or perhaps a feature behaves differently between versions.

The same thing happens with:

*   Node.js
    
*   Java
    
*   Go
    
*   PHP
    
*   .NET
    
*   Terraform
    
*   kubectl
    
*   Helm
    
*   CLI tools
    

If versions are not controlled, you're already introducing uncertainty.

* * *

# Part 3 — Dependencies Can Betray You

Here's another classic example.

A developer runs:

```bash
pip install -r requirements.txt
```

Everything works.

But imagine the requirements file contains:

```text
requests
```

instead of:

```text
requests==2.32.3
```

Now you don't necessarily have a reproducible environment.

Today, the developer may install one version.

Tomorrow, CI may install a newer version.

Next week, production may build with another version.

The application code hasn't changed.

The dependency did.

This is why **dependency management matters**.

The same issue exists with:

```text
package.json
requirements.txt
pom.xml
go.mod
Gemfile
```

And even when versions are pinned, transitive dependencies can still introduce differences if the dependency tree isn't properly locked.

That's why lockfiles and reproducible builds are important.

You want the build to answer:

> **“What exactly did we build?”**

Not:

> **“Whatever versions happened to be available when we ran the build.”**

* * *

# Part 4 — Docker Helps, But It Doesn't Solve Everything

This is where things get interesting.

Someone usually says:

> “That's why we use Docker.”

And they're right.

Docker solves a **huge** part of the problem.

Instead of relying on whatever happens to be installed on the developer's machine, you can define the application environment in a Dockerfile.

For example:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Now the application has a defined runtime.

That's much better than:

> “Install Node, install these packages, install this tool, configure this environment, and hopefully it works.”

But here's the important part:

**Docker does not magically make your entire system identical everywhere.**

The container may be identical.

The environment around the container might not be.

Think about this:

```text
             SAME CONTAINER
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
     Laptop      Staging   Production
        │          │          │
     Local DB     RDS       RDS
     Local DNS    DNS       DNS
     Local API    API       API
     4 CPU        2 CPU     16 CPU
     No proxy     Proxy     Proxy
```

The container is the same.

The world around it isn't.

And that world matters.

* * *

# Part 5 — Configuration Is One of the Biggest Differences

Applications rarely run without configuration.

They need things like:

```text
DATABASE_URL
API_URL
REDIS_HOST
AWS_REGION
LOG_LEVEL
PORT
```

A developer might have:

```text
API_URL=http://localhost:8080
```

while production has:

```text
API_URL=https://api.example.com
```

That's expected.

The problem isn't that configurations differ.

**They should differ.**

The problem is when those differences are unmanaged, undocumented, or accidentally hardcoded.

For example:

```javascript
const database = "localhost:5432";
```

works beautifully on a developer's laptop.

Then someone deploys it to Kubernetes.

The application starts trying to connect to:

```text
localhost:5432
```

But inside the container, `localhost` means:

> **this container**

not your production database.

Now you're debugging a “database failure” that was actually a configuration problem.

This is why configuration should be separated from application code.

* * *

# Part 6 — Your Laptop Has Things Production Doesn't

This one causes more problems than people realize.

A developer's machine can slowly become a snowflake.

Over time, it accumulates:

```text
Custom environment variables
Global packages
CLI tools
Cached dependencies
SSH configuration
Certificates
Local databases
Development services
Aliases
Shell configuration
```

Eventually, the application works because of something the developer doesn't even remember installing.

Then another developer clones the repository.

Runs the application.

And gets:

```text
command not found
```

The first developer says:

> “That's strange. It works for me.”

Of course it does.

Their machine contains years of accumulated configuration.

This is why reproducibility matters.

A new developer should be able to clone the repository and get close to:

```bash
git clone ...
docker compose up
```

rather than spending two days asking:

> “Which version of this tool are you using?”

* * *

# Part 7 — CI Can Pass While Production Still Fails

Now let's move into the part DevOps engineers deal with constantly.

Imagine this pipeline:

```text
Git Push
   ↓
Lint
   ↓
Unit Tests
   ↓
Build
   ↓
Docker Build
   ↓
Security Scan
   ↓
Deploy
```

Everything is green.

You think:

> “We're good.”

Not necessarily.

CI proves that the application passed **the checks you gave it**.

It doesn't automatically prove that production will behave correctly.

Maybe your tests use:

```text
Mock Database
```

while production uses:

```text
Real Database
```

Maybe CI doesn't test:

```text
DNS resolution
```

Maybe CI doesn't test:

```text
IAM permissions
```

Maybe staging has:

```text
100 requests/minute
```

while production suddenly receives:

```text
10,000 requests/minute
```

The pipeline isn't necessarily lying.

It is simply answering a narrower question:

> **“Did the application pass the conditions we tested?”**

That is very different from:

> **“Will this application survive production?”**

* * *

# Part 8 — Staging Can Lie Too

This is one of the most dangerous assumptions.

Teams often say:

> “It passed staging, so production should be fine.”

But how similar is staging to production?

Maybe production has:

```text
20 Kubernetes nodes
```

while staging has:

```text
2 nodes
```

Production might have:

```text
Multiple availability zones
```

while staging has:

```text
One
```

Production might have:

```text
Millions of requests
```

while staging has:

```text
100 test requests
```

Production might connect to:

```text
Real payment providers
Real databases
Real queues
Real third-party APIs
```

while staging uses mocks.

So when staging passes, what you actually know is:

> **The application worked in staging.**

You don't automatically know:

> **The application will behave correctly at production scale.**

That's a very important distinction.

* * *

# Part 9 — The Classic Production Failure

Let's put everything together.

Imagine a developer builds an API.

Locally:

```text
Node.js 22
PostgreSQL 16
Redis available
Local environment variables
Fast network
Plenty of resources
```

Everything works.

The application goes through CI.

CI passes.

Then production:

```text
Node.js 20
PostgreSQL 14
Redis hostname is different
Environment variable is missing
Read-only filesystem
Restricted IAM permissions
Network policy enabled
```

The application starts.

Then users begin seeing:

```text
500 errors
```

The team looks at the application code.

Nothing obvious.

They check the Docker image.

It is the expected image.

They check the deployment.

It looks correct.

Eventually they discover:

```text
REDIS_HOST
```

wasn't configured in production.

The application was working perfectly.

**The environments weren't.**

* * *

# Part 10 — So How Do We Reduce “Works on My Machine”?

The answer isn't:

> “Make every environment exactly identical.”

That's often impossible.

Your laptop shouldn't be identical to production.

Your production environment will always have things that development doesn't.

The goal is to make differences **intentional, controlled, and visible**.

Here are some practical ways to do that.

## 1\. Pin Your Versions

Don't randomly depend on whatever happens to be installed.

Control:

```text
Runtime versions
Dependency versions
Tool versions
Base images
```

For example:

```dockerfile
FROM node:22.14.0
```

is more predictable than:

```dockerfile
FROM node:latest
```

* * *

## 2\. Use Reproducible Builds

A developer should not build one thing while CI builds another.

Your build process should consistently produce the same application artifact from the same source and dependency definitions.

This is one reason CI should build the image that eventually gets deployed.

Instead of:

```text
Developer builds
      ↓
Push code
      ↓
CI builds again
      ↓
Production
```

you can have:

```text
Git commit
    ↓
CI builds image
    ↓
Image tagged with commit SHA
    ↓
Image pushed to registry
    ↓
Same image deployed everywhere
```

Now you know exactly what artifact moved through the pipeline.

* * *

# 3\. Keep Configuration Outside the Image

Don't bake environment-specific configuration into your application image.

Instead:

```text
Same image
     ↓
Development configuration
     ↓
Staging configuration
     ↓
Production configuration
```

The artifact stays the same.

The configuration changes according to the environment.

In Kubernetes, this can involve mechanisms such as:

```text
ConfigMaps
Secrets
External secret managers
Environment variables
```

The important thing is that configuration is managed deliberately.

* * *

# 4\. Make Development Reproducible

Tools like Docker Compose and development containers can help reduce differences between developers.

Instead of telling someone:

> “Install PostgreSQL, Redis, Node, this version of this CLI, and configure these environment variables.”

you can define the development environment.

For example:

```text
Application
     ↓
PostgreSQL
     ↓
Redis
     ↓
Other dependencies
```

and bring them up consistently.

This doesn't make development identical to production.

But it makes development **predictable**.

* * *

# 5\. Test More Than the Application

Unit tests are important.

But production failures don't always come from application logic.

Test things like:

```text
Application → Database
Application → Redis
Application → API
Application → DNS
Application → Authentication
Application → Queue
```

This is where integration and end-to-end testing become valuable.

You want to catch problems that a unit test simply cannot see.

* * *

# 6\. Make Environment Differences Visible

This is probably the most important lesson.

Don't pretend environments are identical.

Document the differences.

For example:

| Environment | Database | Scale | External APIs |
| --- | --- | --- | --- |
| Development | Local PostgreSQL | 1 instance | Mocked |
| Staging | Managed DB | Small | Sandbox |
| Production | Managed DB | High | Real |

Now when something behaves differently, engineers have somewhere to start.

Unknown differences create debugging nightmares.

Known differences become engineering decisions.

* * *

# Part 11 — The Bigger DevOps Lesson

“It works on my machine” isn't really a developer problem.

It's an **engineering consistency problem**.

When an application moves through:

```text
Developer
   ↓
CI
   ↓
Container Registry
   ↓
Staging
   ↓
Production
```

every transition introduces an opportunity for something to change.

The job of good engineering practices is to reduce those unknown changes.

That's why we use:

```text
Docker
CI/CD
Infrastructure as Code
Configuration Management
Dependency Locking
Automated Testing
Observability
Version Control
Immutable Artifacts
```

These aren't just tools.

They are ways of reducing uncertainty.

And that's what DevOps is really trying to achieve.

* * *

# Final Thought

You will probably never completely eliminate:

> **“It works on my machine.”**

And that's okay.

The goal isn't to create a world where every environment is identical.

The goal is to reach a point where when something works on your machine but fails in production, you can answer:

**“What is different?”**

without spending the next six hours guessing.

Because reliable systems aren't built by eliminating every difference.

They're built by making the important differences **intentional, reproducible, and observable.**

And when you can do that consistently, the phrase:

> **“It works on my machine.”**

stops being the beginning of a debugging nightmare.

It becomes a clue.