---
title : "Ingestion Workflow"
date : 2026-07-10
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Ingestion Workflow Overview

In the previous chapters, we completed the storage, database, and foundational security layers. In this Chapter 5.6, the system will be configured with an **Event-driven Architecture** to loosely couple service components (decoupled) while ensuring high durability and scalability.

Given the nature of the Smart Media Analytics system processing massive volumes of multimedia files, AI analysis tasks (image recognition, audio transcription, metadata extraction) require significant computational time. If relying on a traditional Synchronous API call architecture, the system would easily face bottlenecks or timeouts. Therefore, transitioning to an Asynchronous message-based processing model is mandatory.

In this chapter, we will focus on deploying the core orchestration components:
1. **Amazon SQS (Simple Queue Service):** Acts as a buffer and queue containing video/audio processing tasks, safely distributing work to AI Workers without the risk of data loss during traffic spikes.
2. **Amazon EventBridge:** The Event Bus, responsible for listening to state changes (e.g., successful file upload to S3, or AI processing completion/failure) to automatically trigger subsequent processing workflows.
3. **AWS Step Functions (Optional):** A State Machine that orchestrates complex processing workflows, multi-condition branching, and provides visual monitoring of a Media task's entire lifecycle.

*(Technical deployment details for each service will be presented in the following subsections).*
