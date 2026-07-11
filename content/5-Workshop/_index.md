---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying Smart Media Analytics Architecture on AWS

#### Overview

**Smart Media Analytics** (developed by team **CloudForge**) is an end-to-end intelligent media analysis pipeline. Moving this robust architecture to the cloud requires a well-designed, secure, and highly available infrastructure leveraging modern Serverless and Containerized AWS services.

In this workshop, you will learn how to deploy the entire Smart Media Analytics stack on AWS. We will build a secure VPC infrastructure, deploy our AI processing pipeline using Amazon ECS (Fargate), set up our vector databases with RDS PostgreSQL, and host our frontend using AWS App Runner.

We will focus on implementing secure architectural patterns such as utilizing **Private Subnets** for compute layers and connecting to Amazon S3 securely without traversing the public internet using **S3 Gateway Endpoints**.

#### Content

1. [Architecture Overview](5.1-Architecture-overview/)
2. [Prerequisites & Environment](5.2-Prerequisites/)
3. [Network Foundation (VPC)](5.3-Network-vpc/)
4. [Storage & Database (S3, RDS, Redis)](5.4-Database-setup/)
5. [Security (Cognito, Secrets Manager, WAF)](5.5-Security-setup/)
6. [Ingestion Workflow (SQS, EventBridge, Step Functions)](5.6-Ingestion-workflow/)
7. [Compute Setup (ECS Backend & AI Worker)](5.7-Compute-setup/)
8. [AI/ML Integration (Bedrock & Transcribe)](5.8-AI-ML-integration/)
9. [API & Real-time (API Gateway, ALB, WebSocket)](5.9-API-and-realtime/)
10. [Semantic Search](5.10-Semantic-search/)
11. [Frontend Deployment (Amplify, Route53)](5.11-Frontend-deployment/)
12. [CI/CD (GitHub Actions, ECR)](5.12-CICD/)
13. [Observability (CloudWatch, X-Ray)](5.13-Observability/)
14. [Clean up resources](5.14-Cleanup/)
