---
title : "VPC & Networking"
date : 2026-07-09
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Network Foundation (VPC)

In this section, we will build a secure and highly available Virtual Private Cloud (VPC) to host the entire architecture of the Smart Media Analytics system. Instead of using the default network, designing a custom VPC allows us to fully control traffic flow, isolate sensitive services, and optimize data transfer costs.

We will deploy a multi-tier network architecture comprising:
- **Public Subnets**: Hosting the Application Load Balancer to receive requests from end-users (Frontend).
- **Private Subnets**: Hosting internal processing services (ECS Fargate Workers), databases (RDS PostgreSQL), and caching layers (ElastiCache Redis) to ensure they are completely isolated from direct Internet access.
- **NAT Gateway**: Acting as a secure bridge that allows services within the Private Subnets to download patches or pull AI models from external sources safely.
- **S3 Gateway Endpoint**: An internal "fast lane" enabling ECS Fargate to store and retrieve videos from Amazon S3 directly without incurring expensive NAT Gateway data transfer fees.

By leveraging the built-in **VPC and more** wizard on the AWS Console, all these components, along with their associated Route Tables, will be provisioned automatically, saving you significant setup time.

#### Content

- [Create VPC & Subnets](5.3.1-create-vpc-subnets/)
- [Verify NAT Gateway & Routing](5.3.2-create-nat-gateway/)
- [Analyze S3 Gateway Endpoint](5.3.3-create-s3-gateway-endpoint/)
- [Configure Security Groups](5.3.4-security-groups/)
