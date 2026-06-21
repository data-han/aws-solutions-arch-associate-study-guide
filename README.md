# AWS Certified Solutions Architect – Associate (SAA-C03) Study Pack

A self-contained study repo to pass the **SAA-C03** exam: concise scenario-focused notes, high-yield exam facts, and practice/mock questions with explanations.

> Exam version targeted: **SAA-C03** (current as of 2026). Always cross-check against the official [exam guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html).

---

## The exam at a glance

| Item | Detail |
|------|--------|
| Questions | **65** total (50 scored + 15 unscored, unmarked) |
| Time | **130 minutes** (~2 min/question) |
| Question types | Multiple choice (1 of 4) and multiple response (2+ of 5+) |
| Scoring | Scaled **100–1000**, **pass = 720**, compensatory (no per-domain minimum) |
| Guessing | No penalty — **never leave a question blank** |
| Cost | ~$150 USD |
| Validity | 3 years |

### Domains & weighting (spend study time proportionally)

| # | Domain | Weight | Focus |
|---|--------|--------|-------|
| 1 | Design **Secure** Architectures | **30%** | IAM, encryption, network security, data protection |
| 2 | Design **Resilient** Architectures | **26%** | HA, fault tolerance, DR, decoupling |
| 3 | Design **High-Performing** Architectures | **24%** | Right storage/compute/DB/network for performance |
| 4 | Design **Cost-Optimized** Architectures | **20%** | Cheapest option that still meets requirements |

---

## How to use this repo

1. **Read `notes/`** topic by topic (01–10). These are condensed — they assume you'll also watch a course / do hands-on. Focus on *when to use X vs Y*.
2. **Drill `exam-facts/cheatsheet.md` and `service-comparisons.md`** — these are the decision tables the exam tests constantly. Review them daily before the exam.
3. **Do `practice/` by domain**, then work the harder `exam-realistic-set-A/B/C.md`, then take **`mock-exam-1.md`** and **`mock-exam-2.md`** under timed conditions (130 min each). Review every wrong answer back to the relevant note.

### Suggested 4-week plan

| Week | Goal |
|------|------|
| 1 | Notes 01–05 (IAM, compute, storage, DB, VPC) + domain-1 & domain-3 practice |
| 2 | Notes 06–09 (resilience, decoupling, monitoring, cost) + domain-2 & domain-4 practice |
| 3 | Re-read all cheatsheets; note 10 (AI/ML extras); Set A/B/C harder drills; redo every previously-missed question |
| 4 | Both full mock exams (timed) → review gaps → light re-read → sit exam |

---

## How to answer SAA-C03 questions (test strategy)

- **Read the last sentence first** — it tells you what's being optimized (cost? least operational overhead? highest availability? lowest latency?). Then read the scenario.
- **Eliminate distractors.** Usually 2 options are obviously wrong; the real choice is between 2 plausible ones differing on one requirement.
- **Keyword → service mapping** wins most questions (see cheatsheet). E.g. "decouple" → SQS, "fan-out" → SNS, "serverless ETL" → Glue, "millisecond NoSQL at scale" → DynamoDB.
- **Prefer managed / serverless** when the question says "minimize operational overhead."
- **Prefer multi-AZ** for HA, **multi-Region** only when the scenario demands DR/global low latency (it costs more — watch cost-optimized questions).
- When two answers both "work," pick the one matching the *optimized dimension* in the question.

---

## Contents

- `notes/` — topic notes (01–10; note 10 covers AI/ML services + commonly-missed extras)
- `exam-facts/cheatsheet.md` — high-yield facts, service limits, keyword map
- `exam-facts/service-comparisons.md` — decision tables
- `practice/domain1-4` — practice questions per domain (with explanations)
- `practice/mock-exam-1.md` & `mock-exam-2.md` — two full 65-question timed simulations (fresh, non-overlapping)
- `practice/exam-realistic-set-A.md`, `set-B.md` & `set-C.md` — **harder, exam-fidelity** scenario questions (long multi-constraint stems; rationale for why each distractor fails). Set C targets commonly-missed topics (placement groups, AI/ML services, IPv6, Transfer Family, Cognito). Do these once the domain drills feel easy.

> ⚠️ Practice questions here are original, written to mirror exam *style and reasoning*. They are not real exam questions. Don't rely on brain-dumps.
