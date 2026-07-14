---
title : "Workflow Orchestration with Step Functions"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

After setting up the event routing system (EventBridge) and the message queue (SQS), the next step is to build the orchestration layer for complex processing workflows. **AWS Step Functions** acts as a central Orchestrator, managing State Machines to visually control the Media processing workflow with high durability and fault tolerance.

#### 1. Establish EventBridge Connection for Webhook
To allow Step Functions to securely invoke an external API, AWS requires configuring a Connection to store authentication methods. Since the current Backend API does not require token authentication, the team chose the **API Key** method with a dummy value to satisfy the mandatory system constraint:
- Access **Amazon EventBridge** → **API destinations** → **Connections** tab → **Create connection**.
- **Connection name:** `cloudforge-webhook-conn`.
- **API type:** Select `Public` (since our ALB is accessible via Public DNS, or choose `Private` if using an internal VPC).
- **Configure authorization:** Select `Custom configuration`.
- **Authorization type:** Select `API Key` (enter `X-Api-Key` for `API Key Name` and `dummy` for `Value`).

![Create Connection](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_connection_1.png)
![Configure Authorization](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_connection_2.png)

#### 2. Initialize and Configure State Machine Properties
Access the **AWS Step Functions** service → **State machines** → select **Create state machine**. Configure the foundational parameters in the new initialization interface:

- **Template:** Choose `Create from blank` to define the processing flow from scratch.
- **State machine name:** Name the rule `cloudforge-media-workflow`.
- **State machine type:** Select `Standard` (Ensuring a maximum execution time of up to 1 year and supporting detailed execution history storage, perfectly suited for long-term Media analytics tasks).
- **State machine query language:** The system defaults to **JSONata** (A new query standard that optimizes direct data transformation during state transitions without relying on intermediate source code).

![Step Functions Config](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_config.png)

Click **Continue** to move to the visual design interface.

#### 3. Build the Orchestration Diagram
In the Workflow Studio Canvas space, drag and drop logic state blocks to establish the integration with the ECS AI Worker:

1. **Worker Invocation State (Amazon ECS - RunTask):** From the left toolbar (Actions tab), search for **ECS** and drag the **RunTask** block to drop it right below the `Start` point. Rename it to `Launch AI Worker`.
2. **Configure Arguments for RunTask (JSONata):**
   - Click the `Launch AI Worker` block. In the **Configuration** tab on the right, check the **Wait for task to complete** box (Crucial: This forces Step Functions to wait until the AI Worker finishes 100% before moving to the Webhook block).
   - Switch to the **Arguments & Output** tab, where you'll see a JSON editor.
   - Because we chose **JSONata** in the previous step, the new AWS Step Functions interface consolidates all configuration (Cluster, Network, Overrides...) into a single JSON file instead of separate input fields.
   - Delete the default JSON and paste the following code into the **Arguments** box:
```json
{
  "LaunchType": "FARGATE",
  "Cluster": "cloudforge-compute-cluster",
  "TaskDefinition": "cloudforge-ai-worker-task",
  "NetworkConfiguration": {
    "AwsvpcConfiguration": {
      "Subnets": [
        "<YOUR_SUBNET_ID_1>",
        "<YOUR_SUBNET_ID_2>"
      ],
      "SecurityGroups": [
        "<YOUR_SECURITY_GROUP_ID>"
      ],
      "AssignPublicIp": "ENABLED"
    }
  },
  "Overrides": {
    "ContainerOverrides": [
      {
        "Name": "worker-container",
        "Command": [
          "python",
          "entrypoint.py",
          "--config-json",
          "{% $string({'bucket': $states.input.detail.bucket.name, 'video_s3_key': $states.input.detail.object.key}) %}"
        ]
      }
    ]
  }
}
```
   - *(Note: Remember to replace `<YOUR_SUBNET_ID>` and `<YOUR_SECURITY_GROUP_ID>` with the actual VPC Subnet and Security Group IDs configured for the system. The `{% $string(...) %}` syntax is a powerful JSONata feature that automatically extracts the S3 event metadata and converts it to a JSON string for the Worker's entrypoint).*

3. **Backend Webhook Invocation Block (HTTP - Invoke):** To complete the Event-Driven architecture, after the AI Worker finishes scene extraction, the system needs to notify the Backend to continue running heavy AI Models (Whisper, Bedrock) on the same `job_id`. 
   - From the left search bar, type `invoke` or `HTTP`. Drag the **HTTP Endpoint (Call HTTPS APIs)** block right below `Launch AI Worker`. Rename it `Call Backend Webhook` (Note: ABSOLUTELY DO NOT select the Lambda Invoke block).
   - In the **API Endpoint** configuration, because Step Functions strictly requires the secure `https://` protocol and the Backend ALB is in a Private network, the team entered the **API Gateway** URL (which acts as an intermediary receiving HTTPS and forwarding it via VPC Link to the ALB). Syntax: `https://<API-GATEWAY-ID>.execute-api.<REGION>.amazonaws.com/api/v1/ingest/webhook` with Method `POST`.
   - **Authentication:** To integrate HTTP Invoke natively in Step Functions, our team used the **EventBridge Connection** (`cloudforge-webhook-conn` created in the previous step) to securely manage access between Step Functions and the ALB.
   - Still in the **Configuration** tab, scroll down to the **Request body** section and define the payload (using JSONata) to report a success status along with the `job_id`. Since Step Functions has already passed the ECS step, the original input has been modified. We must use the Context object `$states.context.Execution.Input` to retrieve the original EventBridge event:
     ```json
     {
       "job_id": "{% $split($states.context.Execution.Input.detail.object.key, '/')[1] %}",
       "status": "success"
     }
     ```

![Workflow Studio Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/workflow_studio_setup.png)

#### 4. Execute Infrastructure Deployment
Once the inverted Y-shaped diagram accurately completes the orchestration logic, perform the saving steps:
- Move to the top right corner and click the **Create** button.
- **Configure IAM Role:** Because we used JSONata with a custom JSON payload, the AWS Console cannot auto-detect the `ecs:RunTask` dependency. You MUST manually grant permissions to the generated role:
  1. Once saved, click on the **IAM role ARN** link in the State Machine details page to open the IAM Console.
  2. In the IAM Role page, click **Add permissions** -> **Create inline policy**.
  3. Switch to the **JSON** tab, delete the old code, and paste the following snippet (which includes RunTask, PassRole, and EventBridge permissions so Step Functions can wait for the ECS Task to complete):
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "ecs:RunTask",
                     "ecs:StopTask",
                     "ecs:DescribeTasks"
                 ],
                 "Resource": "*"
             },
             {
                 "Effect": "Allow",
                 "Action": [
                     "events:PutTargets",
                     "events:PutRule",
                     "events:DescribeRule"
                 ],
                 "Resource": [
                     "arn:aws:events:*:*:rule/StepFunctionsGetEventsForECSTaskRule"
                 ]
             },
             {
                 "Effect": "Allow",
                 "Action": "iam:PassRole",
                 "Resource": "*"
             }
         ]
     }
     ```
  4. Click **Next**, name the policy `AllowECSRunTask`, and click **Create policy**.
  
![IAM Inline Policy](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/iam_inline_policy.png)

#### 5. Update EventBridge Rule (Event Routing)
Now that the Step Functions state machine is ready, we must configure EventBridge to route S3 events to it:
1. Go back to **Amazon EventBridge** -> **Rules**, select the rule `cloudforge-s3-to-sqs-rule` and click **Edit**.
2. In the **Enhanced builder** interface, click the `>` arrow on the left edge to expand the **Events and Targets** palette.
3. Search for **AWS Step Functions state machine**, then drag and drop this block into the **Targets** area (alongside the existing SQS queue).
4. In the configuration panel on the right, select the state machine `cloudforge-media-workflow`.
5. Click **Update** to save the rule. From now on, every S3 upload event will be fanned out to both SQS and Step Functions simultaneously.

![EventBridge Target Update](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/eventbridge_target_update.png)

#### 6. State Machine Initialization Results
The asynchronous orchestration process is successfully configured, creating a durable Decoupled Architecture.

![Step Functions Workflow](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_diagram.png)

{{% notice tip %}}
**System Design Note (Business Logic Separation):** Moving all branching and orchestration flows to the Step Functions tier completely isolates Business Logic from the application servers' (Workers) source code. By configuring Step Functions to invoke a Webhook, the system ensures that the AI Worker exclusively handles Video processing, while the Backend is solely responsible for aggregating AI results and updating the database.
{{% /notice %}}

***

**Next Step:** The three core components of the Ingestion Workflow tier (SQS, EventBridge, Step Functions) have been successfully initialized. We will move to section [**5.6.4: Test Ingestion Flow**](../5.6.4-test-ingestion-flow/) to perform end-to-end testing of the message routing system before moving to the Compute tier.
