---
title : "Prerequisites & IAM"
date : 2026-07-08
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. AWS Account & IAM Permissions

To deploy the **Smart Media Analytics** project, you need an active AWS account. For this workshop, it is recommended to use an IAM User or Role with `AdministratorAccess` to provision the necessary services (VPC, ECS, S3, RDS, Bedrock, etc.). 

If you are using a restricted environment, ensure you have permissions for the following services:
- **Networking & API:** VPC, ALB (Application Load Balancer), API Gateway, Route 53.
- **Compute & Orchestration:** ECS Fargate, Step Functions, EventBridge.
- **Storage & Data:** S3, RDS PostgreSQL, ElastiCache for Redis, SQS.
- **AI/ML Services:** Amazon Bedrock, Amazon Transcribe.
- **Frontend, Security & Ops:** Amplify, Cognito, Secrets Manager, ECR, CloudWatch, X-Ray.

*Insert a screenshot of your IAM User/Role permissions here:*
![IAM Permissions](../../images/5-Workshop/5.2-Prerequisites/iam_permissions.png)

#### 2. Local Environment Setup

Before starting the deployment, ensure your local machine has the following tools installed:
- **Git**: To clone the project repository.
- **Docker**: To build the frontend and backend images locally before pushing to ECR.
- **AWS CLI**: Configured with your credentials to run deployment commands.

Verify your AWS CLI configuration:
```bash
aws configure
aws sts get-caller-identity
```

*Insert a screenshot of your terminal showing a successful AWS CLI login here:*
![AWS CLI Config](../../images/5-Workshop/5.2-Prerequisites/aws_cli_config.png)

#### 3. Requesting Amazon Bedrock Model Access

Because the project utilizes **Amazon Bedrock (Nova Lite & Titan Embeddings)**, you must explicitly request model access in the AWS Console (these models are not enabled by default).

1. Go to the **Amazon Bedrock** console.
2. Navigate to **Model access** on the left menu.
3. Click **Manage model access** and select **Nova Lite** and **Titan Embeddings**.
4. Click **Save changes** and wait for the access status to become `Access granted`.

*Insert a screenshot of the Amazon Bedrock Model Access page showing the models are enabled here:*
![Bedrock Model Access](../../images/5-Workshop/5.2-Prerequisites/bedrock_access.png)