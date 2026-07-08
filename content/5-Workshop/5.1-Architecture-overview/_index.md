---
title : "Architecture Overview"
date : 2026-07-08
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Smart Media Analytics Architecture (Team CloudForge)

**Smart Media Analytics** is a comprehensive video analysis solution combining the power of AI with Serverless and Microservices architecture on AWS. The system is designed to process large video files, extract intelligent metadata (scene detection, speech-to-text, object recognition), and enable users to perform Semantic Search.

Below is the high-level architecture diagram of the entire system deployed on the Cloud:

![Platform Architecture Overview](../../images/5-Workshop/5.1-Architecture-overview/diagram.png)

#### Key Workflows

1. **Ingestion Workflow (Upload & Pre-processing):**
   - Users upload videos through the Web UI (React).
   - The original video is securely stored in **Amazon S3**.
   - A Job record is created in **RDS PostgreSQL**, and notifications are pushed via **Redis Pub/Sub** to provide real-time status updates to the Frontend via WebSocket.

2. **AI Pipeline Workflow (Analysis & Extraction):**
   - The core compute engine runs on **Amazon ECS (AWS Fargate)** inside a Private Subnet.
   - ECS securely pulls the video from S3 via an S3 Gateway Endpoint to optimize internal bandwidth.
   - **FFmpeg & PySceneDetect** segment the video into scenes and extract keyframes.
   - **Whisper AI** processes the audio track to generate transcripts and captions.
   - Keyframes are sent to a **Vision AI model (Qwen-VL)** for visual description, which is then converted into vector embeddings for search capabilities.

3. **Serving & Semantic Search Workflow:**
   - The Backend API is built with **FastAPI** and protected by a Rate Limiter.
   - All vector embeddings and metadata are stored in **PostgreSQL (using the pgvector extension)**. When a user enters a search query (e.g., "A man riding a bicycle"), the system performs a Cosine Similarity vector search directly within the database.
   - The Backend application is hosted on **AWS App Runner**, utilizing a VPC Connector to securely query the database located inside the secure zone (Private Subnet).