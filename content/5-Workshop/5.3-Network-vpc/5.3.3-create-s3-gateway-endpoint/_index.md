---
title : "Analyze S3 Gateway Endpoint"
date : 2026-07-09
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

In Media / AI Pipeline systems, Workers residing in Private Subnets must continuously read and write massive files (such as high-resolution videos and images) to Amazon S3 every single second.

If this data follows the default routing path (Private Subnet $\rightarrow$ NAT Gateway $\rightarrow$ Internet $\rightarrow$ S3), AWS will apply NAT Gateway data processing charges (approximately $0.045/GB). With hundreds of GBs or TBs of video processing in a real-world scenario, costs can skyrocket. To solve this, the **VPC and more** wizard automatically provisioned an **S3 Gateway Endpoint**.

#### 1. Verify Endpoint Status
1. On the left menu of the VPC service, locate **Endpoints** (under the *Virtual private cloud* category).
2. You should see an Endpoint with a name containing `cloudforge-vpce-s3`.
3. Ensure the **Status** column displays in green as **Available**.
4. Look at the **Service name** column; it will point to the S3 service in the Singapore region: `com.amazonaws.ap-southeast-1.s3`.

   ![S3 Endpoint Verification](../../../../images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_verify.png)

#### 2. The Secret lies in the Route Table
How do servers in a Private Subnet know how to "shortcut" to S3 without traversing the NAT Gateway? The answer lies in automatically injected routing rules:

1. Navigate back to **Route tables** and select one of the system's **Private Route Tables**.
2. Click the **Routes** tab. You will see a new route rule appearing alongside the NAT Gateway rule:
   * **Destination:** `pl-XXXXXX` (This is a *Prefix List* – a curated list containing all the public IP ranges of the AWS S3 service).
   * **Target:** Points directly to `vpce-XXXXXX` (The exact ID of the S3 Gateway Endpoint verified above).

   ![S3 Endpoint Routes](../../../../images/5-Workshop/5.3-Network-vpc/5.3.3-create-s3-gateway-endpoint/s3_endpoint_routes.png)

#### 🎯 Architectural Conclusion:
Thanks to this precise routing rule, whenever our AI Workers or Backend applications execute a video upload/download to S3, AWS will "steer" the packets directly through the internal AWS backbone, **completely bypassing the NAT Gateway**. This architecture guarantees maximum transfer speeds (data center internal network velocity) and **reduces data transfer costs to $0**!

***

**Next Step:** Now that our core network, routing, and cost-optimized shortcuts are perfectly established, we will move on to section **5.3.4: Configure Security Groups** to establish strict firewall locks for each specific service.
