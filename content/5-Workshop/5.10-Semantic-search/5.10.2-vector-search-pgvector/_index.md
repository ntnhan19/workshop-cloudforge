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

Below is the equivalent SQL command structure behind the ORM:
```sql
SELECT 
    asset_id, 
    transcript_snippet, 
    timestamp_start_sec, 
    timestamp_end_sec
FROM 
    scenes
ORDER BY 
    embedding <=> '[0.013, -0.024, ...]' ASC
LIMIT 5;
```
*Technical note: The array `[0.013, -0.024, ...]` represents the Query Embeddings value returned from Amazon Bedrock. The `<=>` operator and `ORDER BY` command ensure the most semantically adjacent results (smallest distance) are prioritized at the top.*

#### Backend Query Logic Implementation
Within the Search Service (running on ECS Fargate), the project team utilizes **SQLAlchemy** and the **pgvector-python** library (`pgvector.sqlalchemy`) to execute the query asynchronously and securely.

Execution source code from `backend/core/embeddings/pgvector_store.py`:

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from pgvector.sqlalchemy import Vector
from models.scene import Scene
from database import SessionLocal

class PGVectorStore:
    # ...
    async def search(self, query_embedding: list[float], n_results: int = 10):
        """
        Execute Vector Similarity Search using Cosine Distance (<=>).
        """
        async with SessionLocal() as db:
            # Query the scenes table, order by cosine distance
            stmt = select(Scene).order_by(
                Scene.embedding.cosine_distance(query_embedding)
            ).limit(n_results)
            
            result = await db.execute(stmt)
            scenes = result.scalars().all()
            
            # Map ORM objects to response dictionary...
            return scenes
```

{{% notice tip %}}
**Performance Optimization:** For large-scale datasets, performing a full table scan (Exact Nearest Neighbor Search) leads to severe performance degradation. The project team configured an **HNSW** (Hierarchical Navigable Small World) index on the `embedding` column during Chapter 5.4, substantially accelerating search speeds (Approximate Nearest Neighbor) with near-perfect accuracy, ensuring query latency remains consistently low.
{{% /notice %}}

***

**Next Step:** The Backend system and core AI data processing pipelines are fully finalized. In the next chapter (**Chapter 5.11: Frontend Deployment**), the project team will deploy the static user interface to Amazon S3 and CloudFront to interact directly with the Cloud architecture we have just constructed.
