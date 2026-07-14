---
title : "Integrate X-Ray Tracing"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.13.2. </b> "
---

Within a distributed architecture (Microservices/Serverless), when a user invokes an API, the data flow may traverse a myriad of services: API Gateway -> Load Balancer -> ECS Container -> Database (Aurora/DynamoDB). Should the API respond sluggishly or return a 500 error, relying solely on text logs in CloudWatch (Lesson 5.13.1) is insufficient to accurately pinpoint the specific service inducing the Bottleneck.

AWS's optimal solution to this challenge is employing **AWS X-Ray** for Distributed Tracing.

This article provides guidance on configuring the X-Ray Daemon to run as a Sidecar pattern on Amazon ECS, alongside integrating the X-Ray SDK libraries into the Backend application source code.

#### Step 1: Augment Permissions for the ECS Task Role
For the containers within the ECS Task to legitimately transmit trace data back to the AWS X-Ray service, they must be provisioned with the corresponding IAM permissions.

1. Access the **AWS IAM Console** -> **Roles**.
2. Locate and open the Role currently assigned as the **Task Role** for the Backend service (Note: This is the Task Role, not the Task Execution Role).
3. Select **Add permissions** -> **Attach policies**.
4. Search for and check the **`AWSXRayDaemonWriteAccess`** policy.
5. Click **Add permissions**.

![IAM Task Role](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/iam_task_role.png)

#### Step 2: Incorporate the X-Ray Daemon Sidecar into Task Definition
On the ECS Fargate environment, the industry-standard methodology for running X-Ray is the **Sidecar** pattern: Executing a secondary container (Daemon) concurrently with the primary container (Backend) within the same Task. This Daemon will listen for, harvest Trace data packets from the application, and propel them to AWS via the UDP protocol.

Access the **Amazon ECS Console** -> **Task Definitions**, select the Task Definition of your Backend API, and click **Create new revision** -> **Create new revision**.

In the new Revision creation interface, retain the configurations for Container 1 (Backend API) and Task size. Scroll down and click **Add more containers** to add the secondary container with the following parameters:

*   **Name**: `xray-daemon`
*   **Image URI**: `public.ecr.aws/xray/aws-xray-daemon:latest`
*   **Essential container**: Yes

![X-Ray Container Setup](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_container_setup.png)

**Port mappings:**
*   **Container port**: `2000`
*   **Protocol**: `UDP` *(Choosing UDP is mandatory to permit the Backend container to communicate internally with the X-Ray Daemon)*.

**Resource allocation limits:**
*   **Memory hard limit**: `256` *(Allocate 256 MB RAM for the Daemon to operate)*.

![X-Ray Port & Resource Setup](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_port_resource.png)

**Log collection:**
*   Check **Use log collection** to push the X-Ray Daemon logs to CloudWatch, simplifying debugging if the Daemon encounters startup issues.

![X-Ray Log Collection](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/xray_log_collection.png)

Once all information is filled out, scroll to the bottom of the page and click **Create**. ECS will generate a new Task Definition revision (containing 2 containers). Next, simply update your ECS Service to deploy this new Revision.

#### Step 3: Integrate X-Ray SDK into Application Source Code (FastAPI)
Once the infrastructure is primed to receive data, the concluding step is to declare the library within the Backend source code (Python/FastAPI) to enable the system to autonomously generate Trace packets for every Request.

1. Install the AWS X-Ray SDK library via the project's `requirements.txt` file:

```text
aws-xray-sdk==2.12.0
```

2. Incorporate a custom X-Ray Middleware into the server initialization file (e.g., `main.py`). This Middleware will automatically measure the processing time of each API Endpoint:

```python
from fastapi import FastAPI
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core.async_context import AsyncContext
from starlette.types import ASGIApp, Receive, Scope, Send

app = FastAPI()

# Configure the X-Ray Recorder and define the service nomenclature on the Service Map
xray_recorder.configure(service='SmartMedia-BackendAPI', context=AsyncContext())

class CustomXRayMiddleware:
    def __init__(self, app: ASGIApp):
        self.app = app

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            return await self.app(scope, receive, send)
            
        segment = xray_recorder.begin_segment('SmartMedia-BackendAPI')
        try:
            segment.put_http_meta('url', scope.get('path'))
            segment.put_http_meta('method', scope.get('method'))
            await self.app(scope, receive, send)
        except Exception as e:
            segment.add_exception(e)
            raise
        finally:
            xray_recorder.end_segment()

# Integrate the measurement Middleware into FastAPI
app.add_middleware(CustomXRayMiddleware)
```

#### Step 4: Observe the Trace Map and Trace details
After the ECS Service has been successfully updated, try dispatching several Requests (for instance, invoking the `/health` or `/search` API) via Postman or a browser.

Navigate to the **AWS CloudWatch Console**, look at the left-hand menu under **Application Signals (APM)**, and select **Trace Map** (or Application Map). You will observe an intuitive visual network graph simulating the data trajectory (Client invoking the Backend system).

Proceed to select the **Traces** section in the left menu. Administrators can click on individual Requests to scrutinize an analytical chart detailing the precise execution duration of each sub-task measured in milliseconds. This facilitates effortless isolation of sluggish SQL queries or processing logic consuming excessive resources.

![AWS X-Ray Service Map](/images/5-Workshop/5.13-Observability/5.13.2-xray-tracing/5.13.2-xray-service-map.png)

***

**Chapter 5.13 Summary:** Furnished with the synergistic combination of Amazon CloudWatch and AWS X-Ray, the Smart Media Analytics system has attained the zenith of operational maturity (Operational Excellence). Administrators now wield total dominion over centralized logging, automated alerts during overload, and intuitive tracing of every distributed data flow.

**Next Step:** With the core architecture fully constructed and observable, the final validation phase can be executed. [**Chapter 5.14: System Walkthrough (Test the Application)**](../../5.14-User-guide/) demonstrates how end-users interact with the system, test media ingestion, and experience semantic search in real-time.
