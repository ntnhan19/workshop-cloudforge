---
title : "EventBridge Setup"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.6.2. </b> "
---

Now that the SQS queue is ready to accept workloads, the next step in the Event-driven architecture is setting up the central router: **Amazon EventBridge**.

In the Smart Media Analytics system, when a multimedia file (Video/Audio) is uploaded to Amazon S3, S3 will emit an event. EventBridge acts as the central router (using the Fan-out pattern) that routes the message containing the file's metadata to multiple destinations: the SQS queue `cloudforge-media-task-queue` (as an audit buffer) and AWS Step Functions (to orchestrate the AI Worker).

#### 1. Create EventBridge Rule (Enhanced Builder)
Access the **Amazon EventBridge** service → **Rules** → **Create rule**. Use the visual Enhanced builder interface to configure the routing flow:

**Step 1: Set Up Trigger (Event Source)**
- In the **Build** tab, under the *Events* section on the left, search for and drag the **S3 (Simple Storage Service) Object Created** block into the **Triggering Events** area.
- **CRITICAL STEP (Prevent Infinite Loops):** You must scroll down to the **Event pattern** section (on the right panel), switch to the **JSON** tab, and paste the following code (remembering to replace `<YOUR_BUCKET_NAME>`). Adding the `{"prefix": "uploads/"}` filter ensures the system only triggers the AI Worker when a user uploads an original video, and **ignores** the thumbnail images generated and uploaded by the AI Worker itself later on:
  ```json
  {
    "source": ["aws.s3"],
    "detail-type": ["Object Created"],
    "detail": {
      "bucket": {
        "name": ["<YOUR_BUCKET_NAME>"]
      },
      "object": {
        "key": [{
          "prefix": "uploads/"
        }]
      }
    }
  }
  ```

![EventBridge Event Pattern](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_event_pattern.png)

**Step 2: Set up Target (Destination)**
- On the left toolbar, search for the **SQS** service (or open the AWS Services category), and drag the **Amazon SQS** block into the **Targets** area.
- In the Target configuration panel, under *Queue*, select the exact **`cloudforge-media-task-queue`** queue initialized in the previous section.
- *(Note: We will add the second target, AWS Step Functions, after creating it in the next chapter).*

![EventBridge Target Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_target.png)

**Step 3: Name the Rule**
- Switch to the **Configure** tab on the navigation bar.
- **Rule name:** Name the rule `cloudforge-s3-to-sqs-rule`.
- **Activation:** Ensure the activation toggle is in the **Active** state.

![EventBridge Rule Naming](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_rule_naming.png)

Finally, double-check the routing diagram and click **Create** in the top right corner to complete the deployment process.

#### 2. Event Routing Deployment Results
When the rule is successfully created and activated, the Asynchronous Data Pipeline is officially established and flows seamlessly from S3 Storage through EventBridge, securely congregating at the SQS Queue.

![EventBridge Rule Created](/images/5-Workshop/5.6-Ingestion-workflow/5.6.2-create-eventbridge-rule/eventbridge_rule_created.png)

{{% notice tip %}}
**System Design Note (Decoupling Architecture):** Decoupling components using EventBridge means the Frontend layer does not need to care about the Backend system's state. Users simply upload files to S3 and receive a success response immediately. All heavy backend tasks are orchestrated by EventBridge and SQS, completely eliminating the risk of bottlenecks or timeouts at the API gateway.
{{% /notice %}}

***

**Next Step:** The data flow from User Upload (S3) → Event Detection (EventBridge) → Waiting for Processing (SQS) is now fully connected. Next, we will introduce an advanced orchestration concept in section [**5.6.3: Step Functions Setup (Optional)**](../5.6.3-create-step-functions/).
