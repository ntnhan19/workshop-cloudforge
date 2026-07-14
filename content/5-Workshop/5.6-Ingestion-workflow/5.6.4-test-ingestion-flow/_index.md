---
title : "Test Ingestion Flow"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

After completing the configuration of all core components in the event-driven orchestration architecture (S3, SQS, EventBridge, Step Functions), the final step is to perform an Integration Test. This process verifies the seamless operation of the Data Pipeline from file reception to the triggering of business logic workflows.

The test scenario is divided into two independent phases to demonstrate the system's high degree of Loose Coupling.

#### 1. Test Event Routing Flow (S3 → EventBridge → SQS)
The objective of this phase is to confirm that Amazon EventBridge correctly catches the state change event from Amazon S3 and securely delivers the metadata payload into the Amazon SQS queue.

**Step 1: Upload a sample data file**
- Access the console interface of **[Amazon S3](https://s3.console.aws.amazon.com/s3/home)** and select the Bucket `cloudforge-media-upload-ntnhan19`.
- Click **Add files**, select a sample multimedia file (e.g., `sample-audio.mp3` or `demo-video.mp4`) from your computer, and click **Upload**.

![S3 Upload Success](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/s3_upload_success.png)

**Step 2: Verify and validate the message in the SQS queue**
- Access the **[Amazon SQS](https://console.aws.amazon.com/sqs/v2/home#/queues)** service → Select the main task queue `cloudforge-media-task-queue`.
- (Optional) In the **Details** section, click the **`► More`** toggle to expand it. You will see the **Messages available** metric increase (e.g., 1), indicating a new event has entered the queue.

![SQS Queue Details](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_queue_list.png)

- Click **Send and receive messages** → Scroll down to the *Receive messages* section and click **Poll for messages**.
- Click on the ID of the newly appeared message to inspect the **Body** tab. The JSON payload sent by EventBridge must accurately contain:
  - The generated event: `ObjectCreated:Put`.
  - The original file name located in the `object.key` field.

![SQS Message Received](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_message_received.png)

#### 2. Validate Orchestration Process (Step Functions Workflow)
The objective of this phase is to verify that EventBridge successfully triggered the Step Functions state machine and passed the correct S3 metadata.

**Step 1: Check Automatic Execution History**
- Go to **[AWS Step Functions](https://console.aws.amazon.com/states/home#/statemachines)** → Select the `cloudforge-media-workflow` state machine.
- You DO NOT NEED to click "Start execution". Since EventBridge is configured to point to Step Functions, the previous S3 upload action automatically spawned a new execution!
- In the **Executions** tab, click on the most recent execution at the top.

![Step Functions Executions](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_executions.png)

**Step 2: Visually Monitor the Workflow (Graph View)**
- The Graph View will now display our complete orchestration logic: `Launch AI Worker` followed by `Call Backend Webhook`.
- The execution flow will transition to the `Launch AI Worker` state and intentionally fail (since we have not deployed the ECS Compute Cluster yet).

![Step Functions Graph View](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_graph.png)

**Step 3: Analyze Input Data**
- Click on the `Launch AI Worker` step on the Graph View.
- On the right panel, inspect the **Step input** tab. You should see the full JSON event payload from S3, proving that EventBridge successfully passed the metadata to Step Functions.

#### 3. Conclusion of the Workflow Orchestration Tier
The experimental results demonstrate that the entire Ingestion Workflow pipeline operates exactly as per the initial architectural design with the Fan-out pattern. Data from the storage tier has been routed to the queue buffer (for auditing) and simultaneously triggered the orchestration scenario (Step Functions).

***

**Next Step:** The orchestration system and event-driven message pipeline of Chapter 5.6 are fully complete. We now have a solid foundation to move on to [**Chapter 5.7: Compute Setup (ECS)**](../../5.7-Compute-setup/) to configure the actual AI Worker containers to execute the business logic.
