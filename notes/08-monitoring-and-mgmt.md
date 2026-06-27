# 08 — Monitoring, Management & Deployment

## CloudWatch (the primary monitoring service)

CloudWatch is AWS's monitoring and observability service. It collects, stores, and lets you act on operational data from your AWS resources and applications.

**Metrics** are numeric measurements collected over time — things like CPU utilization, network bytes in/out, or request count. Most AWS services automatically publish metrics to CloudWatch. Importantly, some metrics are **not** automatically available — memory utilization and disk usage on EC2 instances are not sent to CloudWatch by default because AWS doesn't have access to the OS internals. To get those, you install the **CloudWatch Agent** on the instance, which collects and sends custom metrics.

**Alarms** watch a single metric and trigger actions when it crosses a threshold you define. The three alarm states are `OK`, `ALARM`, and `INSUFFICIENT_DATA`. Actions can include sending a notification to an **SNS topic** (which can trigger an email, a Lambda function, or an SQS message), triggering an **Auto Scaling action** (scale in or out), or taking an EC2 action (stop, terminate, or reboot the instance).

**Logs** is CloudWatch's centralized log storage. Applications running on EC2, Lambda functions, and many AWS services (like VPC Flow Logs, API Gateway) can send their log output to CloudWatch Logs. **Logs Insights** lets you run fast, interactive SQL-like queries across your log data to find patterns and diagnose issues.

**CloudWatch Dashboards** let you build visual displays of metrics and logs, shareable across your team. **Synthetics** (Canaries) lets you run synthetic monitoring scripts that simulate user actions against your application and alert you if they fail. **Container Insights** and **Lambda Insights** provide enhanced monitoring for containerized and serverless workloads.

- Keyword "trigger an alarm or scaling action when a metric crosses a threshold" → **CloudWatch Alarm → SNS / Auto Scaling**.
- Keyword "monitor memory or disk usage on EC2" → **CloudWatch Agent** (not available by default without it).

---

## X-Ray (distributed tracing)

X-Ray traces requests end-to-end as they travel through a distributed application — across Lambda functions, API Gateway, EC2 services, DynamoDB, and more. It builds a **service map** that shows each component and the latency/error rate between them, making it easy to pinpoint exactly which service in a chain is slow or failing.

X-Ray is the answer any time a question describes a microservices or serverless app and asks how to identify *where* a performance bottleneck or error is occurring.

Don't confuse it with:
- **CloudWatch** → metrics and logs (what is happening, how often)
- **CloudTrail** → API audit (who did what, when)
- **X-Ray** → request tracing (which service in my call chain is the problem)

- Keyword "trace requests across microservices / find latency in distributed app / service map / end-to-end visibility" → **X-Ray**.

---

## CloudTrail (API audit log)

CloudTrail records every API call made in your AWS account — including calls made through the console, CLI, SDKs, and other AWS services acting on your behalf. For each event, it captures who made the call (the IAM identity), what action was taken, when it happened, from what IP address, and whether the request succeeded or was denied.

CloudTrail is the **audit and forensics** service. Any time you need to know who made a change, who deleted a resource, or who accessed a piece of data — CloudTrail is the answer.

By default, CloudTrail stores 90 days of management event history. To retain logs for longer or send them to S3 for analysis, you create a **Trail** that continuously delivers log files to an S3 bucket.

Don't confuse CloudTrail with CloudWatch. **CloudWatch** is about *performance and operational health* — metrics, logs, and alarms. **CloudTrail** is about *who did what* — governance, compliance, and forensic investigation.

- Keyword "who made this change / who deleted this resource / API audit / governance / compliance" → **CloudTrail**.

---

## AWS Config (compliance and configuration history)

AWS Config continuously records the configuration state of your AWS resources and evaluates them against **Config Rules** you define. A rule might say "all EBS volumes must be encrypted" or "S3 buckets must not have public access enabled." Config evaluates each resource against the applicable rules and marks it as compliant or non-compliant.

Unlike CloudTrail (which records what API actions were taken), Config tracks the **resulting configuration state** of resources over time — so you can answer "what did this security group's rules look like last Tuesday?" Config can also trigger automatic remediation actions when a resource goes non-compliant.

- Keyword "continuously check whether resources meet compliance requirements / detect configuration drift / configuration history" → **AWS Config**.

---

## Systems Manager (SSM)

SSM is a broad management service that lets you operate your EC2 instances and on-premises servers at scale without needing direct network access. Key capabilities:

- **Parameter Store** — a hierarchical key-value store for configuration data and secrets. Free for standard parameters. Values can be stored encrypted using KMS (as `SecureString` type). Does not support automatic rotation (use Secrets Manager for that).
- **Session Manager** — lets you open a shell session to an EC2 instance through the browser or CLI **without opening SSH port 22, without a bastion host, and without managing SSH keys**. Access is controlled by IAM policies, and sessions are logged to CloudTrail and optionally S3 or CloudWatch Logs. This is the recommended secure, auditable way to access instances.
- **Patch Manager** — automates the process of patching EC2 instances and on-premises servers with OS and application updates on a schedule.
- **Run Command** — lets you execute shell commands or scripts across a fleet of instances at once, without needing SSH access.
- **Automation** — runs predefined or custom runbooks for common operational tasks (e.g., create an AMI, restart a service, remediate a Config finding).

- Keyword "access an EC2 instance without opening SSH / no bastion host required" → **SSM Session Manager**.

---

## Management & governance services

**Trusted Advisor** analyzes your AWS account and provides recommendations across five categories: cost optimization (underused resources, unused RIs), security (open S3 buckets, MFA not enabled), fault tolerance (missing backups, single-AZ deployments), performance (overutilized instances), and service limits (approaching quota limits). Some checks are available on the free tier; the full set requires a Business or Enterprise support plan.

**Cost Explorer and AWS Budgets** are covered in note 09 — see there for detail.

**Service Quotas** is where you view your current service limits (e.g., maximum number of EC2 instances per region) and request increases.

**AWS Health Dashboard** (formerly Personal Health Dashboard and Service Health Dashboard) shows the health of AWS services globally and provides personalized alerts about events affecting your specific resources — useful for proactively understanding the impact of outages and planned maintenance.

**AWS Control Tower** provides a pre-configured, opinionated setup for a secure multi-account AWS environment (called a "landing zone"). It builds on AWS Organizations, sets up guardrails (SCPs and Config rules), creates baseline account structure (management, logging, audit accounts), and handles account vending for new accounts. Use it when you're setting up a multi-account AWS environment from scratch and want AWS-recommended governance baked in.

---

## Infrastructure as Code & deployment

**CloudFormation** is AWS's native IaC service. You define your entire infrastructure as a JSON or YAML template and CloudFormation provisions, updates, or deletes it as a coherent unit called a **stack**. Key advantages: it's repeatable (deploy identical environments in multiple regions or accounts), version-controlled (templates live in source control), and it manages dependencies (provisions resources in the right order). CloudFormation itself is free — you pay only for the resources it creates. It also provides **drift detection** to identify when a resource's actual configuration has diverged from what the template specifies.

**CDK (Cloud Development Kit)** lets you define your infrastructure using real programming languages (TypeScript, Python, Java, C#) instead of YAML/JSON. Under the hood, CDK compiles your code into CloudFormation templates — it's a higher-level abstraction over CloudFormation.

**Elastic Beanstalk** is covered in note 02 — a PaaS that manages the infrastructure (EC2, ASG, ELB) for you when you just want to deploy application code.

**CodePipeline, CodeBuild, CodeDeploy, and CodeCommit** are AWS's CI/CD services. CodeCommit is a managed Git repository. CodeBuild compiles code and runs tests. CodeDeploy automates application deployments to EC2, Lambda, or ECS. CodePipeline orchestrates these into an end-to-end release pipeline.

**Deployment strategies** you should understand for the exam:
- **Rolling** — updates instances in small batches, leaving the rest serving traffic. Reduces capacity during deployment but no separate environment needed.
- **Blue/Green** — creates a completely new environment (green) and shifts traffic to it when it's ready. The old environment (blue) stays running, making rollback as simple as switching traffic back. No downtime and clean rollback, but requires double the resources temporarily.
- **Canary** — a variant of blue/green where a small percentage of traffic (say 5%) goes to the new version first. If no issues arise after a period, you gradually shift more traffic. Reduces blast radius of bad deployments.
- **Immutable** — never modify running instances. Every deployment provisions a brand-new set of instances with the new version alongside the old ones. Once the new instances pass health checks, traffic switches over and the old instances are terminated. No config drift possible because nothing is mutated in place. Elastic Beanstalk supports this explicitly as "immutable updates." More expensive than rolling (you briefly pay for double the instances) but safer — rollback is just terminating the new fleet.

Quick comparison for the exam:

| Strategy | Downtime | Rollback speed | Cost during deploy | Config drift risk |
|---|---|---|---|---|
| In-place / Rolling | Possible | Slow (re-deploy old) | Low | Yes |
| Blue/Green | None | Instant (flip traffic) | 2× temporarily | No |
| Canary | None | Fast (drain small %) | 2× temporarily | No |
| Immutable | None | Fast (terminate new fleet) | 2× temporarily | No |

---

## Keyword map

| Scenario | Answer |
|----------|--------|
| Alert or trigger scaling when a metric crosses a threshold | CloudWatch Alarm |
| Centralized log collection and querying | CloudWatch Logs / Logs Insights |
| Who deleted or changed an AWS resource | CloudTrail |
| Continuously check whether resources are compliant | AWS Config |
| Secure shell access to EC2 without SSH or bastion | SSM Session Manager |
| Repeatable, version-controlled infrastructure provisioning | CloudFormation |
| Multi-account governance and landing zone setup | Control Tower |
| Cost, security, and performance recommendations | Trusted Advisor |
