---
title : "Analyze S3 Gateway Endpoint"
date : 2026-07-09
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

In Media / AI Pipeline systems, Workers residing in Private Subnets or servers in Public Subnets must continuously read and write massive files (such as high-resolution videos and images) to Amazon S3 constantly.

If data from the Private Subnet follows the default routing path (`Private Subnet` → `NAT Gateway` → `Internet` → `S3`), AWS will apply NAT Gateway data processing charges (approximately $0.045/GB). With the large video capacities of our project, these costs would skyrocket. To solve this cost optimization challenge, the **VPC and more** wizard automatically provisioned an **S3 Gateway Endpoint** to steer packets through the internal network.

#### 1. Verify Endpoint Status
1. On the left navigation menu of the VPC service, go to **Endpoints** (located right under *Managed prefix lists*).
2. Check the list and select the Endpoint whose name contains the S3 service code for the Singapore region: `com.amazonaws.ap-southeast-1.s3`.
3. Ensure the **Status** column displays in green as **Available**.

   ![S3 Endpoint Verification](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_verify.png)

#### 2. Analyze Route Table Configurations
The secret to this shortcut is that AWS automatically injects a special routing rule into both groups of route tables (Public and Private) in our project. This keeps all traffic destined for S3 within the AWS internal bandwidth network instead of looping out to the Internet.

##### A. For Private Route Tables (Optimizing NAT Costs)
1. Select a private route table (e.g., `cloudforge-rtb-private1`).
2. On the **Routes** tab, observe the routing rule running parallel to the NAT Gateway:
   - **Destination:** `pl-6fa54006` (The IP Prefix List representing the entire S3 service in the Singapore Region).
   - **Target:** Points directly to the S3 Endpoint ID (`vpce-xxxxxx`).
   - **Meaning:** Any upload/download operations from the AI Worker application (Private) to S3 will go directly through this gateway, **completely bypassing the NAT Gateway**. This maximizes speed and reduces data processing costs through NAT to **$0**.

   ![S3 Endpoint Routes Private](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_routes_private.png)

##### B. For Public Route Table (Optimizing Public Bandwidth)
1. Switch to select the public route table `cloudforge-rtb-public`.
2. On the **Routes** tab, you will also see the rule pointing the `pl-6fa54006` IP range to `vpce-xxxxxx`, appearing independently of the Internet Gateway (`igw-xxxx`).
   - **Meaning:** Even resources in the Public zone (like a Load Balancer or Bastion Host, if any), when interacting with S3, will go through this internal Endpoint. This reduces the load on the Internet Gateway and increases the security of the data flow.

   ![S3 Endpoint Routes Public](/images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_routes_public.png)

***

**Next Step:** Now that our core network, routing, and cost-optimized shortcuts have been verified to work perfectly, we will move on to section **5.3.4: Configure Security Groups** to establish strict firewall locks for each specific service.
