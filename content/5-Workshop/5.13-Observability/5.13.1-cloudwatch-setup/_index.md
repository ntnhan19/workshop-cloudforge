---
title : "Configure Amazon CloudWatch"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.13.1. </b> "
---

This article provides detailed instructions on configuring Amazon CloudWatch to achieve Centralized Logging from the Amazon ECS Fargate container server infrastructure, alongside establishing automated Alarms when resources exhibit signs of overload.

Within an ephemeral containerized environment such as Amazon ECS Fargate, containers can be continuously provisioned, scaled up, or terminated based on load demands. Consequently, persisting logs locally directly on the container is unviable (log data is permanently obliterated upon container termination). The industry-standard solution is to configure the **awslogs** log driver to intercept the system log data stream and forward it directly to the CloudWatch Logs service.

#### Step 1: Configure CloudWatch Log Group Retention
The project team needs to establish the retention setting for the Backend application's default log group to optimize AWS storage costs, averting the default `Never expire` setting which could squander project budget.

1. Access the **Amazon CloudWatch** console.
2. From the left navigation pane, under the **Logs** section, select **Log groups**.
3. The ECS system has autonomously generated a default Log Group named `/ecs/cloudforge-backend-task`. Click on this Log Group's name.
4. Select the **Actions** tab (or directly click on the Retention value displaying `Never expire`), and choose **Edit retention setting**.
5. Select **14 days** (2 weeks) and click **Save changes**.

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

Subsequent to the application update via the CI/CD pipeline, the Backend source code executing within the container will autonomously redirect all log data externally. By navigating to the `/ecs/cloudforge-backend-task` Log Group, administrators can open the **Log streams** to monitor the entire application initialization process in Real-time.

![CloudWatch Log Stream](/images/5-Workshop/5.13-Observability/5.13.1-cloudwatch-setup/5.13.1-log-stream.png)

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

![CloudWatch CPU Alarm](/images/5-Workshop/5.13-Observability/5.13.1-cloudwatch-setup/5.13.1-cpu-alarm.png)

***

**Next Step:** The infrastructure is now equipped with centralized log monitoring capabilities and proactive hardware resource incident detection. In the subsequent article ([**Lesson 5.13.2: Integrate X-Ray Tracing**](../5.13.2-xray-tracing/)), the project team will tackle a significantly more complex challenge: configuring the AWS X-Ray tool for in-depth tracing of data traversing Serverless service layers.
