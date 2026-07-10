---
title : "Setup WebSocket"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.9.3. </b> "
---

If the HTTP API (RESTful) serves as the sturdy backbone for standard data transmission operations, then the **WebSocket** protocol is the "nervous system" relaying real-time signal streams.

Instead of forcing the Frontend to continuously send "check-in" requests to the server (Polling mechanism) - an incredibly expensive method regarding network resources that exerts massive pressure on the database, we will establish a persistent WebSocket connection. This mechanism allows the server to proactively Push notifications directly to the user's browser the exact moment the AI Worker concludes its analysis lifecycle.

#### Structure of WebSocket API
Unlike the traditional HTTP protocol (Stateless), a WebSocket connection maintains a Bi-directional state. AWS API Gateway manages WebSockets via core routes:
- **`$connect`**: Triggered when the browser opens a connection. This is when the connection is established, keeping the user on the real-time "transmission line."
- **`$disconnect`**: Triggered when the user closes the browser or loses connectivity, helping the system clean up old sessions.

#### Provisioning and Optimal Configuration
During practical deployment, the project confronts an AWS architectural characteristic: WebSocket API requires a Network Load Balancer (NLB) to integrate deeply into the internal network via VPC Link. To optimize infrastructure costs and simplify the operational flow for the Workshop while ensuring Real-time functionality, the project opts for the **Mock Integration** solution.

1. Access the **API Gateway** service, click **Build** in the **WebSocket API** block.
2. **Basic Setup:**
   - **API name:** `CloudForge-Media-WS`.
   - **Route selection expression:** `$request.body.action` (Used to route messages from the user based on the "action" field in the JSON payload).
3. **Route Configuration:** Add the two default routes: `$connect` and `$disconnect`.
4. **Integration Setup:**
   - For both routes, select the **Integration type** as **Mock**. 
   - Using Mock Integration acts as a swift "transit station," instantly returning a success response (HTTP 200) so the browser establishes the connection immediately without waiting for complex backend processing.
5. **Deployment:** Create a Stage named `production` and proceed to **Create and deploy**.

*Illustration: The route structure and Mock Integration diagram of the WebSocket API.*
![WebSocket Routes](../../../../images/5-Workshop/5.9-API-and-realtime/5.9.3-websocket-progress/websocket_routes.png)

#### Push Notification Mechanism
The subtlety of this architecture lies here: Although the entry port is "Mock", the real-time data flow operates flawlessly:
- When the AI Worker finishes processing, the information is written to the Database. 
- The Backend API utilizes IAM permissions to call directly into the API Gateway via the connection management Endpoint (**@connections API**). 
- API Gateway relies on the `connectionId` that is kept open to push the resulting JSON payload straight down to the user's screen.

{{% notice tip %}}
**Architectural Advantage:** This solution ensures the user experience (UX) is maximized with instant notifications, while keeping the AWS infrastructure extremely lean, generating no extra costs for unnecessary additional load balancers.
{{% /notice %}}

***

**Next Step:** The real-time communication system is ready. We will advance to the final lesson of the chapter: **End-to-End data flow testing** to confirm the seamless coordination between the HTTP API and WebSocket.
