# 09 — Cost Optimization (Domain 4, 20%)

The exam frames cost questions as: "which option meets the stated requirements at the **lowest cost**?" The key skill is avoiding over-engineering — don't add HA, redundancy, or features the question didn't ask for, because those all cost more. Pick the cheapest option that still satisfies whatever constraints are explicitly stated (availability, performance, security).

---

## Compute cost levers

**Right-sizing** means matching your instance type and size to your actual workload requirements — not over-provisioning "just in case." **AWS Compute Optimizer** analyzes your EC2, Lambda, and ECS usage and provides specific right-sizing recommendations based on actual utilization history.

**Pricing model choice** is the biggest lever for steady workloads. The savings options from most to least discount:

- **Spot Instances** — up to ~90% cheaper than On-Demand. The catch is AWS can reclaim them with a 2-minute notice, so they're only suitable for fault-tolerant, stateless, or interruptible workloads like batch jobs, data processing, and CI pipelines.
- **Savings Plans / Reserved Instances** — up to ~72% off for a 1- or 3-year commitment. Right for workloads that run 24/7 with predictable resource needs. See note 02 for the full RI vs Savings Plans comparison.

**Reserved Instance discount sharing across the organization:** When your AWS accounts are managed under an AWS Organization with consolidated billing, Reserved Instances and Savings Plans purchased in any account automatically share their discount across all accounts in the organization. AWS applies the discount to whichever account has matching usage, prioritizing the account that purchased the RI. This means you do not need to buy separate RIs in each team account — purchasing from the central management account lets the discount flow to wherever the actual usage is. One RI commitment can cover usage across 20 accounts simultaneously.

- **On-Demand** — no commitment, pay by the hour or second. Right for unpredictable, short-lived, or spiky workloads. Most expensive per unit of compute.

**Serverless** options (Lambda, Fargate, DynamoDB On-Demand, Aurora Serverless) charge you only for what you actually use — when idle, you pay nothing. For spiky or low-traffic workloads, this is often cheaper than running an instance 24/7 even if the instance is small.

**Auto Scaling** reduces cost by automatically scaling in (removing instances) when demand drops, so you're not paying for capacity you're not using. For non-production environments (dev, test), use a scheduler (AWS Instance Scheduler) to stop instances outside of working hours — e.g., evenings and weekends — since those are typically only needed during business hours.

---

## Storage cost levers

**S3 lifecycle policies** automatically transition objects to cheaper storage classes as they age. A typical pattern: objects start in S3 Standard, move to Standard-IA after 30 days, then to Glacier after 90 days, then to Glacier Deep Archive after 180 days, and are deleted after a few years. This happens automatically with zero manual work once the policy is set.

**S3 Intelligent-Tiering** is the right choice when you don't know your access patterns. It monitors each object's access frequency and automatically moves it between tiers — no retrieval fees and no operational overhead. There's a small per-object monitoring charge, but it pays for itself if your access patterns are unpredictable.

**One Zone-IA** stores data in only one Availability Zone rather than three, making it ~20% cheaper than Standard-IA. It's appropriate for data you can afford to recreate if the AZ fails — backups you also have elsewhere, temporary outputs, or easily regeneratable data.

For **EBS**: prefer **gp3** over gp2 — gp3 is explicitly cheaper and gives you the same baseline performance with more flexibility to provision IOPS and throughput independently. Delete unattached EBS volumes (volumes that exist but aren't attached to a running instance still incur charges) and remove old snapshots you no longer need.

For file storage, configure EFS lifecycle policies to move files not accessed recently to **EFS-IA**, which is significantly cheaper.

---

## Database cost levers

**Aurora Serverless v2** or right-sizing your RDS instance size avoids over-provisioning for variable database workloads. If your database load is bursty, paying for a large fixed instance that's mostly idle wastes money.

**DynamoDB capacity modes**: On-Demand mode is convenient but more expensive per request. If your traffic is predictable and steady, switching to Provisioned mode with Auto Scaling is cheaper. Use On-Demand for unpredictable or new workloads until you understand the usage pattern.

Using **Read Replicas** to handle read-heavy traffic is more cost-effective than scaling up the primary instance size, because you can add read capacity incrementally and remove it when not needed.

**Athena query cost — convert to Parquet:** Amazon Athena charges based on the amount of data it **scans** per query. You pay per terabyte scanned, regardless of how much data the query actually returns. The most effective way to cut this cost is to store your data in a **columnar format like Parquet** instead of row-based formats like CSV or JSON.

Here is why it helps: in a CSV file, each row contains all the columns. If your query only needs 3 of 50 columns, Athena still reads the entire file to find those 3 columns. In Parquet, columns are stored separately — Athena can physically skip the 47 columns it doesn't need and read only the 3 it does. This typically reduces the amount of data scanned (and therefore the cost) by **3 to 5 times**. The standard approach is to run an **AWS Glue ETL job** that reads your CSV data and writes it back as Parquet, then point Athena at the Parquet files. **Partitioning** your S3 data (organizing it into date- or region-based folder prefixes) provides additional savings by letting Athena skip entire folders that don't match your query filters.

- Keyword "reduce Athena query costs, data is stored as CSV or JSON in S3" → convert to **Parquet** using an AWS Glue ETL job.

---

## Network cost levers (data transfer is an easy-to-overlook cost)

Data transfer within the same AZ over private IPs is generally free. **Cross-AZ traffic** costs a small per-GB fee. **Cross-Region traffic** and **data transfer out to the internet** cost more. This means your architecture choices affect your network bill.

**VPC Gateway Endpoints for S3 and DynamoDB are free** and eliminate the need for NAT Gateway for those services. If your EC2 instances in private subnets are making frequent calls to S3 or DynamoDB, routing that traffic through a NAT Gateway costs you both hourly and per-GB charges. Replace it with a Gateway Endpoint (free) and the cost disappears.

**NAT Gateway** charges by the hour and by GB of data processed. If you have multiple instances downloading data from S3, using S3 Gateway Endpoints eliminates NAT charges for that traffic entirely.

**CloudFront** caches responses at edge locations, reducing the volume of requests that hit your origin and the amount of data transferred from your origin to CloudFront (CloudFront's distribution pricing is often cheaper than direct origin data transfer).

**Transit Gateway vs VPC Peering cost:** If you only need to connect 2 or 3 VPCs, VPC Peering is nearly always cheaper — the peering connection itself has no hourly charge, and you only pay the standard data transfer rate. Transit Gateway charges an hourly fee per VPC attachment plus a per-GB data processing fee. For small-scale connectivity, this makes TGW more expensive. But when you have many VPCs or need transitive routing, TGW's hub-and-spoke model is worth the cost because the alternative (dozens of point-to-point peering connections) becomes unmanageable. Rule of thumb: fewer than 5 VPCs = peering; 10 or more VPCs = Transit Gateway.

**NLB cross-zone load balancing cost:** Unlike ALB (where cross-zone load balancing is always on and free), NLB has cross-zone load balancing **disabled by default**. When you enable it, cross-AZ data transfer charges apply — traffic between the NLB node in one AZ and a target in a different AZ incurs per-GB charges. This is a hidden cost trap: the feature seems like a pure availability improvement, but it adds a data transfer cost that can grow large in high-traffic environments.

When latency permits, keep traffic within a single Region and ideally within a single AZ to minimize transfer costs.

---

## Cost management & visibility tools

Knowing which tool answers which cost question is itself an exam topic:

| Tool | What it's for |
|------|---------------|
| **Cost Explorer** | Visualizes your spend over time, breaks it down by service/account/tag, shows trends, and provides forecasts. Use it to understand where money is going and spot anomalies. |
| **AWS Budgets** | Lets you set a spending budget and sends alerts (via email or SNS) when actual or forecasted spend exceeds thresholds you define. Use it to be notified *before* you overspend. |
| **Cost & Usage Report (CUR)** | The most granular billing dataset available — hourly/daily usage and cost at the resource level, delivered to S3. Typically analyzed with Athena and QuickSight for custom reporting. |
| **Cost Allocation Tags** | Tags you apply to resources (e.g., `Team: DataEngineering`, `Env: Prod`) that appear in billing reports, letting you break down costs by team, project, or environment. |
| **Trusted Advisor** | Identifies specific underused or idle resources, unused Reserved Instance coverage gaps, and service limit warnings. Actionable recommendations, not just graphs. |
| **Compute Optimizer** | Analyzes actual utilization data and recommends specific right-sizing changes for EC2, Lambda, and ECS — moves beyond general advice to specific resource-level recommendations. |
| **Consolidated Billing (Organizations)** | Combines the bills of all accounts in your Organization into one, and AWS calculates volume discounts across the combined usage — you may hit S3 or data transfer price tiers faster with combined usage. |
| **Savings Plans recommendations** | In the Billing console, AWS shows you what Savings Plans commitment would have saved you money based on your past usage. |

---

## Decision heuristics for cost questions

These cover the patterns the exam tests most often:

- **"Unpredictable or spiky load, minimize cost and operational overhead"** → **Serverless** (Lambda, Fargate, DynamoDB On-Demand, Aurora Serverless) — you pay only for what you use and don't manage capacity.
- **"Steady 24/7 predictable workload, minimize cost"** → **Savings Plans or Reserved Instances** — commit to save up to 72%.
- **"Fault-tolerant batch job, absolute cheapest compute"** → **Spot Instances** — up to 90% off, acceptable if the job can be interrupted and restarted.
- **"Infrequently accessed data, minimize storage cost"** → **S3 Glacier or Deep Archive via lifecycle policy** — the cheapest storage tiers.
- **"Reduce data transfer cost for private subnet access to S3"** → **S3 Gateway VPC Endpoint** — free, eliminates NAT Gateway data charges for S3 traffic.
- **"Alert me before I exceed my spend"** → **AWS Budgets** — set a threshold and get an alert before it's exceeded.
- **"Understand what's driving my AWS bill"** → **Cost Explorer** — visualize trends and break down spend by service, account, or tag.

---

## Keyword map

| Scenario | Answer |
|----------|--------|
| Cheapest compute for interruptible workloads | Spot Instances |
| Save on steady, predictable compute | Savings Plans or Reserved Instances |
| Auto-optimize S3 with unknown access patterns | Intelligent-Tiering |
| Archive cheaply, retrieval in hours acceptable | Glacier Deep Archive + lifecycle policy |
| Free private S3 access from EC2, cut NAT Gateway cost | S3 Gateway VPC Endpoint |
| Alert when spend exceeds a threshold | AWS Budgets |
| Analyze and understand spend trends | Cost Explorer |
| One bill and volume discounts across all accounts | Consolidated Billing (Organizations) |
