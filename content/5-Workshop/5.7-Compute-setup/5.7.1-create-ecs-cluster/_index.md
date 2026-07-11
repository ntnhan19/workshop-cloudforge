---
title : "Create ECS Cluster"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.7.1. </b> "
---

Before proceeding with the detailed configuration for processing tasks (Task Definitions), we need to prepare the packaged source code storage space and establish a logical resource management group. This segment will focus on configuring the **Amazon ECR** repository and initializing the **Amazon ECS Cluster**.

Because the project adopts **AWS Fargate** technology, this Cluster will initially operate as an empty logical entity. The actual Compute resources will only be dynamically provisioned by AWS at the exact moment the application is triggered.

#### 1. Initialize Docker Image Repository (Amazon ECR)
We will initialize two separate repositories to completely isolate the source code of the Backend API communication tier and the AI Worker heavy processing tier.

1. Access the **Amazon ECR** service on the AWS Console → Select **Private registry** → **Repositories** → Click **Create repository**.
2. Proceed to configure the parameters on the initialization screen:
   - **Repository name:** Enter the name `cloudforge-backend`.
   - **Image tag mutability:** Select **Immutable** *(Best Practice to prevent overwriting old builds with the same tag, ensuring the consistency of deployment history).*
   - **Encryption settings:** Keep the default **AES-256**.
   - **Image scanning settings:** This option is currently marked as *deprecated* by AWS in favor of centralized vulnerability scanning with Amazon Inspector. Skip this configuration.

![ECR Config](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecr_config.png)
3. Scroll to the bottom of the page and click **Create repository**.
4. Repeat the entire cycle above to create a second repository named `cloudforge-ai-worker`.

Once complete, record the **URI** paths of both repositories (e.g., `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend`) to serve the packaging and source code pushing process to the Cloud in subsequent lessons.

![ECR Repositories Created](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecr_repositories_created.png)

#### 2. Initialize Amazon ECS Cluster
1. Access the **Amazon ECS** service on the AWS Console → Select **Clusters** in the left navigation bar → Click **Create cluster**.
2. **Cluster configuration:** Enter the centralized identifier `cloudforge-compute-cluster` into the Cluster name field.
3. **Infrastructure:** Ensure the **AWS Fargate (serverless)** option is selected by default. The system completely avoids using EC2 instance entities to eliminate operating system operational tasks.
4. **Monitoring (Advanced option):** Select **Use Container Insights** to enable automatically pushing in-depth hardware performance metrics (vCPU, Memory Utilization) to the Amazon CloudWatch monitoring system.

![ECS Cluster Config](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecs_cluster_config.png)
5. Click **Create** and wait a few seconds for the system to approve the initialization cycle.

![ECS Cluster Created](/images/5-Workshop/5.7-Compute-setup/5.7.1-create-ecs-cluster/ecs_cluster_created.png)

{{% notice tip %}}
**Fargate Cost Optimization:** Because the Amazon ECS Cluster running in Fargate mode merely acts as a logical management group, you **incur absolutely no costs** when maintaining an empty Cluster. Compute infrastructure costs will only begin to be calculated based on the actual vCPU and RAM capacity consumed per second when tasks officially transition to the active (`RUNNING`) state.
{{% /notice %}}

***

**Next Step:** The management infrastructure and Docker image repository are ready. Let's move on to lesson **5.7.2: Deploy ECS Backend** to explore the method of setting up the Application Load Balancer (ALB) connected to the application API source code.
