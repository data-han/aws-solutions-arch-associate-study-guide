# 05 — Networking: VPC, Route 53, CloudFront (heavily tested)

## VPC building blocks

A **VPC (Virtual Private Cloud)** is your own isolated virtual network within AWS. When you create a VPC, you assign it a CIDR block (an IP address range like `10.0.0.0/16`). A VPC spans all Availability Zones in a region but is confined to that region.

A **Subnet** is a subdivision of your VPC's IP range, and it lives in exactly **one Availability Zone**. Subnets are classified as:

- **Public subnet** — has a route in its route table pointing to an **Internet Gateway (IGW)**. Instances in a public subnet can be assigned a public IP and communicate directly with the internet.
- **Private subnet** — has no route to the internet. Instances here are isolated from direct internet access.

The **Internet Gateway** is what connects your VPC to the public internet. It's attached to the VPC (not a subnet), and subnets become "public" by having a route that points to it.

A **NAT Gateway** lets instances in a **private subnet initiate outbound connections to the internet** (e.g., to download software updates or call external APIs) while remaining unreachable from the internet — inbound connections from the internet still can't reach them. NAT Gateways are managed by AWS and are highly available **within a single AZ**. To maintain outbound internet access if an AZ goes down, you should deploy one NAT Gateway per AZ and route each AZ's private subnets through their local NAT Gateway.

- Keyword "private instances need to reach the internet (for updates etc.) but stay unreachable from outside" → **NAT Gateway**.

**Route tables** control where network traffic from a subnet is directed. Every subnet is associated with a route table.

An **ENI (Elastic Network Interface)** is a virtual network card that lives in a **subnet** (so it is pinned to one AZ). It holds:

- A **primary private IPv4** address (plus optional secondary private IPs)
- An optional **Elastic/public IP** per private IP
- A **MAC address**
- One or more **security groups** — note that SGs attach to the *ENI*, not the instance. An instance "has a security group" because its ENI does.
- A **source/destination check** flag (disable it on NAT instances or appliances that forward traffic on behalf of others)

An ENI can be **detached and re-attached to another instance in the same AZ**, so its IP/MAC can "float" to a standby instance for quick failover. The primary ENI (eth0) can't be detached; secondary ENIs can. ENIs cannot cross AZs.

**The mental model that ties it together:** whenever an AWS managed service needs to live inside your VPC's IP space, it does so by placing an **ENI in your subnet**. Interface VPC Endpoints (PrivateLink), RDS, ElastiCache, ELB nodes, NAT Gateway, EFS mount targets, Fargate tasks, and a VPC-configured Lambda are all just ENIs under the hood. If a question says "the service gets a private IP in your subnet," it's using an ENI.

---

### IPv6 and the egress-only internet gateway

A VPC always has a private IPv4 CIDR; you can **also** add an IPv6 CIDR block. IPv6 addresses in AWS are **always public/globally-routable** — there's no concept of a "private" IPv6 range like there is with IPv4 (no NAT for IPv6).

That creates a problem: how do you let IPv6 instances make **outbound** connections to the internet while staying **unreachable from inbound**? A NAT Gateway only works for IPv4. The answer is the **Egress-Only Internet Gateway (EIGW)** — it's the IPv6 equivalent of a NAT Gateway: it allows outbound IPv6 traffic from your instances and the return traffic, but blocks any internet-initiated inbound IPv6 connection.

- Keyword "IPv6 instances need outbound internet but must block inbound" → **Egress-Only Internet Gateway** (NAT Gateway is IPv4-only).

---

## Security layers

**Security Groups** are stateful, instance-level firewalls with allow-only rules — they're covered in depth in note 01.

**NACLs** are stateless, subnet-level firewalls supporting both allow and deny rules — also covered in note 01. The key difference that matters for networking scenarios: NACLs block at the subnet boundary before traffic ever reaches an instance, making them the right tool for blanket IP blocks.

---

## Connecting VPCs and on-premises networks

| Method | What it does and when to use it |
|--------|----------------------------------|
| **VPC Peering** | Creates a direct private network connection between two VPCs (in the same or different accounts/regions). Traffic stays on the AWS network. Peering is **non-transitive** — if A is peered with B and B is peered with C, A cannot reach C through B. No overlapping CIDR blocks allowed. Best for a small number of VPCs. |
| **Transit Gateway** | A regional hub that all your VPCs and on-premises networks connect to, creating a hub-and-spoke topology. Traffic between any two connected networks flows through the Transit Gateway. It's **transitive** — connecting to the hub gives you access to all other spokes. Use this when you need to connect many VPCs or build a scalable hybrid network. |
| **Site-to-Site VPN** | Creates an encrypted IPSec tunnel over the public internet between your on-premises network and your VPC. Quick to set up (minutes to hours), lower cost, but latency varies because it uses the public internet. Good for quick connectivity or as a backup link. |
| **Direct Connect (DX)** | A dedicated private physical fiber connection from your data center to AWS, bypassing the public internet entirely. Provides consistent, low-latency, high-bandwidth connectivity. Takes weeks to provision. Does not encrypt traffic by default — combine with a VPN over DX if encryption is required. |
| **DX + VPN** | Run a VPN tunnel over your Direct Connect line to get both the consistent performance of a dedicated connection and the encryption of a VPN. |

- Keyword "connect hundreds of VPCs centrally, transitive routing" → **Transit Gateway**.
- Keyword "consistent low-latency dedicated private connection to on-premises" → **Direct Connect**.
- Keyword "quick, encrypted hybrid connection" → **Site-to-Site VPN**.
- Direct Connect takes weeks to provision — use a VPN as the interim solution while waiting, or as a failover backup.

---

## VPC Endpoints (access AWS services without going through the internet)

By default, when an EC2 instance in a private subnet calls an AWS service like S3 or DynamoDB, that traffic routes out through a NAT Gateway and over the internet. VPC Endpoints let that traffic stay entirely within the AWS network, improving security and potentially reducing cost.

- **Gateway Endpoint** — supports **only S3 and DynamoDB**. It is added as an **entry in your route table** (an S3/DynamoDB *prefix list*) and is completely **free**. It uses **no ENI and no private IP** — it's pure routing. The catch: because it's just a route in *your* VPC's route table, it **only works from inside that VPC**. It can **not** be reached over Direct Connect/VPN (on-premises) or across VPC peering.
- **Interface Endpoint (powered by PrivateLink)** — supports S3 and most other AWS services. It creates an **ENI with a private IP** in your subnet, and DNS resolves the service's hostname to that private IP. Because it's a real ENI/IP, it **can** be reached from **on-premises** (over DX/VPN) and from **peered VPCs**. There's an hourly charge plus a per-GB data charge.

**S3 specifically has both options** — and choosing between them is a common exam point:

| You need to reach S3/DynamoDB privately from… | Use |
|------------------------------------------------|-----|
| EC2 **inside the VPC**, cheapest | **Gateway Endpoint** (free, no ENI) |
| **On-premises** over Direct Connect / VPN, or a **peered VPC** | **Interface Endpoint / PrivateLink** (ENI-based; gateway endpoints can't be reached from outside the VPC) |

Note the asymmetry: **DynamoDB only has a gateway endpoint** (no interface option), so you can't reach DynamoDB privately from on-premises the same way you can S3.

- Keyword "access S3 from a private subnet without a NAT Gateway, cheapest" → **Gateway Endpoint** (free, no ENI).
- Keyword "access S3 privately **from on-premises** over Direct Connect" → **Interface Endpoint (PrivateLink)** — a gateway endpoint can't be reached from outside the VPC.
- Keyword "privately expose a service in one VPC to consumers in other VPCs without peering" → **PrivateLink**.

---

## Route 53 (DNS and domain management)

Route 53 is AWS's managed DNS service that also handles domain registration and health checking. Understanding its **routing policies** is a heavily tested topic:

| Policy | What it does |
|--------|--------------|
| **Simple** | Returns a single record. No intelligence — just a standard DNS lookup. |
| **Weighted** | Distributes traffic across multiple targets in proportions you define (e.g., 80% to one, 20% to another). Use for A/B testing or canary deployments. |
| **Latency** | Routes each user's request to the AWS region that will give them the lowest network latency based on measurements. Use when you have resources in multiple regions and want each user served from the fastest one. |
| **Failover** | An active-passive setup where Route 53 routes traffic to a primary resource and switches to a secondary only if the primary's health check fails. Use for disaster recovery. |
| **Geolocation** | Routes users based on where they are located (continent, country, or US state). Use for content localization or compliance requirements that restrict data to specific locations. |
| **Geoproximity** | Routes based on geographic distance and can be biased (you can attract more or less traffic to a resource by adjusting a bias value). More flexible than Geolocation. |
| **Multivalue** | Returns up to 8 healthy records for the same name. Clients pick one randomly. Not a load balancer, but provides basic client-side distribution across healthy endpoints. |

**Alias records** are a Route 53-specific extension to standard DNS. Unlike a CNAME, an Alias record can point to an AWS resource (ALB, CloudFront distribution, S3 website endpoint, etc.) at the **zone apex** (the root domain like `example.com`, not just a subdomain). There's no extra charge for Alias record queries, and Route 53 automatically follows the target's IP changes.

- Keyword "route users to the nearest/fastest region" → **Latency routing policy**.
- Keyword "active-passive DNS-level disaster recovery" → **Failover routing policy**.

---

## CloudFront (CDN)

CloudFront is AWS's Content Delivery Network. It caches copies of your content at **edge locations** distributed globally so that users are served from a location physically close to them, reducing latency and offloading traffic from your origin.

CloudFront can serve content from multiple **origin types**: an S3 bucket, an ALB, an EC2 instance, or any HTTP endpoint. When a user requests content, CloudFront checks whether it has a cached copy at the nearest edge location. If it does (a cache hit), it serves it immediately without touching the origin. If not (a cache miss), it fetches from the origin, caches the response, and serves it.

**OAC (Origin Access Control)** or the older OAI (Origin Access Identity) restricts an S3 bucket so that only CloudFront can access it — preventing users from bypassing CloudFront and accessing the S3 bucket directly.

CloudFront integrates with **WAF** for Layer 7 protection at the edge, **ACM** for TLS certificates (important: for CloudFront, the certificate must be in the **us-east-1** region even if your content is elsewhere), and **Lambda@Edge / CloudFront Functions** for running lightweight code at the edge (e.g., request manipulation, URL rewrites, authentication checks).

- Keyword "global low-latency content delivery / reduce origin load" → **CloudFront**.

---

## Global Accelerator

Global Accelerator uses the AWS global backbone network to route traffic from users to your application endpoints (which can be in one or multiple regions). It provides **two static anycast IP addresses** — users connect to those IPs, and traffic is routed over AWS's network (not the public internet) to the nearest healthy endpoint.

The key distinction from CloudFront: **CloudFront caches content** and is designed for HTTP/HTTPS workloads where caching helps. **Global Accelerator doesn't cache** — it's a network-layer acceleration service for TCP/UDP traffic, for non-cacheable dynamic content, or when you need fast, deterministic failover across regions and a fixed IP address.

- Keyword "global, static IP, fast multi-region failover, TCP/UDP or non-cacheable traffic" → **Global Accelerator**.

---

## Monitoring & misc

**VPC Flow Logs** capture metadata about IP traffic flowing through your VPC (source/destination IPs, ports, protocol, whether traffic was accepted or rejected). Flow logs can be sent to CloudWatch Logs or S3 and are used for network troubleshooting, security analysis, and compliance auditing. Note that they capture metadata, not the actual packet contents.

Enable **DNS resolution and DNS hostnames** in your VPC settings when using VPC Endpoints — otherwise, services' private DNS names won't resolve to the endpoint's private IP.

---

## Keyword map

| Scenario | Answer |
|----------|--------|
| Private subnet instances need outbound internet (IPv4) | NAT Gateway |
| IPv6 instances need outbound-only internet | Egress-Only Internet Gateway |
| Access S3 privately from a private subnet, avoid NAT cost | Gateway VPC Endpoint (free, no ENI) |
| Access S3 privately from on-premises (DX/VPN) or a peered VPC | Interface Endpoint / PrivateLink (ENI-based) |
| Connect many VPCs and on-prem networks centrally | Transit Gateway |
| Dedicated, consistent, low-latency on-prem connection | Direct Connect |
| Quick encrypted hybrid connectivity | Site-to-Site VPN |
| Route users to the nearest/fastest region | Route 53 Latency routing |
| Active-passive DNS failover for DR | Route 53 Failover routing |
| Cache and deliver content globally | CloudFront |
| Static anycast IP + multi-region failover for TCP/UDP | Global Accelerator |
