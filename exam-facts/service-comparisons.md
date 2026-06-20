# Service Comparison / Decision Tables

The exam is mostly "X vs Y — which fits these requirements?" Drill these tables.

---

## S3 storage classes
| Class | Access | Min duration | Retrieval | Use |
|-------|--------|--------------|-----------|-----|
| Standard | frequent | — | instant | active data |
| Intelligent-Tiering | unknown/changing | — | instant | auto-optimize, no retrieval fee |
| Standard-IA | infrequent | 30 d | instant (fee) | backups, DR |
| One Zone-IA | infrequent, re-creatable | 30 d | instant (fee) | single-AZ, cheaper |
| Glacier Instant Retrieval | archive, rare | 90 d | ms | medical images |
| Glacier Flexible Retrieval | archive | 90 d | min–12h | backups |
| Glacier Deep Archive | rarely | 180 d | 12h | compliance archive, cheapest |

## EBS volume types
| Type | Category | Use |
|------|----------|-----|
| gp3 | SSD general | default, cheaper than gp2, 3000 IOPS baseline |
| gp2 | SSD general | legacy general |
| io2 / io1 | SSD provisioned IOPS | databases, highest/guaranteed IOPS, multi-attach |
| st1 | HDD throughput | big sequential (logs, data warehouse) |
| sc1 | HDD cold | infrequent, cheapest |

## File / shared storage
| Service | Protocol | OS | Scope |
|---------|----------|----|----|
| EFS | NFS | Linux | multi-AZ, many instances |
| FSx for Windows | SMB | Windows | AD-integrated |
| FSx for Lustre | Lustre | Linux | HPC/ML, S3-linked |
| EBS | block | any | single instance/AZ |
| Instance Store | block | any | ephemeral |

## Load balancers
| | ALB | NLB | GWLB |
|--|-----|-----|------|
| Layer | 7 HTTP/S | 4 TCP/UDP | 3 |
| Routing | host/path/header | flow hash | appliance |
| Static IP | no | **yes** (EIP) | — |
| Use | web/microservices/containers | extreme perf, low latency | 3rd-party firewalls/IDS |

## Database selection
| Workload | Service |
|----------|---------|
| Standard relational, managed | RDS |
| High-perf relational, HA, low ops | Aurora |
| Variable relational, scale to low | Aurora Serverless v2 |
| Key-value NoSQL, ms, serverless | DynamoDB |
| In-memory cache/session | ElastiCache (Redis/Memcached) |
| Data warehouse / OLAP | Redshift |
| Ad-hoc SQL on S3 | Athena |
| Graph | Neptune |
| Document (Mongo) | DocumentDB |
| Ledger/immutable | QLDB |
| Time-series | Timestream |
| Cassandra | Keyspaces |

## RDS Multi-AZ vs Read Replica
| | Multi-AZ | Read Replica |
|--|----------|--------------|
| Purpose | **HA/failover** | **read scaling** |
| Replication | synchronous | asynchronous |
| Failover | automatic | manual promote |
| Read traffic | no (standby) | yes |
| Cross-Region | no (standby same Region*) | yes |

## SQS vs SNS vs EventBridge vs Kinesis
| | SQS | SNS | EventBridge | Kinesis Data Streams |
|--|-----|-----|-------------|----------------------|
| Pattern | queue (pull) | pub/sub (push) | event bus | stream |
| Consumers | one logical group | many subscribers | many targets/rules | many, independent |
| Ordering | FIFO option | FIFO option | no | **yes (per shard)** |
| Replay | no | no | no (archive/replay feature) | **yes (retention)** |
| Best for | decouple/buffer | fan-out/notify | AWS/SaaS events, cron | real-time analytics/streaming |

## VPC connectivity
| Method | Encrypted | Dedicated | Use |
|--------|-----------|-----------|-----|
| VPC Peering | private | — | connect 2 VPCs (non-transitive) |
| Transit Gateway | private | — | hub for many VPCs + on-prem |
| Site-to-Site VPN | **yes** | no (internet) | quick hybrid, backup |
| Direct Connect | no (add VPN) | **yes** | consistent low-latency, high bandwidth |
| PrivateLink/Interface Endpoint | private | — | expose/consume services privately |

## CloudFront vs Global Accelerator
| | CloudFront | Global Accelerator |
|--|------------|--------------------|
| Caches content | **yes** | no |
| Protocol | HTTP/S | any TCP/UDP |
| IP | edge | **2 static anycast IPs** |
| Best for | static/dynamic web content | non-HTTP, fast regional failover, static IP |

## DR strategies (cost & RTO ascending)
| Strategy | RTO | Cost |
|----------|-----|------|
| Backup & Restore | hours | $ |
| Pilot Light | 10s of min | $$ |
| Warm Standby | minutes | $$$ |
| Active/Active (Multi-Site) | ~0 | $$$$ |

## EC2 pricing models
| Model | Commit | Savings | Use |
|-------|--------|---------|-----|
| On-Demand | none | — | spiky/short/unknown |
| Spot | none (interruptible) | up to 90% | fault-tolerant batch |
| Reserved | 1–3 yr | up to 72% | steady, specific |
| Savings Plans | 1–3 yr ($/hr) | up to 72% | steady, flexible across EC2/Fargate/Lambda |
| Dedicated Host | varies | — | BYOL per-socket, compliance |

## Encryption / secrets
| Service | Use |
|---------|-----|
| KMS | managed keys, integrated, envelope encryption |
| CloudHSM | dedicated single-tenant HSM, you control keys |
| ACM | TLS/SSL certs (free for AWS resources) |
| Secrets Manager | secrets + auto-rotation ($) |
| SSM Parameter Store | config/secrets, free, no auto-rotation |

## Security/monitoring services (don't confuse)
| Service | Answers |
|---------|---------|
| CloudWatch | metrics, alarms, logs (performance/ops) |
| CloudTrail | API audit — who did what |
| AWS Config | resource config & compliance over time |
| GuardDuty | threat detection (ML on logs) |
| Inspector | vulnerability scanning |
| Macie | sensitive data (PII) discovery in S3 |
| Security Hub | aggregate findings |
| Trusted Advisor | cost/security/perf/limits recommendations |
