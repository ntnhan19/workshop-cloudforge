---
title: "Proposal"
date: 2026-04-17
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Smart Media Analytics
## Building an AI-powered Video Analysis (Semantic Search System)

### 1. Executive Summary
Smart Media Analytics (SMA) is a local-first media management and search system designed for video editors, designers, and research teams needing rapid retrieval of media content. The system automates media ingestion, scene splitting, transcription, and AI-powered contextual description, enabling natural language search.
Initially, SMA runs entirely on local machines (via Docker) to ensure privacy and cut experimental costs. For production scaling, it offers a seamless upgrade path to AWS (S3, Bedrock, RDS).

### 2. Problem Statement
**What’s the Problem?**
Current media libraries are scattered, and meaningless file names make manual scene searching incredibly time-consuming. Relying directly on cloud AI services for the entire pipeline from day one incurs high costs and potential data privacy risks.

**The Solution**
SMA addresses this with an automated pipeline utilizing a React dashboard, FastAPI, ChromaDB, PostgreSQL, MinIO, and local AI models (Ollama, faster-whisper). Users simply type "beach sunset" to jump exactly to the desired video timestamp. When scaling to production, it seamlessly transitions to AWS services like Amazon S3, AWS Bedrock, Amazon Transcribe, and Amazon RDS PostgreSQL, managed and deployed via AWS Amplify and orchestrated by Step Functions and ECS Fargate.

**Benefits and Return on Investment**
The solution significantly shortens the time required to search for footage, read transcripts, and identify needed scenes, boosting the creative team's efficiency. Users can query in natural language and get precise timestamps instead of manually reviewing files. The initial local-first approach saves cloud AI costs during development, while the clear AWS migration path ensures long-term scalability.

### 3. Solution Architecture
The platform employs a hybrid local-to-cloud approach. In the local phase, the media ingest workflow runs in Docker Compose. When deployed to AWS, the architecture uses a serverless and managed services model. 

![SMA Architecture](/images/2-Proposal/SMA_architecture.png)

**AWS Services Used**
- **1. Networking & Routing:** Amazon Route 53, AWS Amplify, Amazon API Gateway, AWS ALB (Application Load Balancer), Amazon VPC (including Internet Gateway, NAT Gateway, and S3 Gateway Endpoint).
- **2. Compute & CI/CD:** Amazon ECS on Fargate (Divided into 2 services: Backend and AI Worker), Amazon ECR (Elastic Container Registry).
- **3. Storage & Database:** Amazon S3, Amazon RDS (PostgreSQL), Amazon ElastiCache (Redis).
- **4. AI & ML:** Amazon Bedrock (Anthropic Claude 3 & Titan Embeddings), Amazon Transcribe.
- **5. Orchestration & Events:** Amazon SQS, Amazon EventBridge, AWS Step Functions.
- **6. Security & Observability:** Amazon Cognito, AWS Secrets Manager, Amazon CloudWatch, AWS X-Ray.

**Component Design**
- **Frontend Layer:** React app managed and distributed by AWS Amplify with Route 53, authenticated via Cognito.
- **API & Routing Layer:** API Gateway or ALB routes traffic to the Backend running on Amazon ECS Fargate.
- **Processing & AI Layer:** Step Functions coordinates AI Worker (ECS Fargate) tasks for video processing, followed by Bedrock (Anthropic Claude 3 & Titan) & Transcribe for AI extraction.
- **Data Layer:** Metadata is persisted in RDS PostgreSQL, cached in ElastiCache. Secured within a VPC with a NAT Gateway and S3 Gateway Endpoint.

### 4. Technical Implementation
**Implementation Phases**
1. **Architectural Research:** Design the local architecture and map the data flow to the AWS services list (1 month pre-internship).
2. **Local-first Build:** Develop and refine the core logic locally using Docker Compose, integrating Ollama and faster-whisper (Month 1).
3. **AWS Cloud-ready Integration:** Incrementally replace local containers with AWS managed services like ECS Fargate, Bedrock, and RDS (Month 2).
4. **Develop, Test, and Deploy:** Finalize the cloud infrastructure using CI/CD, set up CloudWatch monitoring, and release to production (Month 3).

**Technical Requirements**
- **Media Processing:** Containerized processing tasks utilizing FFMPEG for scene splitting.
- **Cloud Infrastructure:** Practical knowledge of AWS Amplify, ECS Fargate, Step Functions, Bedrock, and RDS. Use AWS SDK to code interactions (e.g., S3 uploads, Bedrock invocations).

### 5. Timeline & Milestones
**Project Timeline**
- **Pre-Internship (Month 0):** 1 month for planning and architectural research.
- **Internship (Months 1-3):** 3 months.
  - **Month 1:** Build the local-first pipeline and test local AI models.
  - **Month 2:** Design and adapt the architecture for AWS, setting up core services (S3, RDS, ECS).
  - **Month 3:** Implement Step Functions orchestration, test full flow, and launch.
- **Post-Launch:** Up to 1 year for further model tuning and scaling.

### 6. Budget Estimation
Deploying this comprehensive production architecture with over 20 AWS services, the budget for a small-scale or lab environment is estimated as follows:

| AWS Service | Use Case | Estimated Cost/Month (USD) |
| --- | --- | --- |
| **Amazon RDS (PostgreSQL)** | Primary Database (Multi-AZ, `db.t4g.medium`, 20GB) | $68.00 |
| **AWS NAT Gateway** | Allows Private Subnet internet access (1 NAT, 24/7) | $32.40 |
| **Amazon ElastiCache** | Cache (Multi-AZ, `cache.t4g.micro`, 2 nodes) | $32.00 |
| **Amazon ECS (Fargate)** | Auto Scaling Task for Video Processing (~1 vCPU, 2GB RAM) | $15.00 |
| **Amazon Bedrock & Transcribe** | AI, Speech-to-Text, Embeddings (Base estimate) | $15.00 |
| **AWS ALB** | Application Load Balancer routing to ECS Backend | $16.00 |
| **Amazon CloudWatch & X-Ray** | Logs, Metrics, Alarms, Distributed Tracing | $5.00 |
| **Amazon S3** | Media & Keyframe Storage (Est. ~50GB) | $2.00 |
| **AWS Secrets Manager** | Keys & Credentials Management (5 secrets) | $2.00 |
| **Amazon API Gateway** | REST API & WSS Push (Request-based) | $1.00 |
| **AWS Amplify** | Hosting & CDN for Frontend | $1.00 |
| **Amazon SQS, EventBridge, Step Functions** | Workflow Orchestration, Event Bus, Queues | $1.00 |
| **Amazon Route 53** | DNS Management (1 Hosted Zone) | $0.50 |
| **Amazon Cognito** | User Authentication (JWT) | $0.00 (Free Tier) |
| **Estimated Total** | **Entire System** | **~$190.90** |

### 7. Risk Assessment
**Risk Matrix**
- **Large Media Volumes:** High impact, medium probability.
- **Local AI Inference Speed:** Medium impact, high probability.
- **Cloud Scaling Costs:** High impact, low to medium probability.

**Mitigation Strategies**
- **Media Volumes:** Limit batch ingests, optimize FFMPEG processing.
- **AI Inference:** Cache results aggressively, move to AWS Bedrock if local models are too slow.
- **Costs:** Use AWS budget alerts and monitor resource utilization closely.

**Contingency Plans**
- If local vector search is a bottleneck, migrate to OpenSearch Serverless earlier than planned.
- Retain basic metadata even if full media processing fails, avoiding total library loss.

### 8. Expected Outcomes
**Technical Improvements:**
- SMA enables users to ingest media, automatically generate semantic indices, search via natural language, and jump directly to the exact video timestamp.
- Replaces manual tagging and searching with an automated AI pipeline.

**Long-term Value:**
- Serves as a highly efficient Media Intelligence system for creative teams.
- The local-first architecture ensures quick development, while AWS mapping provides a clear upgrade path to production infrastructure.