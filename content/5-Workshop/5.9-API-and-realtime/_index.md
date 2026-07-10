---
title : "API & Real-time"
date : 2026-07-10
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

### API & Real-time Overview

With an asynchronous data processing architecture, how does the Frontend know when a video has finished being processed by AI? Having the Frontend continuously poll the API would overload the system.

This chapter solves the real-time communication and API management problem:
1. **Amazon API Gateway (WebSocket):** Establish persistent two-way connections. Once the AI Worker finishes processing, it can actively push notifications directly to the user's browser.
2. **API Gateway (REST API):** Wrap the Application Load Balancer endpoints to provide rate limiting (Throttling) and API versioning.

*(Detailed content will be updated in the sub-articles).*
