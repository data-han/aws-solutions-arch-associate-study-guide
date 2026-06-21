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
