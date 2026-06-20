# Mock Exam 1 — Full 65-Question Simulation (SAA-C03)

**Instructions:** Set a timer for **130 minutes**. Answer all 65 questions (mix of multiple-choice and multiple-response, flagged). Pass mark ≈ 72% (≈47/65). Answer key with rationale at the bottom — don't peek until done. Review every miss back to the relevant note.

Domain mix mirrors the real exam: Secure (~30%), Resilient (~26%), High-Performing (~24%), Cost (~20%).

---

**1.** An EC2-based app needs least-privilege access to a specific S3 bucket. What's the best approach?
- A. Embed IAM user keys in the app
- B. Attach an IAM role (instance profile) scoped to that bucket
- C. Make the bucket public
- D. Use the root account credentials

**2.** Which control can explicitly DENY traffic from one IP at the subnet level?
- A. Security group
- B. Network ACL
- C. IAM policy
- D. WAF web ACL

**3.** A company needs automatic rotation of database credentials. Which service?
- A. KMS
- B. Secrets Manager
- C. Parameter Store (String)
- D. IAM

**4.** Which provides agentless threat detection by analyzing CloudTrail, VPC Flow Logs, and DNS logs?
- A. Inspector
- B. GuardDuty
- C. Macie
- D. Config

**5.** A guardrail must prevent member accounts in an Organization from leaving a Region, even for admins. Use:
- A. IAM permissions boundary
- B. Service Control Policy
- C. Bucket policy
- D. Session policy

**6.** To serve a private S3 bucket only via CloudFront:
- A. Make bucket public
- B. Use Origin Access Control + restrictive bucket policy
- C. Enable Transfer Acceleration
- D. Use an Interface Endpoint

**7.** Auditors need to know who terminated an EC2 instance and when. Which service?
- A. CloudWatch
- B. CloudTrail
- C. Config
- D. VPC Flow Logs

**8.** Protect a public web app from SQL injection at the edge:
- A. Shield Standard
- B. WAF
- C. NACL
- D. GuardDuty

**9.** Discover PII across many S3 buckets:
- A. Macie
- B. Inspector
- C. Detective
- D. Config

**10.** A workload requires single-tenant, customer-controlled FIPS 140-2 Level 3 key hardware. Use:
- A. KMS AWS-managed key
- B. CloudHSM
- C. Secrets Manager
- D. ACM

**11.** Centralize sign-on to dozens of AWS accounts and SaaS apps with an external IdP:
- A. IAM users in each account
- B. IAM Identity Center
- C. Cognito user pool
- D. STS only

**12.** (Multiple response — choose TWO) Which protect S3 objects from accidental deletion?
- A. Versioning
- B. One Zone-IA
- C. MFA Delete
- D. Reduced Redundancy
- E. Transfer Acceleration

**13.** Default-secure encryption at rest for S3 with an auditable, customer-controlled key:
- A. SSE-C
- B. SSE-KMS (customer managed key)
- C. No encryption
- D. Client-side only

**14.** A Lambda function needs DB credentials stored cheaply with no rotation requirement. Use:
- A. Secrets Manager
- B. SSM Parameter Store SecureString
- C. Hardcode in code
- D. S3 plaintext file

**15.** Continuously evaluate whether all EBS volumes are encrypted and flag non-compliant ones:
- A. CloudTrail
- B. AWS Config
- C. GuardDuty
- D. Trusted Advisor

**16.** Securely access an EC2 shell with no open SSH port or bastion host:
- A. Open port 22 to 0.0.0.0/0
- B. SSM Session Manager
- C. Direct Connect
- D. NAT Gateway

**17.** Cross-account access to an S3 bucket should be granted by:
- A. Sharing IAM user keys
- B. A bucket policy/role trust to the other account
- C. Making the bucket public
- D. A NACL rule

**18.** A company wants L7 DDoS protection, cost protection, and 24/7 response team support:
- A. Shield Standard
- B. Shield Advanced
- C. WAF only
- D. GuardDuty

**19.** Which statement about security groups is TRUE?
- A. They are stateless
- B. They support deny rules
- C. They are stateful and allow-only
- D. They operate at the subnet level

**20.** Encrypt data in transit to an ALB using a managed, free certificate:
- A. Self-signed cert
- B. AWS Certificate Manager (ACM)
- C. CloudHSM
- D. KMS

**21.** Survive a full AZ failure for a web fleet automatically and simply:
- A. Bigger instance
- B. Auto Scaling group across multiple AZs behind an ALB
- C. Second Region active/active
- D. Single instance with EIP

**22.** Decouple a spiky order system so no orders are lost when consumers fall behind:
- A. SNS direct to instances
- B. SQS queue with consumers scaling on depth
- C. Larger DB
- D. CloudFront

**23.** Cheapest DR with multi-hour RTO/RPO acceptable:
- A. Active/active
- B. Warm standby
- C. Pilot light
- D. Backup & restore

**24.** RDS must auto-fail-over to a synchronous standby with no data loss on AZ failure:
- A. Read replica
- B. Multi-AZ
- C. Hourly snapshots
- D. Larger instance

**25.** Capture messages that fail processing repeatedly for later debugging:
- A. Long polling
- B. Dead-letter queue
- C. Visibility timeout = 0
- D. FIFO

**26.** Relational DB with cross-Region DR and sub-second replication lag:
- A. RDS Multi-AZ
- B. Aurora Global Database
- C. DynamoDB Global Tables
- D. Single read replica

**27.** Stop logging users out when ASG replaces instances:
- A. Disable ASG
- B. Store sessions in ElastiCache/DynamoDB
- C. Sticky sessions only
- D. Larger instances

**28.** Centralized, policy-based backups across EBS/RDS/DynamoDB/EFS with cross-Region copy:
- A. S3 lifecycle
- B. AWS Backup
- C. Manual snapshots
- D. Storage Gateway

**29.** Route users to a standby Region automatically when the primary is unhealthy:
- A. Weighted routing
- B. Latency routing
- C. Failover routing
- D. Simple routing

**30.** (Multiple response — choose TWO) Ways to achieve high availability for a stateless web tier:
- A. Auto Scaling across AZs
- B. Single large instance
- C. Elastic Load Balancer with health checks
- D. Instance store for sessions
- E. One subnet only

**31.** A multi-Region active-active NoSQL store with millisecond latency worldwide:
- A. RDS read replicas
- B. DynamoDB Global Tables
- C. Aurora Multi-AZ
- D. ElastiCache

**32.** Private subnet instances need outbound internet for OS patches but must be unreachable inbound:
- A. Internet Gateway
- B. NAT Gateway
- C. VPC peering
- D. Direct Connect

**33.** A self-healing compute layer that replaces failed instances automatically requires:
- A. Manual monitoring
- B. ASG + ELB health checks
- C. Reserved Instances
- D. A bastion host

**34.** Connect hundreds of VPCs and on-prem networks through a central transitive hub:
- A. VPC peering
- B. Transit Gateway
- C. Internet Gateway
- D. NAT Gateway

**35.** A pilot-light DR design keeps which component running/replicating at all times?
- A. The full production fleet
- B. The core database
- C. Nothing
- D. Only DNS

**36.** Read-heavy app with repeated identical queries overloading RDS — best fix with least change:
- A. ElastiCache caching layer
- B. Vertically scale forever
- C. Switch to Redshift
- D. Enable Multi-AZ

**37.** Microsecond reads for hot DynamoDB items:
- A. GSI
- B. DAX
- C. Global Tables
- D. Larger table

**38.** Lowest-latency global delivery of cacheable images from S3:
- A. Transfer Acceleration
- B. CloudFront
- C. Bigger bucket
- D. Cross-Region replication

**39.** Serverless ad-hoc SQL directly on data in S3:
- A. Redshift
- B. Athena
- C. EMR
- D. RDS

**40.** Shared file system mounted concurrently by many Linux EC2 across AZs:
- A. EBS io2
- B. EFS
- C. Instance store
- D. FSx for Windows

**41.** Load balancer needing a static IP, L4, millions of req/s, ultra-low latency:
- A. ALB
- B. NLB
- C. GWLB
- D. Classic

**42.** EC2-attached volume needing guaranteed high IOPS for a database:
- A. gp2
- B. st1
- C. io2
- D. sc1

**43.** Spiky, unpredictable compute with zero cost when idle:
- A. Reserved EC2
- B. Lambda
- C. Dedicated Host
- D. Fixed fleet

**44.** Ordered real-time stream, multiple independent consumers, replay:
- A. SQS Standard
- B. Kinesis Data Streams
- C. SNS
- D. SQS FIFO

**45.** Global non-HTTP TCP app: static anycast IPs, AWS backbone, fast Region failover:
- A. CloudFront
- B. Global Accelerator
- C. Route 53 weighted
- D. Transit Gateway

**46.** Route HTTP requests to different microservices based on URL path:
- A. NLB
- B. ALB
- C. GWLB
- D. Route 53 only

**47.** Stream data into S3/Redshift with no code and near-real-time delivery:
- A. Kinesis Data Firehose
- B. Kinesis Data Streams
- C. SQS
- D. Glue

**48.** Windows application needing shared SMB storage integrated with Active Directory:
- A. EFS
- B. FSx for Windows File Server
- C. S3
- D. EBS

**49.** HPC/ML workload needing a high-throughput file system linked to S3:
- A. EFS
- B. FSx for Lustre
- C. EBS st1
- D. Instance store

**50.** (Multiple response — choose TWO) Improve database read performance/scalability:
- A. Add read replicas
- B. Add an ElastiCache cache
- C. Reduce instance size
- D. Disable backups
- E. Use a single AZ

**51.** Orchestrate a multi-step workflow across Lambda functions with retries and branching:
- A. SQS
- B. Step Functions
- C. SNS
- D. EventBridge

**52.** Managed REST API front end for Lambda with throttling and authentication:
- A. ALB
- B. API Gateway
- C. CloudFront
- D. AppSync

**53.** Fault-tolerant batch processing at the lowest compute cost:
- A. On-Demand
- B. Reserved
- C. Spot
- D. Dedicated Host

**54.** Steady 24/7 workload for 3 years, max savings, flexible across EC2/Fargate/Lambda:
- A. Spot
- B. Compute Savings Plans
- C. On-Demand
- D. More ASG

**55.** Logs hot for 30 days, rare after, retain 7 years, hours-retrieval OK — cheapest:
- A. All in Standard
- B. Lifecycle to Standard-IA then Glacier Deep Archive, expire at 7y
- C. All in Glacier immediately
- D. One Zone-IA only

**56.** Unknown, changing S3 access pattern, minimize cost, no retrieval fees, no tuning:
- A. Standard-IA
- B. Intelligent-Tiering
- C. Glacier
- D. One Zone-IA

**57.** Private instances pull large data from S3 via NAT, incurring high charges — cut cost:
- A. Bigger NAT
- B. S3 Gateway Endpoint
- C. Public subnet
- D. Interface endpoint for S3

**58.** Alert when forecast monthly spend exceeds a threshold:
- A. Cost Explorer
- B. AWS Budgets
- C. Trusted Advisor
- D. CUR

**59.** Dev EC2 used only business hours — cheapest minimal-effort optimization:
- A. Reserved Instances
- B. Schedule stop/start off-hours
- C. Spot for dev
- D. Permanent downsize

**60.** Many accounts want one bill plus volume discounts:
- A. AWS Budgets
- B. Consolidated billing (Organizations)
- C. Savings Plans
- D. Separate invoices

**61.** Variable, intermittent relational DB workload, idle much of the time — cost-optimal:
- A. Largest RDS instance
- B. Aurora Serverless v2
- C. Multi-AZ always on
- D. DynamoDB provisioned

**62.** Right-sizing recommendations to reduce EC2 over-provisioning:
- A. Compute Optimizer
- B. CloudTrail
- C. Config
- D. GuardDuty

**63.** Cheapest S3 class for infrequently accessed, easily re-creatable data in one AZ:
- A. Standard
- B. One Zone-IA
- C. Glacier Deep Archive
- D. Intelligent-Tiering

**64.** Reduce data-transfer and improve performance for repeated content served from an origin:
- A. CloudFront caching
- B. Larger origin instance
- C. More NAT Gateways
- D. Cross-Region replication

**65.** (Multiple response — choose TWO) Cost-effective ways to handle unpredictable spiky traffic:
- A. AWS Lambda
- B. Fixed large EC2 fleet running 24/7
- C. DynamoDB On-Demand capacity
- D. Reserved Instances for peak
- E. Dedicated Hosts

---

## Answer Key & Rationale

| # | Ans | Why (note ref) |
|---|-----|----------------|
| 1 | B | IAM role, least privilege (01) |
| 2 | B | Only NACL denies at subnet (01/05) |
| 3 | B | Secrets Manager auto-rotates (01) |
| 4 | B | GuardDuty agentless threat detection (01) |
| 5 | B | SCP guardrail across accounts (01) |
| 6 | B | OAC + bucket policy (03/05) |
| 7 | B | CloudTrail = API audit (08) |
| 8 | B | WAF = L7 SQLi/XSS (01) |
| 9 | A | Macie = PII in S3 (01) |
| 10 | B | CloudHSM single-tenant custody (01) |
| 11 | B | IAM Identity Center SSO (01) |
| 12 | A, C | Versioning + MFA Delete (03/06) |
| 13 | B | SSE-KMS CMK = auditable/controlled (01/03) |
| 14 | B | Parameter Store free, no rotation (01) |
| 15 | B | AWS Config compliance rules (08) |
| 16 | B | SSM Session Manager, no SSH (08) |
| 17 | B | Cross-account role/bucket policy (01) |
| 18 | B | Shield Advanced (L7, cost protection, DRT) (01) |
| 19 | C | SG = stateful, allow-only (01) |
| 20 | B | ACM free managed certs (01) |
| 21 | B | ASG multi-AZ + ALB (06) |
| 22 | B | SQS decouple/buffer (06/07) |
| 23 | D | Backup & Restore cheapest (06) |
| 24 | B | RDS Multi-AZ failover (04/06) |
| 25 | B | Dead-letter queue (07) |
| 26 | B | Aurora Global Database (04/06) |
| 27 | B | Externalize sessions (06) |
| 28 | B | AWS Backup centralized (06) |
| 29 | C | Route 53 failover routing (05/06) |
| 30 | A, C | ASG across AZs + ELB health checks (06) |
| 31 | B | DynamoDB Global Tables (04) |
| 32 | B | NAT Gateway outbound only (05) |
| 33 | B | ASG + ELB health checks self-heal (06) |
| 34 | B | Transit Gateway hub (05) |
| 35 | B | Pilot light keeps core DB running (06) |
| 36 | A | ElastiCache caches repeated reads (04) |
| 37 | B | DAX microsecond reads (04) |
| 38 | B | CloudFront edge cache (05) |
| 39 | B | Athena SQL on S3 (04) |
| 40 | B | EFS shared multi-AZ Linux (03) |
| 41 | B | NLB static IP, L4 (02) |
| 42 | C | io2 guaranteed IOPS (03) |
| 43 | B | Lambda, no idle cost (02/09) |
| 44 | B | Kinesis Data Streams ordered/replay (07) |
| 45 | B | Global Accelerator static anycast IP (05) |
| 46 | B | ALB path-based routing (02) |
| 47 | A | Firehose → S3/Redshift no code (07) |
| 48 | B | FSx for Windows SMB + AD (03) |
| 49 | B | FSx for Lustre HPC/ML (03) |
| 50 | A, B | Read replicas + caching (04) |
| 51 | B | Step Functions orchestration (07) |
| 52 | B | API Gateway + Lambda (07) |
| 53 | C | Spot cheapest interruptible (02/09) |
| 54 | B | Compute Savings Plans (09) |
| 55 | B | Lifecycle → IA → Deep Archive (09) |
| 56 | B | Intelligent-Tiering (03/09) |
| 57 | B | S3 Gateway Endpoint free (05/09) |
| 58 | B | AWS Budgets alerts (09) |
| 59 | B | Schedule stop/start (09) |
| 60 | B | Consolidated billing (09) |
| 61 | B | Aurora Serverless v2 (04/09) |
| 62 | A | Compute Optimizer (09) |
| 63 | B | One Zone-IA (03/09) |
| 64 | A | CloudFront caching cuts egress + latency (05/09) |
| 65 | A, C | Lambda + DynamoDB On-Demand (09) |

**Scoring:** Count correct / 65. ≥47 (72%) = passing range. Below that, focus re-study on the domains where you missed most (note refs above point you to the topic).
