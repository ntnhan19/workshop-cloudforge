---
title: "Workshop"
date: 2024-01-01
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
3. [Network & S3 Gateway Setup](5.3-Network-vpc/)
4. [Database Deployment (RDS & Redis)](5.4-Database-setup/)
5. [Deploying Compute (ECS & App Runner)](5.5-Compute-setup/)
6. [Clean up resources](5.6-Cleanup/)