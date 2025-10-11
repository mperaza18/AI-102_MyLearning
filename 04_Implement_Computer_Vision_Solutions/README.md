# Implement computer vision solutions

This section represents the (15 - 20%) according to the official study guide -> [Link](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-computer-vision-solutions-1015)

The topics evaluated are:

1. [Analyze images](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#analyze-images)
2. [Implement custom vision models](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-custom-vision-models)
3. [Analyze videos](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#analyze-videos)


In each topic you will find the details services and information that will be evaluated.

## MS Learning Assests

- Learning: [Develop computer vision solutions in Azure](https://learn.microsoft.com/en-us/training/paths/create-computer-vision-solutions-azure-ai/), 8 Modules - 6 hr 35 min.
- Video: [Preparing for AI-102 - Implement Azure AI vision solutions (Part 3 of 6)](https://learn.microsoft.com/en-us/shows/exam-readiness-zone/preparing-for-ai-102-implement-azure-ai-vision-solutions)
- Labs: [Develop computer vision solutions in Azure](https://microsoftlearning.github.io/mslearn-ai-vision/)


## Resume
Below is a detailed table for each capability you mentioned, including the description, main Azure service or API used, and an example deployed endpoint URL format you would get once the resource is provisioned in Azure.

| # | Capability | Description | Azure Service / Feature  | Example Endpoint (Deployed URL) | Python Example | C# Example|
| - | - | - | - | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 1 | **Analyze Images**                                   | Extracts visual features such as objects, people, tags, colors, and image description. Useful for scene understanding.     | **Azure AI Vision (Analyze Image API)**             | `https://<your-resource-name>.cognitiveservices.azure.com/vision/v3.2/analyze`                                              | [Python Example](https://www.notion.so/Python-Example-2863a5ad5807808484dbcfde564773ec?pvs=21)<br>[Python OLD SDK Example](https://www.notion.so/Python-OLD-SDK-Example-2863a5ad580780fba6cddfc9a7ae2dd7?pvs=21) | [C# Example](https://www.notion.so/C-Example-2863a5ad580780d29d30c69a74a5bfaa?pvs=21) |
| 2 | **Read Text in Images**                              | Performs OCR (Optical Character Recognition) to extract printed or handwritten text from images.                           | **Azure AI Vision – Read API**                      | `https://<your-resource-name>.cognitiveservices.azure.com/vision/v3.2/read/analyze`                                         | [Python Example](https://www.notion.so/Python-Example-2863a5ad58078058bb22ee1ca87c06ec?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad580780b9a5fffbfbcf2354a2?pvs=21) |
| 3 | **Detect, Analyze, and Recognize Faces**             | Detects faces, identifies attributes (age, emotion, pose), and matches them against a known person group.                  | **Azure AI Face Service**                           | `https://<your-resource-name>.cognitiveservices.azure.com/face/v1.0/detect`                                                 | [Python Example](https://www.notion.so/Python-Example-2863a5ad580780eda0bbf6ebd56069e5?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad580780e89f9ed06795ee5618?pvs=21) |
| 4 | **Classify Images**                                  | Categorizes images into predefined or custom classes. You can train your own model.                                        | **Azure AI Custom Vision – Classification**         | `https://<your-customvision-resource>.cognitiveservices.azure.com/customvision/v3.0/classify/iterations/<model-name>/image` | [Python Example](https://www.notion.so/Python-Example-2863a5ad580780c9aea0c1bbb15650ab?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad580780c6b41ed2df274ade45?pvs=21) |
| 5 | **Detect Objects in Images**                         | Identifies and locates multiple objects within an image using bounding boxes.                                              | **Azure AI Custom Vision – Object Detection**       | `https://<your-customvision-resource>.cognitiveservices.azure.com/customvision/v3.0/detect/iterations/<model-name>/image`   | [Python Example](https://www.notion.so/Python-Example-2863a5ad58078058b21dd204daf57bdb?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad58078017b789f9a7319489b6?pvs=21) |
| 6 | **Analyze Video**                                    | Extracts insights such as scenes, faces, emotions, objects, speech-to-text, and OCR from videos.                           | **Azure AI Video Indexer**                          | `https://api.videoindexer.ai/<location>/Accounts/<account-id>/Videos/<video-id>/Index`                                      | [Python Example](https://www.notion.so/Python-Example-2863a5ad58078006b816db77274ad462?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad58078015b006d85ee43c1007?pvs=21) |
| 7 | **Develop Vision-Enabled Generative AI Application** | Combines vision input (image or video) with generative models (e.g., GPT-4o, Florence, or Phi-3-Vision) for multimodal AI. | **Azure AI Foundry / Azure OpenAI (GPT-4o Vision)** | `https://<your-hub>.openai.azure.com/openai/deployments/<deployment-name>/chat/completions?api-version=2024-06-01`          | [Python Example](https://www.notion.so/Python-Example-2863a5ad5807802abab3f23ed9e2edec?pvs=21)                                                                                                                   | [C# Example](https://www.notion.so/C-Example-2863a5ad5807801090a4deaec85465a2?pvs=21) |


## 🧩 Notes for AI-102 Exam Context

- **Analyze, Read, Face APIs** → belong to **Azure AI Vision** and **Face** services.
- **Custom Vision (Classification/Object Detection)** → allows **training your own models** via a web UI or API.
- **Video Indexer** → uses **AI services for media analysis**, and integrates with **Azure Media Services**.
- **Generative Vision Apps** → use **Azure AI Foundry** with **multimodal models** (e.g., GPT-4o, Florence 2, or CLIP).



### 📘 Microsoft Learn References

- [Analyze images with Azure AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/call-analyze-image)
- [Read text in images with Azure AI Vision OCR](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/call-read-api)
- [Use the Azure Face service](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/identity-client-library?tabs=windows%2Cvisual-studio&pivots=foundry-portal)
- [Custom Vision service overview](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/overview)
- [Azure Video Indexer overview](https://learn.microsoft.com/en-us/azure/azure-video-indexer/video-indexer-overview)
- [Azure OpenAI GPT-4o with Vision](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/use-your-data?tabs=ai-search%2Ccopilot)