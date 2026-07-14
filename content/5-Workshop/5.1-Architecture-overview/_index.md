---
title : "Architecture Overview"
date : 2026-07-09
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Smart Media Analytics Architecture (Team CloudForge)

**Smart Media Analytics** is a comprehensive video analysis solution combining the power of Generative AI with a Serverless and Event-Driven architecture on AWS. The system is designed for high scalability, capable of processing large video files, extracting intelligent metadata, and enabling Semantic Search capabilities.

Below is the high-level architecture diagram of the entire system:

![Platform Architecture Overview](/images/5-Workshop/5.1-Architecture-overview/diagram1.png)

#### Key Workflows

1. **Frontend & Authentication:**
   - The Web Application is hosted on **AWS Amplify**, with **Route 53** handling DNS routing.
   - User authentication is managed by **Amazon Cognito**, issuing secure JWT tokens.
   - Client API requests are verified by **API Gateway** and routed through an **Application Load Balancer (ALB)** to the Backend API service running on **Amazon ECS (AWS Fargate)**.

2. **Ingestion & Orchestration:**
   - Users securely upload media files directly to **Amazon S3**.
   - An *Event Notification* is triggered and sent to an **Amazon SQS** ingest queue.
   - The event is forwarded to **Amazon EventBridge**, which in turn triggers **AWS Step Functions**.
   - **Step Functions** acts as the central orchestrator, managing the end-to-end video processing workflow.

3. **AI Processing Pipeline:**
   - Heavy computational tasks are executed on the **ECS AI Worker (AWS Fargate)** within Private Subnets (with Auto Scaling across Availability Zones A & B).
   - The ECS Worker securely pulls videos and saves keyframes to S3 via an **S3 Gateway Endpoint**, ensuring traffic remains on the AWS global network and optimizing data transfer costs.
   - The processing leverages AWS managed AI/ML services:
     - **Amazon Transcribe**: For Speech-to-Text conversion.
     - **Amazon Bedrock (Nova Lite & Titan Embeddings)**: For multimodal analysis and generating vector embeddings.
   - Job progress is published by the Worker to **ElastiCache for Redis** in the private subnet. The **ECS Backend API** subscribes to these events and pushes real-time progress updates directly to the client via a **WebSocket connection (WSS)** managed by the ALB.

4. **Data Persistence & Semantic Search:**
   - Upon completion, the ECS AI Worker securely writes metadata and vector embeddings directly into **RDS PostgreSQL** (configured with Multi-AZ Primary/Replica for high availability).
   - During a search request, the **ECS Backend API** calls Bedrock to generate a Query Vector and accesses the RDS database directly to execute the Vector Search.

5. **CI/CD & Observability:**
   - **GitHub Actions** automates the build pipeline, pushing Docker images to **Amazon ECR**.
   - System health, traces, and performance are monitored using **CloudWatch** (Logs, Metrics, Alarms) and **AWS X-Ray** (Distributed Tracing).

---
**Next Step:** Proceed to **[Prerequisites & IAM](../5.2-prerequisites/)** to configure IAM Roles and your local environment.
