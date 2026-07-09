---
title : "Prerequisites & IAM"
date : 2026-07-09
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. AWS Account & IAM Roles

To allow the system to operate automatically and securely, we need to create IAM Roles so that AWS services can communicate with each other. For this workshop, it is recommended your AWS User has `AdministratorAccess` to provision the infrastructure, and then you will create the following specific roles:

- **ECS-Backend-TaskRole:** Grants the Backend API cluster permission to invoke Bedrock (for vectorizing search queries), read/write to S3, and access internal networks.
- **ECS-Worker-TaskRole:** Grants the heavy AI processing task permission to invoke Bedrock, Transcribe, read/write to S3, and publish job progress to Redis.
- **StepFunctions-Orchestrator:** Grants Step Functions permission to trigger the ECS Worker (RunTask API) and read messages from SQS.

*📸 Screenshot: IAM Roles list in the AWS Console when searching for the project name.*
![IAM Roles](../../images/5-Workshop/5.2-Prerequisites/iam_permissions.png)

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