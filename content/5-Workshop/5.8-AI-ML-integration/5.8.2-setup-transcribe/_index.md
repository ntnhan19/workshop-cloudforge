---
title : "Integrate Amazon Transcribe"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.8.2. </b> "
---

To understand and extract the content of a Video or Audio file, the first and most critical step in the processing pipeline is to extract the conversational stream into raw text. Within the overall architecture of Smart Media Analytics, we utilize the **Amazon Transcribe** service - an Automatic Speech Recognition (ASR) solution based on AWS's deep machine learning technology.

Unlike Amazon Bedrock, which requires an auto-enable mechanism or Use Case declaration from the user, Amazon Transcribe is unlocked by default on all AWS accounts. In this segment, we will delve deep into the automated interaction mechanism between the AI Worker and this audio processing service.

#### Automatic Audio Processing Flow

The AI Worker application is programmed to execute a fully automated cycle in Python (via the `boto3` SDK library) to interact with Amazon Transcribe:

1. **Extract Audio Data:** When the AI Worker consumes a message (Metadata of the newly uploaded Video) from the Amazon SQS queue, it invokes the built-in `FFmpeg` tool within the Docker Container to extract the audio stream from the original Video file.
2. **Intermediate Storage:** The extracted audio file is uploaded to a specialized temporary directory on **Amazon S3** following the path structure `s3://[Bucket-Name]/audio-temp/`.
3. **Trigger Transcribe Job:** The AI Worker sends a `StartTranscriptionJob` API call to Amazon Transcribe, passing the S3 link of the audio file while configuring the Auto Language Identification feature.
4. **Asynchronous Status Polling:** Because the voice conversion process requires computation time and occurs asynchronously, the AI Worker continuously loops to Poll the `GetTranscriptionJob` API to update the system's processing status.
5. **Synchronize Results:** As soon as the status switches to the `COMPLETED` label, Amazon Transcribe outputs a JSON file containing the entire extracted text (Transcript), accompanied by detailed Timestamps for each word.

![Transcribe Flow](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/transcribe_flow.png)

#### Infrastructure Role Permissions (IAM Role)
For the automated process above to run smoothly without permission errors, the **ECS Task Role** of the AI Worker service must be attached with the following authorization policies:
- `transcribe:StartTranscriptionJob` and `transcribe:GetTranscriptionJob` to control the text conversion process.
- `s3:GetObject` and `s3:PutObject` on the designated Bucket for the purpose of writing the input audio file and reading the output JSON analysis result.

{{% notice tip %}}
**Cost Optimization:** The AI Worker proactively extracting and compressing the input audio stream (such as MP3/OGG format) before pushing it to the cloud not only reduces internal network bandwidth but also significantly saves storage costs. Because Amazon Transcribe calculates costs based on each second of file processing, a compact file accelerates I/O processing speed and optimizes billing resources.
{{% /notice %}}

***

**Next Step:** After acquiring all the raw text data extracted from the Video/Audio, we will move to the final segment of the knowledge analysis flow: Using mathematical embedding models to create Vectors (Embeddings), directly serving the deep Semantic Search feature.
