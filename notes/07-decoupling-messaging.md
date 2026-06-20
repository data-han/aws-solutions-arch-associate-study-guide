# 07 — Decoupling & Application Integration (high-yield)

These services come up constantly in resilient and high-performing architecture questions. The most important skill is knowing **when to use SQS vs SNS vs EventBridge vs Kinesis** — each solves a different communication problem.

---

## SQS (Simple Queue Service) — queue-based, pull model

SQS is a message queue that decouples producers (the things sending work) from consumers (the things doing the work). A producer puts a message in the queue and moves on — it doesn't wait for the consumer to finish. The message stays in the queue (for up to **14 days**) until a consumer retrieves and deletes it. This design means a slow or temporarily failed consumer doesn't block or crash the producer.

**Queue types:**

- **Standard queues** offer nearly unlimited throughput. Messages are delivered at least once (a message could be delivered more than once in rare cases, so consumers should handle duplicates). The ordering is best-effort — messages are generally delivered in order but this is not guaranteed.
- **FIFO queues** guarantee exactly-once processing and strict first-in-first-out ordering. They are limited to 300 transactions per second (or 3,000 using batching). Use FIFO when the order of operations matters — for example, processing financial transactions.

**How message processing works:** when a consumer retrieves a message, SQS hides it from other consumers for a period called the **visibility timeout**. If the consumer successfully processes the message, it deletes it from the queue. If the consumer fails or the timeout expires before deletion, the message reappears and another consumer can try again — this is the retry mechanism.

**Long polling** means the consumer waits up to 20 seconds for a message to arrive before returning an empty response. This reduces the number of API calls (and cost) compared to short polling, which returns immediately even if the queue is empty.

A **Dead Letter Queue (DLQ)** is a separate queue where SQS moves messages that have been received more than a configured number of times without being successfully deleted — these are "poison" messages that keep failing. By routing them to a DLQ, you prevent them from clogging the main queue and can investigate them separately.

Consumers **pull** messages by polling the queue. A common pattern is to use an **Auto Scaling Group** scaled on the number of messages in the queue — as work piles up, more consumers are added; as the queue drains, they scale back down.

- Keyword "decouple components, buffer work, smooth out spikes, don't lose messages" → **SQS**.

---

## SNS (Simple Notification Service) — pub/sub, push model

SNS is a publish/subscribe messaging service. A publisher sends a single message to an SNS **topic**, and SNS **pushes** that message to all subscribers simultaneously. Subscribers can be SQS queues, Lambda functions, HTTP/HTTPS endpoints, email addresses, SMS phone numbers, and more.

The most powerful pattern is **SNS fan-out**: you send one message to an SNS topic, and multiple SQS queues each subscribed to that topic receive their own independent copy of the message. This lets multiple downstream systems each process the same event independently — for example, an image upload event fanning out to a queue for thumbnail generation, a queue for moderation, and a queue for analytics.

SNS is push-based — subscribers receive messages immediately when they're published, without polling. This is the key behavioral difference from SQS.

FIFO SNS topics are also available, which pair with SQS FIFO queues for ordered fan-out scenarios.

- Keyword "send one event to multiple subscribers simultaneously / fan-out / notifications" → **SNS**.

---

## EventBridge (event bus)

EventBridge is a serverless event routing service. It receives events from AWS services, your own applications, and third-party SaaS integrations, then applies **routing rules** to forward matching events to targets. Rules can filter events based on the event's content (not just the source), so you can route only the specific event types you care about to the right target.

EventBridge also supports **scheduled events** (cron expressions), making it the right service for triggering actions on a schedule — replacing the need for cron jobs on EC2.

It integrates with dozens of AWS services as targets (Lambda, SQS, SNS, Step Functions, ECS, API Gateway, etc.) and can receive events from hundreds of SaaS partners (Zendesk, Datadog, etc.) without any polling.

EventBridge has richer filtering capabilities than SNS (you can match on specific fields in the event JSON) and is better suited as the backbone of event-driven application architectures. For new event-driven designs, EventBridge is generally the preferred choice over SNS alone.

- Keyword "react to AWS service events / SaaS events / scheduled cron jobs / route events by content" → **EventBridge**.

---

## Kinesis (real-time streaming data)

Kinesis is a suite of services for working with streaming data — continuous, high-volume data that arrives in real time (clickstreams, IoT sensor readings, application logs, financial transactions).

| Service | What it does |
|---------|--------------|
| **Kinesis Data Streams** | Ingests and stores streaming data in ordered, replayable **shards**. Multiple independent consumer applications can read the same stream simultaneously. Data can be retained for up to 365 days and replayed from any point. The consumer builds its own processing logic. |
| **Kinesis Data Firehose** | A fully managed delivery pipeline that continuously loads streaming data into a destination (S3, Redshift, OpenSearch, Splunk) with no code to write. Near-real-time (buffered by seconds to minutes). Use it when you just want the data in a destination without building a consumer. |
| **Kinesis Data Analytics** | Runs SQL or Apache Flink queries directly on a stream in real time, for live analytics and aggregations without storing the raw data first. |

**When to use Kinesis vs SQS:**

- **SQS** is a work queue — producers put tasks in, one consumer group processes them and deletes the messages. There's no replay, no ordering guarantee (unless FIFO), and multiple consumer groups can't independently read the same message.
- **Kinesis Data Streams** is a streaming log — multiple independent consumer applications can all read the same data simultaneously, order within a shard is preserved, and consumers can replay past data within the retention window. Use Kinesis for real-time analytics, event sourcing, or multiple systems consuming the same stream.

- Keyword "real-time streaming, ordered data, multiple consumers, replay capability" → **Kinesis Data Streams**.
- Keyword "load streaming data into S3 or Redshift without writing consumer code" → **Kinesis Firehose**.

---

## Step Functions (workflow orchestration)

Step Functions lets you coordinate multiple AWS services (Lambda, ECS tasks, SNS, SQS, DynamoDB, etc.) into a **state machine** — a visual workflow where each step transitions to the next based on results, with built-in support for branching, parallel execution, waiting, retries on failure, and error handling.

Use Step Functions when you have a multi-step process that needs to be orchestrated with conditional logic, retries, and visibility into where each execution is in the workflow — things that would be messy and fragile to manage manually in code.

- Keyword "coordinate multiple steps, long-running workflow, visual orchestration" → **Step Functions**.

---

## API Gateway

API Gateway is a fully managed service that acts as the front door for your APIs. It handles accepting HTTP/REST/WebSocket requests, authenticating them (via IAM, Cognito user pools, or custom Lambda authorizers), routing them to a backend (Lambda, HTTP, or other AWS services), and returning the response.

It includes built-in throttling to protect your backend from traffic spikes, response caching to reduce latency and backend load, and deployment stages (dev, prod) with independent configurations and traffic shifting.

- Keyword "expose a serverless backend as a REST or HTTP API / throttle / authenticate API calls" → **API Gateway + Lambda**.

---

## AppSync

AppSync is AWS's managed **GraphQL** API service. Instead of REST (multiple endpoints, each returning a fixed shape), GraphQL lets clients specify exactly what data they want in a single request. AppSync also supports real-time subscriptions and offline data sync for mobile apps.

- Keyword "GraphQL API" → **AppSync**.

---

## SES and Amazon MQ

**SES (Simple Email Service)** is for sending high-volume transactional or bulk emails — order confirmations, password resets, marketing campaigns. It handles deliverability, bounce handling, and reputation management.

**Amazon MQ** is a managed message broker for **ActiveMQ and RabbitMQ**. It's specifically for migrating existing on-premises applications that already use standard messaging protocols like JMS or AMQP — you can lift-and-shift without rewriting the messaging layer. For any **new** application, prefer SQS and SNS; Amazon MQ exists purely for compatibility with existing apps.

---

## Decoupling keyword map

| Scenario | Answer |
|----------|--------|
| Buffer requests between components, retry on failure | SQS |
| One event delivered to multiple independent consumers | SNS (fan-out) or SNS → multiple SQS |
| Route AWS service events, SaaS events, or scheduled jobs | EventBridge |
| Real-time stream, ordered data, multiple consumers, replay | Kinesis Data Streams |
| Stream data directly into S3 or Redshift with no code | Kinesis Firehose |
| Orchestrate a multi-step workflow with retries and branching | Step Functions |
| Managed REST/HTTP API in front of Lambda or other services | API Gateway |
| Migrating an app that uses JMS or AMQP messaging | Amazon MQ |
