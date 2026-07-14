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

**Step 1: Trigger the Event Pipeline and Retrieve Job ID**
In the standard architecture, the WebSocket route strictly requires a valid UUID. Therefore, we will trigger the processing pipeline via the REST API so the Backend initiates a Job and allocates a UUID (instead of uploading directly to S3 which does not return an ID).

Use cURL (or PowerShell) to call the Ingest API, replacing `[ALB-DNS]` with your actual ALB DNS:
```bash
curl -X POST http://[ALB-DNS]/api/v1/ingest \
  -H "Content-Type: application/json" \
  -d '{"source_path": "s3://cloudforge-media-upload-bucket/sample-video.mp4"}'
```
*The returned response will contain a `job_id` (e.g., `14492409-f583-41ac-89c4-c66a9351919c`).*

![Trigger REST API](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/trigger_api.png)

**Step 2: Establish the Listening Connection**
Use `wscat` to open a WebSocket connection piercing directly through the Application Load Balancer (ALB). Replace `[ALB-DNS]` and `[job_id]` with the exact UUID string you received in Step 1.
```bash
wscat -c ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]
```
*System state: The Terminal transitions to a Connected state. The Backend Container (FastAPI) natively records and maintains this connection session.*

![Websocket Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/websocket_test.png)

**Step 3: Validate the Real-time Data Flow**
In the Terminal running `wscat`, the system will output JSON payloads as soon as the AI Worker's status changes, culminating in the completion payload (actual processing time depends on the Video size).

The returned result requires no Polling action from the Client:

```json
< {
    "event": "processing",
    "job_id": "a99b7166-7c59-4787-9120-a4b12bf9ac3e",
    "asset_id": null,
    "status": "processing",
    "progress": 0.0,
    "assets_queued": 0,
    "assets_processed": 0,
    "error_message": null
  }
```

![Realtime Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/realtime_test.png)

{{% notice tip %}}
**Architectural Evaluation:** The test results confirm that the Event-Driven architecture and Real-time API are functioning as designed. The processing occurs completely asynchronously, eliminating bottlenecks or wasted connection resources caused by traditional Polling mechanisms.
{{% /notice %}}

***

**Next Step:** The API communication system and background data processing have been validated. In the next chapter ([**Chapter 5.10: Semantic Search**](../../5.10-Semantic-search/)), the project team will exploit the Vector Embeddings data repository to establish the semantic search engine.
