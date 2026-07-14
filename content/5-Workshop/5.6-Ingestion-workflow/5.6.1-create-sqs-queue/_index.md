---
title : "Amazon SQS Setup"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

The first core component in Smart Media Analytics' asynchronous orchestration architecture is **Amazon SQS (Simple Queue Service)**. SQS acts as a secure buffer, receiving multimedia processing requests and distributing them sequentially to the AI Worker server clusters.

The SQS infrastructure needs to be initialized first to serve as the Target for event routing rules, and simultaneously as the workload source for the Compute tier (application servers).

#### 1. Create a Dead-Letter Queue (DLQ)
Following Distributed Systems Best Practices, a secondary queue (Dead-Letter Queue - DLQ) should be established first to isolate failed processing messages (corrupted files, inference model errors), preventing the system from entering an infinite processing loop (poison pill).

From the AWS Console, navigate to the **Amazon SQS** service → **Queues** → select **Create queue**. Configure the following sections directly on the single-page interface:

- **Details section:**
  - **Type:** Select `Standard`.
  - **Name:** Enter `cloudforge-media-dlq`.
- **Configuration & Encryption & Access policy sections:** Keep the system's default configurations.
- **Dead-letter queue - Optional section:** Default to `Disabled` (the DLQ does not need to point to another DLQ).

![Create DLQ](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/create_dlq.png)

Finally, scroll down to the bottom right corner and click **Create queue** to complete initialization.

#### 2. Create the Main Task Queue
Once the DLQ is ready, return to the Queues list interface and click **Create queue** once again to set up the main workload orchestration queue:

- **Details section:**
  - **Type:** Select `Standard` (Ensuring maximum processing throughput, suitable for the asynchronous Media processing architecture).
  - **Name:** Enter `cloudforge-media-task-queue`.

- **Configuration section (Optimized for AI/Media tasks):**
  - **Visibility timeout:** Change from the default to **15 Minutes** (Equivalent to 900 Seconds).

{{% notice tip %}}
**System Design Note:** Because AI models processing video/audio files consume a lot of computational time, the *Visibility timeout* parameter must be set sufficiently long (15 minutes). When an AI Worker receives a message from the queue, this message is temporarily hidden from other Workers. This guarantees the current Worker process has enough time to complete the task without other servers competing for it or processing it redundantly.
{{% /notice %}}

- **Dead-letter queue - Optional section:**
  - **Set this queue to receive undeliverable messages:** Switch to the **Enabled** state.
  - **Choose queue:** Select the exact `cloudforge-media-dlq` queue initialized in step 1.
  - **Maximum receives:** Set the value to `3`. (If a Media task is processed with errors by the Worker more than 3 times, the message will automatically be moved to the DLQ for manual inspection by the engineering team).

![Configure Task Queue](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/configure_task_queue1.png)

![Configure Task Queue](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/configure_task_queue2.png)

Finally, double-check all configuration information and click the **Create queue** button.

#### 3. Deployment Results
Upon successful initialization, the system will allocate a unique URL identifier for each queue. This URL string will be recorded to be configured into the environment variables of the application tier.

![SQS Queue Created](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/sqs_queue_created.png)

***

**Next Step:** After completing the storage and message orchestration (SQS) tiers, we will proceed to configure the router in [**5.6.2: EventBridge Setup**](../5.6.2-create-eventbridge-rule/) to automatically capture data change events from S3 and push them into this queue.
