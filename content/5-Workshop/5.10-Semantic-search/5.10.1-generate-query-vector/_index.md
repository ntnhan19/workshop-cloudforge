---
title : "Generate Query Vector"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.10.1. </b> "
---

For the Vector database to perform comparisons and searches, the user's natural language query must inevitably be transformed into the identical mathematical space as the stored data. The initial step in the Semantic Search lifecycle is the Query Embeddings generation process.

#### Query Processing Flow Architecture
When a user submits a search phrase on the Frontend, the data processing flow is designed as follows:

1. The Frontend issues an HTTP POST request containing the search string (e.g., `{"query": "cloud data security methods"}`) to the API Gateway.
2. The API Gateway routes the request via VPC Link to the Application Load Balancer (ALB).
3. The ALB distributes the request to an API Pod (running on ECS Fargate) designated for handling search tasks (Search Service).
4. The API Pod receives the search string and initiates a call to the Amazon Bedrock service.

![Query Flow](/images/5-Workshop/5.10-Semantic-search/5.10.1-generate-query-vector/5.10.1-query-flow.png)

#### Amazon Bedrock Interaction Setup
The project team configures the API Pod using the AWS SDK (`boto3`) to interact with Amazon Bedrock. Consistent with the Ingestion phase, the designated model is **Amazon Titan Text Embeddings v2** (`amazon.titan-embed-text-v2:0`).

Ensuring model consistency is a pivotal technical requirement: **The query's Vector must be generated from the exact same Model and possess the same number of Dimensions as the original data's Vectors.** Any discrepancy in the model would result in the two Vectors existing in entirely disparate mathematical spaces, rendering the comparison algorithm meaningless.

{{% notice tip %}}
**Technical Configuration:** In this Workshop, the Titan v2 model is configured to output the input text sequence with a fixed dimension of **1536 dimensions** (ensuring a 100% match with the `vector(1536)` data type in the PostgreSQL table structure).
{{% /notice %}}

Below is the Core Logic processing code snippet deployed on the Search Service to interact with the Bedrock API:

```python
import json
import boto3

def generate_query_vector(query_text: str):
    # Initialize Bedrock Runtime Client within the internal VPC network
    bedrock_client = boto3.client(service_name="bedrock-runtime", region_name="ap-southeast-1")
    
    # Configure the Body according to Amazon Titan Text Embeddings v2 specifications
    body = json.dumps({
        "inputText": query_text,
        "dimensions": 1536,
        "normalize": True
    })
    
    # Execute the command to the ML model
    response = bedrock_client.invoke_model(
        body=body,
        modelId="amazon.titan-embed-text-v2:0",
        accept="application/json",
        contentType="application/json"
    )
    
    response_body = json.loads(response.get("body").read())
    
    # Return the float array representing the Query Vector
    return response_body.get("embedding")
```

#### IAM Role Assignments
During this phase, the ECS Task Role of the Search Service (Backend API) is provisioned with the following authorization policy:
- `bedrock:InvokeModel`: Grants the API Pod permission to invoke the Titan model's Embeddings generation function.

*Technical note: Thanks to the internal VPC network architecture, the user's query data is transmitted directly from the Backend API to the Amazon Bedrock Endpoint via a VPC Endpoint without traversing the public Internet, satisfying information security standards.*

***

**Next Step:** Once the API Pod receives the Vector array returned from Amazon Bedrock, this mathematical sequence will be utilized as the Input Parameter for the PostgreSQL database query in lesson 5.10.2.
