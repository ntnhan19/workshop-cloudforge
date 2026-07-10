---
title : "API & Real-time"
date : 2026-07-10
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### API & Real-time Communication Overview

In queue-based asynchronous data processing architectures, a classic challenge always arises: How can the Frontend (user's browser application) know exactly when a heavy background task (like AI analysis) is complete? Having the Frontend continuously send "check-in" requests to the server (Polling) exhausts network resources and wastefully overloads the system.

This chapter completely resolves the Real-time communication and API Management routing problem to deliver the most seamless user experience.

#### Core Benefits of API & Real-time Architecture:
- **Seamless UX:** The WebSocket mechanism allows the server to proactively Push notifications directly to the user's browser the instant the AI Worker finishes processing, with near-zero latency.
- **System Protection (Throttling & Security):** API Gateway acts as a "gatekeeper," hiding the entire underlying internal infrastructure (ALB, ECS) while providing traffic limiting capabilities to mitigate DDoS attacks or API abuse.
- **Layer Decoupling:** The Frontend only needs to communicate with a single Endpoint of the API Gateway, oblivious to the complexity of the Microservices network behind it.

#### API & Real-time Layer Implementation Roadmap:
1. **Amazon API Gateway (HTTP API):** Provision a public communication gateway to catch traffic from the Internet.
2. **Application Load Balancer (ALB) Integration:** Configure a private network link (VPC Link) to securely route traffic from the API Gateway into the Backend system residing deep within the Private Subnet.
3. **Amazon API Gateway (WebSocket):** Establish a persistent two-way network connection strictly serving the Progress Tracking notification flow.
4. **Test Real-time Flow:** Validate the end-to-end cycle from Video upload $\rightarrow$ AI Processing $\rightarrow$ Push notification via WebSocket to the Frontend.

With this chapter, Smart Media Analytics is not only intelligent in its machine learning layer but also profoundly elegant and optimized in how it interacts with the end-user!

### Hands-on Content

- [Create API Gateway](5.9.1-create-api-gateway/)
- [Integrate ALB](5.9.2-create-alb/)
- [Setup WebSocket](5.9.3-websocket-progress/)
- [Test Real-time Flow](5.9.4-test-realtime-flow/)
