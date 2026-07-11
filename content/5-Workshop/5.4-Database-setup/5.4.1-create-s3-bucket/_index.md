---
title : "Create S3 Bucket"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

The first and most fundamental storage component of the entire Smart Media Analytics system is **Amazon S3 (Simple Storage Service)**. This will be the place to receive, store, and preserve all original video and audio files uploaded by users.

To ensure global uniqueness and data security, we will initialize a new Bucket.

#### 1. Amazon S3 Bucket Initialization Process
Access the **Amazon S3** service on the AWS Console → Click **Create bucket**. Configure the details through the visual segments on the interface:

1. **General configuration segment:**
   - **AWS Region:** Select `ap-southeast-1` (Singapore) - Ensure it's in the same region as the VPC network infrastructure to optimize transmission latency.
   - **Bucket name:** Enter `cloudforge-media-upload-<your-name>` *(Note: The Bucket name must be a globally unique identifier across the entire AWS system).*
2. **Object Ownership segment:** Keep the default setting `ACLs disabled (recommended)` for centralized access management via IAM policies.
3. **Block Public Access settings for this bucket segment:** 
   - Check **Block all public access**. This is a mandatory security standard (Best Practice) to completely isolate storage resources from the public Internet environment, only allowing internal access by Backend tier services.
4. **Bucket Versioning & Default encryption segment:** Keep the default configuration parameters (Automatic encryption using the SSE-S3 mechanism).

Scroll to the bottom of the form page and click **Create bucket** to complete the process.

#### 2. Enable Event Forwarding to Amazon EventBridge (Important)
By system default, Amazon S3 will not proactively broadcast event data externally. For the asynchronous orchestration pipeline in Chapter 5.6 to function, we need to enable the notification feature:

1. In the Buckets list, click on the name of the newly created Bucket.
2. Navigate to the **Properties** tab on the horizontal menu bar.
3. Scroll down to the **Amazon EventBridge** segment, click **Edit**.
4. Toggle the state from *Off* to **On** and click **Save changes**.

{{% notice tip %}}
**System Design Note:** Toggling the state to *On* allows Amazon S3 to automatically package metadata in JSON format and push it directly into EventBridge's default event Bus whenever a file upload action (`ObjectCreated`) occurs.
{{% /notice %}}

#### 3. Deployment Results
The unstructured storage infrastructure (Object Storage) has been successfully established and is ready to act as the task provisioning source for the entire automated processing pipeline downstream.

![S3 EventBridge Config](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_eventbridge_enabled.png)

***

**Next Step:** The File Storage system is now ready. We will continue setting up the structured data memory in **5.4.2: Create RDS PostgreSQL** to serve metadata and search vector storage for AI.
