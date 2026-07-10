---
title : "Test Connection"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

After successfully provisioning RDS PostgreSQL and ElastiCache Redis, the final step of the Database Setup section is to **test and validate the connection**. This step ensures that the firewalls (Security Groups) and internal network routing configurations (Private Subnets) are working exactly as designed.

At the same time, we will consolidate all connection parameters to package them as environment variables (`.env`) ready to be injected into the Backend application in the upcoming chapters.

#### 1. Test Redis Connection from EC2
Since the Redis cluster is completely isolated within the Private Subnet, we must use an intermediate EC2 server (Bastion Host) located in the same VPC network to perform the test.

At the EC2 Terminal, proceed with the following steps:

**Step 1: Install the Redis CLI tool**
On the Amazon Linux 2023 OS, the command-line tool package is optimized by AWS and identified as `redis6`:
```bash
sudo dnf install -y redis6
```

**Step 2: Ping the Redis Cluster**
Use the Primary Endpoint collected in the previous section (note: remove the port routing `:6379` at the end of the URL string when assigning it to the `-h` parameter):

```bash
redis6-cli -h <ENTER_YOUR_REDIS_ENDPOINT_HERE> -p 6379 --tls ping
```

> **Real-world Experience (Troubleshooting):**
> - **System command name:** Ensure you use the exact `redis6-cli` command instead of the traditional `redis-cli` to align with the new package management mechanism of Amazon Linux 2023.
> - **`--tls` Security flag:** ElastiCache enables Encryption in transit by default. If the `--tls` flag is missing, the connection from EC2 to Redis will fall into a silent state and time out.
> - **Firewall Clearance:** Ensure the Redis Security Group (`cloudforge-db-redis-sg`) has been configured to open Port 6379 with the Source accepting traffic from the application Security Group (`cloudforge-ecs-app-sg`).

If the network configuration is perfectly accurate, the Terminal will immediately respond:

```plaintext
PONG
```

#### 2. Overall Architecture Validation Results
To prove that the network infrastructure and Zero-Trust security layers are completely seamless, we will run two connectivity status check commands simultaneously to both the Caching (Redis) and Core Database (RDS PostgreSQL) services right on the same command-line window.

*📸 Illustration: Successful connection validation to isolated internal data layers.*
![Test Connection Result](../../../../images/5-Workshop/5.4-Database-setup/5.4.4-test-connection/test_connection.png)

The result returned on the Terminal is technical proof affirming:
- Future application servers (ECS Tasks / AI Workers) have fully valid and secure access to the core data layers.
- The protective rings of the Security Groups work correctly by design: Blocking all unauthorized access from the Internet but precisely opening doors for internal resources to communicate with each other.

#### 3. Sample Environment Configuration File (.env)
After validating that the Endpoints operate stably, we will structure these parameters into a standard sample environment configuration file ready to be injected into the application Containers:

```env
# Database Configuration
DB_HOST=cloudforge-db.xxxxxx.ap-southeast-1.rds.amazonaws.com
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=<ENTER_YOUR_PASSWORD_HERE>
DB_NAME=cloudforge_db

# Redis Caching & Queue Configuration
REDIS_HOST=master.cloudforge-redis.xxxxxx.apse1.cache.amazonaws.com
REDIS_PORT=6379
```

> [!WARNING]
> **Strict Security Warning:** Absolutely never commit a file containing actual password information (`DB_PASSWORD`) to public source code repositories like Public GitHub. For a real enterprise environment, these sensitive details will be centrally managed, encrypted, and dynamically injected through specialized services such as AWS Secrets Manager or Systems Manager Parameter Store.

***

**Next Step:** With our core networking and database foundation ready, we will proceed to section **5.5: Security Setup** to configure a secure Secrets management vault, putting the security warning above into practice!
