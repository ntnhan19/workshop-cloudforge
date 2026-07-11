---
title : "Test Connection"
date : 2026-07-10
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

After successfully provisioning RDS PostgreSQL and ElastiCache Redis, the final step of the Database Setup section is to **test and validate the connection**. This step ensures the firewall system (Security Groups) and routing configurations in the internal network (Private Subnets) are operating exactly as designed.

Simultaneously, we will synthesize all connection parameters to package them into environment variables (`.env`) ready to be injected into the Backend application in upcoming chapters.

#### 1. Test Redis Connection from EC2
Since the Redis cluster is completely isolated within the Private Subnet, we are forced to use the EC2 intermediary server (Bastion Host) located in the same VPC network to perform the test.

From the EC2 instance's Terminal, proceed with the following steps:

**Step 1: Install the Redis CLI tool**
On the Amazon Linux 2023 operating system, the command-line tool package is optimized by AWS and identified as `redis6`:
```bash
sudo dnf install -y redis6
```

**Step 2: Ping the Redis Cluster**
Use the Primary Endpoint gathered in the previous lesson (remember to remove the `:6379` port routing suffix at the end of the URL string when assigning it to the `-h` parameter):
```bash
redis6-cli -h <YOUR_REDIS_ENDPOINT> -p 6379 --tls ping
```

**Troubleshooting Experience:**
- **System command name:** Ensure you use the exact `redis6-cli` command instead of the traditional `redis-cli` to align with the new package management mechanism of Amazon Linux 2023.
- **Security flag `--tls`:** ElastiCache enables Encryption in transit by default. Without the `--tls` flag, the connection from EC2 to Redis will fall silent and hang (Timeout).
- **Firewall Check:** Ensure the Redis Security Group (`cloudforge-db-redis-sg`) has been configured to open Port 6379 with the Source accepting traffic from the application's Security Group (`cloudforge-ecs-app-sg`).

If the network configuration is perfectly accurate, the Terminal will respond with:
```plaintext
PONG
```

#### 2. Overall Architecture Validation Results
To prove that the Zero-Trust network and security infrastructure layer is fully unblocked, we will run two connection status check commands concurrently to both the Caching service (Redis) and the core Database (RDS PostgreSQL) right on the same command-line window.

![Test Connection Result](/images/5-Workshop/5.4-Database-setup/5.4.5-test-connection/test_connection.png)

The result returned on the Terminal is the technical proof confirming:
- Future application servers (ECS Tasks / AI Workers) have completely valid and safe access rights to the core data tiers.
- Security Group defense rings are working as designed: Blocking all unauthorized access from the Internet but opening doors precisely for internal resources to communicate with each other.

#### 3. Sample Environment Configuration File (.env)
After verifying the Endpoints are operating stably, we proceed to structure these parameters into a standard sample environment configuration file format to prepare for Container deployment:
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

{{% notice warning %}}
**Security Best Practice:** Absolutely never store sensitive information (like `DB_PASSWORD`) as plaintext directly in the source code or public repositories (GitHub). For a Production architecture, these parameters must be isolated, centrally managed, and dynamically injected into the application via specialized services like AWS Secrets Manager.
{{% /notice %}}

***

**Next Step:** The foundational network and database systems are ready. We will move to section **5.5: Security Setup** to establish a secure Secrets management vault, putting the security warning above into practice!
