---
title : "Compute Setup (ECS)"
date : 2026-07-10
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Compute Setup Overview

With the internal network, databases, security, and event queues in place, it is time to bring the "heart" of the system to the Cloud: the Backend and AI Worker application source codes.

Instead of managing traditional EC2 virtual machines, we will package the applications using Docker and run them on **Amazon ECS (Elastic Container Service)** combined with the serverless compute engine **AWS Fargate**.

Benefits of Fargate:
- **No server management:** AWS handles underlying CPU/RAM provisioning.
- **Absolute security:** Containers will run entirely within the **Private Subnets** established in Chapter 5.3.
- **Auto-scaling:** Easily scale the number of Containers based on SQS queue depth or CPU load.

In this chapter, we will set up:
1. **Amazon ECR:** Push the Backend and AI Worker Docker Images to the cloud registry.
2. **ECS Cluster & Task Definitions:** Define hardware configurations and inject environment variables (`.env`) from Secrets Manager.
3. **Application Load Balancer (ALB):** Create a gateway from the Internet to the Backend API (wrapped by WAF).

*(Detailed setup steps will be updated in the sub-articles of this section).*
