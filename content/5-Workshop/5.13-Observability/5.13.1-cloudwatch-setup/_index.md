---
title : "Configure Amazon CloudWatch"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.13.1. </b> "
---

This article provides detailed instructions on configuring Amazon CloudWatch to achieve Centralized Logging from the Amazon ECS Fargate container server infrastructure, alongside establishing automated Alarms when resources exhibit signs of overload.

Within an ephemeral containerized environment such as Amazon ECS Fargate, containers can be continuously provisioned, scaled up, or terminated based on load demands. Consequently, persisting logs locally directly on the container is unviable (log data is permanently obliterated upon container termination). The industry-standard solution is to configure the **awslogs** log driver to intercept the system log data stream and forward it directly to the CloudWatch Logs service.

#### Step 1: Provision a CloudWatch Log Group
The project team provisions a dedicated Log Group to categorize and aggregate the entirety of the Backend application's logs into a singular storage space.

1. Access the **Amazon CloudWatch** console.
2. From the left navigation pane, under the **Logs** section, select **Log groups**.
3. Click the **Create log group** button located at the top right corner.
4. Supply the configuration parameters:
   - **Log group name:** `/ecs/cloudforge-backend`
   - **Retention setting:** Select **14 days** (A 2-week log retention period to optimize AWS storage costs, averting the default `Never expire` setting which could squander project budget).
5. Click **Create** to finalize.

*Illustration: Successful provisioning of the centralized log storage space on Amazon CloudWatch.*
![Create Log Group](/images/5-Workshop/5.13-observability/5.13.1-create-log-group.png)
*(Screenshot Guide: Capture the Log groups list screen clearly displaying the /ecs/cloudforge-backend record with the corresponding Retention column indicating 14 days).*

#### Step 2: Augment ECS Task Definition with the awslogs driver
For the ECS server system to discern the destination for log forwarding, the log driver configuration must be tightly integrated within the Task Definition blueprint.

The `logConfiguration` block is explicitly declared within the JSON format of the Task Definition as follows:

```json
"logConfiguration": {
    "logDriver": "awslogs",
    "options": {
        "awslogs-group": "/ecs/cloudforge-backend",
        "awslogs-region": "ap-southeast-1",
        "awslogs-stream-prefix": "ecs"
    }
}
```

Subsequent to the application update via the CI/CD pipeline, the Backend source code executing within the container will autonomously redirect all log data externally. By navigating to the `/ecs/cloudforge-backend` Log Group, administrators can open the **Log streams** to monitor the entire application initialization process in Real-time.

*Illustration: Backend system log data stream displayed in detail on the CloudWatch Log Stream interface.*
![CloudWatch Log Stream](/images/5-Workshop/5.13-observability/5.13.1-log-stream.png)
*(Screenshot Guide: Click into the /ecs/cloudforge-backend Log group, select the most recent log stream, and capture the screen displaying the application's running logs, such as server initialization and Database connections).*

#### Step 3: Establish a CloudWatch Alarm for CPU Alerts
To proactively mitigate network congestion or system overload scenarios before the service crashes, the project team establishes an alerting system (Alarm) that measures the CPU consumption of the ECS Service.

1. From the left navigation pane of the CloudWatch Console, under the **Alarms** section, select **All alarms**.
2. Click **Create alarm** -> Select **Select metric**.
3. Choose the **ECS** category -> **ClusterName, ServiceName**.
4. Locate and select the precise **CPUUtilization** metric corresponding to the `cloudforge-backend-service` operating within the `cloudforge-compute-cluster`. Click **Select metric**.
5. Formulate the alert activation Conditions:
   - **Statistic:** `Average`
   - **Period:** `1 minute` (Granular measurement by the minute).
   - **Threshold type:** `Static`
   - **Whenever CPUUtilization is:** Select `Greater/Equal` and input the value `80` (Triggers the alert state when CPU reaches or exceeds 80%).
6. Click **Next**. At the Configure actions phase, opt to send a notification to a dedicated Amazon SNS Topic (e.g., `DevOps-Alerts`) to autonomously forward alert emails to the operations team's inbox.
7. Designate the Alarm name as `ECS-High-CPU-Alert` and click **Create alarm**.

*Illustration: Configuration of the resource threshold alert on CloudWatch.*
![CloudWatch CPU Alarm](/images/5-Workshop/5.13-observability/5.13.1-cpu-alarm.png)
*(Screenshot Guide: Capture the interface of the condition setup step for the CloudWatch Alarm, displaying the measurement graph accompanied by the red dashed horizontal line representing the Threshold at the 80% mark).*

***

**Next Step:** The infrastructure is now equipped with centralized log monitoring capabilities and proactive hardware resource incident detection. In the subsequent article (**Lesson 5.13.2: Integrate X-Ray Tracing**), the project team will tackle a significantly more complex challenge: configuring the AWS X-Ray tool for in-depth tracing of data traversing Serverless service layers.
