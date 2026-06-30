# AWS Networking — VPC, Subnets, PrivateLink & ENI

> Notes from exploring when AWS resources need to live inside a VPC vs when they don't,
> how PrivateLink works, and the Snowflake + S3 private connectivity pattern.

---

## The Core Mental Model

**Ask one question first:**
> "Am I provisioning a server / cluster, or am I calling an API?"

| Situation | VPC / Subnet needed? |
|---|---|
| Provisioning a server or database | ✅ Yes — it has a private IP, it lives somewhere |
| Calling an AWS managed service API | ❌ No — it's outside your VPC in AWS-land |
| Calling a managed service but want it private | Optional — add a VPC Endpoint (ENI doorway) |
| Reaching a private resource in someone else's VPC | PrivateLink — private tunnel between two VPCs |

---

## Two Categories of AWS Services

### Category 1 — Infrastructure you provision (always in a subnet)

These have persistent private IP addresses. They live somewhere in your network.
You must place them in a subnet.

- EC2 instances
- RDS / Aurora
- ElastiCache
- Redshift (provisioned)
- ECS on EC2 launch type
- Application Load Balancers
- NAT Gateways
- OpenSearch (provisioned)

### Category 2 — AWS managed platform services (outside your VPC)

These live in AWS's own internal infrastructure. You call them via HTTPS API endpoints.
They have no IP address you manage. No subnet required by default.

- Amazon S3
- AWS Glue
- AWS Lambda (default, no VPC config)
- Amazon Athena
- Amazon DynamoDB
- Amazon SQS / SNS
- Amazon EventBridge
- AWS Step Functions
- Amazon Kinesis

> **S3 and Glue are always outside your VPC.**
> Glue jobs talk to S3 via the S3 API — no VPC, no subnet, no NAT Gateway needed
> for a simple Glue → S3 pipeline.

---

## When Platform Services Cross Into VPC Territory

There are three situations where a platform service optionally touches your VPC:

### 1. Lambda or Glue connecting to a private resource

If a Lambda function or Glue job needs to reach something with a private IP
(RDS, ElastiCache, an EC2-hosted service), you attach it to a subnet.

AWS creates an **ENI** (Elastic Network Interface) in that subnet so the function
gets a private IP to reach the private resource. The function itself hasn't "moved"
into your VPC — it just has a network foot there.

```
Lambda function
    │ (needs to reach RDS in private subnet)
    ▼
ENI created in your subnet (e.g. 10.0.1.55)
    │
    ▼
RDS instance in private subnet (e.g. 10.0.1.20)
```

**Lambda in VPC gotcha:** putting Lambda in a **public** subnet does NOT give it
internet access. Lambda ENIs never receive public IPs.
To give a VPC-attached Lambda outbound internet access:
- Put it in a **private** subnet
- Add a **NAT Gateway** in a public subnet
- Add a `0.0.0.0/0` route from the private subnet to the NAT Gateway

All three are required. Missing the route is the most common mistake.

### 2. VPC Endpoints — keeping managed service traffic off the internet

You want traffic from your VPC to a managed service (S3, KMS, Glue) to stay
on AWS's private backbone instead of going via the public internet.

Two types:

| Type | Services | Cost | How it works |
|---|---|---|---|
| **Gateway Endpoint** | S3, DynamoDB only | Free | Route table entry pointing at the endpoint |
| **Interface Endpoint (PrivateLink)** | All other AWS services | Charged (hourly + per GB) | ENI placed in your subnet with a private IP |

> **Gateway Endpoint** = no ENI, no subnet needed for the endpoint itself.
> Just a route table entry. Free.
>
> **Interface Endpoint** = an ENI IS placed in a subnet you choose.
> That ENI is the doorway. The managed service hasn't moved.

### 3. PrivateLink for third-party SaaS (e.g. Snowflake)

Snowflake runs in their own AWS account (their own VPC). PrivateLink creates
a private tunnel between your VPC and theirs without either party "moving."

---

## How PrivateLink Actually Works (Interface Endpoint)

The subnet is not about where the service lives.
**It is about where the doorway (ENI) lives.**

```
Your laptop / on-prem
      │
      │  Corporate VPN / Direct Connect
      ▼
Your VPC  (e.g. 10.0.0.0/16)
  └── Private subnet  (e.g. 10.0.1.0/24)
       └── ENI — Interface Endpoint  (private IP: 10.0.1.45)
                 │
                 │  Private DNS resolves:
                 │  glue.us-east-1.amazonaws.com → 10.0.1.45
                 │
                 │  AWS private backbone (never touches public internet)
                 ▼
           AWS Glue / KMS / Secrets Manager / etc.
           (still in AWS-land, location unchanged)
```

**Three layers that must all work together:**

1. **VPN / Direct Connect** — gets your local machine's traffic into the VPC privately
2. **ENI in a subnet** — gives the managed service a private IP address inside your VPC
3. **Private DNS** — makes the hostname resolve to that private IP, not the public one

> If DNS isn't configured, your machine resolves the hostname to the public IP
> and bypasses the ENI entirely — even if the endpoint exists.

**The ENI IS in a subnet. The managed service is NOT.**
You're not putting Glue in a subnet.
You're putting a doorway (ENI) in a subnet that routes traffic to Glue via the AWS backbone.

---

## Snowflake + AWS PrivateLink — Real Example

There are three distinct private connections in the Snowflake/AWS world.
They are separate setups and serve different traffic flows.

### Connection 1 — Your VPC → Snowflake (queries, JDBC, Snowsight)

This is the "classic" Snowflake PrivateLink for your apps/analysts to reach Snowflake privately.

```
Your VPC
  └── Subnet
       └── Interface Endpoint ENI (10.0.1.45)
                 │
                 │  AWS private backbone
                 ▼
           Snowflake's VPC (their AWS account)
           └── Network Load Balancer → Snowflake compute
```

**Setup sequence:**
1. Request PrivateLink enablement from Snowflake (Enterprise tier+)
   — provide them your AWS Account ID
2. Snowflake gives you a **VPC Endpoint Service name**
   (e.g. `com.amazonaws.vpce.us-east-1.vpce-svc-xxxx`)
3. In your AWS account: create an **Interface VPC Endpoint**
   targeting that service name, in a subnet + security group of your choice
4. Configure **private Route 53 hosted zone** so Snowflake's hostname
   (`account.privatelink.snowflakecomputing.com`) resolves to the ENI's private IP
5. Security group: allow outbound HTTPS (443) from your resources to the endpoint

> One-directional: your VPC can reach Snowflake,
> but Snowflake cannot initiate connections into your VPC.

### Connection 2 — Snowflake → YOUR S3 bucket (external stage, COPY INTO, Snowpipe)

This is how Snowflake reads/writes data in your own S3 bucket.
This is a separate setup from Connection 1.

**Standard (most common) — cross-account IAM:**
- Create an IAM role in your AWS account
- Trust policy allows Snowflake's AWS account to assume it
- Scope permissions to your specific bucket
- Snowflake uses this role to access S3

**Private (no public internet) — `USE_PRIVATELINK_ENDPOINT`:**

Snowflake provisions a PrivateLink endpoint inside Snowflake's VPC to reach S3.
You run this SQL in Snowflake (requires ACCOUNTADMIN):

```sql
SELECT SYSTEM$PROVISION_PRIVATELINK_ENDPOINT(
  'com.amazonaws.us-west-2.s3',
  '*.s3.us-west-2.amazonaws.com'
);
```

Then create the storage integration with the flag:

```sql
CREATE STORAGE INTEGRATION my_s3_private_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::ACCOUNT:role/myrole'
  STORAGE_ALLOWED_LOCATIONS = ('s3://my-bucket/path/')
  USE_PRIVATELINK_ENDPOINT = TRUE
  ENABLED = TRUE;
```

Optional — lock the bucket down so it can ONLY be accessed via that endpoint:

```json
{
  "Sid": "DenyNonPrivateLink",
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"],
  "Condition": {
    "StringNotEquals": {
      "aws:SourceVpce": "vpce-01c31eb5f4a1e817d"
    }
  }
}
```

> **Constraint:** S3 PrivateLink requires the bucket region and
> Snowflake account region to match. Cross-region not supported.

### Connection 3 — Snowflake clients → Snowflake internal stage (backing S3)

When Snowflake stores data in normal tables, it uses its own S3 buckets internally.
If your client (JDBC, SnowSQL) downloads large result sets, it talks to that S3 directly.
To keep that traffic private too — configure an S3 Gateway Endpoint in your VPC
and set `ENABLE_INTERNAL_STAGES_PRIVATELINK` in Snowflake.
Separate setup from both Connection 1 and 2.

---

## Connecting to Glue from a Local Machine

You do not "connect to Glue" like SSH-ing into a server.
Glue is a job execution service. You interact with it by:

- Submitting jobs via AWS console
- AWS CLI: `aws glue start-job-run --job-name my-job`
- Python / boto3 from your laptop
- Glue interactive sessions (Jupyter-like, runs on Glue's infra)

All of these are HTTPS API calls to `glue.<region>.amazonaws.com`.
Your laptop → public internet → AWS Glue API. **No VPC or PrivateLink needed.**

If your organisation blocks all public AWS API calls (strict enterprise policy):
- Set up VPN / Direct Connect into your VPC
- Create an Interface VPC Endpoint for Glue in a subnet
- Configure private DNS so the Glue hostname resolves to the ENI's IP
- Traffic: laptop → VPN → VPC → ENI → AWS backbone → Glue API

```
Normal (no PrivateLink):
Laptop ──── public internet ──── AWS Glue API

With PrivateLink (enterprise policy only):
Laptop ── VPN ── VPC
                  └── ENI (10.0.1.45)
                            └── AWS backbone ── AWS Glue API
```

---

## Simple Glue → S3 Pipeline (the common case)

For a standard data engineering pipeline:

```
Local machine
    │
    │  boto3 / AWS CLI (HTTPS, public API call)
    ▼
AWS Glue API ──── submits job ────► Glue job execution (AWS-managed infra)
                                          │
                                          │  S3 API call
                                          ▼
                                     S3 bucket (AWS-managed, outside any VPC)
```

**No VPC. No subnet. No NAT Gateway. No PrivateLink.**

You only add VPC/subnet config to Glue when:
- The job needs to reach a private resource (RDS, ElastiCache) → Glue Connection with subnet + SG
- Org policy blocks public AWS API calls → Interface Endpoint for Glue

---

## Summary Cheat Sheet

| Resource / Scenario | VPC / Subnet needed? | Why |
|---|---|---|
| EC2 instance | ✅ Yes | Has a private IP, lives in your network |
| RDS / Aurora | ✅ Yes | Provisioned server with private IP |
| ElastiCache | ✅ Yes | Provisioned cluster with private IP |
| NAT Gateway | ✅ Yes (public subnet) | It IS a network resource |
| ALB | ✅ Yes | Lives in subnets you specify |
| S3 | ❌ No | AWS-managed, outside any VPC |
| Glue (→ S3 only) | ❌ No | Both are platform services |
| Lambda (→ S3 / DynamoDB only) | ❌ No | All API calls, no private IP needed |
| Lambda (→ RDS) | ✅ Yes | Needs ENI to reach private RDS IP |
| Glue (→ RDS) | ✅ Yes | Needs Glue Connection with subnet + SG |
| Interface Endpoint (PrivateLink) | ✅ ENI in subnet | The doorway needs to live somewhere |
| Gateway Endpoint (S3/DynamoDB) | ❌ No subnet for endpoint | Route table entry only, no ENI |
| Snowflake via PrivateLink | ✅ ENI in subnet | Interface Endpoint to Snowflake's NLB |
| Snowflake → your S3 (private) | Snowflake side only | `USE_PRIVATELINK_ENDPOINT` in Snowflake SQL |

---

## Key Things to Remember

- **S3 is not in any VPC.** It never is. Neither is Glue, Lambda (by default), Athena, etc.
- **ENI = the doorway.** It sits in YOUR subnet but points to a service that is outside your VPC.
- **Private DNS is required** for PrivateLink to work. Without it, the hostname resolves to the public IP and bypasses the ENI.
- **Gateway Endpoint (S3/DynamoDB) is free** and has no ENI — just a route table entry.
- **Interface Endpoint costs money** (hourly + per GB) and places an ENI in a subnet you choose.
- **Lambda in a public subnet ≠ internet access.** Lambda ENIs never get public IPs. Always use private subnet + NAT Gateway for Lambda internet access.
- **Snowflake PrivateLink is Enterprise tier only** and must be requested through Snowflake support.
- **PrivateLink for S3 requires same region** — Snowflake account region must match S3 bucket region. Cross-region not supported.
