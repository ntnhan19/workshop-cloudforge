---
title: "Blog 4"
date: 2026-07-13
weight: 4
chapter: false
pre: " <b> 3.4. </b> "
---
# Deep Dive into AWS Bedrock & Vector Embedding – What Turns Raw Text into "Coordinates" for AI?

Hello everyone in the AWS Study Group VN community.

Coming from a background of strictly coding traditional Backend workflows in Java or Node.js, recently as I dove deeper into Cloud AI and RAG (Retrieval-Augmented Generation) architectures—and actively optimized this architecture for a real-world project—I realized a fascinating truth: AI doesn't actually read the exact text provided by users!

We often hear about feeding documents to an AI for Q&A. However, behind the scenes, the system doesn't read Strings the way humans read books. It requires a crucial transformation step called Embedding.

Today, I want to dissect the process of using Amazon Bedrock to convert raw data into Vectors – the core of any modern AI system.

![AWS Bedrock Embedding](/images/3-BlogsPosted/3.4-Blog4/blog4.jpg)

To build a complete RAG system on AWS, your data goes through a processing pipeline that backend developers are very familiar with:

### 1. Chunking
You cannot throw a 100-page PDF file entirely into Bedrock and tell it to "Embed this." It's like trying to stuff a whole chicken into your mouth. We must use code to split the document into smaller pieces (chunks) of a few sentences or a paragraph. This chunking helps the AI search for information more accurately later on instead of losing context.

### 2. Calling Amazon Bedrock
After obtaining the small text chunks, the backend system calls the API to send that text to Amazon Bedrock (using specialized models like Titan Text Embeddings or Cohere). Bedrock acts like a "magical black box." It takes a String (e.g., "7-day return policy") and returns a massive array of real numbers (e.g., `[0.012, -0.453, 0.887, ...]`).

### 3. Storing in a Vector Database
In theory, this array of numbers (Vector) could still be stuffed into a JSON or BLOB column in a standard MySQL or PostgreSQL database, and nothing prohibits this. The problem lies elsewhere: when similarity search is needed across millions of vectors, traditional databases lack appropriate indexing mechanisms (like HNSW or IVFFlat). They would have to scan the entire dataset sequentially—which is extremely slow and resource-intensive.

Therefore, in practice, people use specialized databases for high-dimensional spaces like Amazon OpenSearch Serverless, or attach a dedicated extension like pgvector to PostgreSQL for ANN (Approximate Nearest Neighbor) indexing, making searches significantly faster than full scans. Each record then consists of: `[Original Text Segment] + [Vector Array]`.

### How Does the AI Search When a User Asks a Question?
This is the most interesting part of the RAG architecture. When a user types in the chat: "I want outfit ABCXYZ," the system performs the following steps:
1. Sends this question to Bedrock to turn it into a Vector (creating a new coordinate on the map).
2. Takes this coordinate into the Vector Database and runs an algorithm called Cosine Similarity (measuring the angle between two vectors rather than standard straight-line distance). In simple terms: The database uses a protractor to see which documents on that multi-dimensional map are closest to the question's coordinates.
3. Retrieves the top 5 closest results (the exact text segments about short-sleeved shirts, shorts, etc.) and pushes that text to a Large Language Model (LLM) so it can synthesize the most natural answer.

All of this happens in just a few hundred milliseconds!

### Why AWS Bedrock Instead of Hosting the Model Yourself?
From a system architecture perspective, self-hosting Embedding models using open-source libraries on EC2 is a nightmare. You have to handle GPU installation, environment configuration, and especially auto-scaling when request volumes spike.

With Amazon Bedrock, the infrastructure is almost entirely managed by AWS (serverless). You just configure the SDK in Java or Node.js, call the API, and AWS takes care of scaling the underlying infrastructure within your account quota limits. You pay exactly for the requests you make. This allows developers to focus entirely on building business logic instead of acting as plumbers for the infrastructure.

### "Battle Scars" from Integrating Bedrock Embedding
- **Rate Limit (Throttling) Errors**: When writing a script to read thousands of document pages and continuously calling the Bedrock API to convert them into Vectors, it's very easy to hit API limits (account/region quotas). Always remember to implement a Retry mechanism (try again after a few seconds) or use a queue (Amazon SQS) to process them gradually.
- **Chunking Strategy is Critical**: If you cut the text too short, the AI loses context (it doesn't understand what is being talked about). If cut too long, the information gets muddled during the search. A good practice is to let the text segments "overlap" a bit.
- **Vector DBs Can Be Costly**: Storing Vectors consumes significantly more RAM than storing plain text. Calculate the costs carefully when choosing a storage service.
- **API Call Costs Must Also Be Calculated**: Besides Vector DB storage costs, calling Bedrock to embed documents in bulk (especially when re-embedding due to a change in the chunking strategy) is charged per processed token. For large datasets, this adds up quickly, so think carefully before running batch processes over the entire document set multiple times.

### Conclusion
Amazon Bedrock is not just a place to host fancy models like Claude or Llama; it also provides an incredibly powerful Embedding tool to breathe life into raw data. Mastering the Chunking → Embedding → Vector Search process is the key for a Backend Developer to confidently build advanced AI features without needing to be a math or data science expert.

Has anyone here tried building a RAG system and encountered issues when optimizing search results from a Vector DB? I look forward to hearing real-world experiences from all of you!
