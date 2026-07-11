---
title : "Compute Setup (ECS)"
date : 2026-07-10
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Compute Setup Overview

After completing the secure internal network layer, persistent databases, and event-driven message orchestration pipeline, the Smart Media Analytics system needs a robust computing platform to operate the "heart" of the entire project: the Backend API applications and AI Worker tasks executing heavy processing.

Instead of using the traditional EC2 virtual machine management architecture, which requires significant operational effort (OS patching, manual scaling group management), the project adopts a solution of packaging applications with Docker Containers and deploying them on the **Amazon ECS (Elastic Container Service)** platform combined with the **AWS Fargate** serverless compute engine.

#### Core Benefits of the ECS Fargate Architecture:
- **Infrastructure Abstraction (Serverless Compute):** AWS automatically takes responsibility for provisioning, managing, and optimizing the underlying hardware resources (CPU/RAM). Engineers only need to focus on optimizing the application source code.
- **In-depth Security (Isolate by Design):** Compute tasks (Tasks) are configured to launch entirely within the **Private Subnets** partition established in Chapter 5.3, completely isolated from the public Internet.
- **Elastic Scalability:** The system can automatically scale up or down the number of processing tasks (Containers) based on the number of messages accumulated in the SQS queue or hardware load thresholds, helping to maximize operational cost optimization.

#### Compute Tier Deployment Roadmap:
1. **Amazon ECR (Elastic Container Registry):** Initialize a secure cloud repository to manage and store the system's Docker Image distributions.
2. **ECS Backend & Application Load Balancer:** Establish an application load balancer as an intermediate gateway and deploy the public-facing API application on the AWS Fargate platform.
3. **Background AI Worker:** Deploy a completely isolated background service in the Private Subnet to handle heavy analysis tasks from the SQS queue.
4. **Auto Scaling:** Configure intelligent resource scale-out and scale-in policies based on CPU metrics to optimize performance and costs.

Completing this chapter will provide the system with flexible computing capacity, absolute security, and automatic optimization according to the enterprise's actual load scale.

### Hands-on Content

- [Create ECS Cluster](5.7.1-create-ecs-cluster/)
- [Deploy ECS Backend](5.7.2-deploy-ecs-backend/)
- [Deploy ECS AI Worker](5.7.3-deploy-ecs-ai-worker/)
- [Configure Auto Scaling](5.7.4-configure-auto-scaling/)
