# Practice — Domain 3: Design High-Performing Architectures (24%)

> Answers + explanations at the bottom.

---

**Q1.** A read-heavy application puts excessive load on an RDS database with repeated identical queries. What improves read performance with the least application change?
- A. Add an ElastiCache cache in front of the database
- B. Vertically scale the database instance
- C. Switch to DynamoDB
- D. Enable Multi-AZ

**Q2.** A DynamoDB-backed app needs microsecond read latency for hot items. What should you add?
- A. A larger table
- B. DynamoDB Accelerator (DAX)
- C. Global Tables
- D. A GSI

**Q3.** A global audience experiences high latency loading images from an S3-hosted website. What reduces latency most effectively?
- A. Enable S3 Transfer Acceleration
- B. Serve content through CloudFront
- C. Move the bucket closer to users
- D. Use a larger EC2 instance

**Q4.** A data analytics team wants to run ad-hoc SQL queries directly against data stored in S3 without provisioning servers. Which service?
- A. Redshift
- B. Athena
- C. EMR
- D. RDS

**Q5.** A fleet of EC2 instances needs a shared file system accessible concurrently from many Linux instances across AZs. Which storage?
- A. EBS io2 with multi-attach
- B. Amazon EFS
- C. Instance store
- D. S3 mounted as a drive

**Q6.** An application requires a load balancer that handles millions of requests per second at ultra-low latency and provides a static IP. Which load balancer?
- A. Application Load Balancer
- B. Network Load Balancer
- C. Gateway Load Balancer
- D. Classic Load Balancer

**Q7.** A database workload requires consistent, guaranteed high IOPS for an EC2-attached volume. Which EBS type?
- A. gp2
- B. st1
- C. io2 Provisioned IOPS SSD
- D. sc1

**Q8.** An application's traffic is unpredictable and spiky; the team wants compute that scales automatically and incurs no cost when idle. Which compute model?
- A. Reserved EC2 instances
- B. AWS Lambda
- C. Dedicated Hosts
- D. A fixed EC2 fleet

**Q9.** A real-time clickstream must be ingested in order, allow multiple independent consumers, and support replay for analytics. Which service?
- A. SQS Standard
- B. Kinesis Data Streams
- C. SNS
- D. SQS FIFO

**Q10.** A non-HTTP TCP application needs global users routed to the nearest healthy Region over the AWS backbone with static anycast IPs and fast failover. Which service?
- A. CloudFront
- B. AWS Global Accelerator
- C. Route 53 weighted routing
- D. Transit Gateway

---

## Answers & explanations

1. **A** — **ElastiCache** caches repeated reads, offloading the DB with minimal app change. Read replicas would also work but aren't offered; caching is ideal for identical repeated queries.
2. **B** — **DAX** gives microsecond reads for DynamoDB. Global Tables = multi-Region, not latency for hot reads.
3. **B** — **CloudFront** caches at edge locations globally → lowest latency for downloads. Transfer Acceleration optimizes uploads.
4. **B** — **Athena** = serverless SQL directly on S3. Redshift requires loading/provisioning.
5. **B** — **EFS** = shared, multi-AZ, concurrent Linux access. EBS multi-attach is same-AZ and limited.
6. **B** — **NLB** = L4, ultra-low latency, millions req/s, static/Elastic IP.
7. **C** — **io2** provides provisioned, guaranteed IOPS for databases.
8. **B** — **Lambda** scales automatically and costs nothing when idle — ideal for spiky/unpredictable workloads.
9. **B** — **Kinesis Data Streams** = ordered, multiple consumers, replay within retention. SQS doesn't replay or support multiple independent consumer groups the same way.
10. **B** — **Global Accelerator** = static anycast IPs, any TCP/UDP, AWS backbone, fast regional failover. CloudFront is for cacheable HTTP content.
