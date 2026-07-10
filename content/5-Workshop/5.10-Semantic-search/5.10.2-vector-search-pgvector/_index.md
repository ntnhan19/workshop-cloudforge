---
title : "Execute Vector Search with pgvector"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.10.2. </b> "
---

Following the successful transformation of the user's query into a Vector array (Query Embeddings) via Amazon Bedrock in lesson 5.10.1, the subsequent architectural step involves executing the semantic search query directly on the Amazon RDS PostgreSQL Database.

#### Vector Similarity Mechanism
The project utilizes the `pgvector` extension, pre-activated on PostgreSQL, to support the storage and querying of the `vector(1536)` data type.
To measure the degree of similarity between the query and the Transcript segments, the architecture employs the **Cosine Distance** operator (Denoted in pgvector as `<=>`). A smaller Cosine Distance signifies that the two text segments share a closer contextual meaning.

#### SQL Query Structure
The project team formulates an advanced SQL query on the Backend API to calculate and extract the highly correlated (Top-K) results.

Below is the core Raw SQL command structure:
```sql
SELECT 
    video_id, 
    transcript_text, 
    start_time, 
    end_time,
    (embedding <=> '[0.013, -0.024, ...]') AS cosine_distance
FROM 
    video_transcripts
ORDER BY 
    cosine_distance ASC
LIMIT 5;
```
*Technical note: The array `[0.013, -0.024, ...]` represents the Query Embeddings value returned from Amazon Bedrock. The `ORDER BY cosine_distance ASC` command ensures the most semantically adjacent results (smallest distance) are prioritized at the top.*

#### Backend Query Logic Implementation
Within the Search Service (running on ECS Fargate), the aforementioned SQL command is encapsulated and executed via a database connection library. To adhere to security standards, Database credentials are dynamically retrieved from AWS Secrets Manager (configured in Chapter 5.4).

Source code simulating the execution flow (`psycopg2` & `boto3`):

```python
import psycopg2
import boto3
import json
import os

def get_db_credentials():
    # Securely retrieve DB credentials from Secrets Manager
    client = boto3.client('secretsmanager', region_name='ap-southeast-1')
    secret_name = os.environ.get('DB_SECRET_NAME', 'cloudforge-db-credentials')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

def search_similar_transcripts(query_vector: list, limit: int = 5):
    creds = get_db_credentials()
    
    # Initialize connection to RDS PostgreSQL in the Private Subnet
    conn = psycopg2.connect(
        host=creds['host'],
        database=creds['dbname'],
        user=creds['username'],
        password=creds['password']
    )
    cursor = conn.cursor()
    
    # Convert the vector list into pgvector's standard string format
    vector_str = '[' + ','.join(map(str, query_vector)) + ']'
    
    # Execute the Cosine Distance calculation query
    query = """
        SELECT video_id, transcript_text, start_time 
        FROM video_transcripts 
        ORDER BY embedding <=> %s ASC 
        LIMIT %s;
    """
    cursor.execute(query, (vector_str, limit))
    results = cursor.fetchall()
    
    cursor.close()
    conn.close()
    
    return results
```

{{% notice tip %}}
**Performance Optimization:** For large-scale datasets, performing a full table scan (Exact Nearest Neighbor Search) leads to severe performance degradation. The project team configured an **HNSW** (Hierarchical Navigable Small World) index on the `embedding` column during Chapter 5.4, substantially accelerating search speeds (Approximate Nearest Neighbor) with near-perfect accuracy, ensuring query latency remains consistently low.
{{% /notice %}}

***

**Next Step:** The Backend system and core AI data processing pipelines are fully finalized. In the next chapter (**Chapter 5.11: Frontend Deployment**), the project team will deploy the static user interface to Amazon S3 and CloudFront to interact directly with the Cloud architecture we have just constructed.
