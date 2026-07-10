---
title : "Test Ingestion Flow"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

The three core components of our message orchestration architecture (SQS, EventBridge, Step Functions) are now in place. It's time to conduct an End-to-End Test to verify that the Data Pipeline flows seamlessly from start to finish.

The objective of this hands-on lab is to simulate an end-user uploading a video to S3, and verify if SQS successfully receives the notification message without any manual intervention.

#### 1. Upload a Sample File to Amazon S3
First, we will play the role of an end-user uploading a file to the storage system.

1. Access the **Amazon S3** service on the AWS Console.
2. Find and click on the project's video upload Bucket (e.g., `cloudforge-media-upload-xxxxxx`).
3. Click the orange **Upload** button.
4. Click **Add files**, and select any file from your computer (it could be a short `.mp4` video, or simply a dummy `.txt` file).
5. Scroll to the bottom and click **Upload**. Wait a few seconds until the progress bar turns green (Succeeded).

The exact moment this file successfully lands in S3, S3 emits an Event, EventBridge instantly catches it, and routes it straight into the SQS queue. Let's verify this!

#### 2. Verify Messages in the SQS Queue
1. Open a new browser tab and access the **Amazon SQS** service.
2. Click on the main task queue **`cloudforge-media-task-queue`**.
3. In the top right corner, click the **Send and receive messages** button.
4. Scroll down to the **Receive messages** section. You should see the *Messages available* parameter showing a value greater than 0.
5. Click the **Poll for messages** button to pull messages from the queue.
6. A new message will appear in the list below. Click on its **ID**.

In the **Body** tab of the message, you will see a detailed JSON payload sent by EventBridge. If you look closely, you will find the S3 Bucket name and the File name (Object Key) that you just uploaded in Step 1.

{{% notice tip %}}
**The Power of Decoupled Architecture:** This message will wait safely in the SQS queue (up to 4 days by default) until an available AI Worker server comes to receive and delete it. Even if your Backend system doesn't exist yet, or completely crashes, you never have to worry about losing customer data!
{{% /notice %}}

***

**Next Step:** Congratulations! The Serverless Ingestion Pipeline is working perfectly. But who will read these messages waiting in SQS and execute the AI processing blocks? 

Let's move on to the most crucial chapter of the system: **Chapter 5.7: Compute Setup** to deploy the Amazon ECS (Elastic Container Service) application server cluster.
