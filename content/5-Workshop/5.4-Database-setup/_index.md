---
title : "Database Setup"
date : 2026-07-10
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

### Core Data System Overview

In this chapter, we will build a robust and secure data foundation for the Smart Media Analytics application. To strictly comply with the **Zero-Trust** security architecture, all database services will be deployed entirely within the internal network zone (**Private Subnets**) and protected by specialized firewall layers.

The core components to be established include:
1. **Amazon S3:** Infinite Object Storage, acting as the vault to receive and preserve all original video and audio files.
2. **Amazon RDS PostgreSQL:** Serves as the primary relational database, pre-integrated with the `pgvector` extension to support the Semantic Search feature for AI models.
3. **Amazon ElastiCache (Redis):** Acts as ultra-high-speed Caching and a Message Broker to distribute tasks to AI Workers for asynchronous processing.

By the end of this chapter, not only will the data system's secure communication capabilities be verified, but you will also have gathered all the necessary connection parameters to package them into environment variables (`.env`), fully ready for application deployment.

### Hands-on Content

- [Create S3 Bucket](5.4.1-create-s3-bucket/)
- [Create RDS PostgreSQL](5.4.2-create-rds-postgresql/)
- [Enable pgvector](5.4.3-enable-pgvector/)
- [Create ElastiCache (Redis)](5.4.4-create-elasticache-redis/)
- [Test Connection & Consolidate Endpoints](5.4.5-test-connection/)
