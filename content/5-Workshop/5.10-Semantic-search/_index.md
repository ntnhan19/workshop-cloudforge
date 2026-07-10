---
title : "Semantic Search"
date : 2026-07-10
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Semantic Search Overview

Instead of traditional Keyword-based search, Smart Media Analytics allows users to search for videos using natural language. For example: *"Find video clips with a cheerful atmosphere mentioning festivals"*.

To realize this, we will leverage the **RDS PostgreSQL** database equipped with the **pgvector** extension configured in Chapter 5.4.

Chapter focus:
1. **Vector Indexing:** How to store and index feature vectors generated from Amazon Bedrock.
2. **Cosine Similarity:** The vector distance calculation algorithm to find results with the highest semantic similarity to the user's query.

*(Detailed content will be updated in the sub-articles).*
