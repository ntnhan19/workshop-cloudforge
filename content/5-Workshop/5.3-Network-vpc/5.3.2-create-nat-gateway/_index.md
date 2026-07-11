---
title : "Verify NAT Gateway & Routing"
date : 2026-07-09
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

In the previous step, the **VPC and more** wizard automatically configured and provisioned a NAT Gateway for our system. The NAT (Network Address Translation) Gateway is a critical component in a Multi-tier architecture: It acts as an intermediary, allowing resources in the isolated **Private Subnets** (like ECS Fargate Tasks) to initiate outbound connections to the internet (e.g., to pull Docker images or call external APIs), while strictly blocking any unsolicited inbound connections from the Internet to the core infrastructure.

In this section, we will verify the operational status of the NAT Gateway and deeply analyze the Route Tables to understand exactly how our network traffic is controlled.

#### 1. Verify NAT Gateway & Elastic IP
1. On the left navigation pane of the VPC service, select **NAT gateways**.
2. Verify that the **State** of the NAT Gateway (named `cloudforge-nat-...`) is displaying in green as **Available**.
3. Observe the **Elastic IP address** field: This is the single static public IP allocated specifically to you by AWS. From now on, all outbound internet traffic from any Private Subnet server will be "masqueraded" under this public IP address.

   ![NAT Gateway Verification](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/nat_gateway_verify.png)

#### 2. Analyze Route Tables & Subnet Associations
A Route Table contains network rules that dictate where packets from a subnet are directed. The automated system created specialized route tables:

##### A. Public Route Table (Name containing `cloudforge-rtb-public`)
1. Select this public route table and click the **Routes** tab. You will see a routing rule:
   * **Destination:** `0.0.0.0/0` (All internet traffic).
   * **Target:** Points to `igw-XXXX` (Internet Gateway). This is the key that transforms its associated subnets into "Public Subnets".
2. Switch to the **Subnet associations** tab → **Explicit subnet associations** section: Verify that the 2 Public Subnets (`cloudforge-subnet-public-...`) are securely associated here.

   ![Public Route Table](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/public_route_table.png)

##### B. Private Route Tables (Name containing `cloudforge-rtb-private`)
1. Select a private route table and click the **Routes** tab. Observe the routing rule:
   * **Destination:** `0.0.0.0/0` (All outbound traffic).
   * **Target:** Points directly to `nat-XXXX` (The NAT Gateway verified in Step 1). This ensures internal data is safely protected when traversing the Internet.
2. Switch to the **Subnet associations** tab: Verify that the Private Subnets (which will host ECS and RDS) are isolated in this table, completely detached from the Internet Gateway.

   ![Private Route Table](/images/5-Workshop/5.3-Network-vpc/5.3.2-create-nat-gateway/private_route_table.png)

***

**Next Step:** Now that our outbound internet routing is clear, in section **5.3.3**, we will inspect a crucial cost-optimization component: the **S3 Gateway Endpoint**. This ensures that heavy data transfers (like Videos) do not incur unnecessary NAT Gateway data processing fees.
