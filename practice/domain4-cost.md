# Practice — Domain 4: Design Cost-Optimized Architectures (20%)

> Answers + explanations at the bottom.

---

**Q1.** A batch image-processing job is fault-tolerant and can be interrupted and resumed. Which EC2 purchasing option is most cost-effective?
- A. On-Demand
- B. Reserved Instances
- C. Spot Instances
- D. Dedicated Hosts

**Q2.** A company runs a steady, predictable production workload 24/7 for the next 3 years and wants maximum savings with flexibility across EC2, Fargate, and Lambda. Which option?
- A. Spot Instances
- B. Compute Savings Plans
- C. On-Demand
- D. More Auto Scaling

**Q3.** Logs in S3 are accessed frequently for 30 days, rarely after, and must be retained 7 years for compliance with retrieval times of several hours acceptable. What is the most cost-effective approach?
- A. Keep everything in S3 Standard
- B. Lifecycle policy: Standard → Standard-IA → Glacier Deep Archive; expire at 7 years
- C. Store everything in Glacier immediately
- D. Use S3 One Zone-IA only

**Q4.** Access patterns for a dataset in S3 are unknown and change over time. Which storage class minimizes cost without retrieval fees or manual tuning?
- A. Standard-IA
- B. S3 Intelligent-Tiering
- C. Glacier Flexible Retrieval
- D. One Zone-IA

**Q5.** Instances in a private subnet download large amounts of data from S3 through a NAT Gateway, generating high data-processing charges. How do you cut costs?
- A. Use a larger NAT Gateway
- B. Use a Gateway VPC Endpoint for S3
- C. Move instances to a public subnet
- D. Use an Interface Endpoint with PrivateLink

**Q6.** A finance team wants automatic alerts when monthly spend is forecast to exceed a threshold. Which tool?
- A. Cost Explorer
- B. AWS Budgets
- C. Trusted Advisor
- D. Cost and Usage Report

**Q7.** A development environment of EC2 instances is only used during business hours. What reduces cost with minimal effort?
- A. Buy Reserved Instances
- B. Stop/schedule the instances outside business hours
- C. Use Spot Instances for dev
- D. Resize to the smallest instance permanently

**Q8.** A company with many AWS accounts wants a single consolidated bill and volume pricing discounts. What enables this?
- A. AWS Budgets
- B. Consolidated billing in AWS Organizations
- C. Savings Plans
- D. Separate invoices per account

**Q9.** A relational database workload has highly variable, intermittent traffic and is idle much of the time. Which option optimizes cost?
- A. Largest RDS instance
- B. Aurora Serverless v2
- C. Multi-AZ RDS always on
- D. DynamoDB provisioned capacity

**Q10.** Which service provides right-sizing recommendations to reduce EC2 over-provisioning costs?
- A. AWS Compute Optimizer
- B. CloudTrail
- C. AWS Config
- D. GuardDuty

---

## Answers & explanations

1. **C** — **Spot** (up to ~90% off) is ideal for interruptible, fault-tolerant batch work.
2. **B** — **Compute Savings Plans** give up to 72% off for steady commitment with flexibility across EC2/Fargate/Lambda.
3. **B** — A **lifecycle policy** transitioning to cheaper tiers then Deep Archive, with expiration, is the cost-optimal pattern for tiered access + compliance retention.
4. **B** — **Intelligent-Tiering** auto-optimizes for unknown/changing access with no retrieval fees and no manual work.
5. **B** — **S3 Gateway Endpoint** is free and removes NAT data-processing charges for S3 traffic. (S3 uses Gateway, not Interface, endpoints.)
6. **B** — **AWS Budgets** sends threshold/forecast alerts. Cost Explorer analyzes but doesn't alert.
7. **B** — **Scheduling stop/start** off-hours cuts cost simply for dev environments.
8. **B** — **Consolidated billing via Organizations** = one bill + aggregated volume discounts.
9. **B** — **Aurora Serverless v2** scales capacity to match intermittent load, avoiding paying for idle.
10. **A** — **Compute Optimizer** gives right-sizing recommendations.
