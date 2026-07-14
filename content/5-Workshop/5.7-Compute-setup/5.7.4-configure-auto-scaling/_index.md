---
title : "Configure Auto Scaling"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.7.4. </b> "
---

One of the greatest advantages of cloud architecture and AWS Fargate is the ability to automatically scale (Auto Scaling). The system can automatically increase the number of Containers when traffic or workload spikes unexpectedly, and automatically scale-in when the system is idle to optimize costs.

In this segment, we will set up Auto Scaling Policies for both service tiers: **Backend API** and **AI Worker**.

#### 1. Configure Auto Scaling for Backend API
User traffic often fluctuates in real-time. Configuring Auto Scaling based on CPU Utilization is the most common and effective method for the Backend.

1. Access the **Amazon ECS** service, select the `cloudforge-compute-cluster` Cluster.
2. In the **Services** tab, check the `cloudforge-backend-service` service and click the **Update** button.
3. Scroll down to the **Service auto scaling** section and check **Use service auto scaling**.
4. Configure **Task capacity** with Minimum number of tasks as `2` and Maximum number of tasks as `10`.
5. In the **Scaling policies** section, select Policy type as **Target tracking** and set the Policy name to `backend-cpu-scaling-policy`.
6. Set the **ECS service metric** to **ECSServiceAverageCPUUtilization** with a Target value of `70` (When the average CPU exceeds 70%, the system automatically creates additional Containers).
7. Enter `60` for the Scale-out cooldown period and `300` for the Scale-in cooldown period.

![Backend Scaling Config](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/backend_scaling_config.png)

8. Scroll to the bottom of the page and click **Update**.

![Backend Auto Scaling](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/backend_auto_scaling.png)

#### 2. Configure Auto Scaling for AI Worker
The AI Worker's workload depends entirely on the number of messages backlogged in the queue. Within the scope of this Workshop, we will use CPU Utilization to get familiar with the basic Auto Scaling mechanism.

1. Return to the **Services** list of the `cloudforge-compute-cluster` Cluster.
2. Check the `cloudforge-ai-worker-service` service and click the **Update** button.
3. Check **Use service auto scaling** in the Service auto scaling section.
4. Configure **Task capacity** with Minimum number of tasks as `1` and Maximum number of tasks as `5`.
5. In the **Scaling policies** section, select Policy type as **Target tracking** and set the Policy name to `worker-cpu-scaling-policy`.
6. Set the **ECS service metric** to **ECSServiceAverageCPUUtilization** with a Target value of `75`.
7. Enter `60` for the Scale-out cooldown period and `300` for the Scale-in cooldown period.

![Worker Scaling Config](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/worker_scaling_config.png)

8. Scroll to the bottom of the page and click **Update**.

![Worker Auto Scaling](/images/5-Workshop/5.7-Compute-setup/5.7.4-configure-auto-scaling/worker_auto_scaling.png)

{{% notice tip %}}
**Best Practice for Background Worker:** In large-scale Production systems, scaling AI Workers should be based on the **`ApproximateNumberOfMessagesVisible`** metric of Amazon SQS via a CloudWatch Custom Metric. This ensures the system reacts accurately to the backlogged workload (Queue Depth) rather than mere hardware metrics.
{{% /notice %}}

***

**Next Step:** The Compute infrastructure system is complete with smart, automatic scaling capabilities. You can be proud that the CloudForge application is now capable of handling high loads with a true Serverless architecture. Let's move on to the next chapter to [**Integrate AI/ML Services**](../../5.8-AI-ML-integration/) into the system!
