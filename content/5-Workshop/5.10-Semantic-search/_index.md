---
title : "Semantic Search"
date : 2026-07-10
weight : 10
chapter : false
pre : " <b> 5.10. </b> "
---

### Semantic Search Architecture Overview

The core functionality that constitutes the value of the Smart Media Analytics system is the capability to retrieve multimedia data based on contextual meaning, rather than relying on traditional keyword matching methodologies. Clients can query using natural language, for example: *"Find conversations expressing concern about cybersecurity."*

To actualize this capability, the project team establishes an integration architecture between Machine Learning Models (Amazon Bedrock) and a Vector Database (Amazon RDS PostgreSQL paired with the `pgvector` extension).

#### Operational Principle of Vector Search
The project's Semantic Search architecture operates on a mathematical spatial correlation mechanism:
- **Data Representation:** Every text segment (from the Video Transcript) was already encoded into multi-dimensional Vectors during the Ingestion phase (Chapter 5.8.3) and stored within PostgreSQL.
- **Query Representation:** The user's search query is similarly encoded into a multi-dimensional Vector using the exact Embeddings Model employed during Ingestion.
- **Distance Measurement:** The system utilizes the **Cosine Similarity** algorithm supported by `pgvector` to compute the distance between the query Vector and all Vectors residing in the database. Vectors with the closest geometric proximity are returned as the most semantically relevant results.

#### Semantic Search Implementation Roadmap:
1. **Generate Query Embeddings:** Construct an API flow to process search requests, invoking Amazon Bedrock to transform the user's text query into a Vector structure.
2. **Execute Vector Search with pgvector:** Formulate advanced database queries (Advanced SQL Queries) to execute mathematical distance calculations and return result data in real-time.

With this architecture, the project comprehensively resolves the challenge of comprehending the end-user's search intent.

### Hands-on Content

- [Generate Query Vector](5.10.1-generate-query-vector/)
- [Execute Vector Search with pgvector](5.10.2-vector-search-pgvector/)
