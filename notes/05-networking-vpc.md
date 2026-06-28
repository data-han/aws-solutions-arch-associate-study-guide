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
| **Client VPN** | Lets individual remote users (laptops, phones) connect to a VPC over an encrypted OpenVPN tunnel. Each person installs a VPN client and authenticates — then their device gets a private IP and can reach resources inside the VPC as if they were on the corporate network. |
| **Direct Connect (DX)** | A dedicated private physical fiber connection from your data center to AWS, bypassing the public internet entirely. Provides consistent, low-latency, high-bandwidth connectivity. Takes weeks to provision. Does not encrypt traffic by default — combine with a VPN over DX if encryption is required. |
| **DX + VPN** | Run a VPN tunnel over your Direct Connect line to get both the consistent performance of a dedicated connection and the encryption of a VPN. |

### Site-to-Site VPN vs Client VPN

These are commonly confused — the difference is **what is connecting to AWS**:

| | Site-to-Site VPN | Client VPN |
|---|---|---|
| Who connects | An entire **network** (office, data center) | Individual **users/devices** (laptops, phones) |
| Both ends | On-premises router ↔ AWS Virtual Private Gateway | User's VPN client app ↔ AWS Client VPN endpoint |
| Protocol | IPSec | OpenVPN (TLS) |
| Use case | Permanent network-to-network tunnel (hybrid cloud) | Remote workers, developers needing VPC access |
| Setup | Configure on your router/firewall | Users install a VPN client app |
| Scales to | The whole office through one tunnel | Individual users, each with their own session |

**Analogy:**
- **Site-to-Site** = a permanent bridge between two buildings. Everyone in building A can reach building B automatically — they don't do anything special.
- **Client VPN** = a security door each person opens with their own badge. Each person connects individually from wherever they are.

**Exam triggers:**
- "remote employees need to access private resources in the VPC" → **Client VPN**
- "connect entire on-premises office/data center to VPC" → **Site-to-Site VPN**
- "quick encrypted hybrid connection, backup to Direct Connect" → **Site-to-Site VPN**

- Keyword "connect hundreds of VPCs centrally, transitive routing" → **Transit Gateway**.
- Keyword "consistent low-latency dedicated private connection to on-premises" → **Direct Connect**.
- Keyword "quick, encrypted hybrid connection" → **Site-to-Site VPN**.
- Direct Connect takes weeks to provision — use a VPN as the interim solution while waiting, or as a failover backup.

**VPC Peering vs Transit Gateway — cost and scale comparison:** VPC Peering connections have **no hourly infrastructure charge** — you only pay the standard per-GB data transfer rate for traffic flowing through the connection. Transit Gateway charges you a separate **hourly fee per VPC attachment** plus a **per-GB data processing fee**. For a small number of VPCs (say, 2–4), VPC Peering is almost always cheaper. However, VPC Peering is non-transitive and the number of connections grows quadratically as VPCs increase (10 VPCs fully meshed = 45 peering connections). Transit Gateway solves both problems with a hub-and-spoke model, and its cost is justified once you have enough VPCs or routing complexity.

- Keyword "cheapest way to connect 2 or 3 VPCs" → **VPC Peering** (no hourly fee, just data transfer).
- Keyword "connect many VPCs, transitive routing, hub-and-spoke at scale" → **Transit Gateway**.

### AWS RAM — sharing VPC resources across accounts

**AWS Resource Access Manager (RAM)** lets you share specific AWS resources that you own with other AWS accounts — accounts inside your AWS Organization or even external accounts. The most common exam scenario is **shared VPC subnets**.

Here is the problem RAM solves: if you follow the best practice of giving each team its own separate AWS account for security isolation, each account would normally need its own VPC, subnets, NAT Gateways, and Transit Gateway attachments. This creates duplicated networking infrastructure and costs across many accounts.

With RAM, a central networking account creates one VPC with well-designed subnets, then uses RAM to share specific subnets with each team account. The team accounts can launch EC2 instances, RDS databases, and other resources directly into those shared subnets — as if the subnet belonged to their own account — without owning or controlling the underlying VPC.

A second common scenario is sharing a **Transit Gateway**: one central account creates and owns it, then shares it via RAM across the organization. All other accounts attach their own VPCs to the shared Transit Gateway.

- Keyword "share a VPC subnet across multiple AWS accounts, central networking team" → **AWS RAM**.
- Keyword "share a Transit Gateway across all org accounts without each account creating their own" → **AWS RAM**.

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

**OAC (Origin Access Control)** or the older OAI (Origin Access Identity) restricts an S3 bucket so that only CloudFront can access it — preventing users from bypassing CloudFront and accessing the S3 bucket directly. OAC controls access to the origin (your S3 bucket) but does not control which end users can access your CloudFront distribution. For restricting end-user access to specific content, you use Signed URLs or Signed Cookies.

**CloudFront Signed URLs and Signed Cookies** both restrict which authenticated users can access content through your distribution — for example, only paying subscribers can watch your video library. The difference is the scope of what they protect:

- **Signed URL**: grants a specific user access to **one single file** for a defined time window. You generate the signed URL on your backend server after the user is authenticated, and only that URL (with its cryptographic signature) will be served by CloudFront. Use a Signed URL when you need to give a user temporary access to one specific asset — for example, a purchased e-book download link, a generated PDF report, or a time-limited video file.

- **Signed Cookie**: grants a user access to **multiple files or an entire protected section** of your distribution at once. After the user logs in, your backend sets a signed cookie in their browser. While the cookie is valid, the user can access any content that falls within the cookie's policy — for example, all content under `/members/` or all videos in a subscription library. Use a Signed Cookie when a user needs access to many files, because generating a separate Signed URL for each file in a 1,000-video library would be impractical.

- Keyword "restrict CloudFront content to paying subscribers or authenticated users" → **Signed URLs** (for one file) or **Signed Cookies** (for many files or a whole section).

CloudFront integrates with **WAF** for Layer 7 protection at the edge, **ACM** for TLS certificates (important: for CloudFront, the certificate must be in the **us-east-1** region even if your content is elsewhere), and **Lambda@Edge / CloudFront Functions** for running lightweight code at the edge (e.g., request manipulation, URL rewrites, authentication checks).

**CloudFront Price Classes** let you limit which edge locations serve your distribution. By default, CloudFront uses all edge locations worldwide — including more expensive locations in South America and Australia. If your users are concentrated in North America and Europe, you are paying for edge infrastructure in regions where none of your users are. Price Classes let you opt out of the more expensive edge regions:

- **Price Class 100** — US, Canada, and Europe only. The lowest cost option. Choose this when your entire user base is in North America and Europe and you want to minimize CloudFront charges.
- **Price Class 200** — US, Canada, Europe, Asia, Middle East, and Africa. A middle ground.
- **Price Class All** — Every edge location globally, including South America and Australia. Highest cost, but best performance for users anywhere in the world.

- Keyword "reduce CloudFront cost, users are only in the US and Europe" → **Price Class 100**.
- Keyword "global low-latency content delivery / reduce origin load" → **CloudFront**.

---

## Global Accelerator

Global Accelerator uses the AWS global backbone network to route traffic from users to your application endpoints (which can be in one or multiple regions). It provides **two static anycast IP addresses** — users connect to those IPs, and traffic is routed over AWS's network (not the public internet) to the nearest healthy endpoint.

The key distinction from CloudFront: **CloudFront caches content** and is designed for HTTP/HTTPS workloads where caching helps. **Global Accelerator doesn't cache** — it's a network-layer acceleration service for TCP/UDP traffic, for non-cacheable dynamic content, or when you need fast, deterministic failover across regions and a fixed IP address.

- Keyword "global, static IP, fast multi-region failover, TCP/UDP or non-cacheable traffic" → **Global Accelerator**.

---

## How a request actually flows — DNS, edge, and VPC routing

Three things get called "routing" in AWS but they operate at completely different layers and never compete with each other:

```
┌─────────────────────────────────────────────────────────┐
│  1. ROUTE 53 (DNS)        "What IP is example.com?"    │
│     Answers before any packet is sent.                  │
│     Operates on: domain names, on the public internet   │
└────────────────────────┬────────────────────────────────┘
                         │ browser now has the IP → opens TCP connection
┌────────────────────────▼────────────────────────────────┐
│  2. EDGE SERVICE          first AWS service hit         │
│     CloudFront POP, Global Accelerator POP, or ALB      │
│     IGW is transparent here — never a destination       │
└────────────────────────┬────────────────────────────────┘
                         │ packet is inside the VPC
┌────────────────────────▼────────────────────────────────┐
│  3. ROUTE TABLE (VPC routing)  "Where does this go?"   │
│     IP prefix → local / IGW / NAT GW / TGW / endpoint  │
│     Operates on: IP addresses only, no domain names     │
└─────────────────────────────────────────────────────────┘
```

**Route 53 and route tables have nothing to do with each other** despite the similar name. Route 53 = DNS (names → IPs). Route tables = IP forwarding rules inside AWS.

### What is the first AWS service a user's request hits?

After Route 53 returns an IP your browser makes a **TCP connection** (then TLS handshake for HTTPS) to whatever that IP belongs to. **IGW is never the destination** — it's a transparent door on the VPC wall that passes packets through to ENIs. NAT Gateway handles only outbound traffic from private subnets; inbound users never touch it.

| Setup | First hit (edge) | Where TLS terminates |
|-------|-----------------|----------------------|
| CloudFront → ALB → EC2 | CloudFront edge POP (globally close to user) | CloudFront |
| ALB → EC2 (no CloudFront) | ALB (in public subnet; IGW is transparent) | ALB |
| Global Accelerator → ALB | GA anycast POP (static IPs, AWS backbone) | ALB |

**Full flow example (CloudFront setup):**
```
User types example.com
  → Route 53: returns CloudFront IP (via ALIAS record)
  → TCP + TLS to CloudFront edge POP  ← FIRST HIT
      cache hit? → serve immediately, never reaches origin
      cache miss? → fetch from ALB over AWS backbone
  → ALB (public subnet; IGW silently routes packet to ALB's ENI)
  → EC2 (private subnet; route table: 10.0.0.0/16 → local)
```

### Choosing the edge service

| Scenario | Edge |
|----------|------|
| Global users, HTTP/S, caching helps | **CloudFront** |
| Static IPs, TCP/UDP, non-cacheable dynamic traffic | **Global Accelerator** |
| Single-region, no caching needed | **ALB directly** (IGW handles entry, no extra edge layer) |

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
| What is the first service a user's request hits? | CloudFront (if used) → ALB → EC2; IGW is transparent, never the destination |
| Inbound traffic from internet never touches… | NAT Gateway (outbound only) or IGW directly (just a pass-through) |
