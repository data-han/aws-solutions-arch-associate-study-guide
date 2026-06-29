# 01 — IAM & Security (Domain 1, 30% — the biggest domain)

## AWS Organizations

### Structure

The management account creates and owns the organization. Everything sits under a single Root.

```
Management Account (payer, org owner)
└── Root
    ├── OU: Production
    │   ├── Account A  ← existing account, joined via invitation
    │   └── Account B  ← new account, created directly from management account
    ├── OU: Development
    │   └── Account C
    └── OU: Sandbox
        └── Account D
```

- **Management account** creates the org, creates OUs, invites or creates member accounts, and is the consolidated billing payer. SCPs never apply to it.
- **Root** is auto-created when you create the org — it's the top-level container.
- **OUs** are folders. Nest accounts inside them to apply SCPs in bulk. OUs can be nested inside other OUs.
- **Member accounts** are the workload accounts. SCPs can restrict what they can do.

### Adding accounts

**Invite an existing account** — management account sends an invitation; the existing account owner must accept. Use when the account already exists outside the org.

**Create a new account** — management account creates it directly via Organizations console or API; no invitation needed. Use when spinning up a net-new account (new team, new environment). After joining, move the account into the right OU.

### RI sharing and consolidated billing

RI discounts are shared **org-wide automatically** — no OU configuration needed. If Account A buys an RI and Account B has matching usage, the discount applies. The management account can turn off sharing per account in billing settings.

### SCPs (Service Control Policies)

SCPs attach to OUs (or Root) and restrict what member accounts can do. They never grant permissions — IAM still controls actual access within each account. SCPs on the Root apply to every account in the org except the management account.

Common exam pattern: deny `cloudtrail:StopLogging` at Root → enforced on all member accounts, no one can opt out.

---

## IAM core

IAM (Identity and Access Management) is a **global service** — identities you create (users, groups, roles) exist across all AWS regions. The core concept is that everything in AWS starts with zero permissions, and you explicitly grant access through **policies** attached to identities or resources.

**The most important rule:** always apply **least privilege** — grant only the permissions needed for the job, nothing more. This is the default-correct answer to almost any "how do you secure access?" exam question.

An **IAM Role** grants temporary credentials to a trusted entity that assumes it. Roles are the right tool whenever something needs to interact with AWS services — instead of creating a long-lived access key, you let the entity assume a role and receive short-lived credentials automatically. You should use roles (not access keys) for:

- EC2 instances calling AWS services — attach the role as an **instance profile** so the application running on the instance inherits credentials automatically.
- Cross-account access — let an identity in Account A assume a role in Account B.
- AWS services calling other services (e.g., Lambda calling S3).
- Federated or SSO users coming from an external identity provider.

**Never embed access keys in code or on EC2 instances.** If an exam question mentions hardcoded credentials, the answer is always to replace them with an IAM role.

### Policy types

IAM has several policy types that work together — understanding which one to use (and how they combine) is a core exam skill:

| Type | Attached to | Purpose |
|------|-------------|---------|
| **Identity-based** | User / group / role | Defines what actions the identity is allowed to perform |
| **Resource-based** | A resource (S3 bucket, SQS queue, etc.) | Defines who can access this resource — includes a `Principal` field |
| **Permissions boundary** | A user or role | Sets the *maximum* permissions that identity can ever have, even if other policies grant more |
| **SCP (Service Control Policy)** | An OU or account in AWS Organizations | A guardrail that sets the ceiling of permissions for entire accounts — it does not grant anything by itself |
| **Session policy** | An assumed-role session | Further restricts the permissions of a specific session |

The **effective permission** for any action is the **intersection** of all applicable policies — a permission must be explicitly allowed by every relevant policy layer to actually work. An explicit **Deny anywhere always wins**, overriding any Allow.

SCPs are important to understand: they do not grant permissions. They only define the maximum that identities within an account *can* have. An SCP allowing everything still requires IAM policies to actually grant access.

### ABAC — Attribute-Based Access Control

Traditional IAM access control is called RBAC (Role-Based Access Control) — you create a separate role or policy for each job function, project, or team. This works fine at small scale, but if you have 100 teams and 50 projects, you quickly end up creating and maintaining thousands of policies.

**ABAC (Attribute-Based Access Control)** is an alternative approach where you use **tags** to control access dynamically. You tag your AWS resources (EC2 instances, S3 buckets, RDS databases) and your IAM identities (roles, users) with matching attributes, then write a single policy that grants access when the tags on the resource and the principal match.

For example: tag every EC2 instance with `Project: Alpha` and tag the Alpha engineering team's IAM role with `Project: Alpha`. Then write a policy with a condition that says "allow actions on any EC2 instance where the resource's Project tag matches the caller's Project tag." The Alpha team can access Alpha instances automatically — and when you start Project Beta, you just tag the new resources appropriately, without writing any new policies.

ABAC scales much better for large organizations because adding a new project or team only requires tagging new resources and assigning tags to principals — no new IAM policies or roles needed.

- Keyword "manage access at scale across many teams or projects", "grant access based on resource tags, reduce number of IAM policies" → **ABAC with IAM tag conditions**.

---

## AWS Organizations

AWS Organizations lets you centrally manage multiple AWS accounts under one umbrella. Key benefits are **consolidated billing** (all accounts on one bill, and you get volume discount pricing based on combined usage) and the ability to apply **SCPs** as guardrails across entire accounts or Organizational Units (OUs).

Best practice is to separate accounts by environment (separate production and development accounts) with a dedicated management account that doesn't run workloads.

### SCP region restriction — the NotAction trap

A common governance requirement is restricting which AWS regions an organization is allowed to use. You do this with an SCP that denies API calls in non-approved regions. However, there is a critical trap: some AWS services are **global** — they do not operate in any specific region at all. These include IAM, CloudFront, Route 53, STS (Security Token Service), Support, and AWS Organizations itself.

If you write a blanket Deny statement that covers all API actions with a region condition, you accidentally block these global services too — breaking IAM, preventing certificate management, and stopping CloudFront. The fix is to use the IAM policy element called **`NotAction`**.

`NotAction` means "apply this Deny to everything **except** the listed actions." By listing the global services in the `NotAction` block, you exclude them from the restriction. All other service APIs are still blocked in non-approved regions, but global services continue to work normally.

In plain language: instead of saying "deny all actions in bad regions," you say "deny all actions in bad regions, but don't apply this to IAM, CloudFront, Route 53, STS, Support, or Organizations."

**Example — the broken version:**

```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": { "aws:RequestedRegion": "ap-southeast-1" }
  }
}
```

You intend this to block everything outside Singapore. But when a developer calls `iam:CreateUser`, that request has no region — IAM is global. The condition asks "is the region NOT `ap-southeast-1`?" and since there is no region, the answer is effectively yes → Deny fires → IAM is blocked. Nobody can create users, attach policies, or assume roles.

**Example — the correct version:**

```json
{
  "Effect": "Deny",
  "NotAction": ["iam:*", "sts:*", "cloudfront:*", "route53:*", "support:*"],
  "Resource": "*",
  "Condition": {
    "StringNotEquals": { "aws:RequestedRegion": "ap-southeast-1" }
  }
}
```

`NotAction` carves out the global services entirely — the region condition never applies to them. Everything else is still blocked outside Singapore.

Also important: the **management account of an AWS Organization is never affected by SCPs at all** — SCPs don't apply to the management account, no matter what policies are in place.

- Keyword "SCP to restrict regions without breaking IAM or CloudFront" → use `NotAction` in the SCP Deny statement to exclude global services.

---

## IAM Identity Center (successor to AWS SSO)

IAM Identity Center provides **single sign-on** across multiple AWS accounts and business applications from one central place. It integrates with external identity providers like Okta or Active Directory (via SAML), so users log in once with their corporate credentials and get access to the appropriate AWS accounts without separate IAM users in each account.

- Keyword "single sign-on across many AWS accounts" → **IAM Identity Center**.

---

## Federation

Federation lets users who exist outside of AWS (in your company's directory or on a social platform) authenticate and receive temporary AWS credentials.

- **SAML 2.0** is used for enterprise identity federation (connecting your corporate Active Directory or Okta to AWS).
- **OIDC** is used for web identity federation (letting users authenticated by Google, Facebook, etc. access AWS resources).
- **Amazon Cognito** is the managed service for mobile and web application user management. It has two distinct pieces the exam likes to separate:
  - **User Pools** = **authentication** (the directory). They handle sign-up, sign-in, MFA, password reset, and hosted UI, and can federate to Google/Facebook/SAML. The output is a signed JWT identifying the user. Think "who is this user, and let them log in." User Pools are also a native authorizer for **API Gateway**.
  - **Identity Pools (Federated Identities)** = **authorization to AWS resources**. They take an authenticated token (from a User Pool, social login, or SAML) and exchange it via STS for **temporary AWS credentials** scoped by an IAM role — so the app can call S3, DynamoDB, etc. directly. Think "give this logged-in user limited AWS access."
  - Use Cognito when you have up to millions of end users authenticating through your app. Keyword "let mobile app users sign in **and** get temporary AWS credentials" → User Pool (sign-in) **+** Identity Pool (credentials).
- **STS (Security Token Service)** is the underlying AWS service that issues temporary credentials. `AssumeRole` and `AssumeRoleWithWebIdentity` are the two most common STS API calls you'll see in exam scenarios.

- Keyword "millions of app users, social login, mobile/web app" → **Cognito**.

---

## Data protection & encryption

**KMS (Key Management Service)** is AWS's managed key service, integrated with almost every AWS service that supports encryption. When you encrypt data with KMS, it uses **envelope encryption**: KMS generates a data key, encrypts your data with the data key, then encrypts the data key itself with a KMS master key (CMK) — you store the encrypted data key alongside the encrypted data. Keys are **Regional** — a key in us-east-1 cannot decrypt data in eu-west-1 without cross-Region configuration.

KMS offers three key types: **AWS managed keys** (AWS creates and rotates them automatically, you have no direct control, free to use), **customer managed keys (CMKs)** (you create them, set rotation policies, and control the key policy — you decide who can use the key), and **AWS owned keys** (shared keys used by some services internally, not visible to you).

Automatic key rotation is available for CMKs. If your application calls KMS so frequently that it hits API rate limits, you can implement **data key caching** — reuse a data key for a short period rather than calling KMS on every encrypt/decrypt operation.

**KMS key policy — a rule beginners often miss:** Every KMS key has a **key policy** attached directly to it, which is a resource-based policy separate from any IAM identity policies. The critical rule is that **IAM policies alone are never sufficient to grant access to a KMS key**. For a role or user to actually use a KMS key, two conditions must both be true at the same time: (1) the key policy must allow that identity, and (2) the IAM policy must also grant the relevant KMS permissions. If either is missing, access is denied.

When you create a customer-managed key (CMK), AWS automatically adds a default root statement to the key policy. This statement grants the account's root user full access and — importantly — allows IAM policies in the account to manage access to the key. You should **never remove this default root statement**. If you do, IAM policies stop working for that key entirely, and if all the named principals in the key policy are then deleted, the key becomes permanently inaccessible. The only recovery path is contacting AWS Support, which takes time and is not guaranteed to succeed quickly.

- Keyword "IAM policy grants kms:Decrypt but the user still gets 'access denied'" → the key policy is missing the required permission; IAM policy alone is not enough for KMS.

**CloudHSM** provides a dedicated, single-tenant hardware security module that you control entirely. Unlike KMS (where AWS manages the hardware), with CloudHSM you hold the keys and AWS cannot access them. Use it when regulations require customer-controlled key custody or FIPS 140-2 Level 3 compliance.

- Keyword "dedicated HSM / regulatory key custody / FIPS 140-2 L3" → **CloudHSM**.

**Encryption in transit** means using TLS/SSL to protect data moving over the network. AWS Certificate Manager (ACM) handles provisioning and renewing TLS certificates.

**Secrets Manager vs SSM Parameter Store** are both ways to store sensitive configuration, but they serve different needs. Secrets Manager supports **automatic rotation** of secrets such as database passwords — it can call a Lambda function on a schedule to rotate the value and update the dependent service. It costs money per secret per month. Parameter Store is free and simpler — use it for general configuration values and non-rotating secrets (stored as `SecureString` encrypted via KMS). If the question mentions automatic credential rotation, the answer is Secrets Manager.

---

## Network security

**Security Groups** operate at the instance/ENI level and are **stateful** — if outbound traffic is allowed, the return traffic is automatically allowed without needing an explicit inbound rule. They support allow rules only; you cannot write a deny rule in a Security Group.

**NACLs (Network Access Control Lists)** operate at the **subnet level** and are **stateless** — outbound and inbound rules are evaluated independently, so you must explicitly allow both directions including ephemeral (return) ports. Unlike Security Groups, NACLs support both allow and deny rules, which is why they're the right tool when you need to block a specific IP address entirely.

- Keyword "block a specific IP address" → **NACL** (Security Groups cannot deny).

**WAF (Web Application Firewall)** operates at Layer 7 (HTTP/HTTPS) and protects against web attacks like SQL injection, XSS, and bad bots. You attach it to CloudFront, an ALB, API Gateway, or AppSync.

**IMDSv2 (Instance Metadata Service version 2)** addresses a specific and important attack type. Every EC2 instance has access to a special internal IP address (`169.254.169.254`) called the instance metadata endpoint. Your application code queries this endpoint to find out things about the instance itself — its region, its instance ID, and most importantly, the temporary IAM role credentials that give your code permission to call AWS services. This is the mechanism that lets code on EC2 work without hardcoded access keys.

The danger is an attack called **SSRF (Server-Side Request Forgery)**. If your web application fetches a URL supplied by a user (or follows an HTTP redirect), an attacker can point that URL to `169.254.169.254`. Your application dutifully fetches it, the metadata service returns the IAM role credentials, and the attacker captures them from the response — giving them temporary access to your AWS resources.

IMDSv2 closes this hole by requiring a **session token** before any metadata can be retrieved. To get a session token, the caller must first make a specific PUT request to a particular path. Because SSRF attacks typically work by getting the victim application to follow a URL (a GET-style operation), they generally cannot perform this preliminary PUT step. No token means no metadata. You enforce IMDSv2 by setting `HttpTokens=required` in your EC2 launch template or instance configuration, or by using an IAM condition to require it.

- Keyword "SSRF attack stealing EC2 instance role credentials via the metadata service" → enforce **IMDSv2** (`HttpTokens=required`).

**Shield Standard** is free and automatically protects all AWS customers against common Layer 3 and Layer 4 DDoS attacks. **Shield Advanced** is a paid upgrade that adds Layer 7 DDoS mitigation, cost protection during attacks, and access to AWS's DDoS Response Team (DRT).

**AWS Network Firewall** is a managed, stateful firewall service for your VPC — more powerful than NACLs and designed for filtering and inspecting traffic at scale. It can inspect and filter traffic at the VPC level, filter outbound requests by domain name, and block specific threat signatures.

**AWS Firewall Manager** is a centralized management service that sits above individual security services like WAF, Shield Advanced, and Network Firewall. Without Firewall Manager, if you want WAF rules on every Application Load Balancer across 50 AWS accounts, an administrator in each account would need to manually configure it — and any new ALBs created later in those accounts would not automatically get the rules. Firewall Manager lets you define a security policy once at the AWS organization level, and it automatically enforces that policy across all accounts in the organization, including accounts added in the future and new resources created in existing accounts.

- Keyword "enforce WAF rules or Shield protections across all accounts automatically", "org-wide firewall policy that applies to new resources" → **AWS Firewall Manager**.

**AWS Client VPN** is a managed VPN service for giving individual remote users secure access to your VPC. It is based on the OpenVPN protocol, meaning standard OpenVPN client software works with it. Unlike Site-to-Site VPN (which connects an entire office network or data center to AWS as a fixed tunnel), Client VPN is for individual people — a developer working from home who needs to reach a private RDS database, or a consultant who needs temporary access to an internal API. Each user connects from their laptop, gets authenticated, and receives access to the resources their IAM permissions allow.

- Keyword "remote employees need to access private VPC resources from home", "individual user VPN into the VPC" → **AWS Client VPN**. (Distinct from Site-to-Site VPN, which is network-to-network, not user-to-network.)

**GuardDuty** is a threat detection service that analyzes VPC Flow Logs, DNS query logs, and CloudTrail events using machine learning to identify suspicious behavior — things like unusual API calls, instances communicating with known malicious IPs, or credential misuse. No agents required — it works entirely from existing log sources.

**Inspector** automatically scans EC2 instances, container images in ECR, and Lambda functions for known software vulnerabilities and unintended network exposure.

**Macie** uses ML to automatically discover and classify sensitive data (like PII — personal identifiable information) stored in S3, and alerts you to potential data exposures.

**AWS Config** continuously monitors your AWS resource configurations and evaluates them against rules you define (e.g., "all EBS volumes must be encrypted," "S3 buckets must not be public"). It records configuration history so you can see how a resource looked at any point in time, and can auto-remediate non-compliant resources.

**CloudTrail** records every API call made in your AWS account — who made it, when, from what IP, and whether it succeeded. It is the audit and forensics service. Any question asking who made a change, who deleted a resource, or who accessed data → CloudTrail.

**Security Hub** aggregates security findings from GuardDuty, Inspector, Macie, and other tools into a single dashboard, making it easier to manage your security posture across accounts.

---

## Quick keyword map

| Scenario | Answer |
|----------|--------|
| EC2 needs to access S3 securely | IAM role (instance profile) |
| Grant access to another AWS account | Cross-account IAM role / resource policy |
| Auto-rotate database credentials | Secrets Manager |
| Detect PII in S3 | Macie |
| Audit who made an API call | CloudTrail |
| Threat detection without installing agents | GuardDuty |
| Block a malicious IP at the subnet level | NACL deny rule |
| Filter SQL injection on a web app | WAF |
| Centralize login across many accounts | IAM Identity Center |
| Customer-controlled key custody / dedicated HSM | CloudHSM |
| SSRF attack stealing EC2 role credentials via metadata | Enforce IMDSv2 (HttpTokens=required) |
| Enforce WAF/Shield rules across all org accounts automatically | AWS Firewall Manager |
| Individual remote employees need VPN access to the VPC | AWS Client VPN |
| Grant access based on resource tags, scale to many teams | ABAC with IAM tag conditions |
| SCP restricts regions without breaking IAM or CloudFront | SCP with NotAction listing global services |
| Share RI discounts across all accounts in a company | AWS Organizations with consolidated billing — sharing is automatic org-wide |
| Enforce guardrails across all member accounts | SCPs attached to OUs |
| Add an existing AWS account to the org | Invite it — account owner must accept |
| Create a net-new AWS account inside the org | Create directly from management account — no invitation needed |

---

## AWS Organizations

### Structure

The management account creates and owns the organization. Everything sits under a single Root.

```
Management Account (payer, org owner)
└── Root
    ├── OU: Production
    │   ├── Account A  ← existing account, joined via invitation
    │   └── Account B  ← new account, created directly from management account
    ├── OU: Development
    │   └── Account C
    └── OU: Sandbox
        └── Account D
```

- **Management account** creates the org, creates OUs, invites or creates member accounts, and is the consolidated billing payer. SCPs never apply to it.
- **Root** is auto-created when you create the org — it's the top-level container.
- **OUs** are folders. You nest accounts inside them to apply SCPs in bulk. OUs can be nested inside other OUs.
- **Member accounts** are the workload accounts. SCPs can restrict what they can do.

### Adding accounts

Two ways to get accounts into the org:

**Invite an existing account**
- Management account sends an invitation to the existing account's email/ID
- Owner of that account must log in and accept
- Use when the account already exists outside the org

**Create a new account**
- Management account creates it directly via Organizations console or API
- No invitation needed — account is immediately a member
- Use when spinning up a net-new account (e.g. new team, new environment)

After an account joins, you move it into the right OU from the management account.

### RI sharing and consolidated billing

RI discounts are shared **org-wide automatically** — no OU configuration needed. If Account A buys an RI and Account B has matching usage, the discount applies. The management account can turn off sharing per account in billing settings.

### SCPs

SCPs attach to OUs (or Root) and restrict what member accounts can do. They never grant permissions — IAM still controls actual access within each account. SCPs on the Root apply to every account in the org except the management account.

Common exam pattern: deny `cloudtrail:StopLogging` at Root → enforced on all member accounts, no one can opt out.
| IAM grants KMS permission but access still denied | Check the KMS key policy — IAM alone is insufficient |
