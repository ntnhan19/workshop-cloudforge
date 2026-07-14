---
title : "Create RDS PostgreSQL"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
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

![DB Subnet Group](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/db_subnet_group.png)

#### 2. Provision the PostgreSQL Database
1. Switch to the **Databases** menu on the left and click **Create database**.
2. From the dropdown menu, click on **Full configuration** to reveal all deep system configurations.
3. **Engine options:** Select **PostgreSQL** (leave the default version suggested by the system).

![RDS Engine Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_engine.png)

4. **Templates & Durability:**
   - **Templates:** Select **Sandbox** (or *Dev/Test*) to maximize cost savings for the workshop environment.
   - **Deployment options:** Ensure the system defaults to **Single-AZ DB instance deployment**.

![RDS Template Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_template.png)

5. **Settings:**
   - **DB instance identifier:** Change the default name to `cloudforge-db`.
   - **Credentials management:** Check **Self managed** to manage your own password.
   - **Master username:** Keep `postgres`.
   - **Master password / Confirm password:** Enter a secure password (e.g., `CloudForge2026!`).

![RDS Settings](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_settings.png)

6. **Instance configuration & Storage:**
   - **DB instance class:** Keep the smallest burstable class suggested (e.g., `db.t3.micro`).
   - **Allocated storage:** Reduce the large default value to **20 GiB** (the minimum allowed).
   - **Storage autoscaling:** Uncheck *Enable storage autoscaling*.

![RDS Instance Configuration](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_instance.png)

7. **Connectivity (Internal Network - Extremely Important):**
   - **Virtual private cloud (VPC):** Select `cloudforge-vpc`.
   - **DB Subnet group:** Select the `cloudforge-db-subnet-group` created in step 1.
   - **Public access:** Select **No** *(Ensure the database is completely isolated from the Internet)*.
   - **VPC security group:** Select **Choose existing** → Remove the `default` group and select only **`cloudforge-db-redis-sg`** (The Zero-Trust firewall you created in section 5.3.4).

![RDS Connectivity Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_connectivity.png)

8. Expand the **Additional configuration** section at the bottom:
   - **Initial database name:** Enter `cloudforge_db` (so AWS creates an empty DB automatically).
   - **Backup:** Uncheck *Enable automated backups* to speed up the provisioning process for the workshop.

![RDS Additional Configuration](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_additional_configuration.png)

9. Scroll to the bottom and click **Create database**. *(The actual provisioning process usually takes 3-5 minutes).*

***

**Next Step:** After the RDS instance finishes provisioning (Status: **Available**), we will move to section [**5.4.3**](../5.4.3-enable-pgvector/) to enable the `pgvector` extension, preparing the database to store embeddings.
