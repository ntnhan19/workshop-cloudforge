---
title : "Workflow Orchestration with Step Functions"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.6.3. </b> "
---

After setting up the event routing system (EventBridge) and the message queue (SQS), the next step is to build the orchestration layer for complex processing workflows. **AWS Step Functions** acts as a central Orchestrator, managing State Machines to visually control the Media processing workflow with high durability and fault tolerance.

#### 1. Initialize and Configure State Machine Properties
Access the **AWS Step Functions** service → **State machines** → select **Create state machine**. Configure the foundational parameters in the new initialization interface:

- **Template:** Choose `Create from blank` to define the processing flow from scratch.
- **State machine name:** Name the rule `cloudforge-media-workflow`.
- **State machine type:** Select `Standard` (Ensuring a maximum execution time of up to 1 year and supporting detailed execution history storage, perfectly suited for long-term Media analytics tasks).
- **State machine query language:** The system defaults to **JSONata** (A new query standard that optimizes direct data transformation during state transitions without relying on intermediate source code).

![Step Functions Config](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_config.png)

Click **Continue** to move to the visual design interface.

#### 2. Build the Orchestration Diagram (Workflow Studio v2)
In the Workflow Studio Canvas space, drag and drop logic state blocks to establish the Skeleton Architecture for the processing flow:

1. **Initial State (Pass State - Start Processing):** Drag a `Pass` block and drop it right below the `Start` point, rename it to `Start Processing`.
2. **Branching State (Choice State - Check Status):** Drag a `Choice` block and drop it right below the initial processing block, rename it to `Check Status`.
   - *Configure branching condition (JSONata):* Under Rule #1, set the Condition to `true` (the system will display it as `{% true %}`). This is a mandatory placeholder value to complete the branching routing skeleton before integrating actual data from the Worker servers.
3. **Successful Processing Branch (Pass State - Update Metadata):** In the `Rule #1` branch of the Choice block, drag a `Pass` block and configure its name as `Update Metadata` (Defining the task to record the result data into the Database).
4. **Failed Processing Branch (Pass State - Notify Error):** In the remaining `Default` branch, drag a `Pass` block and configure its name as `Notify Error` (Defining the task to trigger the error alerting system).

![Workflow Studio Setup](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/workflow_studio_setup.png)

#### 3. Execute Infrastructure Deployment
Once the inverted Y-shaped diagram accurately completes the orchestration logic, perform the saving steps:
- Move to the top right corner and click the **Create** button.
- **Confirm IAM Role:** The system will automatically generate a dedicated IAM Role to grant permissions allowing Step Functions to securely interact with related AWS resources. Click **Confirm** to complete.

#### 4. State Machine Initialization Results
The asynchronous orchestration process is successfully configured, creating a durable Decoupled Architecture.

![Step Functions Workflow](/images/5-Workshop/5.6-Ingestion-workflow/5.6.3-create-step-functions/step_functions_diagram.png)

{{% notice tip %}}
**System Design Note (Business Logic Separation):** Moving all branching and orchestration flows to the Step Functions tier completely isolates Business Logic from the application servers' (Workers) source code. When the system needs additional features (e.g., adding a video compression step or sending an SMS), engineers only need to reconfigure the diagram on Step Functions without having to recompile or redeploy the source code of the entire processing server cluster.
{{% /notice %}}

***

**Next Step:** The three core components of the Ingestion Workflow tier (SQS, EventBridge, Step Functions) have been successfully initialized. We will move to section **5.6.4: Test Ingestion Flow** to perform end-to-end testing of the message routing system before moving to the Compute tier.
