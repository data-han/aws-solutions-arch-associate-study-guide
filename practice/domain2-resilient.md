# Practice — Domain 2: Design Resilient Architectures (26%)

> Answers + explanations at the bottom.
1B, 2B, 3D, 4B, 5C, 6B, 7B, 8B, 9B, 10C
---

**Q1.** A web tier on EC2 behind an ALB must survive the failure of an entire Availability Zone with no manual intervention. What design achieves this most simply?
- A. Run two instances in one AZ with a standby
- B. Use an Auto Scaling group spanning multiple AZs behind the ALB
- C. Use a larger instance type
- D. Add a Route 53 failover record to a second Region

**Q2.** An order-processing system loses messages when the processing fleet is overwhelmed during sales spikes. How should you decouple it?
- A. Increase EC2 instance size
- B. Place an SQS queue between the front end and the processing fleet, scaling consumers on queue depth
- C. Use SNS to push orders directly to instances
- D. Add a read replica to the database

**Q3.** A company wants the cheapest disaster recovery option and can tolerate several hours of recovery time (RTO) and data loss (RPO). Which strategy?
- A. Multi-site active/active
- B. Warm standby
- C. Pilot light
- D. Backup and restore

**Q4.** An RDS MySQL database must automatically fail over to a standby with no data loss if its AZ goes down. What should be configured?
- A. A cross-Region read replica
- B. Multi-AZ deployment
- C. Multiple read replicas in the same AZ
- D. Automated snapshots every hour

**Q5.** Messages that repeatedly fail processing must be retained for later debugging instead of being lost. What should you configure?
- A. Increase visibility timeout
- B. Enable long polling
- C. A dead-letter queue (DLQ)
- D. FIFO queue

**Q6.** A globally used application needs a relational database with cross-Region disaster recovery and sub-second replication lag. Which option?
- A. RDS Multi-AZ
- B. Aurora Global Database
- C. DynamoDB Global Tables
- D. Read replica in the same Region

**Q7.** A stateless application stores user session data on each EC2 instance's local disk, so users get logged out when Auto Scaling replaces an instance. How do you fix this resiliently?
- A. Disable Auto Scaling
- B. Store session state in ElastiCache (Redis) or DynamoDB
- C. Enable sticky sessions only
- D. Use a larger instance

**Q8.** A company needs centralized, policy-driven backups across EBS, RDS, DynamoDB, and EFS with cross-Region copies. Which service?
- A. S3 lifecycle policies
- B. AWS Backup
- C. Manual snapshots
- D. Storage Gateway

**Q9.** To protect against accidental and malicious deletion of critical objects in S3, which combination is best?
- A. One Zone-IA storage class
- B. Versioning plus MFA Delete (and optionally Object Lock)
- C. Cross-Region replication only
- D. Bucket ACLs

**Q10.** An application must route users to a secondary Region automatically if the primary Region's endpoint becomes unhealthy. Which Route 53 routing policy?
- A. Weighted
- B. Latency
- C. Failover
- D. Geolocation

---

## Answers & explanations

1. **B** — ASG across multiple AZs + ALB gives automatic, self-healing multi-AZ HA. A second Region (D) is more than required and costlier; single-AZ designs (A) don't survive AZ loss.
2. **B** — **SQS** buffers spikes and prevents message loss; consumers scale on queue depth. SNS (C) doesn't buffer/persist for pull consumers the same way.
3. **D** — **Backup & Restore** is cheapest and fits hours-scale RTO/RPO.
4. **B** — **Multi-AZ** provides synchronous standby + automatic failover. Read replicas are async and don't auto-fail-over.
5. **C** — A **DLQ** captures messages exceeding max receive count for later analysis.
6. **B** — **Aurora Global Database** = relational, cross-Region, <1s lag. DynamoDB (C) is NoSQL.
7. **B** — Externalize session state to **ElastiCache/DynamoDB** so any instance serves any user. Sticky sessions alone (C) still lose state when the instance dies.
8. **B** — **AWS Backup** centralizes cross-service, policy-based backups with cross-Region copy.
9. **B** — **Versioning + MFA Delete** (and Object Lock for WORM) protect against deletion/overwrite.
10. **C** — **Failover** routing = active-passive based on health checks.
