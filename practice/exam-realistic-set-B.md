# Exam-Realistic Set B — 25 Hard, Scenario-Style Questions

Same difficulty calibration as Set A. These lean harder into the **"all options work, pick the best"** pattern and the multi-constraint stems the real exam uses. Watch the qualifier.

> ⚠️ Original questions written to mirror exam style — not real exam content.

---

**B1.** A company runs a web app on EC2 in an Auto Scaling group behind an ALB across two Availability Zones. The database is a single RDS instance in one AZ. During an AZ outage, the app stayed up but the database became unreachable. What is the **most cost-effective** change to eliminate this single point of failure?
- A. Convert the RDS instance to a Multi-AZ deployment
- B. Create a read replica in the second AZ
- C. Migrate to Amazon DynamoDB global tables
- D. Take more frequent automated snapshots

**B2.** A SaaS company must isolate workloads for hundreds of customers and apply different security guardrails per environment, while keeping a **single consolidated bill**. Which approach is best?
- A. One AWS account with separate VPCs per customer
- B. AWS Organizations with separate accounts per workload/environment and SCPs
- C. One account with IAM groups per customer
- D. Separate AWS Organizations per customer

**B3.** An application uploads images to S3, and each upload must trigger a Lambda function to generate thumbnails. The solution must be **fully serverless and event-driven with no polling**. What should the architect configure?
- A. An S3 event notification that invokes the Lambda function
- B. A CloudWatch alarm that triggers Lambda
- C. A scheduled EventBridge rule that scans the bucket every minute
- D. An SQS queue that the Lambda function polls continuously

**B4.** A company's Lambda functions connect directly to an RDS database and exhaust the database's connections during traffic spikes. What should the architect add to **manage connections efficiently with minimal code changes**?
- A. Increase the RDS instance size
- B. Add Amazon RDS Proxy between Lambda and the database
- C. Move the database to DynamoDB
- D. Add a read replica

**B5.** A financial firm must retain audit logs of **every AWS API call** across all accounts, store them immutably, and detect if anyone tries to disable logging. Which combination meets this? **(Choose TWO.)**
- A. Enable an organization trail in AWS CloudTrail delivering to a central S3 bucket
- B. Enable VPC Flow Logs in every VPC
- C. Apply S3 Object Lock and an SCP preventing CloudTrail from being disabled
- D. Use Amazon Macie to scan the logs
- E. Enable AWS Config only

**B6.** An application needs to serve read traffic from the geographically **nearest of several Regions** to minimize latency, automatically routing users to the closest healthy endpoint. Which Route 53 routing policy should the architect use?
- A. Weighted
- B. Latency-based
- C. Geolocation
- D. Failover

**B7.** A company wants to run a relational database that **automatically scales capacity up and down** to match an intermittent, unpredictable workload and avoid paying for idle capacity. Which option is best?
- A. Amazon RDS for MySQL on a large instance
- B. Amazon Aurora Serverless v2
- C. Amazon Redshift
- D. Amazon RDS with Multi-AZ

**B8.** A solutions architect must allow an application in Account A to read objects from an S3 bucket owned by Account B, following **least-privilege best practices**. What is the recommended approach?
- A. Make the bucket public
- B. Create an IAM role in Account B that Account A can assume, or a bucket policy granting Account A's role access to the specific prefix
- C. Share Account B's IAM user access keys with Account A
- D. Use a NAT gateway to bridge the accounts

**B9.** An online game needs a leaderboard that supports **real-time ranking with sub-millisecond reads and sorted-set operations**. Which service is best suited?
- A. Amazon DynamoDB
- B. Amazon ElastiCache for Redis
- C. Amazon RDS
- D. Amazon S3

**B10.** A company runs an application across two Regions for disaster recovery. It needs a **single static IP entry point** that routes users to the healthy Region and fails over within seconds for a **non-HTTP TCP protocol**. Which service should the architect choose?
- A. Amazon CloudFront
- B. AWS Global Accelerator
- C. Route 53 weighted routing
- D. An internet-facing NLB in each Region only

**B11.** A company must process a stream of IoT sensor data and **load it into Amazon S3 and Amazon Redshift in near real time with no servers to manage**. Which service should the architect use?
- A. Amazon Kinesis Data Firehose
- B. Amazon Kinesis Data Streams with a custom consumer
- C. Amazon SQS
- D. AWS Batch

**B12.** A static marketing website experiences global traffic. The company wants the **lowest cost and least operational overhead** to host and serve it with low latency. What should the architect recommend?
- A. EC2 instances behind an ALB in multiple Regions
- B. An S3 bucket configured for static website hosting, fronted by Amazon CloudFront
- C. Elastic Beanstalk with a load balancer
- D. A fleet of Lightsail instances

**B13.** A company must encrypt EBS volumes and their snapshots for a compliance audit, with **keys managed by AWS but auditable**. The simplest way to ensure all new volumes are encrypted by default is to:
- A. Manually enable encryption when creating each volume
- B. Enable EBS encryption by default in the account/Region using a KMS key
- C. Use instance store volumes instead
- D. Encrypt data only at the application layer

**B14.** An application's batch tier reads messages from an SQS queue. Occasionally a malformed message causes the consumer to fail repeatedly, blocking the queue. How should the architect prevent this **without losing the problematic messages**?
- A. Reduce the visibility timeout to zero
- B. Configure a dead-letter queue with a maxReceiveCount
- C. Switch to a FIFO queue
- D. Delete messages that fail once

**B15.** A company needs to grant temporary, federated access to its AWS resources for users authenticated by its corporate **SAML 2.0 identity provider**, without creating IAM users for each employee. What should the architect use?
- A. Amazon Cognito user pools
- B. IAM Identity Center / IAM federation with the SAML IdP and IAM roles
- C. An IAM user per employee
- D. Access keys shared via email

**B16.** A workload requires **shared, low-latency file storage** accessed concurrently by hundreds of Linux EC2 instances, and storage must scale automatically. Which service should the architect choose?
- A. Amazon EBS Multi-Attach
- B. Amazon EFS
- C. Amazon S3
- D. Instance store

**B17.** A company wants to reduce the blast radius of a compromised set of credentials and ensure **temporary credentials with a maximum permission ceiling** for a contractor's IAM role, even if a broader policy is attached. What should the architect apply?
- A. A resource-based policy
- B. An IAM permissions boundary on the role
- C. A larger managed policy
- D. An access control list

**B18.** A solutions architect needs the **most cost-effective** storage for data that is accessed once or twice a quarter, must be available within milliseconds when needed, and is stored long-term. Which S3 storage class fits best?
- A. S3 Standard
- B. S3 Standard-IA
- C. S3 Glacier Deep Archive
- D. S3 One Zone-IA for irreplaceable data

**B19.** An application tier must call a downstream service that is sometimes slow. The architect wants to **decouple them and let the application keep accepting requests** while work is processed asynchronously, with automatic retries. Which design is best?
- A. Synchronous REST calls with a longer timeout
- B. Place an SQS queue between the tiers; workers process asynchronously with retries and a DLQ
- C. Increase the downstream service's instance size
- D. Add a CloudFront distribution

**B20.** A company runs steady, predictable production workloads on EC2 24/7 and wants to **maximize savings over the next 3 years** while retaining flexibility to change instance families and move some workloads to Fargate. Which purchase option is best?
- A. On-Demand Instances
- B. Standard Reserved Instances for specific instance types
- C. Compute Savings Plans
- D. Spot Instances

**B21.** A solutions architect must protect a public REST API built on API Gateway and Lambda from being overwhelmed by excessive requests, and **authenticate callers**. Which features should the architect use? **(Choose TWO.)**
- A. API Gateway throttling and usage plans
- B. A larger Lambda memory setting
- C. Amazon Cognito user pools or a Lambda authorizer for authentication
- D. A NAT gateway
- E. EC2 Auto Scaling

**B22.** A company's compliance team needs to be **automatically alerted and have the resource remediated** whenever an S3 bucket is created without encryption. Which service provides config-rule evaluation with auto-remediation?
- A. AWS CloudTrail
- B. AWS Config with remediation actions
- C. Amazon GuardDuty
- D. Amazon Inspector

**B23.** An application stores infrequently accessed but **mission-critical, irreplaceable** backups that must survive the loss of an Availability Zone. Which S3 storage class should the architect avoid, and what should be used instead?
- A. Use One Zone-IA for lowest cost
- B. Use Standard-IA (multi-AZ); avoid One Zone-IA because it stores data in a single AZ
- C. Use Reduced Redundancy Storage
- D. Use instance store snapshots

**B24.** A team needs to migrate an on-premises Oracle database to Amazon Aurora PostgreSQL with **minimal downtime**, converting the schema in the process. Which combination should the architect use?
- A. AWS DataSync only
- B. AWS Database Migration Service (DMS) with the Schema Conversion Tool (SCT)
- C. A manual mysqldump export/import
- D. Snowball Edge

**B25.** A company wants to centrally provision identical, version-controlled infrastructure across multiple accounts and Regions, and detect when someone changes resources manually. Which service should the architect use?
- A. AWS CloudFormation with drift detection (and StackSets for multi-account/Region)
- B. Manual setup documented in a wiki
- C. AWS Config only
- D. Elastic Beanstalk

---

## Answer Key — with why each distractor fails

**B1 — A.** **Multi-AZ RDS** adds a synchronous standby with automatic failover — directly removes the DB single point of failure, most cost-effectively. *B:* a read replica is async, doesn't auto-fail-over for writes. *C:* a full re-platform to DynamoDB is excessive and changes the data model. *D:* snapshots don't provide availability during an outage.

**B2 — B.** **Organizations + separate accounts + SCPs** gives strong isolation, per-environment guardrails, and consolidated billing. *A/C:* single-account isolation is weak and hard to govern at scale. *D:* multiple Organizations breaks the single consolidated bill.

**B3 — A.** **S3 event notification → Lambda** is native, event-driven, no polling. *B:* CloudWatch alarms are metric-based, not per-object events. *C:* scheduled scanning is polling and adds latency/cost. *D:* continuous SQS polling is not "no polling."

**B4 — B.** **RDS Proxy** pools and reuses connections, ideal for Lambda bursts, minimal code change. *A:* bigger instance raises the limit but doesn't manage connection churn. *C:* re-platform is excessive. *D:* read replicas don't solve connection exhaustion on writes.

**B5 — A and C.** An **organization CloudTrail trail** captures all API calls centrally; **Object Lock + an SCP** make logs immutable and prevent disabling logging. *B:* Flow Logs capture network metadata, not API calls. *D:* Macie scans for PII, not relevant. *E:* Config tracks resource config, not every API call.

**B6 — B.** **Latency-based routing** sends users to the lowest-latency Region. *A:* weighted splits by percentage, not latency. *C:* geolocation routes by user location, not measured latency/health the same way. *D:* failover is active-passive DR.

**B7 — B.** **Aurora Serverless v2** auto-scales capacity for intermittent/unpredictable relational workloads, avoiding idle cost. *A/D:* fixed instances pay for idle. *C:* Redshift is analytical, not OLTP.

**B8 — B.** **Cross-account IAM role / bucket policy scoped to the prefix** = least privilege. *A:* public bucket violates least privilege. *C:* sharing keys is a major anti-pattern. *D:* NAT is irrelevant to cross-account IAM.

**B9 — B.** **ElastiCache for Redis** offers sub-millisecond reads and native sorted sets for leaderboards. *A:* DynamoDB is single-digit ms (DAX µs) but lacks native sorted-set ranking semantics. *C/D:* not designed for real-time ranking.

**B10 — B.** **Global Accelerator** gives static anycast IPs, any TCP/UDP, and second-level Region failover over the AWS backbone. *A:* CloudFront caches HTTP, not arbitrary TCP. *C:* DNS failover is slower (TTL-bound) and not a single static IP. *D:* two NLBs alone give no unified static entry point or automatic cross-Region failover.

**B11 — A.** **Firehose** delivers streaming data to S3/Redshift in near real time with no servers. *B:* Data Streams needs you to build/manage consumers. *C:* SQS isn't a streaming-to-warehouse delivery service. *D:* Batch is for batch compute, not streaming delivery.

**B12 — B.** **S3 static hosting + CloudFront** = cheapest, lowest-ops, low-latency global static site. *A/C/D:* running servers for a static site adds cost and operational overhead.

**B13 — B.** **Enable EBS encryption by default** with a KMS key ensures all new volumes and snapshots are encrypted automatically, KMS-auditable. *A:* manual is error-prone. *C:* instance store isn't persistent/snapshot-able the same way. *D:* app-layer only doesn't satisfy volume/snapshot encryption.

**B14 — B.** A **DLQ with maxReceiveCount** moves poison messages aside after N failures without losing them. *A:* zero visibility timeout worsens reprocessing storms. *C:* FIFO doesn't fix malformed messages. *D:* deleting on first failure loses data.

**B15 — B.** **IAM federation / Identity Center with SAML + roles** provides temporary federated access without per-user IAM users. *A:* Cognito targets app end-users, not corporate workforce SAML federation to AWS. *C:* per-employee IAM users is the anti-pattern being avoided. *D:* emailing keys is insecure.

**B16 — B.** **EFS** = shared, elastic, low-latency NFS for many Linux instances concurrently. *A:* EBS Multi-Attach is same-AZ and limited to specific volume types/instances, not hundreds. *C:* S3 isn't a POSIX file system. *D:* instance store is ephemeral and not shared.

**B17 — B.** A **permissions boundary** caps the maximum permissions of the role regardless of attached policies. *A:* resource-based policies govern resource access, not the identity's ceiling. *C:* a bigger policy widens, not limits. *D:* ACLs are legacy object/network controls.

**B18 — B.** **Standard-IA** = millisecond access, lower cost for infrequent access, multi-AZ durability. *A:* Standard costs more for infrequent data. *C:* Deep Archive can't deliver millisecond retrieval. *D:* One Zone-IA risks data loss on AZ failure — wrong for irreplaceable data.

**B19 — B.** **SQS + async workers + DLB/retries** decouples and keeps the front end responsive. *A:* longer timeouts keep them coupled and fragile. *C:* scaling up doesn't decouple. *D:* CloudFront is a CDN, irrelevant here.

**B20 — C.** **Compute Savings Plans** maximize savings for steady usage while staying flexible across instance families and Fargate. *A:* On-Demand is most expensive. *B:* standard RIs lock to instance types — less flexible. *D:* Spot is interruptible, unsuitable for steady production.

**B21 — A and C.** **API Gateway throttling/usage plans** cap request rates; **Cognito or a Lambda authorizer** authenticates callers. *B:* memory doesn't control request volume. *D/E:* NAT and EC2 Auto Scaling are irrelevant to a serverless API's throttling/auth.

**B22 — B.** **AWS Config + remediation** evaluates the encryption rule and auto-remediates. *A:* CloudTrail records the API call but doesn't evaluate/remediate compliance. *C/D:* GuardDuty/Inspector are threat/vuln detection, not config compliance.

**B23 — B.** Use **Standard-IA** (multi-AZ); **avoid One Zone-IA** for irreplaceable data because it lives in a single AZ. *A:* directly risks loss on AZ failure. *C:* RRS is deprecated/less durable. *D:* instance store snapshots aren't a backup storage class.

**B24 — B.** **DMS + SCT** migrates with minimal downtime (ongoing replication) and converts the heterogeneous Oracle→Aurora PostgreSQL schema. *A:* DataSync is for files, not live DB cutover. *C:* mysqldump is wrong engine and high downtime. *D:* Snowball is bulk file transfer, not live DB migration.

**B25 — A.** **CloudFormation (+ StackSets) with drift detection** provisions identical, version-controlled infra across accounts/Regions and detects manual changes. *B:* manual docs aren't IaC. *C:* Config detects changes but doesn't provision. *D:* Beanstalk deploys apps, not arbitrary multi-account infra.
