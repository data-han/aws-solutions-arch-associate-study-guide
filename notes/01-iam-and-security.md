# 01 — IAM & Security (Domain 1, 30% — the biggest domain)

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

---

## AWS Organizations

AWS Organizations lets you centrally manage multiple AWS accounts under one umbrella. Key benefits are **consolidated billing** (all accounts on one bill, and you get volume discount pricing based on combined usage) and the ability to apply **SCPs** as guardrails across entire accounts or Organizational Units (OUs).

Best practice is to separate accounts by environment (separate production and development accounts) with a dedicated management account that doesn't run workloads.

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

**Shield Standard** is free and automatically protects all AWS customers against common Layer 3 and Layer 4 DDoS attacks. **Shield Advanced** is a paid upgrade that adds Layer 7 DDoS mitigation, cost protection during attacks, and access to AWS's DDoS Response Team (DRT).

**AWS Network Firewall** is a managed, stateful firewall service for your VPC — more powerful than NACLs and designed for filtering and inspecting traffic at scale.

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
