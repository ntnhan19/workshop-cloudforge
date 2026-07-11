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
3. Navigate to the exact directory containing the Backend source code (where the Dockerfile is located):
   ```bash
   cd backend
   ```
4. Proceed to compile (Build) the Docker Image and directly attach an identifier tag (Tag) mapping it to the remote repository on AWS:
   ```bash
   docker build -t 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest .
   ```
5. Push the packaged product to the cloud:
   ```bash
   docker push 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest
   ```

*Illustration: The Backend application's Image has been successfully pushed to Amazon ECR with the latest tag.*
![ECR Image Pushed](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecr_image_pushed.png)

#### 2. Initialize Application Load Balancer (ALB)
Since the Backend application's Containers will be completely isolated within the Private Subnet zone for security, we need to establish an Application Load Balancer (ALB) situated in the Public Subnets zone to act as an intermediary shield receiving and distributing traffic from the Internet.

**Step 1: Initialize Target Group**
1. Access the **EC2** service → **Target Groups** → **Create target group**.
2. **Choose a target type:** Select **IP addresses** (This is mandatory for the AWS Fargate architecture because each Task will possess a separate internal IP address belonging to an Elastic Network Interface - ENI).
3. **Target group name:** Enter `cloudforge-backend-tg`.
4. **Protocol & Port:** Select HTTP and configure port `8080`.
5. **VPC:** Select the correct `cloudforge-vpc`.
6. For **Health checks**, keep the default configuration and click **Next** → **Create target group** (Skip the manual IP registration step as the ECS Service will automatically manage this flow).

**Step 2: Configure Load Balancer**
1. On the left EC2 menu, select **Load Balancers** → Click **Create load balancer** → Select **Application Load Balancer**.
2. **Load balancer name:** Enter `cloudforge-backend-alb`.
3. **Scheme:** Check **Internet-facing** to allow receiving public traffic.
4. **Network mapping:**
   - Select the project's VPC (`cloudforge-vpc`).
   - In the Mappings section, check both Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`), while mapping them accurately to the corresponding **Public Subnets** of each Zone.
5. **Security groups:** Select a pre-configured Security Group that allows opening port 80 (HTTP) and port 443 (HTTPS) from any source (`0.0.0.0/0`).
6. **Listeners and routing:** For HTTP Protocol Port 80, set the Default action to forward (Forward to) the `cloudforge-backend-tg` Target Group just created in Step 1.
7. Click **Create load balancer**.

*Illustration: Application Load Balancer (ALB) successfully initialized in Active state.*
![ALB Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_created.png)

#### 3. Establish Task Definition
The Task Definition acts as an architectural blueprint detailing the hardware limits, the Docker image used, and the security parameters of the application.

1. Access the **Amazon ECS** service → **Task definitions** → Click **Create new task definition**.
2. **Task definition configuration:** Define the name `cloudforge-backend-task`.
3. **Infrastructure requirements:**
   - **Launch type:** Select **AWS Fargate**.
   - **Task size:** Specify cost-optimized resources including `0.5 vCPU` and `1 GB` Memory.
   - **Task execution role:** Choose to create a new one or designate `ecsTaskExecutionRole` (Grants the task permissions to pull images from ECR and push logs to CloudWatch).
4. **Container configuration:**
   - **Container name:** `backend-container`.
   - **Image URI:** Paste the exact ECR link string: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest`.
   - **Port mappings:** Configure Container Port as `8080`, Protocol `TCP`, App protocol select `HTTP`.
5. **Environment variables (Security Best Practice):**
   - Instead of hardcoding database connection information, in the environment variables section, map the `DB_HOST`, `DB_PASSWORD` variables to retrieve secure values directly from the AWS Secrets Manager service using the `ValueFrom` mechanism.
6. Click **Create**.

*Illustration: Task Definition initialization complete with full hardware configuration parameters and Image URI.*
![Task Definition Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_definition_created.png)

#### 4. Deploy ECS Service Operations
The Service plays a coordinating role, ensuring the continuous maintenance of a stable number of active Containers and automatically connecting them to the ALB load balancer.

1. At the **Amazon ECS** dashboard, select the `cloudforge-compute-cluster` Cluster.
2. Switch to the **Services** tab → Click **Create**.
3. **Environment:** Select the Launch type feature and designate **FARGATE**.
4. **Deployment configuration:**
   - **Application type:** Select **Service**.
   - **Task definition:** Select the `cloudforge-backend-task` Family with the newest version (`latest`).
   - **Service name:** Enter the identifier name `cloudforge-backend-service`.
   - **Desired tasks:** Enter the value `2` (Configure to run 2 containers concurrently in parallel across 2 different AZs to ensure High Availability for the API system).
5. **Networking:**
   - **VPC:** Select `cloudforge-vpc`.
   - **Subnets:** Accurately designate the 2 isolated **Private Subnets** network zones. (Absolutely do not select a Public Subnet to comply with Zero-Trust safety standards).
   - **Security group:** Select an SG that allows receiving Inbound traffic solely from the ALB's Security Group running through port `8080`.
6. **Load balancing:**
   - **Load balancer type:** Select **Application Load Balancer**.
   - **Load balancer:** Select `cloudforge-backend-alb`.
   - **Container to load balance:** The system automatically recognizes `backend-container:8080:8080`. Click **Add to load balancer**.
   - **Target group:** Select the pre-established `cloudforge-backend-tg`.
7. Scroll to the bottom and click **Create**.

*Illustration: Backend API tier successfully deployed with Tasks reaching the RUNNING state.*
![ECS Service Success](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_success.png)

{{% notice tip %}}
**Architectural Note (Isolate by Design):** With this decoupled design model, the Backend API system possesses two strict layers of protection. All potential attack traffic from the Internet will be blocked or filtered at the ALB load balancer located in the Public Subnet zone (Can be additionally wrapped with AWS WAF). The clean data stream is then silently routed through the internal network to enter the Containers nestled deep within the Private Subnet network zone, completely eliminating the risk of direct vulnerability exploitation from the external environment.
{{% /notice %}}

***

**Next Step:** The Backend API system has been fully deployed and can receive orchestration requests via the ALB endpoint. We will continue setting up the project's analytical "brain" component in lesson **5.7.3: Deploy ECS AI Worker** to configure the dedicated server cluster processing background tasks from the SQS queue.
