# Exam-Day Cheatsheet — High-Yield Facts

Review this the morning of the exam. These are the facts and keyword→service mappings the SAA-C03 tests over and over.

---

## 🔑 Keyword → Service (the single most useful table)

| If the question says… | Pick |
|------------------------|------|
| EC2 needs to access AWS services securely | **IAM role** (instance profile), never access keys |
| Decouple / buffer / smooth spikes / don't lose work | **SQS** |
| One message → many consumers / fan-out / notify | **SNS** |
| React to AWS/SaaS events, cron schedule, content routing | **EventBridge** |
| Real-time streaming, ordered, replay, analytics | **Kinesis Data Streams** |
| Stream → S3/Redshift, no code | **Kinesis Data Firehose** |
| Orchestrate multi-step workflow | **Step Functions** |
| Serverless compute, event-driven, <15 min | **Lambda** |
| Containers without managing servers | **Fargate** |
| Need Kubernetes | **EKS** |
| Static IP on a load balancer | **NLB** |
| Route HTTP by path/host | **ALB** |
| Global low-latency content (cacheable, HTTP) | **CloudFront** |
| Global static IP + fast multi-Region failover (TCP/UDP) | **Global Accelerator** |
| Private subnet → internet outbound | **NAT Gateway** |
| Private access to S3/DynamoDB, no NAT | **Gateway VPC Endpoint** (free) |
| Connect many VPCs + on-prem centrally | **Transit Gateway** |
| Dedicated private on-prem link, consistent | **Direct Connect** |
| Quick encrypted hybrid link | **Site-to-Site VPN** |
| Nearest-Region DNS routing | **Route 53 latency policy** |
| Active-passive DR via DNS | **Route 53 failover policy** |
| Shared file system, many Linux EC2 | **EFS** |
| Shared Windows SMB file storage | **FSx for Windows** |
| HPC/ML high-throughput file storage | **FSx for Lustre** |
| Object storage, web scale | **S3** |
| Unknown S3 access pattern, optimize cost | **S3 Intelligent-Tiering** |
| Cheapest archive, hours retrieval OK | **S3 Glacier Deep Archive** |
| Block storage for one EC2 / DB, high IOPS | **EBS (io2)** |
| HA / automatic failover for RDS | **RDS Multi-AZ** |
| Offload read traffic from DB | **Read Replica** |
| Serverless NoSQL, millisecond, huge scale | **DynamoDB** |
| Microsecond reads for DynamoDB | **DAX** |
| Multi-Region active-active NoSQL | **DynamoDB Global Tables** |
| Cache DB reads / session store | **ElastiCache (Redis)** |
| SQL directly on S3, no infra | **Athena** |
| Data warehouse / BI / OLAP | **Redshift** |
| Auto-rotate DB credentials | **Secrets Manager** |
| Detect PII in S3 | **Macie** |
| Threat detection from logs, no agents | **GuardDuty** |
| Vulnerability scanning EC2/ECR/Lambda | **Inspector** |
| Who made an API call (audit) | **CloudTrail** |
| Resource compliance / config drift | **AWS Config** |
| SSH-less access to EC2 | **SSM Session Manager** |
| Repeatable infrastructure (IaC) | **CloudFormation** |
| Filter SQLi/XSS on web app | **WAF** |
| Centralize SSO across accounts | **IAM Identity Center** |
| Customer-controlled HSM key custody | **CloudHSM** |
| Migrate PB of data, slow network | **Snowball** |
| Ongoing scheduled on-prem↔AWS sync | **DataSync** |
| Budget overspend alert | **AWS Budgets** |
| Analyze spend trends | **Cost Explorer** |

---

## ⚙️ Stateful vs stateless / single vs multi

| Concept | Key fact |
|---------|----------|
| **Security Group** | **Stateful**, instance-level, **allow only** |
| **NACL** | **Stateless**, subnet-level, allow **+ deny** (only way to block an IP) |
| **RDS Multi-AZ** | HA/failover (sync standby), **NOT** read scaling |
| **RDS Read Replica** | Read scaling (async), **NOT** automatic failover |
| **EBS** | Single-AZ, one instance (io2 multi-attach exception) |
| **EFS** | Multi-AZ, many instances, Linux NFS |
| **Instance Store** | Ephemeral, lost on stop/terminate |
| **S3 / DynamoDB** | Gateway endpoint (free); everything else = interface endpoint |
| **IAM** | Global service; KMS keys are Regional |

---

## 📊 Numbers worth memorizing

| Thing | Value |
|-------|-------|
| Exam | 65 Q, 130 min, pass **720**/1000 |
| S3 max object size | 5 TB (multipart >5 GB) |
| S3 durability | 11 nines (99.999999999%) |
| Lambda max timeout | 15 min; mem up to 10 GB; /tmp up to 10 GB |
| SQS message retention | up to 14 days; visibility timeout default 30s |
| SQS FIFO throughput | 300 TPS (3000 batched) |
| RDS read replicas | 5 (Aurora 15) |
| RDS automated backup | 1–35 days, point-in-time |
| Aurora storage | 6 copies / 3 AZs, up to 128 TB |
| DynamoDB latency | single-digit ms (DAX → µs) |
| Glacier Deep Archive | retrieval ~12h, min 180 days |
| Standard-IA / Glacier min duration | 30 / 90 days |
| EBS gp3 baseline | 3000 IOPS / 125 MB/s |
| Spot savings | up to 90% · RI/Savings Plans up to 72% |

---

## 🧠 Common traps / disambiguation

- **CloudWatch (performance/metrics/logs)** vs **CloudTrail (who-did-what API audit)** vs **Config (resource compliance)**.
- **SQS (decouple, delete after read, no replay)** vs **Kinesis (stream, ordered, multiple consumers, replay)**.
- **CloudFront (cache HTTP content)** vs **Global Accelerator (static IP, any TCP/UDP, fast failover, no caching)**.
- **Multi-AZ (HA)** vs **Read Replica (read scale)** vs **Multi-Region (DR/global)**.
- **SCP (ceiling, doesn't grant)** vs **IAM policy (grants)**. **Explicit Deny always wins.**
- **Secrets Manager (auto-rotation, $)** vs **Parameter Store (free, no rotation)**.
- **NAT Gateway (outbound internet, $)** vs **VPC Endpoint (private AWS service access)**.
- "Minimize **operational overhead**" → prefer **managed / serverless**.
- "Most **cost-effective**" → don't pick multi-Region/over-provisioned unless required.
- "Bring your own license / per-socket" → **Dedicated Host**.

---

## ✅ Question-answering ritual
1. Read the **last line** → identify the optimized dimension (cost / least ops / HA / latency / security).
2. Cross out 2 obvious distractors.
3. Match remaining options against the **optimized dimension** + keyword map.
4. Prefer **managed/serverless, Multi-AZ, least-privilege, decoupled** unless the question says cheapest.
5. Never leave blank.
