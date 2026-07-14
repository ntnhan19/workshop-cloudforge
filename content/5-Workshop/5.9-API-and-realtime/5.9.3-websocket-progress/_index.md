---
title : "Setup WebSocket"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.9.3. </b> "
---

If the HTTP API (RESTful) serves as the sturdy backbone for standard data transmission operations, then the **WebSocket** protocol is the "nervous system" relaying real-time signal streams.

Instead of forcing the Frontend to continuously send "check-in" requests to the server (Polling mechanism) - an incredibly expensive method regarding network resources that exerts massive pressure on the database, we will establish a persistent WebSocket connection. This mechanism allows the server to proactively Push notifications directly to the user's browser the exact moment the AI Worker concludes its analysis lifecycle.

#### The Hidden Power of Application Load Balancer (ALB)
During practical deployment, setting up WebSockets via API Gateway requires a complex architecture (e.g., requiring a Network Load Balancer - NLB to connect a VPC Link, or writing state management code for `@connections`).

However, the project's FastAPI Backend codebase is pre-programmed to manage Native WebSockets natively using the `fastapi.WebSocket` library and Redis PubSub. The brilliant part is that the **Application Load Balancer (ALB) we deployed in Chapter 5.7 supports routing the WebSocket protocol (ws:// and wss://) completely natively** on the same HTTP port (Port 80) without requiring any additional configuration!

#### Stateful Push Notification Mechanism
Thanks to the native support of the ALB, the real-time data flow operates incredibly smoothly and directly:
1. **Connection Setup:** The Frontend browser will not call the API Gateway, but instead open a WebSocket connection piercing directly through the ALB: `ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]`.
2. **Stateful Connection:** The ALB transparently forwards this WebSocket session to an ECS Fargate Container running FastAPI. This container will keep the connection open.
3. **Pub/Sub Broadcasting:** When the AI Worker finishes processing the Video, it reports the progress to Redis. The FastAPI Backend "listening" to Redis will immediately catch this message.
4. **Pushing Results:** The Backend pushes the JSON payload containing the `ANALYSIS_COMPLETED` status directly through the open WebSocket transmission line straight to the user's screen.

{{% notice tip %}}
**Architectural Advantage:** The solution of using Native WebSockets via ALB extremely simplifies the AWS infrastructure (completely eliminating the cumbersome API Gateway WebSocket), minimizes network latency (Network Hop), and maximizes operational cost savings for businesses.
{{% /notice %}}

***

**Next Step:** The real-time communication system is ready. We will advance to the final lesson of the chapter: [**End-to-End data flow testing**](../5.9.4-test-realtime-flow/) to confirm the seamless coordination between the HTTP API and WebSocket.
