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

Although the activation mechanism has been automated, AWS regulations for Anthropic models require First-time users to still provide Use Case information.

1. Search for and access the **Amazon Bedrock** service (Ensure you are in the `ap-southeast-1` Region).
2. Access the **Model access** section on the left navigation bar.
3. The system will display a notification confirming the auto-enable policy: *"Model access page has been retired"*.
4. To use the Claude 3 model, during the first API call by the AI Worker (or when you test in the Bedrock Playground interface), if the system asks for *Use case details*, you can use the content: *"Testing Generative AI capabilities for an educational workshop and cloud deployment pipeline simulation"*.

![Bedrock Auto Access](/images/5-Workshop/5.8-AI-ML-integration/5.8.1-enable-bedrock-access/bedrock_model_access.png)

{{% notice tip %}}
**Best Practice:** AWS shifting to the "Auto-enable upon first invocation" model frees system architects from worrying about manual Provisioning processes, perfectly aligning with the automation and flexible infrastructure philosophy of Cloud-native development.
{{% /notice %}}

***

**Next Step:** With Amazon Bedrock ready in an auto-enabled state, we will move on to setting up the **Amazon Transcribe** service to process the audio data stream in the next section.
