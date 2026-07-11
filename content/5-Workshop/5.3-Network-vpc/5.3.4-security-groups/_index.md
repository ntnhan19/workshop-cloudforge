---
title : "Configure Security Groups"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

Security Groups act as virtual firewalls at the resource level. To ensure the highest security standards, we will implement a **Zero-Trust** model: system components will only accept traffic from specifically authorized sources, absolutely preventing wide-open internet access.

In this section, we will create 3 core Security Groups within the `cloudforge-vpc`:

#### 1. ALB-SG (Application Load Balancer Firewall)
**Purpose:** This is the only gateway open to the Internet to receive user traffic.
1. Navigate to the **EC2 Console** or **VPC Console** and select **Security Groups** on the left menu.
2. Click the orange **Create security group** button.
3. **Security group name:** `cloudforge-alb-sg`
4. **Description:** `Allow HTTP/HTTPS from Internet`
5. **VPC:** Select `cloudforge-vpc`.
6. Under **Inbound rules**, configure the following:
   - Type: `HTTP` | Port: `80` | Source: `0.0.0.0/0` (Anywhere IPv4)
   - Type: `HTTPS` | Port: `443` | Source: `0.0.0.0/0` (Anywhere IPv4)
7. **Outbound Rules:** Keep the Default (Allow All Traffic).
8. Click **Create security group**.

![ALB Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/alb_sg.png)

#### 2. ECS-App-SG (Backend & AI Worker Firewall)
**Purpose:** Only accept requests filtered through the Load Balancer and allow internal containers to communicate with each other.
1. Create a new security group named `cloudforge-ecs-app-sg` inside `cloudforge-vpc`.
2. **Inbound rules:**
   - Type: `Custom TCP` | Port: `8000` (Backend API Port) | Source: Select the ID of `cloudforge-alb-sg`. *(Protects the API, only accepting traffic from the ALB)*.
   - Type: `All TCP` | Port: `0-65535` | Source: Select the ID of `cloudforge-ecs-app-sg` itself. *(Allows cross-communication between internal containers)*.

![ECS App Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/ecs_app_sg.png)

#### 3. DB-Redis-SG (Database & Cache Firewall)
**Purpose:** Tightly lock down the Data tier, allowing only processing servers (ECS containers) to access and read/write data.
1. Create a new security group named `cloudforge-db-redis-sg` inside `cloudforge-vpc`.
2. **Inbound rules:**
   - Type: `PostgreSQL` | Port: `5432` | Source: Select the ID of `cloudforge-ecs-app-sg`.
   - Type: `Custom TCP` | Port: `6379` (Redis Port) | Source: Select the ID of `cloudforge-ecs-app-sg`.

![DB Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/sg_db_redis.png)

***

**Next Step:** With our solid network foundation and Zero-Trust security layers completed, we will proceed to section **5.4: Database Setup** to provision the database clusters.
