---
title : "Create API Gateway"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.9.1. </b> "
---

For the Frontend application (a static SPA hosted on S3) to securely interact with the Backend API system hidden within the Private Subnets network, we need a robust and intelligent transit entry point at the public boundary.

This is the exact role of **Amazon API Gateway**. In this section, we will build a high-performance HTTP API to catch all incoming requests from the Internet.

#### Why HTTP API?
Amazon API Gateway supports various protocols (REST, HTTP, WebSocket). For the standard communication flow of the project, the **HTTP API** is preferred due to its outstanding characteristics:
- **Response Speed:** Considerably lower inherent latency compared to traditional REST APIs.
- **Cost Optimization:** Operational costs are up to 71% cheaper than REST APIs, making it highly ideal for high-load Microservices architectures.
- **Native Integration:** Flawlessly supports integration with the VPC Link feature to push traffic directly to the Application Load Balancer (ALB) inside the internal network.

#### Provisioning the HTTP API
1. Access the **API Gateway** service via the search bar on the AWS Management Console.
2. On the main dashboard, locate the block named **HTTP API** and click the **Build** button.
3. In the first configuration step (Step 1), there's no need to set up an Integration right away. Enter the API name as `CloudForge-Media-API` for easy identification.
4. Click **Next** to skip through the default Route configuration steps.
5. In the final step, review the information and click **Create** to let the system provision the API gateway.

![Create HTTP API](/images/5-Workshop/5.9-API-and-realtime/5.9.1-create-api-gateway/create_http_api.png)

#### Configuring Cross-Origin Resource Sharing (CORS)
Because our API will be invoked by a Frontend browser residing on a different Domain, the browser will automatically block these requests for security reasons (CORS error) unless we explicitly configure it.

1. On the left navigation Menu of `CloudForge-Media-API`, select the **CORS** section.
2. Click the **Configure** button and set the security parameters as follows:
   - **Access-Control-Allow-Origin:** Add `*` (Allows all origins. *Note: In actual Production, you need to specifically designate the application's domain to ensure security*).
   - **Access-Control-Allow-Methods:** Add `*` (Allows all HTTP methods like GET, POST, PUT, DELETE).
   - **Access-Control-Allow-Headers:** Add `*` (Allows passing any custom Headers).
3. Click **Save** to apply the cross-origin resolution rule set.

{{% notice tip %}}
**Operational Core:** At this moment, this HTTP API is merely an "empty door" facing the Internet. It has been allocated a valid URL address (Invoke URL) but is completely unaware of how to navigate the data stream into the system.
{{% /notice %}}

***

**Next Step:** For this API door to truly serve its purpose, we will establish a "Connecting Bridge" (VPC Link) to funnel the data stream through the virtual private network and hit the Backend infrastructure directly in the next lesson.
