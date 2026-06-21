# 03 — Storage: S3, EBS, EFS, FSx, Storage Gateway

## Amazon S3 (object storage — most-tested storage service)

S3 is AWS's core object storage service. It stores unlimited numbers of objects, with each individual object allowed up to **5 TB** in size. For objects larger than 100 MB you should use multipart upload (it's required above 5 GB) — this splits the object into parts, uploads them in parallel, and S3 reassembles them.

Bucket names must be **globally unique** across all AWS accounts. The data itself is stored regionally (you choose the region), and S3 provides **11 nines (99.999999999%) durability** — meaning S3 redundantly stores your data across multiple AZs within that region, making data loss extremely unlikely.

S3 also provides **strong read-after-write consistency** for all operations since December 2020, meaning that after you successfully write or update an object, any subsequent read immediately reflects the new version.

### Storage classes (cost questions love these)

S3 offers multiple storage tiers — you choose based on how often you expect to access the data and whether you need instant retrieval:

| Class | Use | Notes |
|-------|-----|-------|
| **S3 Standard** | Frequently accessed data | Default class; highest availability and performance |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns | Automatically moves objects between access tiers based on usage; small monthly monitoring fee but **no retrieval fees** |
| **Standard-IA** | Infrequent access but rapid retrieval when needed | Cheaper per-GB storage than Standard, but charges a retrieval fee; objects must stay at least 30 days |
| **One Zone-IA** | Infrequent, re-creatable data | Same as Standard-IA but stored in a single AZ only (less durable / resilient); cheaper than Standard-IA |
| **Glacier Instant Retrieval** | Archive data you rarely access but need in milliseconds | Cheapest option that still gives immediate (ms) retrieval |
| **Glacier Flexible Retrieval** | Archive where you can wait minutes to hours | Retrieval takes minutes to 12 hours; significantly cheaper storage cost |
| **Glacier Deep Archive** | Long-term archive, cheapest of all | Retrieval takes up to 12 hours; minimum storage duration 180 days |

**Lifecycle policies** let you define rules that automatically transition objects between storage classes (e.g., move to Standard-IA after 30 days, then to Glacier after 90) or expire (delete) objects entirely after a set time — this lets you manage cost without manual intervention.

- Keyword "unknown access pattern, optimize cost automatically" → **Intelligent-Tiering**.
- Keyword "cheapest archival, retrieval in hours OK" → **Glacier Deep Archive**.

### S3 security & features

**Access control** works in layers. Block Public Access is enabled by default on new buckets and acts as a safety net that overrides any policy or ACL that would make data public — even if a bucket policy accidentally allows public access, Block Public Access will override it. The main access-control tools are **bucket policies** (resource-based IAM policies attached to the bucket) and **IAM policies** (user/role-based). ACLs are a legacy mechanism; avoid them on new setups.

**Encryption at rest** has three server-side options: **SSE-S3** (AWS manages everything with S3-owned keys — now the default), **SSE-KMS** (AWS KMS manages the keys, giving you audit trails via CloudTrail and the ability to set key policies), and **SSE-C** (you provide your own key with each request; S3 uses it but never stores it). Client-side encryption is also possible if you encrypt before uploading.

**Versioning** keeps every version of every object, protecting against accidental overwrites and deletes. Once enabled, deleting an object just adds a delete marker — previous versions remain and can be restored. **MFA Delete** adds an extra layer requiring MFA to permanently delete a version. **Object Lock** enforces WORM (write-once-read-many) protection — objects can't be overwritten or deleted for a defined retention period, used for compliance and legal hold scenarios.

**Replication**: **CRR (Cross-Region Replication)** copies objects to a bucket in a different AWS region — useful for disaster recovery or reducing read latency for geographically distributed users. **SRR (Same-Region Replication)** copies to a bucket in the same region — useful for log aggregation or keeping a separate dev copy. Both require versioning enabled on source and destination.

**Presigned URLs** grant temporary, time-limited access to a private S3 object without making the object public — you generate the URL server-side (using AWS credentials) and hand it to a user or application.

**S3 Transfer Acceleration** speeds up uploads over long distances by routing data through the nearest CloudFront edge location and using AWS's optimized backbone network to reach the bucket's region. Useful when users are far from the bucket's region.

**S3 Event Notifications** can trigger a Lambda function, SQS queue, SNS topic, or EventBridge rule in response to events like `s3:ObjectCreated` — for example, automatically processing or resizing an image when it's uploaded.

**Static website hosting** lets you serve a bucket's content as a website. Combined with CloudFront, this is the standard pattern for global static sites (CloudFront handles caching at edge locations and adds HTTPS).

Other useful features: **Requester Pays** (the requester, not the bucket owner, pays for data transfer), **S3 Access Points** (simplify access management at scale — each team/app gets its own access point with its own policy instead of one giant bucket policy), and **S3 Select** (run SQL queries against a single object to retrieve only the subset of data you need, rather than downloading the whole thing).

---

## EBS (block storage, attached to EC2)

EBS provides persistent block storage volumes for EC2 instances — think of them as virtual hard drives. They are **locked to a single Availability Zone**, so an EBS volume in `us-east-1a` can only be attached to an EC2 instance also in `us-east-1a`. To move a volume to a different AZ or region, you take a **snapshot** (stored in S3) and restore it in the target AZ/region.

| Type | Best for |
|------|----------|
| **gp3 / gp2** | General-purpose SSD — boot volumes and most standard workloads. gp3 provides a baseline of 3,000 IOPS regardless of volume size, and is cheaper than gp2; prefer gp3 for new volumes |
| **io2 / io1** | High-performance SSD with **provisioned IOPS** — databases and latency-sensitive workloads where you need guaranteed, consistent IOPS. io2 supports Multi-Attach |
| **st1** | Throughput-optimized HDD — large sequential reads/writes like log processing, data warehouse, or big data. High throughput, lower IOPS |
| **sc1** | Cold HDD — the cheapest EBS option; for data that is infrequently accessed and where lowest cost matters more than performance |

**Multi-Attach** (io1/io2 only) allows a single EBS volume to be attached to multiple EC2 instances simultaneously, but only within the same AZ. Used for clustered applications that coordinate access themselves.

EBS encryption uses KMS to encrypt data at rest on the volume, all data in transit between the volume and the instance, and all snapshots — and any volumes restored from those snapshots are also encrypted.

- Keyword "highest IOPS / database needs guaranteed IOPS" → **io2**.

---

## Instance Store

Instance Store volumes are physically attached to the host server running your EC2 instance, giving you extremely high IOPS and throughput — much higher than EBS. The critical trade-off is that they are **ephemeral**: the data is lost if the instance is stopped, terminated, or if the underlying host fails. Use them only for temporary data you don't need to persist: caches, scratch space, or buffers.

---

## EFS (Elastic File System — shared Linux file storage)

EFS is a managed NFS file system that can be mounted by many EC2 instances simultaneously, even across multiple AZs. It is **elastic** — storage capacity scales up and down automatically as you add and remove files, and you pay only for what you use.

EFS has two storage classes: **Standard** (for actively used files) and **IA (Infrequent Access)** (cheaper, for files not accessed often). A **One Zone** variant stores data in a single AZ for lower cost. You can configure lifecycle policies to move files automatically between Standard and IA.

EFS is Linux-only (NFS protocol). For Windows shared storage, use FSx for Windows File Server instead.

- Keyword "shared file system across many Linux instances / AZs" → **EFS**.

---

## FSx (managed third-party file systems)

FSx is AWS's service for running popular third-party file system software as a fully managed service — meaning AWS handles provisioning, patching, and backups.

- **FSx for Windows File Server** — A fully managed Windows file system supporting the **SMB protocol** and integrated with **Active Directory**. Use this when you have Windows workloads (e.g., .NET apps, home directories) that need shared file storage with Windows permissions and AD authentication.

- **FSx for Lustre** — A high-performance parallel file system designed for compute-intensive workloads like **HPC (High-Performance Computing)** and **ML training**. It can be linked to an S3 bucket so it presents S3 objects as files and can write results back to S3. Use this when you need extremely high throughput and low latency for processing large datasets.

- **FSx for NetApp ONTAP / OpenZFS** — Managed ONTAP or OpenZFS file systems, mainly useful for migrating on-premises NetApp or ZFS workloads to AWS with minimal changes.

- Keyword "Windows shared storage (SMB/AD)" → **FSx for Windows**. "HPC or ML high throughput" → **FSx for Lustre**.

---

## Hybrid storage

### Storage Gateway

**Storage Gateway** is a service that bridges your on-premises environment with AWS cloud storage. You run it as a virtual appliance (VM) in your data center, and it exposes familiar storage protocols — file shares, block volumes, or tape drives — to your local applications. Under the hood it stores the data in S3, Glacier, or as EBS snapshots. This means existing on-prem applications can keep working without being rewritten to use AWS APIs. There are three types:

- **File Gateway** — Presents cloud storage as a standard **NFS** (Linux) or **SMB** (Windows) network file share to on-prem servers. Files written to the share are stored as objects in **S3**. A local cache on the gateway appliance keeps recently accessed files available at low latency. Use this when you want existing file-based applications to read and write to S3 without changing the application code.

- **Volume Gateway** — Presents cloud-backed storage as **iSCSI block volumes** — to your on-prem servers, it looks exactly like a local hard disk. It operates in two modes: *cached mode* stores your full dataset in S3 but caches recently accessed blocks on-prem for low-latency reads; *stored mode* keeps the full dataset on-prem (for lowest latency) and asynchronously backs it up to S3 as EBS snapshots for disaster recovery. Use this for block-level storage and DR backups of on-prem volumes.

- **Tape Gateway** — Emulates a **Virtual Tape Library (VTL)**, making it appear to your existing backup software (Veeam, Veritas, etc.) as if there's a physical tape library attached. Backup jobs run unchanged, but the virtual tapes are stored in **S3** (while in active use) and can be archived to **S3 Glacier** for long-term retention. Use this to retire physical tape hardware while keeping your current backup workflows intact.

### DataSync

DataSync is a fast, online data transfer service for copying large amounts of data between on-premises storage and AWS (S3, EFS, FSx), or between AWS storage services. It handles encryption, data integrity validation, scheduling, and bandwidth throttling for you. Use DataSync for one-time migrations or ongoing scheduled sync jobs — situations where you want to move or continuously replicate data over the network.

### AWS Transfer Family

Transfer Family is a **fully managed SFTP / FTPS / FTP** (and AS2) service that lands files directly into **S3 or EFS**. The classic scenario: a partner or legacy system uploads files over SFTP, and you need that data in S3 without running and patching your own SFTP server fleet. You keep the existing SFTP workflow and credentials; AWS manages the server, scaling, and availability.

- Keyword "partners upload via SFTP/FTPS, store in S3, no servers to manage" → **AWS Transfer Family** (don't confuse with DataSync, which is bulk AWS-managed sync, not a protocol endpoint for third parties).

### Snowball / Snowmobile

Snowball and Snowmobile are physical devices for **offline** bulk data transfer. AWS ships you a hardware appliance, you load your data onto it locally, and then ship it back to AWS for ingestion. This is used when the dataset is so large (petabytes) or your network connection so slow that transferring over the internet would take weeks or months.

- Keyword "migrate PB of data, slow/limited network" → **Snowball**. "Ongoing scheduled online sync" → **DataSync**.
- Keyword "on-prem app needs NFS/SMB file share backed by S3" → **File Gateway**. "Replace physical tape backups" → **Tape Gateway**.

---

## Storage decision cheat sheet

| Need | Service |
|------|---------|
| Object store, web-scale, cheap | S3 |
| Block storage for one EC2 / DB | EBS |
| Shared Linux file system (multi-AZ) | EFS |
| Shared Windows (SMB/AD) file system | FSx for Windows |
| HPC / ML scratch space | FSx for Lustre |
| Cheapest long-term archive | S3 Glacier Deep Archive |
| On-prem app needs cloud-backed storage | Storage Gateway |
| Move large data online / scheduled sync | DataSync |
| Offline bulk migration (slow network / huge data) | Snowball |
| Partners upload via managed SFTP/FTPS into S3/EFS | AWS Transfer Family |
