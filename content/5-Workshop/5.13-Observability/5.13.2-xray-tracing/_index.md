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

#### Step 2: Incorporate the X-Ray Daemon Sidecar into Task Definition
On the ECS Fargate environment, the industry-standard methodology for running X-Ray is the **Sidecar** pattern: Executing a secondary container (Daemon) concurrently with the primary container (Backend) within the same Task. This Daemon will listen for, harvest Trace data packets from the application, and propel them to AWS via the UDP protocol.

Access the **Amazon ECS Console** -> **Task Definitions**, opt to create a new revision, and append a secondary container definition block:

```json
{
    "name": "xray-daemon",
    "image": "public.ecr.aws/xray/aws-xray-daemon:latest",
    "cpu": 32,
    "memoryReservation": 256,
    "portMappings": [
        {
            "containerPort": 2000,
            "protocol": "udp"
        }
    ]
}
```

*Note: Opening UDP port 2000 is mandatory to permit the Backend container to communicate internally with the X-Ray Daemon Container.*

#### Step 3: Integrate X-Ray SDK into Application Source Code (FastAPI)
Once the infrastructure is primed to receive data, the concluding step is to declare the library within the Backend source code (Python/FastAPI) to enable the system to autonomously generate Trace packets for every Request.

1. Install the AWS X-Ray SDK library via the project's `requirements.txt` file:

```text
aws-xray-sdk==2.12.0
```

2. Incorporate the X-Ray Middleware into the server initialization file (e.g., `main.py`). This Middleware will automatically measure the processing time of all API Endpoints:

```python
from fastapi import FastAPI
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.starlette.middleware import XRayMiddleware

app = FastAPI()

# Configure the X-Ray Recorder and define the service nomenclature on the Service Map
xray_recorder.configure(service='SmartMedia-BackendAPI')

# Integrate the measurement Middleware into FastAPI
app.add_middleware(XRayMiddleware, app=app, recorder=xray_recorder)

# --- Declare your business logic Routes here ---
@app.get("/api/health")
def health_check():
    return {"status": "Healthy"}
```

#### Step 4: Observe the Service Map and Traces
Following a source code commit and allowing the CI/CD pipeline (GitHub Actions) to autonomously deploy the novel iteration to ECS, try dispatching several Requests from a browser/Postman to the system's API.

Navigate to the **AWS X-Ray Console** -> **Service map**. You will observe an intuitive visual network graph simulating the data trajectory (Client invoking the Backend, Backend querying PostgreSQL).

Transition to the **Traces** tab; administrators can click on individual Requests to scrutinize a Gantt chart analyzing the precise execution duration of each sub-task measured in milliseconds. This facilitates effortless isolation of sluggish SQL queries or processing logic consuming excessive CPU.

![AWS X-Ray Service Map](/images/5-Workshop/5.13-observability/5.13.2-xray-service-map.png)
*(Screenshot Guide: Capture the Service Map screen on the AWS Console displaying the visual communication nodes from Client -> Backend -> Database).*

***

**Chapter 5.13 Summary:** Furnished with the synergistic combination of Amazon CloudWatch and AWS X-Ray, the Smart Media Analytics system has attained the zenith of operational maturity (Operational Excellence). Administrators now wield total dominion over centralized logging, automated alerts during overload, and intuitive tracing of every distributed data flow.

**Next Step:** Congratulations! You have superlatively finalized the construction of the entire core architecture for the CloudForge project. In **Chapter 5.14: Resource Cleanup**, we will execute the infrastructure decommissioning procedures to avert unintended incidental AWS expenditures following the Workshop's conclusion.
