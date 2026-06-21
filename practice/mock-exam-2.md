# Mock Exam 2 — Full 65-Question Simulation (SAA-C03)

**Instructions:** Set a timer for **130 minutes**. Answer all 65 questions before checking the key. Pass mark ≈ 72% (≈47/65). Questions are interleaved across domains like the real exam; multi-response questions say **(Choose TWO).** Review every miss back to the referenced note.

Domain mix mirrors the real exam: Secure (~30%), Resilient (~26%), High-Performing (~24%), Cost (~20%). These are **fresh** questions — no overlap with mock-exam-1 or the Set A/B/C drills.

---

**1.** A company needs S3 objects encrypted at rest with keys it controls and can audit via CloudTrail, but wants AWS to handle the cryptographic operations. Which option?
- A. SSE-S3
- B. SSE-KMS with a customer managed key
- C. SSE-C
- D. Client-side encryption with a custom KMS-free library

**2.** A fleet of EC2 web servers behind an ALB must scale automatically and survive an AZ outage. What is the minimum design?
- A. One large instance with a CloudWatch reboot alarm
- B. An Auto Scaling group spanning ≥2 AZs behind the ALB
- C. Two instances in one AZ behind the ALB
- D. Instances in two AZs without Auto Scaling

**3.** A solution must guarantee that an SQS message is processed **in order** and **exactly once** for a payment pipeline. Which queue type?
- A. SQS standard queue
- B. SQS FIFO queue
- C. SNS standard topic
- D. Kinesis Data Firehose

**4.** A startup wants the cheapest compute for a nightly, fault-tolerant data-processing job that can restart if interrupted. Which option?
- A. On-Demand instances
- B. Reserved Instances
- C. Spot Instances
- D. Dedicated Hosts

**5.** Which service answers "which IAM principal deleted this S3 bucket and when?"
- A. AWS Config
- B. CloudTrail
- C. CloudWatch Metrics
- D. VPC Flow Logs

**6.** An application in a private subnet must call Amazon S3 frequently. The team wants to eliminate NAT gateway data-processing charges for this traffic. What should they add?
- A. An interface VPC endpoint for S3
- B. A gateway VPC endpoint for S3
- C. A second NAT gateway
- D. An internet gateway

**7.** A relational database must fail over automatically within ~60 seconds if its AZ fails, with no application logic changes. What should be enabled?
- A. RDS read replica
- B. RDS Multi-AZ
- C. Larger instance class
- D. DynamoDB global tables

**8.** A company needs a managed NoSQL database delivering single-digit millisecond reads with automatic scaling and zero capacity planning for a spiky workload. Which configuration?
- A. DynamoDB with provisioned capacity, no auto scaling
- B. DynamoDB with on-demand capacity
- C. RDS for MySQL
- D. ElastiCache for Memcached

**9.** A company wants a guardrail preventing any account in its Organization from disabling encryption settings, enforceable even on account admins. What should they use?
- A. IAM permissions boundary
- B. Service Control Policy
- C. IAM group policy
- D. Resource-based policy

**10.** A web tier stores user session state locally, preventing smooth scale-in. Where should sessions be moved for low-latency, highly available access? **(Choose TWO.)**
- A. ElastiCache for Redis
- B. DynamoDB
- C. Instance store
- D. A single EC2 "session server"
- E. The ALB only via sticky sessions

**11.** A solution must deliver static and dynamic web content globally with low latency and cache static assets near users. Which service?
- A. Global Accelerator
- B. CloudFront
- C. Route 53 latency routing
- D. An NLB

**12.** A company needs the absolute cheapest S3 storage for compliance archives accessed maybe once a year, where a 12-hour retrieval is acceptable. Which class?
- A. S3 Standard-IA
- B. S3 Glacier Instant Retrieval
- C. S3 Glacier Deep Archive
- D. S3 One Zone-IA

**13.** A latency-sensitive trading application needs the lowest possible network latency between its tightly-coupled EC2 nodes. Which placement strategy?
- A. Spread placement group
- B. Cluster placement group
- C. Partition placement group
- D. Multiple AZs behind an NLB

**14.** An EC2 instance must access AWS services with temporary, automatically rotated credentials and no stored secrets. What should be used?
- A. IAM user access keys in the AMI
- B. An IAM role via instance profile
- C. Credentials in Secrets Manager fetched at boot
- D. Hardcoded keys rotated quarterly

**15.** A company wants to ingest a high-throughput clickstream where multiple independent consumers read the same records in order and can replay the last 24 hours. Which service?
- A. SQS standard
- B. Kinesis Data Streams
- C. SNS
- D. Kinesis Data Firehose

**16.** A solution must block traffic from a specific malicious /24 IP range at the subnet boundary. What should be configured?
- A. Security group deny rule
- B. Network ACL deny rule
- C. IAM policy deny
- D. Route 53 health check

**17.** A company runs Windows applications needing a shared file system over SMB with Active Directory integration. Which service?
- A. Amazon EFS
- B. Amazon FSx for Windows File Server
- C. Amazon FSx for Lustre
- D. S3 File Gateway

**18.** A reporting workload runs complex analytical SQL over petabytes of structured historical data for BI dashboards. Which service is purpose-built?
- A. Amazon Athena
- B. Amazon Redshift
- C. Amazon RDS
- D. Amazon DynamoDB

**19.** A company must retain S3 objects so they cannot be deleted or overwritten for 7 years, even by the root user. What should be enabled?
- A. Versioning only
- B. S3 Object Lock in compliance mode
- C. A deny-delete bucket policy
- D. MFA Delete only

**20.** A spiky, event-driven backend should run code only when triggered, with no idle cost and minimal operations. Which compute?
- A. EC2 Auto Scaling group
- B. AWS Lambda
- C. ECS on EC2
- D. Reserved Instances

**21.** A company needs to connect 60 VPCs and its on-premises network with transitive routing through a central hub. Which service?
- A. VPC peering between all VPCs
- B. AWS Transit Gateway
- C. Multiple internet gateways
- D. A NAT gateway mesh

**22.** A mobile app must let users sign in with social identity providers and then upload to user-specific S3 prefixes using temporary AWS credentials. Which Cognito components? **(Choose TWO.)**
- A. Cognito user pool
- B. Cognito identity pool
- C. IAM Identity Center
- D. STS GetSessionToken with stored keys
- E. A custom OAuth server on EC2

**23.** A company wants to be alerted before its monthly AWS spend exceeds $10,000. Which tool?
- A. Cost Explorer
- B. AWS Budgets
- C. Trusted Advisor
- D. Compute Optimizer

**24.** A microservices app must send one order event to three independent systems, each processing at its own pace without losing messages if offline. Which design?
- A. SNS topic with three direct HTTP subscribers
- B. SNS topic fanning out to one SQS queue per system
- C. One shared SQS queue
- D. Synchronous calls to each system

**25.** A workload needs the highest, most consistent IOPS for a transactional database volume on EC2. Which EBS type?
- A. gp3
- B. io2
- C. st1
- D. sc1

**26.** A team needs agentless threat detection analyzing CloudTrail, VPC Flow Logs, and DNS logs. Which service?
- A. Amazon Inspector
- B. Amazon GuardDuty
- C. Amazon Macie
- D. AWS Config

**27.** A company must migrate 300 TB to S3 over a slow internet link that would take months. What is appropriate?
- A. DataSync over the internet
- B. AWS Snowball Edge
- C. Multipart upload script
- D. Site-to-Site VPN + DataSync

**28.** A solution needs a static IP for a TCP load balancer with ultra-low latency for millions of requests per second. Which load balancer?
- A. ALB
- B. NLB
- C. GWLB
- D. Classic LB

**29.** A development environment is only used weekdays 8am–6pm. Management wants the simplest cost reduction with no commitment. What should be done?
- A. Buy Reserved Instances
- B. Use AWS Instance Scheduler to stop instances off-hours
- C. Move to Spot Instances
- D. Permanently downsize the instances

**30.** A company wants to detect PII stored in free-text customer feedback before sharing it with analysts. Which service detects the PII entities in the text?
- A. Amazon Macie
- B. Amazon Comprehend
- C. Amazon Textract
- D. Amazon GuardDuty

**31.** A relational database in two Regions must accept writes in both and replicate in under a second, with fast regional failover. Which option?
- A. RDS Multi-AZ
- B. Aurora Global Database with write forwarding
- C. RDS cross-Region read replica
- D. DynamoDB global tables

**32.** A serverless API must authenticate end users via email/password and social login, returning a token the API validates. Which authorizer?
- A. Cognito user pool authorizer
- B. Cognito identity pool
- C. An IAM role on API Gateway
- D. A Lambda calling LDAP

**33.** A company wants the cheapest DR strategy where hours of downtime are acceptable. Which strategy?
- A. Backup & Restore
- B. Pilot Light
- C. Warm Standby
- D. Multi-Site Active/Active

**34.** A team needs to extract form fields and table data from thousands of scanned PDFs daily. Which service?
- A. Amazon Rekognition
- B. Amazon Textract
- C. Amazon Comprehend
- D. Amazon Transcribe

**35.** EC2 instances assigned only IPv6 must reach the internet for updates but stay unreachable inbound. What should be configured?
- A. NAT gateway
- B. Egress-only internet gateway
- C. Internet gateway with SG rules
- D. NAT instance

**36.** A company wants centralized backups with policy-based retention across EBS, RDS, DynamoDB, and EFS from one place. Which service?
- A. AWS Backup
- B. S3 lifecycle policies
- C. Manual snapshots
- D. Data Lifecycle Manager only

**37.** A read-heavy RDS database has reporting queries slowing transactions. The fix must offload reads with minimal app change and no write-path impact. What should be done?
- A. Multi-AZ and read from standby
- B. Create a read replica and point reports to it
- C. Scale up the instance
- D. Migrate to DynamoDB

**38.** A company needs partners to upload files via managed SFTP that land directly in S3, with no servers to operate. Which service?
- A. AWS DataSync
- B. AWS Transfer Family
- C. File Gateway
- D. EC2 SFTP server

**39.** A solution needs a serverless way to orchestrate a sequence: invoke a Lambda, wait for human approval, then invoke another, with retries and visual tracking. Which service?
- A. SQS
- B. AWS Step Functions
- C. EventBridge
- D. SNS

**40.** A company wants product recommendations on its site based on user behavior, with minimal ML work. Which service?
- A. Amazon Personalize
- B. Amazon Forecast
- C. Amazon Comprehend
- D. Amazon Kendra

**41.** Which is the most cost-effective option for steady, predictable EC2 usage running 24/7 for three years, with flexibility across instance families and Fargate?
- A. On-Demand
- B. Compute Savings Plan
- C. Spot Instances
- D. Standard Reserved Instance locked to one family

**42.** A company must give auditors immutable proof of every API action for compliance, retained for years in S3. What should be configured?
- A. CloudWatch Logs only
- B. A CloudTrail trail delivering to S3 (with Object Lock)
- C. AWS Config only
- D. VPC Flow Logs to S3

**43.** A high-performance computing job needs a shared file system with extremely high throughput, linked to data in S3. Which service?
- A. Amazon EFS
- B. Amazon FSx for Lustre
- C. Amazon FSx for Windows
- D. Amazon EBS Multi-Attach

**44.** A company wants natural-language enterprise search across millions of internal documents, returning precise answers. Which service?
- A. Amazon OpenSearch Service
- B. Amazon Kendra
- C. Amazon CloudSearch
- D. Amazon Athena

**45.** A solution must reduce DynamoDB read latency from milliseconds to microseconds for a read-heavy workload. What should be added?
- A. A read replica
- B. DynamoDB Accelerator (DAX)
- C. ElastiCache for Memcached
- D. Global tables

**46.** A static website in a private S3 bucket must be served only through CloudFront, never directly. What should be used?
- A. Make the bucket public
- B. Origin Access Control (OAC) + bucket policy restricting to the distribution
- C. S3 Transfer Acceleration
- D. Presigned URLs for every object

**47.** A company already uses Apache Kafka and wants a managed equivalent on AWS with minimal code changes. Which service?
- A. Kinesis Data Streams
- B. Amazon MSK
- C. SQS
- D. Kinesis Firehose

**48.** A solution must run containers with no servers or control-plane nodes to manage, and without requiring Kubernetes. Which option?
- A. EKS on EC2
- B. ECS on Fargate
- C. ECS on EC2
- D. Self-managed Kubernetes

**49.** A company wants interactive BI dashboards for executives sourced from Athena and Redshift, serverless and per-user. Which service?
- A. Amazon QuickSight
- B. Amazon OpenSearch Dashboards
- C. Self-hosted Grafana
- D. Amazon Redshift query editor

**50.** A workload's memory utilization must be alarmed on, but the metric isn't in CloudWatch by default. What's required?
- A. Enable detailed monitoring
- B. Install the CloudWatch agent for memory as a custom metric
- C. Use instance metadata
- D. Enable Flow Logs

**51.** A company needs cross-Region disaster recovery for a relational database with sub-second replication lag and promotion in under a minute. Which option?
- A. RDS Multi-AZ
- B. Aurora Global Database
- C. DynamoDB global tables
- D. Daily snapshots copied cross-Region

**52.** A streaming pipeline must deliver records to S3 and Redshift with no consumer code and no servers, buffering by time/size. Which service?
- A. Kinesis Data Streams + KCL
- B. Kinesis Data Firehose
- C. MSK
- D. SQS + Lambda

**53.** A company needs to route AWS service events and run scheduled cron-style jobs, filtering events by content to different targets. Which service?
- A. SNS
- B. Amazon EventBridge
- C. SQS
- D. Step Functions

**54.** A company wants up to ~72% savings on a steady workload but the ability to **sell** the commitment if it no longer needs it. Which option?
- A. Standard Reserved Instance (RI Marketplace)
- B. Compute Savings Plan
- C. Convertible RI
- D. Spot Instances

**55.** A call center wants to transcribe support calls and analyze sentiment with no ML expertise. Which combination? **(Choose TWO.)**
- A. Amazon Transcribe
- B. Amazon Polly
- C. Amazon Comprehend
- D. Amazon Translate
- E. SageMaker custom model

**56.** A company must securely access EC2 instances for administration without opening port 22, using a bastion, or managing SSH keys. Which service?
- A. SSM Session Manager
- B. A public bastion host
- C. Direct SSH with a security group
- D. AWS Client VPN only

**57.** A solution needs the cheapest storage for infrequently accessed, easily re-creatable data where single-AZ durability is acceptable. Which S3 class?
- A. S3 Standard
- B. S3 Standard-IA
- C. S3 One Zone-IA
- D. S3 Glacier Deep Archive

**58.** A company needs a hybrid connection with consistent low latency and high bandwidth that avoids the public internet. Which option?
- A. Site-to-Site VPN
- B. AWS Direct Connect
- C. VPC peering
- D. Transit Gateway alone

**59.** A solution must protect a public web app against SQL injection and bad bots at Layer 7. Which service?
- A. Security groups
- B. AWS WAF
- C. Shield Standard
- D. NACL

**60.** A company runs a large self-managed Cassandra cluster on EC2 in one AZ and wants a single rack failure to affect at most one subset of nodes. Which placement group?
- A. Cluster
- B. Spread
- C. Partition
- D. None

**61.** A company wants to give remote employees managed, persistent cloud desktops without managing VDI infrastructure. Which service?
- A. Amazon AppStream 2.0
- B. Amazon WorkSpaces
- C. EC2 with RDP
- D. AWS Outposts

**62.** A solution needs to centrally enforce table- and column-level permissions on an S3 data lake queried by Athena, Redshift Spectrum, and EMR. Which service?
- A. Bucket policies
- B. AWS Lake Formation
- C. IAM policies per service
- D. AWS Config

**63.** A company wants the cheapest EBS option for infrequently accessed, throughput-light data where lowest cost matters more than performance. Which type?
- A. gp3
- B. io2
- C. st1
- D. sc1

**64.** A company needs near-zero-downtime cross-Region failover with a fixed set of static IPs for a non-HTTP TCP application, routed over the AWS backbone. Which service?
- A. CloudFront
- B. AWS Global Accelerator
- C. Route 53 weighted routing
- D. An NLB per Region only

**65.** A company wants to reduce origin load and data-transfer-out costs for a globally accessed, cacheable website. Which service?
- A. Global Accelerator
- B. CloudFront
- C. A larger origin instance
- D. Cross-Region replication

---

## Answer Key — with rationale and note references

| # | Ans | Why (note) |
|---|-----|------------|
| 1 | B | SSE-KMS CMK = customer-controlled keys + CloudTrail audit, AWS does the crypto (01/03) |
| 2 | B | ASG across ≥2 AZs behind ALB = elastic + AZ-resilient (02/06) |
| 3 | B | FIFO queue = ordered + exactly-once (07) |
| 4 | C | Spot cheapest for fault-tolerant/restartable batch (02/09) |
| 5 | B | CloudTrail = who did what API call (01/08) |
| 6 | B | Gateway endpoint for S3 is free, removes NAT charges (05/09) |
| 7 | B | Multi-AZ = automatic synchronous failover (04/06) |
| 8 | B | DynamoDB on-demand = ms latency, no capacity planning (04) |
| 9 | B | SCP caps permissions across accounts incl. admins (01) |
| 10 | A, B | Redis or DynamoDB = external, HA, low-latency session store (06) |
| 11 | B | CloudFront caches static + serves dynamic globally (05) |
| 12 | C | Glacier Deep Archive cheapest, 12h retrieval OK (03/09) |
| 13 | B | Cluster placement group = lowest inter-node latency (02) |
| 14 | B | IAM role/instance profile = temporary rotating creds, no secrets (01) |
| 15 | B | Kinesis Data Streams = ordered, multi-consumer, replay (07) |
| 16 | B | NACL deny at subnet level (only one that denies by IP) (01/05) |
| 17 | B | FSx for Windows = SMB + AD (03) |
| 18 | B | Redshift = petabyte OLAP/BI warehouse (04) |
| 19 | B | Object Lock compliance mode = WORM, even root can't delete (03/06) |
| 20 | B | Lambda = event-driven, no idle cost (02/09) |
| 21 | B | Transit Gateway = transitive hub for many VPCs + on-prem (05) |
| 22 | A, B | User pool (sign-in) + identity pool (temp AWS creds) (01) |
| 23 | B | AWS Budgets alerts before threshold (09) |
| 24 | B | SNS → one SQS per consumer = durable fan-out, independent pace (07) |
| 25 | B | io2 = provisioned, consistent high IOPS (03) |
| 26 | B | GuardDuty = agentless threat detection on those logs (01) |
| 27 | B | Snowball Edge for huge data over slow link (03) |
| 28 | B | NLB = static IP, L4, ultra-low latency, high throughput (02) |
| 29 | B | Instance Scheduler stop/start = simplest, no commitment (09) |
| 30 | B | Comprehend detects PII entities in text (10) |
| 31 | B | Aurora Global Database + write forwarding = relational active-active (04) |
| 32 | A | Cognito user pool authorizer on API Gateway (01) |
| 33 | A | Backup & Restore = cheapest, hours RTO (06) |
| 34 | B | Textract extracts form/table data from documents (10) |
| 35 | B | Egress-only IGW = IPv6 outbound-only (05) |
| 36 | A | AWS Backup = centralized cross-service backups (06) |
| 37 | B | Read replica offloads reads, no write impact, minimal change (04) |
| 38 | B | AWS Transfer Family = managed SFTP into S3 (03) |
| 39 | B | Step Functions = orchestration with approval, retries, visual (07) |
| 40 | A | Personalize = behavior-based recommendations (10) |
| 41 | B | Compute Savings Plan = steady + cross-family/Fargate flexibility (02/09) |
| 42 | B | CloudTrail trail → S3 (Object Lock) = immutable API audit (08) |
| 43 | B | FSx for Lustre = high-throughput HPC, S3-linked (03) |
| 44 | B | Kendra = natural-language enterprise search (10) |
| 45 | B | DAX = microsecond DynamoDB reads (04) |
| 46 | B | OAC + bucket policy locks S3 to CloudFront (01/05) |
| 47 | B | MSK = managed Kafka, minimal change (07/10) |
| 48 | B | ECS on Fargate = no servers, no control plane, no K8s (02) |
| 49 | A | QuickSight = serverless BI dashboards (10) |
| 50 | B | CloudWatch agent publishes memory as custom metric (08) |
| 51 | B | Aurora Global Database = sub-second cross-Region DR (04/06) |
| 52 | B | Firehose = zero-code delivery to S3/Redshift (07) |
| 53 | B | EventBridge = event routing + cron + content filtering (07) |
| 54 | A | Standard RI can be sold on the RI Marketplace (02) |
| 55 | A, C | Transcribe (audio→text) + Comprehend (sentiment) (10) |
| 56 | A | SSM Session Manager = no SSH/bastion/keys (08) |
| 57 | C | One Zone-IA = cheap, single-AZ, re-creatable data (03/09) |
| 58 | B | Direct Connect = consistent low-latency private link (05) |
| 59 | B | WAF = L7 SQLi/bot protection (01) |
| 60 | C | Partition placement group isolates rack failures per partition (02) |
| 61 | B | WorkSpaces = managed persistent virtual desktops (10) |
| 62 | B | Lake Formation = fine-grained data-lake permissions (10) |
| 63 | D | sc1 (Cold HDD) = cheapest EBS (03) |
| 64 | B | Global Accelerator = static anycast IPs + fast TCP failover (05) |
| 65 | B | CloudFront caching cuts origin load + egress cost (05/09) |

**Scoring:** Count correct / 65. ≥47 (72%) = passing range. Below that, note which domain your misses cluster in and re-read those notes before retesting.
