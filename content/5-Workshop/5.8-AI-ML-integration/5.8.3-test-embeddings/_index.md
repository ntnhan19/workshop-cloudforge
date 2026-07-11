---
title : "Test Embeddings"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.8.3. </b> "
---

After successfully extracting raw text via the Amazon Transcribe service, our system faces a new data challenge: How can a computer truly comprehend the deep "semantics" of a conversation, rather than mechanically relying on traditional keyword matching?

The optimal solution for the architecture of Smart Media Analytics is the use of **Embeddings Models**. In this segment, we will document how the AI Worker communicates with Amazon Bedrock to transform natural language into mathematical Vectors.

#### How Text Embeddings Work

In the realm of Artificial Intelligence, "Embedding" is the process of encoding a piece of text into a sequence of real numbers (a multi-dimensional Vector). Texts with similar meanings will generate Vectors that are located close to each other in mathematical space.

The AI Worker application utilizes the **Amazon Titan Text Embeddings v2** model (a Serverless model automatically enabled in Section 5.8.1) to execute the following pipeline:

1. **Data Chunking:** The JSON text file containing the entire transcript returned from Amazon Transcribe is broken down into manageable short segments (Chunks). This optimizes accuracy and ensures it does not exceed Bedrock's input capacity limit (Context window).
2. **Vector Generation API Call:** The AI Worker packages these text segments and sends them to Amazon Bedrock via the `InvokeModel` API function.
3. **Receiving the Vector Array:** Bedrock calculates and responds with a data array containing thousands of floating-point values (e.g., `[0.012, -0.453, 0.887, ...]`). This serves as the distinct "semantic signature" of that specific text segment.
4. **Knowledge Storage:** This Vector array is subsequently inserted into the Vector Database (PostgreSQL paired with `pgvector`) alongside its Metadata, such as the video ID and timestamps.

![Embeddings Flow](/images/5-Workshop/5.8-AI-ML-integration/5.8.3-test-embeddings/embeddings_flow.png)

#### Infrastructure Role Permissions (IAM Role)
For the communication process with Amazon Bedrock to function smoothly without being blocked, the **ECS Task Role** of the AI Worker has been pre-granted the following authorization policy:
- `bedrock:InvokeModel` (Allows calling and executing the AI model).

{{% notice tip %}}
**Architectural Advantage:** Utilizing Bedrock's Embeddings API directly from the Amazon ECS environment within the same VPC ensures that internal system data never traverses the public Internet. This strictly adheres to the most rigorous enterprise security and compliance standards.
{{% /notice %}}

***

**Next Step:** With all Vector data successfully structured, our Background Processing flow is now considered complete. In the next chapter (**Chapter 5.9: API & Real-time**), we will return to the Backend layer to configure real-time communication flows, instantly notifying the user as soon as the AI Worker finishes its analysis!
