---
title : "Prerequisites & IAM"
date : 2026-07-09
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

To comply with the principle of **Least Privilege** and ensure secure communication between Microservices, we will set up specific IAM Groups and precise JSON-based Roles for the ECS Fargate containers and Step Functions orchestrator.

#### 0. Create IAM Groups & Users (Workshop Environment)

Before creating service roles, set up IAM groups to manage console access for your team.

1. Navigate to **IAM > User groups > Create group**.
2. Create an admin group:
   - **User group name:** `CloudForge-Admins`
   - **Permissions policy:** `AdministratorAccess`
   
   ![Create Admin Group](/images/5-Workshop/5.2-Prerequisites/iam_group_admin.png)

3. Create a developer group (for team members to deploy without full account access):
   - **User group name:** `CloudForge-Developers`
   - **Permissions policy:** `PowerUserAccess`
   
   ![Create Dev Group](/images/5-Workshop/5.2-Prerequisites/iam_group_dev.png)

4. Go to **IAM > Users > Create user**, create users for your team members (e.g., `cf-vy`, `cf-luan`), and assign them to the appropriate groups.

   ![Assign Users](/images/5-Workshop/5.2-Prerequisites/iam_assign_users.png)

#### 1. Create IAM Roles for Compute & Workflow

In AWS ECS Fargate, a container requires two types of roles: an **Execution Role** (for the Fargate agent to pull images and push logs) and a **Task Role** (for your actual code to access other AWS services).

**Role 1: ECS Task Execution Role (System level)**
1. Go to **IAM > Roles > Create role**. Select **AWS service > Elastic Container Service > Elastic Container Service Task**.
2. Attach the managed policy: `AmazonECSTaskExecutionRolePolicy`.
3. Name it: `ecsTaskExecutionRole` and click Create.

   ![ECS Execution Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_execution_role.png)

**Role 2: ECS-Backend-TaskRole (API & Vector Search)**
This role allows your FastAPI/Node.js backend to convert text to vectors via Bedrock and read/write to S3.
1. Go to **IAM > Policies > Create policy (JSON tab)**.
2. Paste the following JSON:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3Access",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudforge-media-storage/*"
        },
        {
            "Sid": "AllowBedrockVectorGeneration",
            "Effect": "Allow",
            "Action": "bedrock:InvokeModel",
            "Resource": "*"
        }
    ]
}
```
3. Name the policy `cloudforge-backend-policy` and create it.
4. Go to **Roles > Create role** (Service: Elastic Container Service Task). Attach the `cloudforge-backend-policy`.
5. Name the role: `ECS-Backend-TaskRole`.

![ECS Backend Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_backend_role.png)

**Role 3: ECS-Worker-TaskRole (Heavy AI Processing)**
This role allows the asynchronous background worker to extract scenes, call Bedrock for multimodal analysis, and call Transcribe for Speech-to-Text.

1. Go to **IAM > Policies > Create policy (JSON tab)**.
2. Paste the following JSON:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowFullS3MediaAccess",
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
            "Resource": ["arn:aws:s3:::cloudforge-media-storage", "arn:aws:s3:::cloudforge-media-storage/*"]
        },
        {
            "Sid": "AllowAIAndMLServices",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "transcribe:StartTranscriptionJob",
                "transcribe:GetTranscriptionJob"
            ],
            "Resource": "*"
        }
    ]
}
```
3. Name the policy `cloudforge-ai-worker-policy`. Create it, then attach it to a new role named `ECS-Worker-TaskRole`.

![ECS Worker Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_worker_role.png)

**Role 4: StepFunctions-Orchestrator**
This role allows the Step Functions State Machine to trigger the ECS AI Worker and consume SQS messages.

1. Go to **IAM > Policies > Create policy (JSON tab)**.
2. Paste the following JSON:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowRunECSTask",
            "Effect": "Allow",
            "Action": "ecs:RunTask",
            "Resource": "arn:aws:ecs:*:*:task-definition/cloudforge-ai-worker:*"
        },
        {
            "Sid": "AllowPassRoleToECS",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": [
                "arn:aws:iam::*:role/ecsTaskExecutionRole",
                "arn:aws:iam::*:role/ECS-Worker-TaskRole"
            ]
        },
        {
            "Sid": "AllowSQSReceive",
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
            ],
            "Resource": "arn:aws:sqs:*:*:cloudforge-ingest-queue"
        }
    ]
}
```
3. Name it `cloudforge-orchestrator-policy`.
4. Create a role (Trusted Entity: Step Functions) named `StepFunctions-Orchestrator` and attach this policy.

![Final IAM Roles](/images/5-Workshop/5.2-Prerequisites/iam_final_roles.png)

#### 2. Local Environment Setup
Before starting the deployment, ensure your local machine has the following tools installed:

- **Git:** To clone the project repository.
- **Docker:** To build the frontend and backend images locally before pushing to ECR.
- **AWS CLI:** Configured with your credentials to run deployment commands.

Verify your AWS CLI configuration:
```bash
aws configure
aws sts get-caller-identity
```
![AWS CLI Config](/images/5-Workshop/5.2-Prerequisites/aws_cli_config.png)

#### 3. Amazon Bedrock Model Access (New AWS Update)

Previously, Amazon Bedrock required manual activation for specific foundation models. However, as per the latest AWS update, the Model Access page has been retired. 

Serverless foundation models (including **Nova Lite** and **Titan Embeddings** used in our project) are now **automatically enabled** across all AWS commercial regions when first invoked in our account. Therefore, no manual intervention is required in the AWS Console for this step.

![Bedrock Model Access Auto](/images/5-Workshop/5.2-Prerequisites/bedrock_access_retired.png)
