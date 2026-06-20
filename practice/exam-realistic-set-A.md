# Exam-Realistic Set A — 25 Hard, Scenario-Style Questions

**How this differs from the domain drills:** these mirror real SAA-C03 *difficulty* — long stems, multiple buried constraints, and distractors that work but aren't optimal. The trick on every one: find the **qualifier** (cheapest / least ops / most resilient / lowest latency) and eliminate options that meet the function but not the qualifier.

> ⚠️ Original questions written to mirror exam style — not real exam content. Cover the answers; full rationale (incl. why each distractor fails) at the bottom.

---

**A1.** A company runs a three-tier web app. The app tier on EC2 in a private subnet must call Amazon S3 and Amazon DynamoDB. Currently traffic routes through a NAT gateway, and the company wants to **reduce cost and keep traffic off the public internet** with the least operational effort. What should a solutions architect recommend?
- A. Create interface VPC endpoints (PrivateLink) for both S3 and DynamoDB
- B. Create gateway VPC endpoints for S3 and DynamoDB and update the route tables
- C. Move the app tier to a public subnet and attach Elastic IPs
- D. Add a second NAT gateway in another AZ for redundancy

**A2.** A media company stores 500 TB of video. Files are accessed heavily for the first 7 days, occasionally for 30 days, then almost never, but must be retained for 5 years and retrievable within 12 hours. The company wants the **lowest cost**. What should the architect implement?
- A. Store in S3 Standard with a lifecycle policy to S3 Standard-IA after 30 days
- B. Store in S3 Standard, transition to S3 Standard-IA after 7 days, then to S3 Glacier Deep Archive after 30 days
- C. Store everything in S3 Glacier Flexible Retrieval immediately
- D. Store in S3 Intelligent-Tiering with no lifecycle rules

**A3.** An application uses an Amazon RDS for PostgreSQL database. During business hours, read queries from a reporting tool slow down customer transactions. The company wants to **offload reporting reads with minimal application changes and no impact to write performance**. What should the architect do?
- A. Enable Multi-AZ deployment and point reports at the standby
- B. Create a read replica and direct the reporting tool to its endpoint
- C. Increase the instance size to the largest available
- D. Migrate the database to Amazon DynamoDB

**A4.** A company needs to run a nightly batch job that processes data for about 3 hours. The job is fault-tolerant and can restart if interrupted. The company wants the **most cost-effective** compute. What should the architect recommend?
- A. A fleet of On-Demand EC2 instances in an Auto Scaling group
- B. Reserved Instances sized for the nightly peak
- C. Spot Instances in an Auto Scaling group, with checkpointing
- D. A Dedicated Host

**A5.** A solutions architect must design a solution so that an application can **send a single event to multiple independent downstream systems**, each of which processes the event at its own pace and must not lose messages if it is temporarily offline. Which design meets these requirements?
- A. Publish to an Amazon SNS topic with each downstream system subscribed directly via HTTP
- B. Publish to an Amazon SNS topic that fans out to one Amazon SQS queue per downstream system
- C. Write events to a single Amazon SQS queue shared by all systems
- D. Invoke each downstream system synchronously from the producer

**A6.** A company's security policy requires that all data in an S3 bucket be encrypted at rest, that the encryption keys be **rotated automatically every year**, and that all key usage be **audited**. Which approach meets these requirements with the least operational overhead?
- A. SSE-S3 with Amazon S3-managed keys
- B. SSE-KMS with an AWS managed key
- C. SSE-KMS with a customer managed key and automatic rotation enabled
- D. SSE-C with customer-provided keys

**A7.** An ecommerce platform experiences unpredictable traffic spikes during flash sales. The backend writes orders to a database that must deliver **single-digit millisecond performance at any scale with no capacity planning**. Which database should the architect choose?
- A. Amazon RDS for MySQL with Multi-AZ
- B. Amazon Aurora provisioned
- C. Amazon DynamoDB with on-demand capacity
- D. Amazon Redshift

**A8.** A company hosts a public website on EC2 behind an Application Load Balancer. Security requires blocking traffic from a list of known-malicious IP ranges and protecting against SQL injection. Which combination meets these requirements? **(Choose TWO.)**
- A. Attach AWS WAF to the ALB with managed and IP-set rules
- B. Configure security group rules to deny the malicious IPs
- C. Use a network ACL to deny the malicious IP ranges
- D. Enable Amazon GuardDuty on the ALB
- E. Attach AWS Shield Standard to the EC2 instances

**A9.** A solutions architect needs a hybrid connection between an on-premises data center and AWS that provides **consistent, low-latency, high-bandwidth** throughput for a large data replication workload. The connection must avoid the public internet. What should the architect recommend?
- A. A Site-to-Site VPN over the internet
- B. AWS Direct Connect
- C. A VPC peering connection
- D. A NAT gateway with increased bandwidth

**A10.** An application running on EC2 must store user session state so that the fleet can scale in and out without logging users off. The solution must be **highly available and provide microsecond-to-millisecond latency**. What should the architect use?
- A. Store sessions on each instance's local EBS volume
- B. Store sessions in Amazon ElastiCache for Redis
- C. Enable ALB sticky sessions only
- D. Store sessions in Amazon S3

**A11.** A company is migrating a Windows-based application to AWS. The application requires a **shared file system using the SMB protocol with Active Directory integration**. Which service should the architect choose?
- A. Amazon EFS
- B. Amazon FSx for Windows File Server
- C. Amazon S3 with File Gateway
- D. Amazon FSx for Lustre

**A12.** A company wants to ensure that, across all accounts in its AWS Organization, **no one — including account administrators — can create resources outside the eu-west-1 Region**. What is the most effective way to enforce this?
- A. Apply an IAM permissions boundary to every user
- B. Apply a Service Control Policy at the organization root that denies actions outside eu-west-1
- C. Use an IAM policy attached to each account's admin group
- D. Configure AWS Config rules to flag non-compliant resources

**A13.** A solutions architect is designing a disaster recovery plan for a critical application. The business requires a **recovery time objective (RTO) of a few minutes** and is willing to pay for a scaled-down but fully functional copy of the environment running continuously in a second Region. Which DR strategy is this?
- A. Backup and restore
- B. Pilot light
- C. Warm standby
- D. Multi-site active/active

**A14.** A company runs a containerized microservices application and wants to **avoid managing any servers or control plane nodes**, while not being locked into Kubernetes. Which option requires the least operational overhead?
- A. Amazon EKS on EC2 worker nodes
- B. Amazon ECS on AWS Fargate
- C. Amazon ECS on EC2
- D. Self-managed Kubernetes on EC2

**A15.** An application stores sensitive files in S3. Compliance requires that objects **cannot be deleted or overwritten for a fixed retention period**, even by administrators. What should the architect enable?
- A. S3 Versioning only
- B. S3 Object Lock in compliance mode
- C. A restrictive bucket policy
- D. Cross-Region Replication

**A16.** A solutions architect must give an EC2-based application temporary, rotating credentials to access AWS services, with **no long-lived secrets stored on the instance**. What is the recommended approach?
- A. Create an IAM user and store access keys in the application config
- B. Attach an IAM role to the instance through an instance profile
- C. Store access keys in AWS Secrets Manager and fetch them at startup
- D. Hardcode credentials and rotate them monthly

**A17.** A global application serves dynamic and static content to users worldwide. The company wants to **reduce latency, offload the origin, and protect against common web exploits**. Which combination should the architect use? **(Choose TWO.)**
- A. Amazon CloudFront in front of the origin
- B. AWS Global Accelerator in front of the origin
- C. AWS WAF associated with the distribution
- D. A larger EC2 instance for the origin
- E. Route 53 multivalue answer routing

**A18.** A company needs to ingest a high-throughput, real-time clickstream. Multiple independent applications must consume the **same data in order**, and the company needs the ability to **replay** records from the last 24 hours. Which service meets these requirements?
- A. Amazon SQS Standard queue
- B. Amazon Kinesis Data Streams
- C. Amazon SNS
- D. Amazon Kinesis Data Firehose

**A19.** An application on EC2 must achieve the **highest, most consistent IOPS** for a transactional database volume. Which EBS volume type should the architect select?
- A. General Purpose SSD (gp3)
- B. Provisioned IOPS SSD (io2)
- C. Throughput Optimized HDD (st1)
- D. Cold HDD (sc1)

**A20.** A company wants to move 200 TB of archival data from its on-premises data center to Amazon S3. The site has a **slow internet connection** that would take months to transfer the data. What is the most appropriate solution?
- A. AWS DataSync over the internet
- B. AWS Snowball Edge
- C. An S3 multipart upload script
- D. A Site-to-Site VPN with DataSync

**A21.** A solutions architect needs to decouple a tightly coupled order-processing application so that a sudden surge of orders does not overwhelm or crash the processing tier, and **no orders are lost**. Which approach best meets this requirement?
- A. Add an Amazon SQS queue between the front end and the processing tier; scale consumers on queue depth
- B. Increase the size of the processing EC2 instances
- C. Add an Application Load Balancer in front of the processing tier
- D. Move the processing tier to a larger Auto Scaling group only

**A22.** A team needs a serverless way to run a sequence of steps — call a Lambda function, wait for a manual approval, then call another function — with **built-in error handling, retries, and visual tracking**. Which service should they use?
- A. Amazon SQS
- B. AWS Step Functions
- C. Amazon EventBridge
- D. AWS Lambda destinations

**A23.** A company runs a fleet of EC2 instances behind a load balancer and needs the load balancer to provide a **static IP address** and operate at **ultra-low latency for a TCP-based protocol**. Which load balancer should the architect choose?
- A. Application Load Balancer
- B. Network Load Balancer
- C. Gateway Load Balancer
- D. Classic Load Balancer

**A24.** A reporting workload runs complex analytical SQL queries across petabytes of structured data for business intelligence dashboards. Which service is **purpose-built** for this?
- A. Amazon RDS for PostgreSQL
- B. Amazon Athena
- C. Amazon Redshift
- D. Amazon DynamoDB

**A25.** A development team's EC2 instances are only needed Monday–Friday, 8am–6pm. Management wants to **cut costs with minimal engineering effort** and no commitment. What should the architect recommend?
- A. Purchase Reserved Instances for the dev fleet
- B. Use AWS Instance Scheduler to stop instances outside business hours
- C. Switch the dev fleet to Spot Instances
- D. Move the workload to the smallest possible instance type permanently

---

## Answer Key — with why each distractor fails

**A1 — B.** Gateway endpoints (S3 & DynamoDB) are **free**, keep traffic on the AWS backbone, and replace the NAT path. *A:* interface endpoints work but cost per hour + per GB — not the cheapest, and S3/DynamoDB specifically support free gateway endpoints. *C:* public subnet increases exposure (fails "off the internet"). *D:* adds cost, doesn't address the requirement.

**A2 — B.** Tiered lifecycle: hot → Standard-IA after 7 days → Deep Archive after 30 days matches the access curve and 12-hour retrieval tolerance at lowest cost. *A:* never archives cold data — far more expensive over 5 years. *C:* Glacier from day 1 hurts the heavy early access (retrieval fees/latency). *D:* Intelligent-Tiering carries monitoring fees and won't reach Deep Archive pricing without configuration.

**A3 — B.** A **read replica** offloads reporting reads with just an endpoint change and no write-path impact. *A:* the Multi-AZ standby is **not readable** — common trap. *C:* scaling up is costly and doesn't isolate reporting load. *D:* a full re-platform is excessive and changes the data model.

**A4 — C.** **Spot** is cheapest for fault-tolerant, restartable batch; checkpointing handles interruptions. *A:* On-Demand is far more expensive. *B:* RIs are for steady 24/7 usage, not a 3-hour nightly job. *D:* Dedicated Hosts are for licensing/compliance, most expensive.

**A5 — B.** **SNS → one SQS queue per consumer** = fan-out + durability + independent pace, with no message loss if a consumer is offline (the queue holds messages). *A:* SNS direct HTTP delivery has no durable buffer — an offline consumer loses messages. *C:* a single shared queue means consumers compete and each event is processed once, not delivered to all. *D:* synchronous coupling is exactly what we're avoiding.

**A6 — C.** **SSE-KMS + customer managed key with automatic rotation** gives yearly rotation **and** CloudTrail audit of key usage with minimal ops. *A:* SSE-S3 gives no per-key audit/rotation control. *B:* AWS managed keys rotate but offer less control and limited policy/audit granularity. *D:* SSE-C makes you manage keys yourself — high overhead, no AWS-side audit.

**A7 — C.** **DynamoDB on-demand** = single-digit ms at any scale, no capacity planning, ideal for unpredictable spikes. *A/B:* RDS/Aurora need capacity sizing and don't match "any scale, no planning" as cleanly. *D:* Redshift is analytical (OLAP), not transactional.

**A8 — A and C.** **WAF** blocks SQL injection (and can use IP-set rules); a **NACL** can explicitly deny IP ranges at the subnet. *B:* security groups can't deny — allow-only. *D:* GuardDuty detects, it doesn't block, and doesn't attach to an ALB. *E:* Shield Standard is automatic DDoS protection, not IP/SQLi filtering.

**A9 — B.** **Direct Connect** = dedicated, consistent low-latency, high-bandwidth, private. *A:* VPN rides the public internet with variable latency. *C:* VPC peering connects VPCs, not on-prem. *D:* NAT is for outbound internet, irrelevant.

**A10 — B.** **ElastiCache for Redis** = HA (Multi-AZ/replication), fast in-memory session store, decoupled from instances. *A:* local EBS dies with the instance — sessions lost on scale-in. *C:* sticky sessions still lose state when the target is removed. *D:* S3 isn't designed for low-latency session reads.

**A11 — B.** **FSx for Windows File Server** = SMB + AD integration. *A:* EFS is NFS/Linux. *C:* File Gateway is hybrid S3-backed, not a native Windows file system. *D:* Lustre is HPC, not general Windows SMB.

**A12 — B.** A **Region-deny SCP** at the org root is the only option that caps permissions for *everyone including admins* across all accounts. *A:* permissions boundaries are per-identity and easy to miss/bypass at scale. *C:* per-account IAM policies can be changed by admins. *D:* Config only detects after the fact, it doesn't prevent.

**A13 — C.** A continuously running, scaled-down full copy = **warm standby** (RTO minutes). *A:* backup/restore is hours. *B:* pilot light keeps only core (e.g., DB) running, app spun up on demand — slower. *D:* active/active runs full prod in both Regions (near-zero RTO, higher cost than described).

**A14 — B.** **ECS on Fargate** = no servers, no control plane, no Kubernetes. *A/C:* you manage EC2 worker nodes. *D:* fully self-managed = most overhead.

**A15 — B.** **Object Lock in compliance mode** enforces WORM so even root can't delete within the retention period. *A:* versioning alone doesn't prevent deletion. *C:* bucket policies can be changed by admins. *D:* replication is for DR, not immutability.

**A16 — B.** **IAM role via instance profile** delivers temporary auto-rotating credentials with nothing stored. *A/D:* long-lived keys on the instance. *C:* still stores/handles credentials and adds overhead vs. a role.

**A17 — A and C.** **CloudFront** reduces latency and offloads the origin for static+dynamic content and integrates **WAF** for exploit protection. *B:* Global Accelerator doesn't cache content (no origin offload). *D:* bigger origin doesn't reduce global latency. *E:* multivalue routing is basic DNS LB, not caching/protection.

**A18 — B.** **Kinesis Data Streams** = ordered, multiple independent consumers, replay within retention. *A:* SQS doesn't preserve order across consumers or replay. *C:* SNS is pub/sub push, no replay/ordering. *D:* Firehose just delivers to destinations — no multi-consumer replay.

**A19 — B.** **io2** = provisioned, consistent high IOPS for databases. *A:* gp3 is good general-purpose but lower guaranteed IOPS. *C/D:* HDD types are throughput/cold, low IOPS.

**A20 — B.** **Snowball Edge** physically ships the data — right when the network is too slow. *A/D:* online transfer over a slow link takes months. *C:* same network bottleneck.

**A21 — A.** **SQS** decouples and buffers the surge; messages persist and consumers scale on depth — no loss. *B/D:* scaling compute doesn't prevent loss if the tier still gets overwhelmed and there's no durable buffer. *C:* an ALB distributes load but provides no durable buffer/retry.

**A22 — B.** **Step Functions** orchestrates sequenced steps with built-in retries, error handling, human-approval (callback) patterns, and visual tracking. *A/C:* queue/event bus don't orchestrate ordered multi-step workflows with approval gates. *D:* destinations route async results, not full orchestration.

**A23 — B.** **NLB** = static IP + ultra-low-latency L4 (TCP). *A:* ALB is L7, no static IP. *C:* GWLB is for third-party appliances. *D:* Classic is legacy, no static IP guarantee.

**A24 — C.** **Redshift** is the purpose-built petabyte-scale data warehouse for BI/analytical SQL. *A:* RDS is OLTP. *B:* Athena suits ad-hoc S3 querying, not sustained heavy BI dashboards at PB scale. *D:* DynamoDB is NoSQL, not analytical SQL.

**A25 — B.** **Scheduling stop/start** (Instance Scheduler) cuts cost with minimal effort and no commitment. *A:* RIs require commitment and suit steady usage. *C:* Spot can be reclaimed and adds complexity for a predictable dev schedule. *D:* permanent downsize may break the workload and doesn't address idle hours.
