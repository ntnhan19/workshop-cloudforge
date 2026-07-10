---
title : "Ingestion Workflow"
date : 2026-07-10
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Ingestion Workflow Overview

In previous chapters, we completed the storage, database, and security layers. In this Chapter 5.6, we will set up the **messaging and event orchestration** system to loosely couple the components in a highly durable, event-driven manner.

The Smart Media Analytics system must process large volumes of multimedia files. AI processing takes time, so synchronous API calls are not viable. Instead, we use an asynchronous architecture based on queues.

In this chapter, we will focus on:
1. **Amazon SQS (Simple Queue Service):** A message queue holding video/audio processing tasks waiting for AI Workers to consume.
2. **Amazon EventBridge:** An Event Bus that routes state events (success/failure) and triggers subsequent workflows.
3. **AWS Step Functions:** (Optional) Visually orchestrates complex, multi-step processing workflows.

*(Detailed setup steps will be updated in the sub-articles of this section).*
