---
title : "Initialize Database Schema (Auto-Migration)"
date : 2026-07-12
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

One of the most common challenges when deploying a system to a Cloud Environment is **initializing tables and relationships (Database Schema)**.

In real-world deployment models on AWS, since the database (RDS) is typically isolated within a Private Subnet for maximum security, we **cannot connect directly from our personal computers** to run table creation commands (such as `alembic upgrade head`).

### How do we solve this problem?
Cloud Architects usually choose one of three approaches:
1. **Use Bastion Host / VPN with CI/CD:** GitHub Actions or GitLab CI connects through a network bridge to execute commands.
2. **Independent Migration Task:** Use AWS Step Functions or AWS CodeBuild to trigger a single-run ECS Task specifically for initializing the DB.
3. **Container Auto-Migration:** Embed the table creation logic directly into the application's startup process (Lifespan). When the container boots up, it automatically detects the Database and creates tables if they don't exist.

In this Workshop, to provide a seamless experience and focus on architecture rather than complex Command Line operations, we apply **Approach 3 (Auto-Migration)**.

### FastAPI Auto-Migration Configuration
If you check the source code in `backend/main.py`, you will see this feature is built into the application's `lifespan` event:

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Auto-migrate database tables for the workshop
    try:
        from database import engine, Base
        import models.asset
        import models.ingest_job
        import models.scene
        async with engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)
        logger.info("Database tables initialized successfully via Auto-Migration.")
    except Exception as e:
        logger.error(f"Failed to auto-migrate database: {e}")
```

Whenever the ECS Task starts (when you deploy the application in the next chapter), this code will automatically contact RDS and execute the initialization of all tables (`ingest_jobs`, `assets`, `search`, etc.).

![Auto Migration Result](/images/5-Workshop/5.4-Database-setup/5.4.6-auto-migration/auto_migration.png)

{{% notice tip %}}
**Security Best Practice for Production:** Although Auto-Migration is extremely convenient for Labs, POCs, or Workshops, when deploying Enterprise-scale applications, DevOps Engineers prefer **Approaches 1 or 2** combined with Database version control tools (like Alembic, Flyway, Liquibase) to strictly control Database changes rather than creating tables automatically.
{{% /notice %}}

***

**Next Step:** The network and foundational database are fully ready (Tables will be automatically created when the Backend runs). We will move on to [**5.5: Security Setup**](../../5.5-Security-setup/) to configure a secure Secrets management vault.
