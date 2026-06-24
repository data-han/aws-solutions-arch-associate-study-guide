# 10 — AI/ML Managed Services & Commonly-Missed Extras

SAA-C03 is an architecture exam, not an ML exam — but it reliably includes a **handful of "which AI service solves this?" questions**. You don't need to know how the models work; you need the **keyword → service** mapping. Pick the *purpose-built managed service* over "build it yourself on SageMaker/EC2" whenever the scenario matches one of these.

---

## AI/ML services (the keyword-mapping ones)

| Service | One-line purpose | Trigger keywords |
|---------|------------------|------------------|
| **Amazon Rekognition** | Image & video analysis | "detect objects/faces/text in images", "content moderation", "facial analysis", "celebrity recognition" |
| **Amazon Textract** | Extract text/data from scanned **documents** | "extract text from PDFs/forms/tables", "digitize invoices/IDs", "OCR with structure" |
| **Amazon Comprehend** | NLP on text | "sentiment analysis", "entity/key-phrase extraction", "detect PII in text", "topic modeling" |
| **Amazon Transcribe** | Speech → text | "convert audio/voice to text", "call-center transcription", "captions/subtitles" |
| **Amazon Polly** | Text → lifelike speech | "convert text to audio/voice", "read articles aloud", "voice response" |
| **Amazon Translate** | Machine translation | "translate text between languages", "localize content automatically" |
| **Amazon Lex** | Conversational chatbots (the engine behind Alexa) | "build a chatbot", "voice/text conversational interface" |
| **Amazon Kendra** | ML-powered **enterprise search** | "natural-language search across internal documents", "intelligent enterprise search" |
| **Amazon Personalize** | Real-time recommendations | "product/content recommendations", "personalization engine like Amazon.com" |
| **Amazon Forecast** | Time-series forecasting | "predict future demand/inventory/sales from historical data" |
| **Amazon Fraud Detector** | ML fraud detection | "detect online/payment fraud", "fake account detection" |
| **Amazon SageMaker** | Build/train/deploy **custom** ML models | "train and host a custom model", "full ML pipeline", "data scientists need notebooks" |

**The most common trap:** the question describes a task one of the *purpose-built* services does out of the box (e.g., "transcribe support calls and analyze sentiment"), and one distractor says "build and train a custom model on SageMaker." The purpose-built combo (**Transcribe → Comprehend**) is the low-overhead, correct answer. SageMaker is only correct when the scenario explicitly needs a **custom** model that no managed service provides.

**Common chained pattern:** audio → **Transcribe** → text → **Comprehend** (sentiment/PII) → store/analyze. Or document → **Textract** → text → **Comprehend**.

---

## Analytics & data services you might see

These extend note 04's data section — lighter weight but exam-relevant:

| Service | Purpose |
|---------|---------|
| **Amazon QuickSight** | Serverless **BI dashboards / visualization** (the "make a dashboard for business users" answer; pairs with Athena/Redshift/CUR). |
| **AWS Lake Formation** | Builds and secures a **data lake** on S3 with centralized, fine-grained permissions over data cataloged by Glue. Keyword "central permissions for a data lake." |
| **AWS Glue** | Serverless **ETL** + the **Data Catalog** (central metadata store). Keyword "serverless ETL", "crawl and catalog data for Athena." |
| **Amazon Athena** | Serverless **SQL directly on S3** (see note 04). Pairs with Glue Catalog + QuickSight. |
| **Amazon OpenSearch Service** | Search & **log analytics** at scale; the destination for full-text search and operational-log dashboards (ELK-style). |
| **Amazon EMR** | Managed **Hadoop/Spark** for big-data processing (see note 02/04). |
| **Amazon MSK** | Managed **Apache Kafka** — for teams already standardized on Kafka (vs. Kinesis for new AWS-native streaming). |

- "Dashboards/visualization for business users" → **QuickSight**.
- "Secure a data lake with central fine-grained access" → **Lake Formation**.
- "Search/log analytics, full-text" → **OpenSearch**.
- "Already use Kafka, want managed" → **MSK**; "new AWS-native stream" → **Kinesis**.

---

## End-user computing & misc services

| Service | Purpose / keyword |
|---------|-------------------|
| **Amazon WorkSpaces** | Managed **virtual desktops (DaaS)** — "give remote employees a cloud desktop." |
| **Amazon AppStream 2.0** | **Stream a single application** to users' browsers (vs. a full desktop). |
| **AWS Amplify** | Fast hosting + backend for **web/mobile front-end** apps. |
| **AWS Device Farm** | Test apps on **real mobile devices** in the cloud. |
| **Amazon SES** | Bulk/transactional **email** (see note 07). |
| **Amazon Pinpoint** | Targeted multi-channel **customer messaging/campaigns** (email/SMS/push) — marketing-oriented vs. SES's raw sending. |

---

## Migration services

When moving servers and databases from on-premises to AWS, AWS provides specialized tools for each stage: discovering what you have, moving it efficiently, and tracking progress centrally.

**AWS Application Discovery Service** helps you plan a migration before it starts. It automatically inventories your on-premises data center — discovering servers, their hardware specifications (CPU, memory, disk), running processes, and the network connections between them. This dependency mapping is especially valuable: you need to know which application servers communicate with which databases before you start moving things, so you can migrate interdependent systems together rather than breaking connections mid-migration. Trigger: "discover and map on-premises servers before migrating to AWS", "identify server dependencies for migration planning."

**AWS Application Migration Service (MGN)** is the recommended tool for lifting and shifting physical servers and virtual machines to EC2. It works by installing a lightweight agent on each source server that continuously replicates the server's disk blocks to AWS in the background. When you are ready to cut over, MGN launches an exact EC2 replica of the server. The actual cutover typically causes less than 15 minutes of downtime. Trigger: "lift-and-shift on-premises servers to EC2", "rehost servers with minimal downtime and no re-architecting."

**AWS Migration Hub** provides a single centralized dashboard where you can track the progress of all your migrations, regardless of which tools are being used — MGN, AWS Database Migration Service, or third-party tools. Instead of logging into each tool separately to check status, Migration Hub aggregates everything into one status view showing which servers are being discovered, which are replicating, which are ready to cut over, and which are complete. Trigger: "track all migration progress from one central place."

---

## Governance and compliance appendix services

These services appear on the exam mostly as distractors, but occasionally one of them is the correct answer. Understanding the one-sentence purpose of each is sufficient.

**AWS Artifact** is a self-service portal where you can download AWS's own compliance documentation — things like AWS's SOC 1, SOC 2, and SOC 3 reports, PCI DSS attestation of compliance, ISO 27001 certificates, and other third-party audits of AWS's infrastructure. This is specifically for when your auditor needs proof that AWS's data centers are certified, not for proving your own compliance. Think of it as "download the report card that AWS got from its own auditors." Trigger: "our auditor needs AWS's own SOC 2 report", "download AWS compliance certifications."

**AWS Audit Manager** is the opposite of Artifact — instead of AWS's compliance evidence, Audit Manager helps you collect and organize your own compliance evidence. It continuously monitors your AWS account and automatically maps resource configurations, CloudTrail activity logs, and Security Hub findings to recognized compliance frameworks like PCI DSS, HIPAA, SOC 2, and GDPR. Before an audit, instead of spending weeks manually gathering screenshots and export files as evidence, Audit Manager has already assembled it on an ongoing basis. Trigger: "automate evidence collection for our own compliance audit", "continuously gather proof that our environment meets PCI DSS requirements."

**AWS Service Catalog** lets a central IT or platform team publish a curated library of approved CloudFormation templates — called "products" — that end users can deploy themselves through a self-service portal without needing full CloudFormation permissions or deep AWS expertise. The platform team controls what is available and what guardrails apply (instance types, regions, encryption settings); business teams browse the approved catalog and deploy what they need in a governed way. Trigger: "self-service portal for approved infrastructure", "governed provisioning without giving developers full CloudFormation access."

**AWS License Manager** helps you track and enforce software license compliance, particularly for BYOL (Bring Your Own License) scenarios. Some software vendors license their products by the physical core or socket count (Oracle Database, Microsoft SQL Server) rather than by the number of instances. License Manager tracks how many vCPUs your EC2 instances are using and can block you from launching additional instances that would exceed your purchased license count — preventing accidental license violations that could result in expensive audit findings. Trigger: "track BYOL per-core or per-socket license usage on EC2", "prevent license violations."

---

## Additional integration and media services

**Amazon AppFlow** is a fully managed, no-code integration service that transfers data between popular SaaS applications (Salesforce, ServiceNow, Slack, Zendesk, SAP) and AWS services (S3, Redshift). You configure a "flow" entirely through the console — choose the source application, the destination, define any field mappings or filters, and set a schedule. No Lambda functions, no custom ETL code, no infrastructure to maintain. This is specifically for when a business team wants data from an external SaaS application to land in AWS with minimal engineering effort. Trigger: "transfer Salesforce records to S3 without writing any code", "SaaS application data into AWS, no engineering effort required."

**Amazon Kinesis Video Streams** is a managed service for ingesting, storing, and processing live video streams from connected devices — security cameras, drones, dash cameras, industrial sensors, and smartphones. The video data can be fed in real time into machine learning models (such as Amazon Rekognition Video) for object detection, anomaly detection, or facial analysis, or stored for later playback and offline analysis. Trigger: "ingest live video from cameras or IoT devices for ML analysis or storage."

**Amazon Elastic Transcoder** is a media conversion service that converts video files stored in S3 into formats and resolutions suitable for playback on specific devices — phones, tablets, web browsers, and smart TVs. For example, you might upload a high-resolution source video and use Elastic Transcoder to create multiple lower-resolution versions for users on mobile data connections. Note: AWS Elastic Transcoder is the older service; AWS Elemental MediaConvert is the newer, more powerful successor recommended for new projects. However, Elastic Transcoder still appears on the SAA-C03 exam for basic format conversion scenarios. Trigger: "convert uploaded video files to formats that play on phones, tablets, and browsers."

---

## Keyword map (AI / analytics quick reference)

| Scenario | Answer |
|----------|--------|
| Detect objects/faces/moderate images & video | Rekognition |
| Extract text/tables from scanned documents | Textract |
| Sentiment / entities / PII in **text** | Comprehend |
| Audio/voice → text | Transcribe |
| Text → spoken audio | Polly |
| Translate between languages | Translate |
| Build a chatbot | Lex |
| Natural-language search over internal docs | Kendra |
| Product/content recommendations | Personalize |
| Forecast demand from time-series history | Forecast |
| Train/host a **custom** ML model | SageMaker |
| BI dashboards for business users | QuickSight |
| Central permissions over an S3 data lake | Lake Formation |
| Full-text search / log analytics | OpenSearch |
| Managed Kafka | MSK |
| Cloud virtual desktops for employees | WorkSpaces |
| Transfer SaaS (Salesforce/ServiceNow) data to S3, no code | AppFlow |
| Ingest live video from devices/cameras for ML or storage | Kinesis Video Streams |
| Convert uploaded video to play on phones/tablets/browsers | Elastic Transcoder |
| Download AWS's own SOC 2 / PCI / ISO compliance reports | AWS Artifact |
| Automate YOUR own compliance evidence collection | AWS Audit Manager |
| Lift-and-shift servers to EC2 with minimal downtime | MGN (Application Migration Service) |
| Discover and map on-premises servers before migration | Application Discovery Service |
| Track all migration progress from one central dashboard | Migration Hub |
| Self-service portal for approved CloudFormation templates | Service Catalog |
| Track and enforce BYOL per-core software licenses on EC2 | License Manager |
