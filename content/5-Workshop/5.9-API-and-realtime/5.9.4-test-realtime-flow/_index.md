---
title : "Test Real-time Flow"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.9.4. </b> "
---

After completing the setup of the distributed components (S3, SQS, ECS Fargate, AI Services, and API Gateway), the project team proceeds to test the End-to-End Data Flow.

The objective of this testing phase is to verify whether the WebSocket connection can receive status notifications from the Backend the precise moment the AI Worker completes the Video processing pipeline.

#### Testing Tools
To interact directly with the WebSocket protocol from the Command Line Interface (CLI), the project utilizes the **`wscat`** tool.
Install the tool via NPM:
```bash
npm install -g wscat
```

#### End-to-End (E2E) Testing Scenario

**Step 1: Establish the Listening Connection**
Use `wscat` to open a connection piercing directly through the Backend's Application Load Balancer (ALB). Replace `[ALB-DNS]` with the actual DNS Name of your ALB, and `[job_id]` with a random identifier string (e.g., `test-job-123`).
```bash
wscat -c ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]
```
*System state: The Terminal transitions to a Connected state. The Backend Container (FastAPI) natively records and maintains this connection session.*

**Step 2: Trigger the Event Pipeline**
Use the AWS CLI to upload a sample Video file into the S3 Bucket (configured in Chapter 5.6).
```bash
aws s3 cp sample-video.mp4 s3://cloudforge-media-upload-bucket/
```
*System state: The completed upload operation triggers EventBridge, sending a Message to SQS, which subsequently triggers the AI Worker on ECS Fargate to invoke Transcribe and Bedrock services.*

**Step 3: Validate the Real-time Data Flow**
In the Terminal running `wscat`, the system will output a JSON payload as soon as the AI Worker concludes its processing lifecycle (actual processing time depends on the Video size). 

The returned result requires no Polling action from the Client:

```json
< {
    "action": "ANALYSIS_COMPLETED",
    "videoId": "sample-video.mp4",
    "status": "SUCCESS",
    "summary": "The video discusses Cloud Native solutions on AWS...",
    "timestamp": "2026-07-10T10:00:00Z"
  }
```

![Realtime Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/realtime_test.png)

{{% notice tip %}}
**Architectural Evaluation:** The test results confirm that the Event-Driven architecture and Real-time API are functioning as designed. The processing occurs completely asynchronously, eliminating bottlenecks or wasted connection resources caused by traditional Polling mechanisms.
{{% /notice %}}

***

**Next Step:** The API communication system and background data processing have been validated. In the next chapter (**Chapter 5.10: Semantic Search**), the project team will exploit the Vector Embeddings data repository to establish the semantic search engine.
