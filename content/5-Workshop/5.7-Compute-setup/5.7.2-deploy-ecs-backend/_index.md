---
title : "Deploy ECS Backend"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

The Backend API tier acts as the routing core, responsible for receiving HTTP/HTTPS request streams from users, executing business logic, and interacting with the database and message queue layers. In this segment, we will package the Backend source code into a Docker Image, push it to the Amazon ECR repository, configure an Application Load Balancer (ALB), and deploy the application on the Serverless AWS Fargate platform.

#### 1. Package and push source code to Amazon ECR
To transfer the local source code to the Cloud infrastructure, we compile the application into a Docker Image and push it into the `cloudforge-backend` repository initialized in the previous lesson via the command-line tool (AWS CLI).

1. Open the Terminal at the root directory of the entire project on your local computer.
2. Log in and authenticate access to the account's Amazon ECR Private Registry:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
3. DO NOT change directories to `backend`. Stay in the root directory of the project (where you can see both `backend` and `ai_pipeline` folders).
4. Compile (Build) the Docker Image by specifying the Dockerfile path and setting the build context to the root directory:
   ```bash
   docker build -t 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest -f backend/Dockerfile .
   ```
5. Push the packaged product to the cloud:
   ```bash
   docker push 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest
   ```

![ECR Image Pushed](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecr_image_pushed.png)

#### 2. Initialize Application Load Balancer (ALB)
Since the Backend application's Containers will be completely isolated within the Private Subnet zone for security, we need to establish an Application Load Balancer (ALB) situated in the Public Subnets zone to act as an intermediary shield receiving and distributing traffic from the Internet.

**Step 1: Initialize Target Group**
1. Access the **EC2** service → **Target Groups** → **Create target group**.
2. **Choose a target type:** Select **IP addresses** (This is mandatory for the AWS Fargate architecture because each Task will possess a separate internal IP address belonging to an Elastic Network Interface - ENI).
3. **Target group name:** Enter `cloudforge-backend-tg`.
4. **Protocol & Port:** Select HTTP and configure port `8000`.
5. **VPC:** Select the correct `cloudforge-vpc`.
6. In the **Health checks** section, change the **Health check path** from `/` to `/health` (because the Backend exposes its health status there). Click **Next** → **Create target group** (Skip the manual IP registration step because the ECS Service will manage this flow automatically).

![Target Group Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/target_group_created.png)

**Step 2: Configure Load Balancer**
1. On the left EC2 menu, select **Load Balancers** → Click **Create load balancer** → Select **Application Load Balancer**.
2. **Load balancer name:** Enter `cloudforge-backend-alb`.
3. **Scheme:** Check **Internet-facing** to allow receiving public traffic.
4. **Network mapping:**
   - Select the project's VPC (`cloudforge-vpc`).
   - In the Mappings section, check both Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`), while mapping them accurately to the corresponding **Public Subnets** of each Zone.
5. **Security groups:** Select the **`cloudforge-alb-sg`** Security Group (This SG was pre-configured to allow receiving HTTP/HTTPS traffic from any Internet source).
6. **Listeners and routing:** Under HTTP Port 80, for the Default action, select Forward to the `cloudforge-backend-tg` Target Group created in Step 1.

![ALB Routing Config](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_routing_config1.png)

![ALB Routing Config](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_routing_config2.png)

7. Click **Create load balancer**.

![ALB Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_created.png)

#### 3. Establish Task Definition
The Task Definition acts as an architectural blueprint detailing the hardware limits, the Docker image used, and the security parameters of the application.

1. Access the **Amazon ECS** service → **Task definitions** → Click **Create new task definition**.
2. **Task definition configuration:** Define the name `cloudforge-backend-task`.
3. **Infrastructure requirements:**
   - **Launch type:** Select **AWS Fargate**.
   - **Task size:** Specify cost-optimized resources including `0.5 vCPU` and `1 GB` Memory.
   - **Task role:** Select the dedicated IAM Role for the Backend application (e.g., `ECS-Backend-TaskRole`). *(Crucial Note: Ensure this IAM Role has a Policy attached that grants `s3:PutObject` and `s3:DeleteObject` permissions to your bucket. Otherwise, the upload process will fail with AccessDenied)*.
   - **Task execution role:** Select to create new or specify `ecsTaskExecutionRole` (Grants the task permissions to pull images from ECR and push logs to CloudWatch).
4. **Container configuration:**
   - **Container name:** `backend-container`.
   - **Image URI:** Paste the exact ECR link string: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest`.
   - **Port mappings:** Configure Container Port as `8000`, Protocol `TCP`, App protocol select `HTTP`.
5. **Environment variables:**
   - Declare the mandatory environment variables for the Backend to operate:
     - `AWS_DEFAULT_REGION`: `ap-southeast-1`
     - `AWS_S3_BUCKET`: `cloudforge-media-upload-ntnhan19`
     - `DATABASE_URL`: `postgresql+psycopg://postgres:<your-password>@<your-rds-endpoint>:5432/cloudforge_db`
     - `REDIS_URL`: `redis://master.cloudforge-redis...` *(Or `rediss://` if you have In-Transit Encryption enabled in ElastiCache)*
     - `STORAGE_BACKEND`: `s3`

![Task Def Environment 1](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment1.png)
![Task Def Environment 2](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment2.png)
![Task Def Environment 3](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment3.png)
6. Click **Create**.

![Task Definition Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_definition_created.png)

#### 4. Deploy ECS Service Operations
The Service plays a coordinating role, ensuring the continuous maintenance of a stable number of active Containers and automatically connecting them to the ALB load balancer.

1. At the **Amazon ECS** dashboard, select the `cloudforge-compute-cluster` Cluster.
2. Switch to the **Services** tab → Click **Create**.
3. **Service details:**
   - **Task definition family:** Select `cloudforge-backend-task`.
   - **Task definition revision:** Leave blank (The system will automatically use the latest revision - LATEST).
   - **Service name:** Name it `cloudforge-backend-service`.
4. **Environment:**
   - **Compute options:** Select **Capacity provider strategy** with **FARGATE** (or just leave the Cluster default).
5. **Deployment configuration:**
   - **Desired tasks:** Enter `2` (Launch 2 containers simultaneously for redundancy and load balancing).
6. **Networking:**
   - **VPC:** Select `cloudforge-vpc`.
   - **Subnets:** You must select the 2 **Private Subnets** (Security: Backend should not be directly exposed to the Internet).
   - **Security group:** Select **Use an existing security group** and EXACTLY CHOOSE **`cloudforge-ecs-app-sg`** (This is the SG dedicated to the application, only allowing traffic from the Load Balancer on port 8000).
7. **Load balancing:**
   - Select the **Application Load Balancer** type.
   - **Application Load Balancer:** Select **Use an existing load balancer** and point to `cloudforge-backend-alb` (created in Step 2).
   - **Listener:** Select **Use an existing listener** and point to the configured `80:HTTP` Listener.
   - **Target group:** Select **Use an existing target group** and point to `cloudforge-backend-tg`.
8. Click **Create** to proceed with the deployment. This process may take 2-3 minutes for the Service status to transition to `Running`.

![ECS Service Networking](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_networking.png)

![ECS Service Success](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_success.png)

{{% notice tip %}}
**Architectural Note (Isolate by Design):** With this decoupled design model, the Backend API system possesses two strict layers of protection. All potential attack traffic from the Internet will be blocked or filtered at the ALB load balancer located in the Public Subnet zone (Can be additionally wrapped with AWS WAF). The clean data stream is then silently routed through the internal network to enter the Containers nestled deep within the Private Subnet network zone, completely eliminating the risk of direct vulnerability exploitation from the external environment.
{{% /notice %}}

***

**Next Step:** The Backend API system has been fully deployed and can receive orchestration requests via the ALB endpoint. We will continue setting up the project's analytical "brain" component in lesson [**5.7.3: Deploy ECS AI Worker**](../5.7.3-deploy-ecs-ai-worker/) to configure the dedicated server cluster processing background tasks from the SQS queue.
