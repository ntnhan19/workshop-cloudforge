---
title : "Configure Amazon Bedrock"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.8.1. </b> "
---

One of the most critical architectural building blocks constituting the analytical capabilities of Smart Media Analytics is **Amazon Bedrock** - a fully managed cloud service that provides access to market-leading Foundation Models (FMs) through a unified API interface.

#### Latest Model Access Policy Update from AWS
Previously, AWS required system architects to manually request model activation to control risks. However, following the latest platform update from AWS, the **traditional Model Access management page has been retired**.

Currently, Serverless foundation models (such as Amazon Titan Text Embeddings v2, which we use to generate Vectors) will be **Automatically enabled** across all commercial Regions upon their first API invocation. This completely eliminates operational bottlenecks, allowing the AI Worker system to connect and use them immediately.

#### Note Regarding Anthropic Models (Claude 3)
In this project, we utilize the **Anthropic Claude 3** model family to process deep reasoning logic and summarize Media content.

{{% notice info %}}
**Important Note:** Starting from this chapter (5.8), the entire communication process with AI services (Amazon Bedrock, Amazon Transcribe) is handled completely automatically by the Python source code inside the `ai_worker` Container that you deployed in the previous chapter. 
**You do not need to manually perform any steps on the AWS Console throughout Chapter 5.8.** Please skim through to understand how the data flow mechanism works!
{{% /notice %}}

{{% notice tip %}}
**Best Practice:** AWS shifting to the "Auto-enable upon first invocation" model frees system architects from worrying about manual Provisioning processes, perfectly aligning with the automation and flexible infrastructure philosophy of Cloud-native development.
{{% /notice %}}

***

**Next Step:** With Amazon Bedrock ready in an auto-enabled state, we will move on to setting up the [**Amazon Transcribe**](../5.8.2-setup-transcribe/) service to process the audio data stream in the next section.
