# 06 — Resilience, HA, Fault Tolerance & DR (Domain 2, 26%)

## Core principles

**High Availability (HA)** means your system can survive the failure of individual components or entire Availability Zones and continue serving requests with minimal or no downtime. Achieving HA in AWS typically means deploying across **multiple AZs**, eliminating single points of failure, and combining redundancy with automation.

**Fault Tolerance** is a stronger guarantee than HA — a fault-tolerant system continues operating normally through failures rather than just failing over. This usually requires more redundancy and over-provisioning than basic HA.

**Scalability** comes in two forms. **Horizontal scaling** (also called scaling out) means adding more instances of a component — adding more EC2 instances behind a load balancer, for example. This is the preferred AWS pattern because it's elastic (you can scale down too), doesn't require downtime, and there's no upper ceiling. **Vertical scaling** (scaling up) means replacing a component with a bigger one (larger instance type) — it has a practical ceiling, often requires a restart, and is less flexible.

**Loose coupling** is the design principle of ensuring that individual components of a system can fail, scale, or change independently without cascading failures. You achieve this with SQS queues and SNS between components (so a slow consumer doesn't block the producer), and with ELB between tiers (so the web tier doesn't need to know which app servers exist). See note 07 for details.

**Stateless application tiers** are essential for horizontal scaling. If an application server stores session data locally (in memory or on local disk), then every request from a user must go to the same server — this limits your ability to add and remove servers freely. By storing state externally (in DynamoDB, ElastiCache, or S3), any server in the fleet can handle any user's request, and Auto Scaling can add and remove instances without losing user sessions.

---

## Designing for HA — the building blocks

The core pattern is: deploy everything across **multiple AZs**, use managed services that are multi-AZ by default, and let AWS's automation handle recovery.

- Spread your **Auto Scaling Group** across at least two (ideally three) AZs so that an AZ failure reduces capacity but doesn't take down the application.
- Place an **Elastic Load Balancer** in front of the ASG — the ELB distributes traffic across healthy instances in all AZs and stops sending requests to any instance that fails its health check. The ASG replaces failed instances automatically.
- Use **RDS Multi-AZ** for your database so that a database instance failure or AZ failure triggers automatic failover to the standby.
- **EFS** replicates data across multiple AZs by design. **S3** provides built-in durability across AZs with no configuration needed.
- Deploy **one NAT Gateway per AZ** (not one shared NAT Gateway) — a single NAT Gateway is an AZ-level dependency; if that AZ fails, private instances in other AZs lose their outbound internet access.
- Use **Route 53 health checks + failover routing** to route DNS away from a failed region entirely.

Avoid these common single-points-of-failure: a single EC2 instance without Auto Scaling, a single NAT Gateway, a single AZ, or any resource that can't be replaced automatically.

---

## Disaster Recovery strategies

DR strategies exist on a spectrum between lowest cost (but slowest recovery) and highest cost (but near-instant recovery). The two key metrics are:

- **RTO (Recovery Time Objective)** — how quickly you need to recover (how long can the system be down?).
- **RPO (Recovery Point Objective)** — how much data loss is acceptable (how old can your last backup be?).

Lower RTO and RPO always means higher cost. Know this spectrum:

| Strategy | RTO / RPO | Cost | How it works |
|----------|-----------|------|--------------|
| **Backup & Restore** | Hours | $ | You take regular backups to S3 or Glacier. On disaster, you provision fresh infrastructure and restore from the backup. Slowest recovery, but cheapest — you're only paying for storage, not running duplicate infrastructure. |
| **Pilot Light** | Tens of minutes | $$ | Your core data tier (database) is always running and replicating to the DR region. Application servers and other components are turned off. On disaster, you start up the rest of the stack and point it at the warm database. The "pilot light" is always on so the flame can be fanned quickly. |
| **Warm Standby** | Minutes | $$$ | A scaled-down but fully functional copy of your production environment is always running in the DR region. It can handle a reduced load. On disaster, you scale it up to full production capacity. Faster than Pilot Light because everything is already running; you just need to scale. |
| **Multi-Site Active/Active** | Near-zero | $$$$ | Full production capacity runs simultaneously in two or more regions, handling live traffic at all times. If one region fails, the other absorbs all traffic. No recovery time — the failover is just rebalancing existing traffic. The most expensive because you're running and paying for duplicate production capacity continuously. |

- Keyword "cheapest DR, hours of downtime acceptable" → **Backup & Restore**.
- Keyword "near-zero downtime, cost is not a constraint" → **Multi-Site Active/Active**.
- Keyword "database always running in DR region, application spun up on demand" → **Pilot Light**.

---

## Service-level resilience patterns

| Service | How to make it resilient |
|---------|--------------------------|
| **RDS** | Multi-AZ for automatic failover. Read Replicas (optionally cross-region) for read scaling and DR. |
| **Aurora** | 6 storage copies across 3 AZs and auto-failover built in. Aurora Global Database for cross-region DR with sub-second replication lag. |
| **DynamoDB** | Global Tables for multi-region active-active replication — every region is a primary. |
| **S3** | Built-in multi-AZ durability. Use Cross-Region Replication for a copy in another region as DR. |
| **EBS** | Take regular snapshots. Snapshots can be copied cross-region so you can restore volumes in a different region after a disaster. |
| **DNS / routing** | Route 53 health checks detect endpoint failures and failover routing policies switch DNS to healthy endpoints automatically. CloudFront can route around origin failures. |

---

## Decoupling for resilience

Tightly coupled architectures are fragile — if a downstream component slows down or fails, it can stall or crash upstream components. The fix is to introduce an **asynchronous queue** (SQS) between them.

When a producer puts a message in an SQS queue, it immediately returns — it doesn't wait for a consumer to process the message. The message sits in the queue until a consumer is ready. This means a slow consumer just falls behind rather than crashing the producer, and the messages aren't lost. If the consumer fails entirely, messages stay in the queue and will be processed when the consumer recovers.

A **Dead Letter Queue (DLQ)** captures messages that a consumer has repeatedly failed to process — after a configured number of failed attempts, the message is moved to the DLQ instead of being retried forever. This lets you investigate and replay problematic messages without blocking the main queue.

Asynchronous processing via queues also naturally smooths out traffic spikes — producers can generate work at peak rates, and the queue absorbs the burst while consumers work through it at a sustainable rate, preventing the spike from overwhelming downstream systems.

---

## Backup & data protection

**AWS Backup** is a centralized, policy-driven backup service that manages backups across multiple AWS services — EBS volumes, RDS databases, DynamoDB tables, EFS file systems, FSx, and more — from a single place. You define backup plans (frequency, retention) and AWS Backup handles execution, lifecycle, and cross-region or cross-account copying of backups for DR purposes.

For S3 specifically, **Versioning** protects against accidental overwrites (every version is preserved), **MFA Delete** requires multi-factor authentication to permanently delete a version (protection against malicious deletion), and **Object Lock** enforces WORM (write-once-read-many) protection so objects cannot be deleted or overwritten for a defined retention period — used for compliance and immutability requirements.

---

## Keyword map

| Scenario | Answer |
|----------|--------|
| Survive an AZ failure | Deploy across multiple AZs (ASG + ELB + Multi-AZ DB) |
| Cheapest DR with hours of RTO acceptable | Backup & Restore |
| Minimal downtime DR | Warm Standby or Active-Active |
| Decouple components so one failure doesn't cascade | SQS queue between tiers |
| Don't lose messages from a failing consumer | Dead-Letter Queue (DLQ) |
| Centralize backups across multiple AWS services | AWS Backup |
| Multi-region active-active database | DynamoDB Global Tables or Aurora Global Database |
| Self-healing compute tier | Auto Scaling Group + ELB health checks |
