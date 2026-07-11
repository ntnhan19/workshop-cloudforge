---
title : "Integrate Internal ALB"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---

After successfully provisioning the public API Gateway port, the subsequent challenge is how to Forward requests from this gateway, piercing through the private network boundary to accurately hit the Backend's **Application Load Balancer (ALB)** residing in the Private Subnets.

To resolve this issue while maintaining stringent security, AWS provides a specialized connectivity solution called **VPC Link**. In this segment, we will establish a VPC Link and configure integration routing for the HTTP API.

#### Concept of VPC Link in HTTP API
For the HTTP API family, a VPC Link acts as a "virtual network bridge" directly connecting the API Gateway resources (which sit outside the VPC) with an endpoint inside your Private network. This mechanism yields the following advantages:
- **Absolute Security:** Traffic from the API Gateway into the ALB flows entirely through AWS's internal transmission lines, eliminating the need to expose the ALB to the public Internet.
- **Infrastructure Decoupling:** Clients only see the API Gateway's Endpoint; the entire routing structure of the ALB and ECS behind it is completely concealed.

#### Step 1: Provisioning the VPC Link
1. On the left navigation bar of the API Gateway service, select **VPC links**.
2. Click the orange **Create button** in the top right corner.
3. Configure the VPC Link parameters as follows:
   - **Choose a VPC link version:** Check **VPC link V2** (V2 is mandatory as it's the version supporting the new HTTP API protocol).
   - **Name:** Enter `cloudforge-vpc-link`.
   - **VPC:** Select the correct project VPC (`cloudforge-vpc`).
   - **Subnets:** Check 2 **Private Subnets** in two different Availability Zones (`cloudforge-subnet-private1-ap-southeast-1a` and `cloudforge-subnet-private2-ap-southeast-1b`) to ensure high availability.
   - **Security groups:** Check **`default`** (or `cloudforge-ecs-app-sg` works as well, to ensure the VPC Link has Outbound permission to push the data stream to the ALB).
4. Click **Create** and wait about 1-2 minutes for the status to change to **Available**.

![Create VPC Link](/images/5-Workshop/5.9-API-and-realtime/5.9.2-create-alb/create_vpc_link.png)

#### Step 2: Configure Routing (Routes & Integration) for the API
Once the VPC Link bridge is in place, we need to configure the API to know when to utilize this bridge to push data to the Backend.

1. Return to the details page of the `CloudForge-Media-API` API. Look at the left menu, select **Routes**.
2. Click **Create** to form a new route. In the **Path** box, enter the catch-all path `/{proxy+}` and select the method as **ANY** (This helps the API catch all requests like `/api/v1/auth`, `/api/v1/videos`... and forward them intact to the Backend). Click **Create**.
3. After the Route is created, click on the newly generated `ANY /{proxy+}` line. Looking at the right pane, click the **Attach integration** button.
4. Click **Create integration** and fill in the ALB connection details:
   - **Integration type:** Select **Private resource**.
   - **Integration method:** Select **ANY**.
   - **Target service:** Select **VPC link**.
   - **VPC link:** Select the exact `cloudforge-vpc-link` created in Step 1.
   - **Target Type:** Select **Application Load Balancer**.
   - **Load balancer:** Select the correct ALB of the backend service (`cloudforge-backend-alb`).
   - **Listener:** Select the listener port `80` (HTTP) or `443` (HTTPS) of the ALB depending on the Backend configuration.
5. Scroll to the very bottom and click **Create**.

![Attach ALB Integration](/images/5-Workshop/5.9-API-and-realtime/5.9.2-create-alb/attach_alb_integration.png)

{{% notice tip %}}
**Operational Core:** From this moment on, the standard communication flow (REST API HTTP) is unobstructed. Any request sent to `https://[API-Gateway-URL]/api/v1/...` will be automatically funneled through the VPC Link, pass through the ALB, and be distributed straight to the Backend Containers running in ECS Fargate.
{{% /notice %}}

***

**Next Step:** The system now has an unobstructed standard API call flow. However, to fulfill the instant video analysis status notification feature, we need to establish an additional parallel two-way connection channel. Let's move on to the next lesson to build the **API Gateway WebSocket** infrastructure!
