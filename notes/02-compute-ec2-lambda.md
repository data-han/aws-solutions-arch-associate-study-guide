# 02 — Compute: EC2, Auto Scaling, ELB, Lambda, Containers

## EC2 instance families

EC2 instance types follow a naming pattern where the first letter (or letters) tell you the family — what the instance is optimized for:

- **T** — Burstable general-purpose (cheap, dev/test workloads). Earns CPU credits when idle, spends them when busy.
- **M** — General-purpose balanced (balanced CPU, memory, and network for most workloads).
- **C** — Compute-optimized (high CPU-to-memory ratio; HPC, batch, video encoding).
- **R / X** — Memory-optimized (high RAM; in-memory databases, large caching layers).
- **P / G / Inf** — GPU/ML accelerated (training, inference, graphics rendering).
- **I / D** — Storage-optimized (very high local IOPS or large local disk; databases, data warehousing).

---

## EC2 purchasing options

The exam regularly asks which pricing option fits a given scenario. The key variable is how predictable and interruptible the workload is:

| Option | Use when | Savings vs On-Demand |
|--------|----------|----------------------|
| **On-Demand** | Short-lived, unpredictable, or spiky workloads where you can't commit | Baseline |
| **Reserved (RI)** | Steady, predictable workloads running 24/7 for 1–3 years | Up to ~72% |
| **Savings Plans** | Flexible 1–3 yr commitment (you commit to a $/hr spend, not a specific instance) | Up to ~72% |
| **Spot** | Fault-tolerant, interruptible workloads (batch jobs, CI, big data processing) | Up to ~90% |
| **Dedicated Host** | Workloads where licensing is tied to physical sockets/cores (BYOL), or strict compliance requiring physical isolation | Most expensive |
| **Dedicated Instance** | Hardware isolation from other customers, but less visibility/control than Dedicated Host | Expensive |

**Spot Instances** can be reclaimed by AWS with only a 2-minute warning. They're only appropriate for stateless or fault-tolerant work — anything you can restart from scratch if interrupted.

- Keyword "bring your own license (BYOL) / per-socket or per-core licensing / compliance requiring dedicated hardware" → **Dedicated Host**.

---

### Reserved Instances vs Savings Plans — what to pick

Both give up to ~72% off for a 1- or 3-year commitment on **steady** workloads. The exam tests which one fits based on your flexibility needs and what services you're running.

| | Reserved Instances (RI) | Savings Plans (SP) |
|--|--|--|
| What you commit to | A specific instance configuration | A **$/hour** spend amount |
| What it covers | EC2 (plus separate RI types for RDS, Redshift, ElastiCache, OpenSearch, DynamoDB) | **EC2, Fargate, and Lambda only** |
| Flexibility | Lower | Higher — auto-applies to any matching usage |
| Can sell unused commitment | **Yes — Standard RIs on the RI Marketplace** | No |
| Capacity reservation | Only **Zonal RI** (specific AZ) | No capacity guarantee |

**Two RI flavors:**

- **Standard RI** — Up to ~72% off. Locked to an instance family and region, but you can change AZ or instance size within that family. Cannot change instance family. Can be sold in the RI Marketplace if you no longer need it.
- **Convertible RI** — Up to ~66% off. You can exchange it to change instance family, OS, or tenancy at any point during the term. Cannot be sold on the Marketplace.

For Regional vs Zonal: a **Regional RI** gives AZ and size flexibility within the region but does not reserve capacity. A **Zonal RI** reserves capacity in a specific AZ but locks you to that AZ with no size flexibility.

**Two Savings Plan flavors:**

- **Compute Savings Plan** — Up to ~66% off. The most flexible option — applies automatically across any region, any instance family, any size/OS/tenancy, and also covers Fargate and Lambda. Pick this when your usage spans multiple regions or includes containers and serverless.
- **EC2 Instance Savings Plan** — Up to ~72% off. Locked to one instance family in one region, but flexible on size, AZ, and OS within that constraint. The cheapest committed option when you know your family and region.

**Decision shortcuts for the exam:**

- Workload uses **Fargate or Lambda** → **Savings Plan** (RIs don't cover these services).
- Need flexibility to move across **different regions or instance families** → **Compute Savings Plan**.
- Steady EC2 with a **known instance family and region**, want the maximum discount → **EC2 Instance Savings Plan** or **Standard RI**.
- Might **change instance family** later → **Convertible RI** (or Compute Savings Plan).
- Want the ability to **sell or exit** the commitment → **Standard RI** (Savings Plans cannot be sold).
- Need a **guaranteed capacity reservation** in a specific AZ → **Zonal RI** (Savings Plans provide no capacity guarantee).
- Reserving **RDS, Redshift, ElastiCache, DynamoDB, or OpenSearch** → these use their own **Reserved Instances or Reserved Nodes** — Savings Plans do not apply to database or analytics services.

---

## Placement groups (how EC2 instances are physically positioned)

Placement groups control how EC2 instances are laid out on the underlying hardware. The exam tests which one fits a stated goal — **performance vs. resilience**:

| Type | Layout | Use when |
|------|--------|----------|
| **Cluster** | Packs instances close together on the **same rack** in a single AZ | You need the **lowest network latency and highest throughput between instances** — HPC, tightly-coupled compute. Trade-off: if the rack fails, you can lose the whole group (no resilience). |
| **Spread** | Places each instance on **distinct racks** (separate hardware, power, network), max **7 instances per AZ** | You have a small number of **critical instances that must not share a failure domain** — e.g., individual members of a cluster you can't afford to lose together. |
| **Partition** | Groups instances into **partitions**, each on its own set of racks; up to 7 partitions per AZ, many instances each | **Large distributed/replicated workloads** that are partition-aware — HDFS, Cassandra, Kafka. A rack failure takes down at most one partition. |

- Keyword "lowest latency / highest throughput between instances, HPC" → **Cluster**.
- Keyword "few critical instances, isolate from each other's hardware failures" → **Spread** (7 per AZ max).
- Keyword "large distributed data store (Hadoop/Cassandra/Kafka), partition-aware HA" → **Partition**.

**EFA (Elastic Fabric Adapter)** is a network interface for HPC/ML that enables very low-latency, high-throughput inter-node communication (OS-bypass) — pair it with a cluster placement group. **Enhanced networking (ENA)** gives higher bandwidth and lower latency than standard, used by most modern instance types.

**EC2 Hibernate** saves the in-memory (RAM) state to the EBS root volume on stop and restores it on start, so the instance resumes where it left off (faster than a cold boot) — useful for long-booting applications. The root volume must be encrypted, and there's a max hibernation duration.

---

## Auto Scaling (ASG)

An Auto Scaling Group maintains a desired number of EC2 instances across **multiple AZs**, automatically replacing failed instances and scaling the count up or down based on demand. It's the foundation of self-healing, elastic architectures.

**Scaling policies** determine when and how to scale:

- **Target tracking** — you specify a metric and a target value (e.g., keep average CPU at 50%), and Auto Scaling adjusts the fleet to maintain it. The simplest and most commonly recommended policy.
- **Step scaling** — scale by different amounts depending on how far the metric exceeds a threshold.
- **Simple scaling** — a single add/remove action when an alarm fires; older and less flexible.
- **Scheduled scaling** — scale at specific times you define; useful for predictable traffic patterns like a daily business hours spike.
- **Predictive scaling** — uses ML to forecast traffic based on historical patterns and pre-scales before the demand arrives, rather than reacting after the fact.

A **Launch Template** defines the instance configuration (AMI, instance type, security group, etc.) that the ASG uses when launching new instances. It replaces the older launch configuration.

A **cooldown period** prevents the ASG from launching or terminating additional instances immediately after a scaling action, giving the new instances time to start handling traffic before the next scaling decision is made. **Warm-up** is a similar concept for instances that need time to become fully functional after launch.

Combining an ASG with an ELB lets the load balancer's health checks drive instance replacement — if the ELB marks an instance unhealthy, the ASG terminates and replaces it automatically.

- Keyword "handle unpredictable traffic automatically / self-healing" → **ASG + ELB**.

---

## Elastic Load Balancing

AWS offers three types of load balancers, each operating at a different network layer:

| LB | Layer | Best for |
|----|-------|----------|
| **ALB (Application Load Balancer)** | Layer 7 (HTTP/HTTPS) | Content-based routing by host or URL path, microservices, containers, WebSocket, WAF integration, Lambda targets |
| **NLB (Network Load Balancer)** | Layer 4 (TCP/UDP) | Extreme performance (millions of requests/sec), ultra-low latency, **static or Elastic IP addresses**, TLS passthrough |
| **GWLB (Gateway Load Balancer)** | Layer 3 | Deploying third-party virtual network appliances (firewalls, intrusion detection systems) transparently inline |

**Sticky sessions** (session affinity) keep a user's requests going to the same target — the ALB does this by inserting a cookie that identifies the target. Useful for stateful applications that store session data locally.

All load balancers support cross-zone load balancing (distributing traffic evenly across instances in all AZs) and health checks (automatically removing targets that fail checks from the rotation).

**NLB cross-zone load balancing — a hidden cost trap:** For ALB, cross-zone load balancing is always enabled and the cross-AZ data transfer is included at no extra charge. For NLB, cross-zone load balancing is **disabled by default**, and when you turn it on, AWS charges for the data transferred between the NLB node and the targets located in other AZs. This catches teams off guard — they enable cross-zone load balancing on an NLB expecting only an availability improvement, and then see unexpected data transfer charges on their bill.

- Keyword "static IP address for a load balancer" → **NLB**.
- Keyword "route by URL path or hostname" → **ALB**.

---

## Lambda (serverless compute)

Lambda lets you run code in response to events without provisioning or managing any servers. You pay only for the number of requests and the duration your code runs (billed in 1ms increments). Key limits to know: maximum execution timeout is **15 minutes**, memory can go up to 10 GB (CPU scales proportionally with memory), and `/tmp` ephemeral storage is up to 10 GB.

**Concurrency** controls how many function instances run simultaneously. **Reserved concurrency** guarantees a specific number of concurrent executions for a function (and prevents it from consuming the account's shared pool). **Provisioned concurrency** pre-initializes a set of execution environments so they're warm and ready to respond instantly — this eliminates cold starts for latency-sensitive workloads.

Lambda functions are triggered by events from many AWS services: API Gateway (HTTP requests), S3 (object events), DynamoDB Streams (data changes), SQS (messages), SNS (notifications), EventBridge (scheduled or event-driven), and Kinesis (stream records).

Lambda can be configured to run inside a VPC, which is required when your function needs to connect to private resources like an RDS database in a private subnet. **Lambda Layers** let you package shared libraries or dependencies separately and reuse them across multiple functions.

**Lambda in a VPC — a very common exam trap:** You might assume that placing a Lambda function in a public subnet (one with a route to an Internet Gateway) would give the function internet access, just like an EC2 instance in a public subnet. This is completely wrong and catches many people out. Lambda ENIs (the network interfaces that Lambda creates inside your subnet) **never receive a public IP address**, so placing a Lambda function in a public subnet provides absolutely no internet access.

The correct architecture when you need a Lambda function to access both private VPC resources (like RDS) AND the internet (like an external API) is: place the Lambda function in a **private subnet**, and deploy a **NAT Gateway in a public subnet**. The private subnet routes outbound internet traffic to the NAT Gateway, the NAT Gateway has a public IP, and internet traffic flows through it. The full chain is: Lambda → private subnet → route table → NAT Gateway → Internet Gateway → internet.

To be clear: Lambda in a public subnet = no internet. Lambda in a private subnet + NAT Gateway = internet access.

- Keyword "Lambda in VPC needs to call an external API or reach the internet" → place Lambda in a **private subnet** + NAT Gateway in a public subnet. A public subnet alone does NOT work.
- Keyword "run code without managing servers / event-driven / minimize operational overhead" → **Lambda**.
- Long-running tasks (over 15 minutes), always-on services, or workloads requiring persistent local state → **not** Lambda; use ECS/Fargate or EC2.

---

## Containers

**ECS (Elastic Container Service)** is AWS's native container orchestration service. It manages the scheduling and running of containers for you. ECS has two launch types: **EC2 launch type**, where you manage the underlying EC2 instances yourself (you're responsible for patching, scaling the cluster nodes, etc.), and **Fargate launch type**, where AWS manages the infrastructure entirely and you only define the container's CPU and memory requirements.

**EKS (Elastic Kubernetes Service)** is managed Kubernetes on AWS. Use it when you need to run the Kubernetes ecosystem specifically — either because your team already knows Kubernetes, you need portability across clouds, or you rely on specific Kubernetes APIs and tooling.

**Fargate** is the serverless compute engine for containers — it works with both ECS and EKS. You don't manage nodes, don't patch servers, and don't think about cluster capacity. You just define what resources your container needs, and Fargate runs it.

**ECR (Elastic Container Registry)** is AWS's managed container image registry, where you store and version your Docker images.

**Decision guide:** Do you need Kubernetes specifically? → EKS. Otherwise, do you want the simplest setup with no server management? → ECS with Fargate. Do you need full control over the host (GPU access, Spot mix, custom kernel)? → ECS or EKS on EC2.

---

## Other compute services

**AWS Batch** handles large-scale batch computing workloads — it manages job queues, dynamically provisions the right amount of compute, and runs jobs in an optimal order. Use it for anything that runs as a scheduled or triggered batch job at scale.

**Elastic Beanstalk** is AWS's Platform-as-a-Service (PaaS) offering. You upload your application code and Beanstalk automatically handles provisioning EC2 instances, load balancers, Auto Scaling, and deployment. The underlying resources still exist (you can inspect and modify them), but you don't need to set them up yourself. Use it when you want to deploy quickly without managing infrastructure manually.

**AWS Outposts** brings AWS hardware and services into your own data center, letting you run AWS APIs and services on-premises for workloads that need to stay local (regulatory requirements, ultra-low latency to on-prem systems).

**Wavelength and Local Zones** are extensions of AWS regions placed in telecom networks (Wavelength) or in specific metro areas (Local Zones), used when you need single-digit millisecond latency to end users in a specific location.

---

## Keyword map

| Scenario | Answer |
|----------|--------|
| Cheapest for interruptible batch jobs | Spot Instances |
| Steady 24/7 workload, save cost | Reserved Instances / Savings Plans |
| Event-driven, no servers | Lambda |
| Containers with no server management | Fargate |
| Need a static IP on a load balancer | NLB |
| Route HTTP traffic by URL path or hostname | ALB |
| Scale proactively for a predictable daily spike | Scheduled or Predictive scaling |
| Per-socket/core licensing compliance | Dedicated Host |
| Lowest inter-instance latency (HPC) | Cluster placement group (+ EFA) |
| Isolate few critical instances from shared hardware failure | Spread placement group |
| Partition-aware big-data cluster (Cassandra/HDFS/Kafka) | Partition placement group |
