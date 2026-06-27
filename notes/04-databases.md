# 04 — Databases: RDS, Aurora, DynamoDB, ElastiCache, Redshift

## Picking the right database

The exam frequently gives you a scenario and asks which database service fits. Start here:

| Need | Service |
|------|---------|
| Relational, managed, standard engines (MySQL, Postgres, etc.) | **RDS** |
| Relational, cloud-native, higher performance and HA | **Aurora** |
| NoSQL key-value, single-digit millisecond latency, serverless scale | **DynamoDB** |
| In-memory caching / session store | **ElastiCache** |
| Data warehouse / complex analytical queries (OLAP) | **Redshift** |
| Graph relationships | **Neptune** |
| Document store (MongoDB-compatible) | **DocumentDB** |
| Time-series data | **Timestream** |
| Immutable ledger / financial audit trail | **QLDB** |
| Wide-column (Cassandra-compatible) | **Keyspaces** |

### Why you can't just use Redis for everything

Redis is fast and flexible, but it has hard limits that make it the wrong tool outside its lane:

| Constraint | Why it matters |
|---|---|
| **Memory-only (expensive)** | RAM costs ~10–50× more per GB than SSD. Storing TBs of data in Redis is prohibitively expensive. Aurora/DynamoDB store on disk cheaply. |
| **No real querying** | Redis has no SQL, no joins, no filtering. "All orders from users in Singapore over $500" requires pulling data out and filtering in app code. One SQL query handles this in Aurora. |
| **Primitive data model** | No tables, no relationships, no schema. Complex relational data (users → orders → products) is unnatural and hard to maintain in Redis. |
| **Not fully ACID** | Redis persistence is best-effort. Under a crash, you can lose the last second of writes. Financial transactions need Aurora/RDS with true ACID guarantees. |
| **RAM scale ceiling** | Each Redis shard is bounded by the RAM of that node. DynamoDB is disk-backed and scales to virtually unlimited size. |

**Use Redis when:** speed is the primary constraint, the data fits in memory, and you don't need SQL or relational structure.

| Need | Right tool |
|---|---|
| Complex queries, joins, ACID transactions | **Aurora / RDS** |
| Massive scale, flexible NoSQL, single-digit ms | **DynamoDB** |
| Sub-millisecond, sessions, leaderboards, counters | **ElastiCache Redis** |
| Analytical queries across huge datasets | **Redshift / Athena** |

---

## RDS (managed relational databases)

RDS supports MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora as managed engines — AWS handles provisioning, patching, backups, and hardware, so you focus on your schema and queries.

**Multi-AZ** creates a **synchronous standby replica** in a different Availability Zone. If the primary instance fails, AWS automatically fails over to the standby and updates the DNS endpoint — your application reconnects without you doing anything. Multi-AZ is about **high availability and automatic failover**, not read scaling. The standby cannot serve read traffic while the primary is healthy.

**Read Replicas** use **asynchronous replication** to create additional copies of your database that can serve read traffic, offloading read queries from the primary. You can have up to 5 Read Replicas for standard RDS engines (up to 15 for Aurora). Read Replicas can be in different regions, which also helps with disaster recovery since they can be promoted to standalone databases if needed.

**Backups**: RDS takes automated backups daily and captures transaction logs throughout the day, giving you point-in-time recovery to any second within your backup retention window (1–35 days). You can also take manual snapshots at any time, which are kept until you explicitly delete them.

RDS supports **storage auto-scaling**, so your storage grows automatically when it's running low without manual intervention. Encryption at rest is configured at creation time via KMS and cannot be changed after the fact without creating an encrypted snapshot and restoring from it.

**RDS Proxy** sits between your application and RDS and maintains a connection pool, reusing database connections efficiently. This is particularly valuable for Lambda functions, which can open thousands of short-lived connections and overwhelm a database. It also improves failover speed because applications connect to the Proxy's stable endpoint rather than the database endpoint directly.

- Keyword "HA / automatic failover for RDS" → **Multi-AZ**.
- Keyword "offload read traffic / scale reads" → **Read Replica**.

---

## Aurora

Aurora is AWS's cloud-native relational database engine compatible with MySQL and PostgreSQL. It was engineered from the ground up for cloud-scale performance — AWS claims approximately **5x the throughput of MySQL** and **3x the throughput of PostgreSQL** on the same hardware. Storage automatically grows in 10 GB increments up to 128 TB without you doing anything.

The key architectural difference from RDS is Aurora's storage layer: your data is **replicated across 6 copies spread over 3 Availability Zones** at all times. If a storage node fails, Aurora detects it and repairs the data automatically in the background using the other copies. This makes it self-healing without any manual intervention.

Aurora supports up to **15 Read Replicas** with minimal replication lag (typically under 100ms). In the event of a primary failure, Aurora automatically promotes the lowest-lag replica to become the new primary — failover is much faster than standard RDS Multi-AZ.

**Aurora Serverless v2** removes the need to choose or manage instance sizes. Capacity automatically scales up and down in fine-grained increments based on actual demand, making it cost-effective for variable or unpredictable workloads that would otherwise sit idle at a fixed instance size.

**Aurora Global Database** replicates your database to up to 5 secondary AWS regions with a typical lag of under 1 second. This enables disaster recovery across regions (you can promote a secondary region to primary in under 1 minute) and low-latency reads for globally distributed users.

- Keyword "MySQL/Postgres but need higher performance and HA with low operational overhead" → **Aurora**.
- Keyword "spiky or unpredictable database workload, scale to near-zero" → **Aurora Serverless v2**.
- Keyword "cross-region DR for relational database, sub-second replication" → **Aurora Global Database**.

---

## DynamoDB (serverless NoSQL)

DynamoDB is a fully managed, serverless NoSQL database that delivers **single-digit millisecond latency** at any scale — it doesn't matter if you have 1 user or 100 million, the response time stays consistent. There are no servers to provision, patch, or manage.

**Capacity modes** control how you pay and how DynamoDB handles traffic:

- **On-Demand mode** — you pay per request (read or write). No capacity planning needed; DynamoDB scales instantly to handle any traffic level. Best for unpredictable or spiky workloads.
- **Provisioned mode** — you specify the number of Read Capacity Units (RCUs) and Write Capacity Units (WCUs) you need. Cheaper than On-Demand for predictable, steady traffic. You can enable Auto Scaling to adjust provisioned capacity automatically within bounds you set.

**DAX (DynamoDB Accelerator)** is a fully managed, in-memory cache that sits in front of DynamoDB. It reduces read latency from single-digit milliseconds to **microseconds** — use it for read-heavy workloads where even DynamoDB's native latency isn't fast enough.

**Global Tables** provide multi-region, multi-active replication — your table is automatically replicated to and writable in multiple AWS regions simultaneously. Any write in one region propagates to all others within seconds. Use this for globally distributed active-active applications.

**TTL (Time to Live)** lets you set an expiry timestamp on items; DynamoDB automatically deletes expired items in the background. Useful for session data, temporary records, or any data with a natural expiry.

**DynamoDB Streams** capture a time-ordered log of every change to items in a table (inserts, updates, deletes) and make it available to consumers in near-real-time. A common pattern is to trigger a **Lambda function** from a stream to react to data changes — for example, updating a search index when a record changes.

**Indexes**: A **Local Secondary Index (LSI)** uses the same partition key as the base table but a different sort key, and must be created at the same time the table is created. A **Global Secondary Index (GSI)** can have a completely different partition key and sort key, and can be added to an existing table at any time. GSIs are more flexible and more commonly used.

- Keyword "serverless NoSQL, millisecond latency, massive scale" → **DynamoDB**.
- Keyword "even faster reads (microseconds)" → **DAX**.
- Keyword "multi-region active-active" → **Global Tables**.

---

## ElastiCache (in-memory caching)

ElastiCache provides managed in-memory data stores for caching. The primary use case is reducing latency and load on your primary database by caching frequently read results in memory — instead of hitting the database every time, your application checks the cache first.

**Redis is not just a cache.** This is a common misconception. Redis is a full in-memory data structure store — it can act as a cache, a session store, a message broker, a real-time leaderboard, and a pub/sub system. The data lives in memory (so reads are sub-millisecond), but Redis can also persist data to disk, which means it survives restarts and is not purely ephemeral like Memcached.

### Redis vs Memcached

| Feature | Redis | Memcached |
|---|---|---|
| Data structures | Rich — strings, lists, sets, **sorted sets**, hashes, bitmaps | Simple key-value only |
| Persistence | Yes — can snapshot to disk (RDB) or write an append-only log (AOF) | No — data lost on restart |
| Replication | Yes — primary + read replicas | No |
| Multi-AZ failover | Yes — automatic | No |
| Pub/sub messaging | Yes | No |
| Multi-threading | Single-threaded per shard | Multi-threaded |
| Use case | Sessions, leaderboards, pub/sub, complex caching, queues | Pure horizontal read caching, simplest setup |

**Sorted sets explained simply** — every member has a numeric score, and Redis keeps the entire collection ranked by score automatically. One command gives you the top-N players, one command gives you a specific player's rank. This is why Redis is the answer for leaderboards — no other AWS service has this built in.

**When Redis is used as a primary data store (not just a cache):**
- Leaderboards — sorted sets hold the live ranking
- Session state — the session data lives in Redis, not a relational DB
- Rate limiting — Redis atomically increments a counter per user per second
- Pub/sub — producers push messages, consumers subscribe

In all these cases Redis is the *source of truth* for that data, not just a copy of something in a database behind it.

### Common use cases

- **Database read caching** — cache query results so the DB gets fewer hits
- **Session store** — any server in an ASG can read any user's session from Redis
- **Leaderboard** — sorted sets give real-time ranking at sub-millisecond speed
- **Rate limiting** — atomic counters track request rates per user

| Scenario | Answer |
|---|---|
| Leaderboard / real-time ranking / sorted set | **ElastiCache Redis** |
| Session store for stateless fleet | **ElastiCache Redis** or DynamoDB |
| Simple, pure read cache, no persistence needed | **Memcached** |
| Sub-millisecond DynamoDB reads specifically | **DAX** (not ElastiCache) |

- Keyword "reduce read load on a database / cache query results" → **ElastiCache**.
- Keyword "session state for a stateless application fleet" → **ElastiCache (Redis)** or **DynamoDB**.
- Keyword "leaderboard / sorted / real-time ranking / sub-millisecond" → **ElastiCache Redis**.
- Keyword "pub/sub messaging in-memory" → **ElastiCache Redis**.

---

## Redshift (data warehouse, OLAP)

Redshift is AWS's managed data warehousing service, designed for **petabyte-scale analytical queries** across large datasets. It uses a columnar storage format, which is much more efficient for the kinds of aggregation queries (SUM, AVG, GROUP BY across millions of rows) that power business intelligence and reporting workloads.

Redshift is **not designed for transactional (OLTP) workloads** — don't use it for your application's main database. It's for running heavy analytical queries, typically on historical data.

**Redshift Spectrum** allows Redshift to query data directly in S3 without loading it into Redshift first — useful for querying infrequently accessed historical data that lives in your data lake.

- Keyword "complex analytical queries / BI / data warehouse / OLAP" → **Redshift**.

---

## Other data services

**Athena** lets you run standard SQL queries directly against data stored in S3 — no database to set up, no infrastructure to manage. You pay per terabyte scanned. It's serverless and works best for ad-hoc queries over data already in S3 (logs, exports, event data). It's commonly paired with **AWS Glue**, which provides a managed data catalog (a central metadata store for your data) and ETL (extract, transform, load) capabilities to prepare data for querying.

**EMR (Elastic MapReduce)** is a managed big-data platform that runs Apache Hadoop, Spark, Hive, and other frameworks. Use it for large-scale data processing jobs that need the full Hadoop/Spark ecosystem.

**DMS (Database Migration Service)** helps you migrate databases to AWS with minimal downtime. It supports both **homogeneous migrations** (e.g., MySQL to MySQL on RDS) and **heterogeneous migrations** (e.g., Oracle to Aurora PostgreSQL). For heterogeneous migrations, you use the **Schema Conversion Tool (SCT)** to convert the schema and stored procedures from the source dialect to the target dialect before migrating the data.

---

## Quick disambiguation

- **Read Replica vs Multi-AZ**: Read Replicas are for **scaling read throughput** — they serve reads. Multi-AZ is for **high availability and failover** — the standby doesn't serve traffic. Don't confuse them.
- **Athena vs Redshift**: Athena is serverless and great for **ad-hoc SQL on S3 data** without loading it anywhere. Redshift is a full data warehouse for **repeated, complex analytical workloads** where you load and structure data for fast querying.
- **ElastiCache vs DAX**: ElastiCache is a general-purpose in-memory cache for **any database or application**. DAX is purpose-built as a **DynamoDB-specific** cache and is the right choice when you need sub-millisecond DynamoDB reads.
