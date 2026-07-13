---
title : "Resource Clean Up"
date : 2026-07-10
weight : 14
chapter : false
pre : " <b> 5.14. </b> "
---

### Clean Up Overview

**CRITICAL COST WARNING!**

The Smart Media Analytics system utilizes many powerful AWS services like RDS, NAT Gateway, WAF, and ECS Fargate. Although some resources fall under the Free-tier, maintaining them 24/7 will generate significant bills if you forget to delete them after finishing the practice.

In this final chapter, we will execute resource destruction steps in a safe sequence (reverse of creation) to ensure no "orphan resources" are left behind, safeguarding your wallet.

#### Step 1: Empty and Delete S3 Buckets
AWS requires S3 buckets to be empty before they can be deleted.
1. Go to the **S3 Console**.
2. Find your media storage bucket (e.g., `cloudforge-media-storage-...`).
3. Select the bucket and click **Empty**. Type `permanently delete` to confirm.
4. Once empty, select the bucket again and click **Delete**.

#### Step 2: Delete Frontend and CDN
1. Go to the **AWS Amplify Console**.
2. Select your frontend app and click **Actions > Delete app**.
3. Go to the **CloudFront Console**.
4. Select your distribution, click **Disable**, and wait for its status to change to *Disabled* (this may take a few minutes).
5. Once disabled, select it and click **Delete**.

#### Step 3: Delete Compute Resources (ECS & ECR)
1. Go to the **Amazon ECS Console**.
2. Navigate to **Clusters** and select your cluster (e.g., `cloudforge-compute-cluster`).
3. Under the **Services** tab, select your backend service, click **Delete**, and confirm by typing `delete`.
4. Once the service is deleted, click **Delete cluster** in the top right corner.
5. Go to the **Amazon ECR Console**.
6. Select your repository (e.g., `cloudforge-backend`) and click **Delete**.

#### Step 4: Delete Databases (RDS & ElastiCache)
1. Go to the **Amazon RDS Console**.
2. Select your database instance (e.g., `cloudforge-db`).
3. Go to **Modify**, scroll down, and uncheck **Enable deletion protection**, then apply changes immediately.
4. Select the database again and click **Delete**. **Uncheck** the option to create a final snapshot. Type `delete me` to confirm.
5. Go to the **Amazon ElastiCache Console**.
6. Navigate to **Redis clusters**, select your cluster, and click **Delete**.

#### Step 5: Delete Security and Network (WAF & VPC)
1. Go to the **AWS WAF Console**.
2. Navigate to **Web ACLs** (ensure you are in the *Global (CloudFront)* region).
3. Select your Web ACL, disassociate it from any remaining resources, and click **Delete**.
4. Go to the **VPC Console**.
5. Navigate to **NAT Gateways**. Select your NAT Gateway and click **Delete**. *You must wait for the state to become 'Deleted' (about 3-5 minutes) before proceeding.*
6. Navigate to **Elastic IPs**, select the allocated IP, click **Actions > Release Elastic IP address**.
7. Navigate to **Your VPCs**, select your custom VPC, click **Actions > Delete VPC**. This will automatically clean up associated subnets, route tables, and internet gateways.

