---
title : "Create RDS PostgreSQL"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

Amazon RDS (Relational Database Service) makes it easy to set up, operate, and scale a relational database in the cloud. For this project, we will use the **PostgreSQL** engine because it perfectly supports the `pgvector` extension – a core requirement for storing vector embeddings for our AI semantic search feature.

#### 1. Create a DB Subnet Group
Before launching the database, RDS requires us to define a **Subnet Group** to specify which Private Subnets within our VPC the database servers can be placed into.

1. Navigate to the **RDS Console** and select **Subnet groups** on the left menu.
2. Click the orange **Create DB subnet group** button.
3. **Name:** `cloudforge-db-subnet-group`
4. **Description:** `Subnet group for CloudForge Databases`
5. **VPC:** Select `cloudforge-vpc`.
6. **Add subnets:** Select 2 Availability Zones (`ap-southeast-1a` and `ap-southeast-1b`), then check the 2 corresponding Private Subnets designated for the Database layer.
7. Click **Create**.

*📸 Illustration: Creating the DB Subnet Group*
![DB Subnet Group](../../../../images/5-Workshop/5.4-Database-setup/5.4.1-create-rds-postgresql/db_subnet_group.png)

#### 2. Provision the PostgreSQL Database
1. Switch to the **Databases** menu on the left and click **Create database**.
2. **Choose a database creation method:** Select **Standard create**.
3. **Engine options:** Select **PostgreSQL** (choose the latest supported version, e.g., 15.x or 16.x).
4. **Templates:** Select **Free tier** or **Dev/Test** (to optimize costs for this workshop).
5. **Settings:**
   - **DB instance identifier:** `cloudforge-db`
   - **Master username:** `postgres`
   - **Master password:** *Enter a secure password (e.g., CloudForge2026!)*. *(Note: Remember this password as we will need it to log in later)*.
6. **Connectivity (Extremely Important):**
   - **Virtual private cloud (VPC):** Select `cloudforge-vpc`.
   - **DB Subnet group:** Select the `cloudforge-db-subnet-group` you just created.
   - **Public access:** Select **No** *(Ensure the database is completely isolated from the Internet)*.
   - **VPC security group:** Select **Choose existing** $\rightarrow$ Remove the `default` group and select **`cloudforge-db-redis-sg`** (The Zero-Trust firewall you created in section 5.3.4).
7. Expand the **Additional configuration** section:
   - **Initial database name:** Enter `cloudforge_db` (so AWS creates an empty DB automatically).
   - You may uncheck *Enable automated backups* to speed up the provisioning process for the workshop.
8. Scroll to the bottom and click **Create database**. *(Note: The actual provisioning process usually takes 5-10 minutes).*

*📸 Illustration: Setting up Connectivity to lock the DB inside the Private Subnet using the Zero-Trust Security Group.*
![RDS Connectivity Setup](../../../../images/5-Workshop/5.4-Database-setup/5.4.1-create-rds-postgresql/rds_connectivity.png)

***

**Next Step:** While waiting for the RDS instance to finish provisioning, we will move to section **5.4.2** to enable the `pgvector` extension, preparing the database to store embeddings.
