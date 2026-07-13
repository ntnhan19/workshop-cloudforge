---
title : "Create ElastiCache (Redis)"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

In our AI Pipeline architecture, Amazon ElastiCache (Redis) plays two crucial roles:
1. **Speed (Caching):** Temporarily stores frequent query results to reduce the load on the primary Database.
2. **Queue Management (Message Broker):** Acts as an intermediary to distribute heavy tasks (such as generating AI embeddings or processing media) to AI Workers for asynchronous processing.

Similar to RDS, we will place this Redis cluster into the Private Subnet to ensure compliance with the Zero-Trust security architecture.

#### 1. Create a Subnet Group for ElastiCache
1. Navigate to the **ElastiCache Console**, and select **Subnet groups** on the left menu (Under *Configurations*).
2. Click the **Create subnet group** button.
3. **Name:** `cloudforge-redis-subnet-group`
4. **Description:** `Subnet group for CloudForge Redis`
5. **VPC ID:** Select `cloudforge-vpc`.
6. **Availability Zones:** Select `ap-southeast-1a` and `ap-southeast-1b`.
7. **Subnet ID:** *Important Note:* The system might automatically select all 4 subnets. Click the **Manage** button, **uncheck the 2 Public Subnets**, and keep only the 2 Private Subnets of our project checked.
8. Click **Create**.

![Redis Subnet Group](/images/5-Workshop/5.4-Database-setup/5.4.4-create-elasticache-redis/redis_subnet_group.png)

#### 2. Provision the Redis Cluster
Since the new AWS ElastiCache interface uses a step-by-step wizard, follow these configurations:

1. Switch to the **Redis OSS caches** menu (under *Resources*), and click the orange **Create cache** button. *(If a prompt introducing Valkey appears, select **Continue with Redis OSS**).*
2. **Step 1: Settings:**
   * **Configuration:** Select **Redis OSS** → **Node-based cluster** → **Cluster cache**.
   * **Cluster mode:** Select **Disabled**.
   * **Cluster info:**
     * **Name:** `cloudforge-redis`
     * **Description:** `Redis cluster for Caching and Message Queue`
     * **Node type:** Select `cache.t3.micro` or `cache.t4g.micro` (Cost optimization).
     * **Number of replicas:** `0`.
   * **Connectivity:** Under the *Subnet groups* section, select **Choose existing subnet group** → Select the `cloudforge-redis-subnet-group` you just created.
   * Click **Next** to proceed to step 2.
3. **Step 2: Advanced settings (Extremely Important):**
   * **Security groups:** Click **Manage** → Uncheck the `default` group, select only the **`cloudforge-db-redis-sg`** group → Click **Save**. *(Common Pitfall: Forgetting to attach this SG will cause ECS tasks to fail to connect)*.
   * **Encryption in transit:** By default, AWS might **Enable** this. If it is enabled, you MUST use `rediss://` instead of `redis://` in your application connection string. Otherwise, your app will experience a connection timeout (Hang) because the server expects a TLS handshake.
   * **Backup:** Scroll down and **uncheck** *Enable automatic backups* to shorten the creation time.
   * Click **Next**.
4. **Step 3: Review and create:**
   * Review your parameters and click **Create** at the bottom of the page. *(This process will take about 3-5 minutes).*

![Redis Connectivity](/images/5-Workshop/5.4-Database-setup/5.4.4-create-elasticache-redis/redis_connectivity.png)

#### 3. Retrieve Endpoint Information
After the Redis cluster's status changes to **Available**, click on the name `cloudforge-redis`.
Under the **Cluster details** section, copy the URL provided in the **Primary endpoint** field (which looks like `cloudforge-redis.xxxxxx.0001.apse1.cache.amazonaws.com:6379`) to save this connection configuration for the application.

***

**Next Step:** The core data system is now complete. We will proceed to section **5.4.4** to consolidate all the necessary Endpoints, preparing everything to deploy the Backend source code to the Cloud.
