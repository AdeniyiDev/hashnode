---
title: "Stop Giving Every Engineer AdministratorAccess
"
datePublished: 2026-09-07T13:28:54.150Z
cuid: cmtra156a00000agm453kehgo
slug: stop-giving-every-engineer-administratoraccess
cover: https://cdn.hashnode.com/uploads/covers/69dad9e0aadf1107e26d5b69/2f5cd039-19c5-4a57-b9c1-d48e2eda232d.png

---

There's a shortcut that happens in a lot of AWS environments.

A new engineer joins the team.

They need access to AWS.

Someone creates an IAM user.

Then comes the easiest solution:

```text
AdministratorAccess
```

Done.

The engineer can access everything.

They can create EC2 instances.

Modify security groups.

Delete S3 buckets.

Change IAM policies.

Access databases.

Create access keys.

Modify networking.

Delete infrastructure.

No more:

> "I don't have permission."

No more waiting for someone from the cloud team.

Everything works.

And that's exactly the problem.

* * *

# 1\. AdministratorAccess Is Not a Developer Role

Let's say an engineer needs to:

```text
Deploy an application
Push an image to ECR
Read CloudWatch logs
Check ECS services
```

Why should that engineer also be able to:

```text
Delete production databases
Modify IAM policies
Create users
Change security boundaries
Delete S3 buckets
Modify VPC networking
Disable security controls
```

They probably shouldn't.

But when you attach:

```text
AdministratorAccess
```

you're effectively saying:

> **"You can do almost anything in this AWS account."**

That's not a job description.

That's a blank cheque.

* * *

# 2\. The Real Problem Isn't Trust

Someone will usually say:

> "But I trust my engineers."

That's not the point.

Security isn't built around assuming everyone is malicious.

It's built around limiting what happens when something goes wrong.

An engineer can make a mistake.

An access key can leak.

A laptop can be compromised.

A browser session can be hijacked.

A malicious package can steal credentials.

A developer can accidentally run the wrong command.

Consider this:

```text
Engineer
   ↓
AWS credentials
   ↓
Compromised laptop
   ↓
Attacker gets credentials
   ↓
AdministratorAccess
   ↓
AWS account
```

The attacker doesn't need to become an AWS expert.

They already inherited the engineer's permissions.

* * *

# 3\. One Mistake Can Become a Production Incident

Imagine an engineer wants to remove an old S3 bucket.

They run:

```bash
aws s3 rb s3://old-bucket --force
```

They accidentally select the wrong bucket.

Instead of:

```text
old-bucket
```

they target:

```text
production-assets
```

If they have AdministratorAccess, AWS doesn't stop them because:

> "This looks dangerous."

The permission exists.

The action is allowed.

The command succeeds.

Now you have:

```text
Production
   ↓
S3 bucket deleted
   ↓
Application failures
   ↓
Incident
```

The problem wasn't necessarily that the engineer was careless.

The bigger problem was:

> **The account had more permissions than the job required.**

* * *

# 4\. Least Privilege Exists for a Reason

The principle is simple:

> **Give a person or service only the permissions required to perform its job.**

Suppose an engineer only needs to deploy to EKS.

They might need permissions around:

```text
EKS
ECR
CloudWatch
```

They probably don't need unrestricted access to:

```text
IAM
Route 53
Billing
RDS deletion
KMS administration
Organizations
Security Hub
```

The exact permissions depend on the role.

But the mindset should be:

```text
What does this engineer need to do?
            ↓
What AWS actions are required?
            ↓
Give only those permissions
```

Not:

```text
What permissions can I give them?
            ↓
AdministratorAccess
```

* * *

# 5\. "But Creating Fine-Grained IAM Policies Takes Time"

Yes.

It does.

That's one reason teams take shortcuts.

Imagine an engineer says:

> "I can't deploy because I don't have permission."

You investigate.

They need:

```text
ecr:PutImage
eks:DescribeCluster
logs:GetLogEvents
```

Then another permission fails.

Then another.

Eventually someone gets frustrated.

And says:

> "Just give them AdministratorAccess."

Problem solved.

At least temporarily.

But now you've exchanged:

```text
Developer convenience
```

for:

```text
Security risk
```

The better approach is to improve the access model instead of repeatedly increasing permissions.

* * *

# 6\. Don't Build IAM Around Individual People

Here's another mistake.

You create a policy specifically for:

```text
James
```

Then another for:

```text
David
```

Then:

```text
Sarah
```

Eventually you have dozens of policies.

That's difficult to manage.

A better approach is to think in terms of **roles**.

For example:

```text
Developer
     ↓
Developer Role

DevOps Engineer
     ↓
DevOps Role

Read-Only Engineer
     ↓
ReadOnly Role

Security Team
     ↓
Security Role
```

Then permissions are attached to the role.

People assume the role when they need access.

This makes the access model easier to understand.

* * *

# 7\. AWS IAM Roles Are Better Than Sharing Access Keys

Imagine your team has:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

sitting inside:

```text
.env
```

Or worse:

```text
GitHub repository
```

Or:

```text
Jenkins credentials
```

Now you have a serious problem.

Long-lived credentials can be copied.

They can leak.

They can be forgotten.

They can remain active after someone leaves the company.

Instead, wherever possible, use temporary credentials through IAM roles and identity federation.

The model becomes:

```text
Engineer
   ↓
Identity Provider / SSO
   ↓
IAM Role
   ↓
Temporary credentials
   ↓
AWS
```

This gives you much better control over:

```text
Who accessed AWS
Which role they assumed
When they accessed it
What they were allowed to do
```

* * *

# 8\. Separate Human Access From Application Access

This is extremely important.

Your engineer shouldn't use the same permissions as your application.

For example:

```text
Engineer
   ↓
Human IAM Role
```

while:

```text
Application
   ↓
Application IAM Role
```

The application might only need:

```text
s3:GetObject
sqs:SendMessage
secretsmanager:GetSecretValue
```

It doesn't need:

```text
iam:*
ec2:*
rds:*
```

Likewise, an engineer shouldn't automatically inherit the application's permissions either.

Different identities should have different responsibilities.

* * *

# 9\. Your CI/CD Pipeline Doesn't Need AdministratorAccess Either

This one is often overlooked.

You create Jenkins.

Jenkins needs to deploy your application.

Someone creates an AWS user:

```text
jenkins
```

Then attaches:

```text
AdministratorAccess
```

Why?

Because:

> "Jenkins needs permission to deploy."

But what does Jenkins actually need?

Maybe:

```text
ECR
EKS
S3
CloudFormation
CloudWatch
```

depending on the deployment architecture.

Giving Jenkins full administrative access means:

```text
Pipeline compromised
       ↓
AWS AdministratorAccess
       ↓
Potential account-wide damage
```

Your CI/CD system becomes one of your biggest attack paths.

Treat machine identities just as seriously as human identities.

* * *

# 10\. Production Access Should Be Different

Here's where things get more interesting.

Should every engineer have unrestricted production access?

Probably not.

You can design access around environments:

```text
Development
     ↓
Broader access

Staging
     ↓
Controlled access

Production
     ↓
Restricted access
```

For example:

```text
Developer
   ↓
Development Account
   ↓
Can deploy

Developer
   ↓
Production Account
   ↓
Read-only
```

A senior engineer or platform team might have a controlled production role.

Some sensitive operations might require approval.

The goal isn't to make production impossible to operate.

It's to make dangerous actions intentional.

* * *

# 11\. Multi-Account AWS Makes This Easier

If everything lives inside one AWS account:

```text
Development
Staging
Production
Security
Logging
```

then access boundaries become harder to enforce.

A stronger structure might look like:

```text
AWS Organization
│
├── Development Account
│
├── Staging Account
│
├── Production Account
│
├── Security Account
│
└── Logging Account
```

Now you can control access at the account boundary.

An engineer can have broad permissions in:

```text
Development
```

while having limited permissions in:

```text
Production
```

This creates another layer of protection.

* * *

# 12\. MFA Should Be Non-Negotiable for Privileged Access

Imagine someone obtains an engineer's AWS credentials.

If the credentials are enough to assume a highly privileged role:

```text
Stolen credentials
       ↓
AWS
       ↓
Privileged access
```

That's bad.

MFA adds another barrier.

A better model is:

```text
Username / SSO
      +
MFA
      ↓
Privileged Role
      ↓
AWS
```

Especially for administrative or production access.

The objective is simple:

> **Stealing one thing shouldn't automatically be enough to take over the environment.**

* * *

# 13\. Use Permissions Boundaries and Guardrails Where Appropriate

Sometimes even administrators shouldn't be able to do absolutely everything.

This is where AWS security controls can work together.

For example:

```text
IAM Policies
     +
Permissions Boundaries
     +
Service Control Policies
     +
MFA
     +
Account Separation
```

These controls can create layers around your environment.

Think of it like a building.

You don't rely on one locked door.

You might have:

```text
Building entrance
      ↓
Office entrance
      ↓
Server room
      ↓
Rack
      ↓
Locked server
```

AWS security should be approached similarly.

One control failing shouldn't necessarily expose everything.

* * *

# 14\. Logging Matters as Much as Permissions

Suppose someone makes a dangerous change.

You need to know:

```text
Who did it?

When?

From where?

What API call was made?

What resource was changed?
```

This is where services such as CloudTrail become important.

You might discover:

```text
DeleteDBInstance
```

was called at:

```text
02:14 AM
```

by:

```text
ProductionRole
```

Now you have something useful to investigate.

Without good logging, you may only know:

> "The database disappeared."

That's not enough.

Access control tells you **what someone can do**.

Logging helps tell you **what someone actually did**.

You need both.

* * *

# 15\. Don't Confuse Read Access With Write Access

Not everyone who needs AWS access needs the ability to change things.

For example:

```text
Monitoring Engineer
        ↓
Read-only

Developer
        ↓
Limited write

DevOps Engineer
        ↓
Deployment + infrastructure access

Security Engineer
        ↓
Security-focused access
```

This is much safer than:

```text
Everyone
   ↓
AdministratorAccess
```

Sometimes the safest permission is simply:

```text
ReadOnlyAccess
```

If someone only needs to investigate an issue, why give them permission to modify the environment?

* * *

# 16\. Temporary Privilege Is Better Than Permanent Privilege

Here's a useful question:

> **Does this engineer need production administrator access all day?**

Probably not.

Maybe they need it for:

```text
20 minutes
```

to perform a specific incident response task.

Instead of:

```text
Permanent AdministratorAccess
```

you can use a model where privileged access is:

```text
Requested
   ↓
Approved
   ↓
Assume privileged role
   ↓
Perform task
   ↓
Access expires
```

This is often called **just-in-time access** or **temporary privilege**.

It reduces the amount of time highly privileged credentials are available.

* * *

# 17\. What About the DevOps Engineer?

This is where people sometimes push back.

They say:

> "But I'm a DevOps engineer. I need access to everything."

Maybe you need broad access.

But "broad" doesn't have to mean:

```text
*
```

across the entire AWS account.

You might legitimately need access to:

```text
EC2
EKS
ECR
VPC
CloudWatch
IAM
S3
Route 53
Terraform-managed infrastructure
```

But even then, you can separate:

```text
Daily engineering role
```

from:

```text
Break-glass administrative role
```

Your normal workflow uses the least privilege role.

The emergency role exists for situations where broader access is genuinely required.

That is much better than using AdministratorAccess for every task.

* * *

# 18\. The Break-Glass Role

Every serious environment should have a way to recover when normal access isn't enough.

For example:

```text
Normal Access
      ↓
Limited permissions

Something goes seriously wrong
      ↓
Break-glass role
      ↓
Highly privileged access
```

But the break-glass role should be:

```text
Rarely used
Strongly protected
MFA-protected
Monitored
Audited
```

The purpose isn't to make administrators powerless.

It's to make powerful access **intentional**.

* * *

# 19\. The Question You Should Ask Before Granting Access

Don't ask:

> **"Does this person need AWS access?"**

Ask:

> **"What exactly does this person need to do?"**

For example:

### Requirement

```text
Deploy Docker images to ECR
```

### Required permissions

Potentially something around:

```text
ECR authentication
Image push
Repository inspection
```

Not:

```text
AdministratorAccess
```

Another example:

### Requirement

```text
Read application logs
```

Potentially:

```text
CloudWatch Logs read permissions
```

Not:

```text
EC2 administrator
RDS administrator
IAM administrator
```

Start with the task.

Then work backward to the permissions.

* * *

# 20\. A Practical Access Model

A simple team could start with something like:

```text
                    AWS Organization
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Development       Staging         Production
          |                |                |
          ↓                ↓                ↓
     Developer Role   Deployment Role   Restricted Role
          |                |                |
       Broader           Limited         Very Limited
       access             access           access
```

Then have:

```text
Break-Glass Admin Role
          ↓
Emergency use only
```

and:

```text
Read-Only Role
          ↓
Investigation / monitoring
```

The exact permissions depend on your organization.

There is no universal "perfect IAM policy."

The important thing is that access is designed around responsibilities.

* * *

# 21\. What You Should Stop Doing

If you find these in your AWS environment, they deserve attention:

```text
❌ Every engineer has AdministratorAccess

❌ Shared AWS accounts

❌ Shared access keys

❌ Root account used for daily work

❌ Long-lived credentials everywhere

❌ Jenkins has AdministratorAccess

❌ Developers have unrestricted production access

❌ No MFA for privileged access

❌ No CloudTrail monitoring

❌ Ex-employees still have active credentials
```

These aren't just configuration problems.

They are potential attack paths.

* * *

# 22\. Start With an IAM Access Review

You don't need to redesign your entire AWS environment overnight.

Start with a review.

Ask:

```text
Who has access?

What roles do they have?

Which roles have AdministratorAccess?

Who can access production?

Which access keys are still active?

Which users haven't used their permissions?

Which service accounts have excessive permissions?

Where are long-lived credentials being used?
```

Then categorize the results.

For example:

```text
Critical
    ↓
AdministratorAccess
    ↓
Production access
    ↓
IAM modification

High
    ↓
Broad write permissions

Medium
    ↓
Unused permissions

Low
    ↓
Unused identities
```

Fix the highest-risk access first.

* * *

# 23\. Don't Remove Permissions Blindly

There's an important balance here.

If you suddenly remove half of an engineer's permissions:

```text
Production
   ↓
Everything breaks
```

Now people will hate the security process.

Instead:

```text
Observe
   ↓
Understand usage
   ↓
Reduce permissions
   ↓
Test
   ↓
Monitor
   ↓
Adjust
```

Security controls should support engineering rather than constantly fighting it.

The goal is not:

> **"Make access painful."**

The goal is:

> **"Make dangerous access difficult to misuse."**

* * *

# 24\. The Bigger Lesson

AdministratorAccess feels convenient because it removes friction.

You don't have to ask:

```text
What permission is missing?
```

You don't have to investigate:

```text
Which AWS API requires this action?
```

You don't have to design:

```text
IAM policies
```

You just give someone everything.

But that convenience comes with a cost.

The more permissions an identity has, the larger the blast radius becomes when something goes wrong.

Think about it this way:

```text
Limited Role
     ↓
Compromised account
     ↓
Limited damage
```

versus:

```text
AdministratorAccess
     ↓
Compromised account
     ↓
Potentially entire AWS environment
```

That's the difference least privilege is trying to create.

* * *

# Final Thought

You don't give someone the keys to the entire building because they need to enter one room.

AWS should be treated the same way.

If an engineer needs to:

```text
Deploy
Debug
Monitor
Read logs
Manage infrastructure
```

give them the permissions required for those responsibilities.

Not everything else.

Because the question isn't:

> **"Do I trust this engineer?"**

The better question is:

> **"If this account is compromised or this command is wrong, how much damage can it cause?"**

That's what IAM is really about.

**Least privilege isn't about trusting your engineers less.**

It's about making sure one mistake, one leaked credential, or one compromised machine doesn't become an AWS-wide incident.