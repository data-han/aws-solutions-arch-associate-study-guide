# Exam-Realistic Set C — 25 Hard Questions on Commonly-Missed Topics

**Why this set exists:** Sets A and B drill the high-frequency core. Set C deliberately targets the topics most study packs (and the first version of this one) under-cover: **placement groups, the AI/ML services, IPv6/egress-only IGW, AWS Transfer Family, Cognito user vs identity pools, OpenSearch/QuickSight/Lake Formation, MSK vs Kinesis, and a few sharp disambiguations.** Same exam-fidelity style — long stems, a buried qualifier, distractors that "work" but aren't optimal.

> ⚠️ Original questions written to mirror exam style — not real exam content. Cover the answers; full rationale (incl. why each distractor fails) at the bottom.

---

**C1.** A genomics company runs a tightly-coupled HPC simulation across 20 EC2 instances that exchange large volumes of data with each other and are extremely sensitive to inter-node network latency. Which configuration gives the **best inter-instance network performance**?
- A. A spread placement group across 3 AZs
- B. A cluster placement group with instances using Elastic Fabric Adapter (EFA)
- C. A partition placement group with 7 partitions
- D. Instances spread across multiple AZs behind a Network Load Balancer

**C2.** A company runs a self-managed Apache Cassandra cluster of 18 nodes on EC2. They want to **minimize the chance that a single hardware/rack failure takes down multiple nodes**, while still running all nodes in one AZ for low latency. Which placement strategy fits best?
- A. Cluster placement group
- B. Spread placement group
- C. Partition placement group
- D. No placement group; rely on Auto Scaling

**C3.** A media app lets users upload photos. The company must **automatically detect and block inappropriate (explicit) images** before they're published, with no ML expertise on the team. What should the architect use?
- A. Train a custom image classifier on Amazon SageMaker
- B. Amazon Rekognition content moderation
- C. Amazon Textract
- D. Amazon Comprehend

**C4.** A call center records customer support calls as audio files in S3. The company wants to **transcribe each call and then analyze the transcript for customer sentiment**, with the least operational overhead. Which combination should the architect recommend? **(Choose TWO.)**
- A. Amazon Transcribe
- B. Amazon Polly
- C. Amazon Comprehend
- D. Amazon Translate
- E. A custom NLP model on SageMaker

**C5.** A logistics company has 5 years of historical shipment-volume data and wants to **predict next quarter's demand per region** to plan capacity. They have no data-science team. What is the lowest-overhead solution?
- A. Amazon Forecast
- B. Amazon Kendra
- C. Amazon Personalize
- D. A regression model built and hosted on SageMaker

**C6.** An insurance company receives thousands of scanned claim forms (PDFs and images) daily and needs to **extract the form fields and table data** into a structured format for downstream processing. Which service is purpose-built for this?
- A. Amazon Rekognition
- B. Amazon Textract
- C. Amazon Comprehend
- D. Amazon Transcribe

**C7.** A workload runs EC2 instances that are assigned **only IPv6 addresses**. The instances must be able to **download OS patches from the internet** but must remain **unreachable from inbound internet connections**. What should the architect configure?
- A. A NAT gateway in a public subnet
- B. An egress-only internet gateway
- C. An internet gateway with security group rules
- D. A NAT instance with IPv6 forwarding

**C8.** A company exchanges files with external business partners who upload data using **SFTP**. The company wants the uploaded files to land directly in Amazon S3 and does **not** want to operate or patch any SFTP servers. Which service should the architect use?
- A. AWS DataSync
- B. AWS Transfer Family
- C. AWS Storage Gateway (File Gateway)
- D. An EC2 instance running an SFTP daemon writing to S3

**C9.** A mobile application needs end users to **sign up and sign in** (including via Google and Apple), and after authentication the app must **directly and securely access user-specific objects in Amazon S3**. Which approach is correct?
- A. Use a Cognito user pool only and embed IAM access keys in the app
- B. Use a Cognito user pool for sign-in and a Cognito identity pool to vend temporary AWS credentials via an IAM role
- C. Create one IAM user per app user
- D. Use IAM Identity Center for the end users

**C10.** A company wants to give **API Gateway** a managed way to authenticate end users with email/password and social login, returning a token the API can validate, without building an auth service. What should they use?
- A. A Cognito user pool as the API Gateway authorizer
- B. A Cognito identity pool
- C. An IAM role attached to API Gateway
- D. AWS Directory Service

**C11.** A retailer wants to add a **product recommendation engine** ("customers who bought this also bought…") to its website based on user behavior, similar to Amazon.com, with minimal ML work. Which service?
- A. Amazon Personalize
- B. Amazon Forecast
- C. Amazon Comprehend
- D. Amazon Kendra

**C12.** An enterprise wants employees to **ask natural-language questions and search across millions of internal documents** (wikis, PDFs, SharePoint) and get precise answers, not just keyword hits. Which service is purpose-built?
- A. Amazon OpenSearch Service
- B. Amazon Kendra
- C. Amazon CloudSearch with a custom ranking function
- D. Amazon Comprehend

**C13.** A company ingests application and infrastructure logs and needs **full-text search and interactive dashboards/visualizations over the log data** for operational troubleshooting. Which service is the best fit?
- A. Amazon Athena
- B. Amazon OpenSearch Service
- C. Amazon QuickSight
- D. Amazon Redshift

**C14.** A business-intelligence team needs **interactive dashboards** for non-technical executives, sourcing data from Athena and Redshift, with per-user access and no servers to manage. Which service?
- A. Amazon QuickSight
- B. Amazon OpenSearch Dashboards
- C. Amazon Managed Grafana
- D. A self-hosted BI tool on EC2

**C15.** A data-platform team is building a **data lake on S3** consumed by multiple analytics teams. They need **centralized, fine-grained (table- and column-level) access control** across Athena, Redshift Spectrum, and EMR. What should they use?
- A. S3 bucket policies per team
- B. AWS Lake Formation
- C. IAM policies on each analytics service
- D. AWS Config rules

**C16.** A company already runs **Apache Kafka** on-premises and wants to move to a managed streaming service on AWS with **minimal changes to its Kafka-based producers and consumers**. Which service minimizes migration effort?
- A. Amazon Kinesis Data Streams
- B. Amazon MSK (Managed Streaming for Apache Kafka)
- C. Amazon SQS
- D. Amazon Kinesis Data Firehose

**C17.** A real-time clickstream pipeline must ingest streaming data and **automatically deliver it to Amazon S3 and Amazon Redshift with no consumer code to write and no servers to manage**, buffering by time/size. Which service?
- A. Amazon Kinesis Data Streams with a custom KCL consumer
- B. Amazon Kinesis Data Firehose
- C. Amazon MSK
- D. Amazon SQS with a Lambda consumer

**C18.** A company wants to provide **remote employees with persistent, managed virtual desktops** accessible from anywhere, without shipping laptops or managing VDI infrastructure. Which service?
- A. Amazon AppStream 2.0
- B. Amazon WorkSpaces
- C. Amazon EC2 with RDP
- D. AWS Outposts

**C19.** A SaaS provider needs to **stream a single internal Windows application** to external users' web browsers without giving them a full desktop or installing anything locally. Which service?
- A. Amazon WorkSpaces
- B. Amazon AppStream 2.0
- C. Amazon Lightsail
- D. CloudFront with a custom origin

**C20.** A solutions architect must process a fleet of EC2 instances' **memory utilization** and alarm when it exceeds 90%. By default this metric is not available in CloudWatch. What is required?
- A. Enable detailed monitoring on the instances
- B. Install the CloudWatch agent to publish memory as a custom metric
- C. Read memory usage from EC2 instance metadata
- D. Enable VPC Flow Logs

**C21.** A company needs an **active-active** relational database spanning two AWS Regions, where **both Regions accept writes** with sub-second replication and fast regional failover. Which option best meets this?
- A. RDS Multi-AZ
- B. RDS cross-Region read replicas
- C. Aurora Global Database with write forwarding
- D. DynamoDB Global Tables

**C22.** An application sends one event per new order that must reach **three independent microservices**, each of which must **process events in strict order** and must **not lose events** if it is briefly offline. Which design fits best?
- A. An SNS standard topic with three Lambda subscribers
- B. An SNS FIFO topic fanning out to three SQS FIFO queues, one per microservice
- C. A single SQS standard queue shared by all three services
- D. Amazon Kinesis Data Firehose to each service

**C23.** A company wants to **automatically convert blog articles into natural-sounding audio** (a "listen to this article" feature) in multiple languages. Which combination is most appropriate? **(Choose TWO.)**
- A. Amazon Translate
- B. Amazon Transcribe
- C. Amazon Polly
- D. Amazon Comprehend
- E. Amazon Lex

**C24.** A financial firm runs a **latency-sensitive, tightly-coupled** trading workload on a cluster placement group. They are worried that if the underlying rack fails, the **entire group** goes down. Which statement reflects the correct trade-off and a valid mitigation?
- A. Cluster groups span AZs automatically, so rack failure is not a concern
- B. Cluster groups maximize throughput/low latency but share a failure domain; for resilience, run a second cluster group in another AZ and replicate
- C. Switch to a spread group with no performance impact
- D. Add more instances to the same cluster group to create redundancy

**C25.** A company stores PII in free-text customer feedback in S3 and must **automatically detect and redact the PII** before sharing the data with analysts. Which service detects the PII entities?
- A. Amazon Macie
- B. Amazon Comprehend
- C. Amazon Textract
- D. Amazon GuardDuty

---

## Answer Key — with why each distractor fails

**C1 — B.** A **cluster placement group + EFA** packs instances on the same rack and uses OS-bypass networking for the lowest latency / highest throughput between nodes — exactly what tightly-coupled HPC needs. *A:* spread maximizes isolation, the opposite of low-latency clustering. *C:* partition is for large partition-aware data stores, not latency-critical HPC. *D:* multi-AZ + NLB adds latency, not reduces it.

**C2 — C.** **Partition** placement group puts each partition on its own rack; a rack failure takes down at most one partition, ideal for a large Cassandra/HDFS/Kafka cluster (18 nodes exceeds spread's 7-per-AZ limit). *A:* cluster shares a failure domain — a rack loss could kill many nodes. *B:* spread caps at 7 instances per AZ, too few for 18 nodes. *D:* no placement group gives no rack-isolation guarantee.

**C3 — B.** **Rekognition** has built-in content moderation for explicit/inappropriate imagery — no training needed. *A:* training a custom model is far more overhead for a solved problem. *C:* Textract is documents/OCR. *D:* Comprehend is text NLP, not images.

**C4 — A and C.** **Transcribe** turns the call audio into text, then **Comprehend** runs sentiment analysis on the transcript — fully managed, no ML build. *B:* Polly is text→speech (wrong direction). *D:* Translate is languages, not sentiment. *E:* a custom SageMaker model is unnecessary overhead.

**C5 — A.** **Forecast** is purpose-built for time-series prediction from historical data with no data-science team. *B:* Kendra is enterprise search. *C:* Personalize is recommendations. *D:* SageMaker works but is much higher overhead than the managed forecasting service.

**C6 — B.** **Textract** extracts text, form fields, and table structure from scanned documents. *A:* Rekognition reads images/scenes, not structured document data. *C:* Comprehend analyzes text you already have. *D:* Transcribe is audio.

**C7 — B.** **Egress-only internet gateway** is the IPv6 equivalent of a NAT gateway: outbound-only for IPv6. *A/D:* NAT (gateway or instance) is IPv4-only. *C:* a plain internet gateway allows inbound too, violating the requirement.

**C8 — B.** **AWS Transfer Family** is fully managed SFTP landing directly in S3 — no servers to operate. *A:* DataSync is bulk AWS-managed transfer/sync, not an SFTP endpoint for third parties. *C:* File Gateway is on-prem NFS/SMB caching, not a partner SFTP endpoint. *D:* self-managed SFTP is exactly the operational burden being avoided.

**C9 — B.** **User pool** authenticates (incl. social IdPs); **identity pool** exchanges that token for temporary, role-scoped AWS credentials so the app can hit S3 directly. *A:* embedding access keys is insecure. *C:* one IAM user per end user doesn't scale and isn't the pattern. *D:* IAM Identity Center is for workforce SSO to AWS accounts, not millions of app end users.

**C10 — A.** A **Cognito user pool authorizer** on API Gateway validates the user pool JWT — managed sign-in with social login, no custom auth. *B:* identity pools vend AWS credentials, they aren't an API Gateway authorizer. *C:* an IAM role doesn't authenticate end users. *D:* Directory Service is managed AD, overkill/wrong fit.

**C11 — A.** **Personalize** delivers Amazon.com-style real-time recommendations from user behavior. *B:* Forecast is time-series demand. *C:* Comprehend is text NLP. *D:* Kendra is search.

**C12 — B.** **Kendra** is ML-powered natural-language enterprise search that returns precise answers across document repositories. *A:* OpenSearch is powerful but keyword/relevance search requiring more tuning, not natural-language Q&A out of the box. *C:* CloudSearch is older keyword search. *D:* Comprehend extracts entities, it doesn't search a corpus.

**C13 — B.** **OpenSearch Service** is built for full-text search and log analytics with dashboards (the ELK pattern). *A:* Athena is ad-hoc SQL on S3, not interactive search/dashboards. *C:* QuickSight is BI dashboards but not a full-text log search engine. *D:* Redshift is a data warehouse for structured analytics.

**C14 — A.** **QuickSight** is the serverless BI/dashboard service for business users, native to Athena/Redshift, with per-user access. *B:* OpenSearch Dashboards target log/search data. *C:* Managed Grafana is ops/metrics oriented. *D:* self-hosted adds the server management the question rules out.

**C15 — B.** **Lake Formation** provides centralized, fine-grained (table/column) permissions over an S3 data lake, enforced across Athena, Redshift Spectrum, and EMR. *A/C:* bucket and per-service IAM policies are coarse and decentralized — hard to manage column-level access. *D:* Config is compliance detection, not access control.

**C16 — B.** **MSK** is managed Apache Kafka — existing Kafka producers/consumers work with minimal change. *A/D:* Kinesis is AWS-native and would require rewriting the Kafka integration. *C:* SQS is a queue, not a Kafka-compatible stream.

**C17 — B.** **Firehose** is the zero-code, fully managed delivery stream to S3/Redshift with built-in buffering. *A:* Data Streams + KCL means writing and running a consumer. *C:* MSK needs consumers/management. *D:* SQS+Lambda is custom plumbing.

**C18 — B.** **WorkSpaces** = managed persistent virtual desktops (DaaS). *A:* AppStream streams a single app, not a full persistent desktop. *C:* EC2+RDP is unmanaged VDI. *D:* Outposts is on-prem AWS hardware, unrelated.

**C19 — B.** **AppStream 2.0** streams a single application to a browser. *A:* WorkSpaces gives a full desktop. *C:* Lightsail is simple VPS hosting. *D:* CloudFront caches content, it doesn't stream an interactive app.

**C20 — B.** Memory (and disk) aren't visible to the hypervisor, so you **install the CloudWatch agent** to publish them as custom metrics. *A:* detailed monitoring only increases frequency of existing metrics, it doesn't add memory. *C:* metadata doesn't expose memory usage. *D:* Flow Logs are network metadata.

**C21 — C.** **Aurora Global Database with write forwarding** lets secondary Regions accept writes (forwarded to the primary) with sub-second replication and fast cross-Region failover — the closest relational active-active. *A:* Multi-AZ is single-Region HA. *B:* read replicas are read-only. *D:* DynamoDB Global Tables are active-active but **NoSQL**, not relational as asked.

**C22 — B.** **SNS FIFO → SQS FIFO per consumer** gives ordered, durable, independent fan-out: each microservice gets its own ordered queue and won't lose events while offline. *A:* SNS standard + Lambda has no ordering and no durable buffer per consumer. *C:* one shared standard queue means the services compete for messages (not each gets all) and no ordering. *D:* Firehose delivers to data stores, it's not a per-service ordered work feed.

**C23 — A and C.** **Translate** converts the article into the target languages, then **Polly** synthesizes natural speech audio. *B:* Transcribe is speech→text (wrong direction). *D:* Comprehend analyzes text, doesn't produce audio. *E:* Lex builds chatbots.

**C24 — B.** Correct trade-off: cluster groups give peak performance but **share a failure domain**; the standard mitigation is a **second cluster group in another AZ** with replication. *A:* cluster groups are single-AZ, they do **not** span AZs. *C:* spread changes the performance profile (it can't pack instances), so "no impact" is false. *D:* more instances in the same group doesn't escape the shared rack/AZ failure domain.

**C25 — B.** **Comprehend** detects (and can redact) PII entities in free **text**. *A:* Macie discovers sensitive data in S3 but is oriented to data classification/discovery at the object level, not inline entity redaction of text in a pipeline. *C:* Textract extracts text from documents, it doesn't classify PII. *D:* GuardDuty is threat detection.
