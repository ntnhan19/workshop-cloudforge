---
title : "Create VPC & Subnets"
date : 2026-07-09
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

To ensure high availability and robust security for our microservices, we will design a custom Virtual Private Cloud (VPC). We will strictly separate public-facing resources (like the Application Load Balancer) from internal processing logic and databases (ECS, RDS, Redis).

Instead of manually creating each component, we will use the **VPC and more** wizard to provision a fully routed, multi-tier network spanning two Availability Zones.

#### Step-by-Step Implementation

1. Navigate to the **VPC Console** in AWS.
2. Click the orange **Create VPC** button.
3. Under **Resources to create**, select **VPC and more**.
4. Configure the network specifications as follows:
   * **Name tag auto-generation:** Enter `cloudforge`.
   * **IPv4 CIDR block:** `10.0.0.0/16` (Provides 65,536 private IP addresses).
   * **Number of Availability Zones (AZs):** Select `2` (e.g., `ap-southeast-1a` and `ap-southeast-1b` for High Availability).
   * **Number of public subnets:** Select `2` (For the Application Load Balancer and NAT Gateway).
   * **Number of private subnets:** Select `2` (For ECS Fargate Workers, RDS PostgreSQL, and ElastiCache).
   * **NAT gateways ($):** Click **In 1 AZ** (or **Zonal**, depending on the latest AWS UI update). *(Note: For production, 1 per AZ is recommended, but for this project, 1 AZ is sufficient and cost-effective).*
   * **VPC endpoints:** Select **S3 Gateway**. *(Crucial: This allows our ECS AI Workers in private subnets to pull/push videos to S3 directly via internal AWS networks, entirely bypassing NAT Gateway data transfer costs).*
   * **DNS options:** Ensure both *Enable DNS resolution* and *Enable DNS hostnames* are checked.

5. On the right side of the screen, review the **Preview** panel. It will display a beautiful, auto-generated logical map of your new network infrastructure.

   ![VPC Preview Map](/images/5-Workshop/5.3-Network-vpc/5.3.1-create-vpc-subnets/vpc_preview_map.png)

6. Click **Create VPC** at the bottom of the page.
7. Wait a few minutes for AWS to provision all the resources (VPC, Subnets, Internet Gateway, NAT Gateway, Route Tables, and S3 Endpoint). Once complete, you will see a "Success" screen.

   ![VPC Success](/images/5-Workshop/5.3-Network-vpc/5.3.1-create-vpc-subnets/vpc_success_creation.png)
