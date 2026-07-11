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
The objective of this phase is to verify the validity and business logic branching capabilities of the state machine upon receiving an orchestration request.

**Step 1: Trigger State Machine execution**
- Access the **[AWS Step Functions](https://console.aws.amazon.com/states/home#/statemachines)** service → Select the state machine `cloudforge-media-workflow`.
- Click the **Start execution** button. In the Input dialog, you can enter a dummy JSON payload (e.g., `{"status": "success"}`) and click **Start execution** again.

![Step Functions Input Mock](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_input_mock.png)

![Step Functions Execution List](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_execution_list.png)

**Step 2: Visually monitor the workflow (Graph View)**
- Based on the placeholder condition (`{% true %}`) established during the skeleton building segment, the execution flow will automatically transition sequentially from `Start Processing` → pass the evaluation point `Check Status` → accurately converge on the successful task branch `Update Metadata`.
- The overall status bar displays green with a **Succeeded** label.

![Step Functions Success](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_success.png)

**Step 3: Analyze Input/Output Data**
- Click on any specific step on the Graph View (e.g., `Update Metadata`).
- On the right panel, inspect the **Step input** and **Step output** tabs to clearly see how data is passed between states. *(Note: Since these are currently just placeholder "Pass States", the JSON data you entered in Step 1 will simply pass directly from Input to Output without modification).*

![Step Functions Details](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_details.png)

#### 3. Conclusion of the Workflow Orchestration Tier
The experimental results demonstrate that the entire Ingestion Workflow pipeline operates exactly as per the initial architectural design. Data from the storage tier has been routed and securely isolated in the queue buffer, and the orchestration scenario is ready to connect with actual computing applications.

***

**Next Step:** The orchestration system and event-driven message pipeline of Chapter 5.6 are fully complete. We now have a solid foundation to move on to **Chapter 5.7: Compute Setup (ECS)** to configure the Container server clusters executing AI models that pull data from SQS.
