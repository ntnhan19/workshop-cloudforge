---
title : "Deploy ECS AI Worker"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

In contrast to the Backend API system, which is responsible for receiving and responding to direct request streams from end-users, the **AI Worker** system acts as the background processor of the entire project. This system operates completely isolated, without opening any ports to the Internet. Instead, it continuously polls for event messages from the Amazon SQS queue, thereby triggering Media analysis cycles and interacting with artificial intelligence models.

In this segment, we will package the AI Worker source code, push it to the Amazon ECR Private repository, and deploy it as an isolated **Background Service** on the AWS Fargate serverless infrastructure.

#### 1. Package and push source code to Amazon ECR
This packaging and compilation process is executed directly at the AI Worker's dedicated source code directory and mapped to the created `cloudforge-ai-worker` repository.

1. Open the integrated Terminal in the IDE at the root directory of the project.
2. Log in and authenticate to Amazon ECR (If the previous AWS CLI session has expired):
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
3. Navigate to the directory containing the AI Worker's source code:
   ```bash
   cd ai-worker
   ```
4. Compile the Docker Image based on the dedicated Dockerfile configuration and assign the specified tag:
   ```bash
   docker build -t 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest .
   ```
5. Push the completed Image to the ECR cloud:
   ```bash
   docker push 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest
   ```

![ECR Worker Image Pushed](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/ecr_worker_image_pushed.png)

#### 2. Establish Task Definition
Due to the specific requirement of executing computational tasks related to Media data stream processing and running machine learning model algorithms, the AI Worker's virtual hardware configuration will be set higher than the standard API tier.

1. Access the **Amazon ECS** service → **Task definitions** → Click **Create new task definition**.
2. **Task definition configuration:** Set the identifier name `cloudforge-ai-worker-task`.
3. **Infrastructure requirements:**
   - **Launch type:** Select **AWS Fargate**.
   - **Task size:** Allocate optimal processing resources including `1 vCPU` for the processor and `2 GB` for RAM capacity.
   - **Task execution role:** Specify the `ecsTaskExecutionRole` role, which possesses attached policies for Secrets Manager access to load system environment variables.
   - **Task role (Crucial):** Select EXACTLY the IAM Role **`ECS-Worker-TaskRole`** which has been pre-granted core security permissions including: Interacting with Amazon SQS queues, processing files on Amazon S3, updating the Database, and crucially, calling AI services like Amazon Bedrock and Transcribe.

![Worker Task Role](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_task_role.png)
4. **Container configuration:**
   - **Container name:** Define the name `worker-container`.
   - **Image URI:** Paste the exact link from ECR: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest`.
   - **Port mappings:** LEAVE COMPLETELY BLANK. Because the Worker operates specifically on an Outbound model (Actively calling out of the system to fetch jobs), not opening an Inbound port helps completely eliminate the network Attack Surface.
5. **Environment variables:**
   - Declare the mandatory environment variables for the AI Worker to operate:
     - `AWS_DEFAULT_REGION`: `ap-southeast-1`
     - `AWS_S3_BUCKET`: `cloudforge-media-upload-ntnhan19`
     - `DATABASE_URL`: `postgresql+psycopg://postgres:<your-password>@<your-rds-endpoint>:5432/cloudforge_db`
     - `PYTHONPATH`: `/app`
     - `STORAGE_BACKEND`: `s3`

![Worker Environment](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_environment.png)
6. Click **Create** to save the V1 configuration.

![Worker Task Definition](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_task_definition.png)

#### 3. AI Worker Execution (Event-Driven)

Unlike the Backend, the **AI Worker** in this architecture is designed following the **Event-Driven Architecture** model. We will **NOT CREATE AN ECS SERVICE** for the AI Worker.

Reason: The AI Worker is a script-based task that will automatically shut down immediately after processing a video (run once and exit). If you intentionally create an ECS Service, the system will try to keep it running continuously, leading to constant restart loops (Task exited with code 1 or 0) and triggering the AWS **Service deployment circuit breaker**.

Instead of running continuously in the background, the AI Worker will be invoked automatically as independent tasks via **AWS Step Functions** (`ecs:RunTask`) whenever a new video is uploaded to S3 (as configured in step 5.6).

You have successfully created the **Task Definition** for the AI Worker. The infrastructure for the Worker is now fully ready!

***

**Next Step:** The entire core processing infrastructure system including Network connectivity, Storage database, Asynchronous event orchestration, and Compute cluster (ECS Backend & Worker) of the Smart Media Analytics project has been completely built and linked. We will move on to the next chapter to execute [**Auto Scaling Configuration**](../5.7.4-configure-auto-scaling/) and ensure high availability.
