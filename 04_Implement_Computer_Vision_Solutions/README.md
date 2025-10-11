# 👁️ Implement Computer Vision Solutions

> **Master Azure AI vision capabilities for intelligent image and video analysis**

[![Exam Weight](https://img.shields.io/badge/Exam%20Weight-15--20%25-blue)](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-computer-vision-solutions-1015)
[![Study Guide](https://img.shields.io/badge/Official-Study%20Guide-orange)](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-computer-vision-solutions-1015)

## 📋 Overview

This section covers **15-20%** of the AI-102 exam content, focusing on implementing computer vision solutions using Azure AI services for image analysis, custom vision models, and video processing.

## 🎯 Key Learning Objectives

The following topics are evaluated in this section. Each links to detailed requirements in the [official study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-computer-vision-solutions-1015):

### 🖼️ 1. Analyze Images
**🔗 [Study Guide Link](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#analyze-images)**

Extract visual features, detect objects, read text, and analyze faces in images using Azure AI Vision services.

### 🎯 2. Implement Custom Vision Models  
**🔗 [Study Guide Link](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#implement-custom-vision-models)**

Build and deploy custom image classification and object detection models using Azure Custom Vision.

### 🎬 3. Analyze Videos
**🔗 [Study Guide Link](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-102#analyze-videos)**

Process and extract insights from video content using Azure Video Indexer and other video analysis services.

> 💡 **Study Tip:** Each topic requires hands-on experience with Azure AI Vision APIs and understanding of different computer vision scenarios.

## 📚 Microsoft Learning Resources

### 🎓 Core Training Materials
| Resource Type | Title | Duration | Link |
|---------------|-------|----------|------|
| 📖 **Learning Path** | Develop computer vision solutions in Azure | 6h 35m (8 modules) | [🔗 Start Learning](https://learn.microsoft.com/en-us/training/paths/create-computer-vision-solutions-azure-ai/) |
| 🎥 **Video** | Preparing for AI-102 - Implement Azure AI vision solutions | ~45 min | [🔗 Watch Video](https://learn.microsoft.com/en-us/shows/exam-readiness-zone/preparing-for-ai-102-implement-azure-ai-vision-solutions) |
| 🛠️ **Labs** | Hands-on computer vision labs | Self-paced | [🔗 Practice Labs](https://microsoftlearning.github.io/mslearn-ai-vision/) |

### 🎯 Learning Path Focus Areas
- **🖼️ Image Analysis:** Object detection, OCR, and visual feature extraction
- **🎯 Custom Models:** Training and deploying custom classification and detection models  
- **👁️ Face Recognition:** Face detection, verification, and identification
- **🎬 Video Processing:** Video indexing, analysis, and content extraction
- **🤖 Multimodal AI:** Combining vision with generative AI capabilities


## 🔧 Azure Computer Vision Capabilities

Comprehensive reference for each computer vision capability, including services, endpoints, and code examples.

| # | Capability | Description | Azure Service / Feature  | Example Endpoint | Code Examples |
|---|------------|-------------|---------------------------|------------------|---------------|
| 1 | **🖼️ Analyze Images** | Extracts visual features such as objects, people, tags, colors, and image description. Useful for scene understanding. | **Azure AI Vision (Analyze Image API)** | `https://<your-resource-name>.cognitiveservices.azure.com/vision/v3.2/analyze` | [📝 View Examples](./Analyze_Image.md) |
| 2 | **📄 Read Text in Images** | Performs OCR (Optical Character Recognition) to extract printed or handwritten text from images. | **Azure AI Vision – Read API** | `https://<your-resource-name>.cognitiveservices.azure.com/vision/v3.2/read/analyze` | [📝 View Examples](./Read_Text_In_Images.md) |
| 3 | **👤 Detect, Analyze, and Recognize Faces** | Detects faces, identifies attributes (age, emotion, pose), and matches them against a known person group. | **Azure AI Face Service** | `https://<your-resource-name>.cognitiveservices.azure.com/face/v1.0/detect` | [📝 View Examples](./Detect_Analyze_Recognize_Faces.md) |
| 4 | **🏷️ Classify Images** | Categorizes images into predefined or custom classes. You can train your own model. | **Azure AI Custom Vision – Classification** | `https://<your-customvision-resource>.cognitiveservices.azure.com/customvision/v3.0/classify/iterations/<model-name>/image` | [📝 View Examples](./Classify_Images.md) |
| 5 | **🎯 Detect Objects in Images** | Identifies and locates multiple objects within an image using bounding boxes. | **Azure AI Custom Vision – Object Detection** | `https://<your-customvision-resource>.cognitiveservices.azure.com/customvision/v3.0/detect/iterations/<model-name>/image` | [📝 View Examples](./Detect_Objects_In_Images.md) |
| 6 | **🎬 Analyze Video** | Extracts insights such as scenes, faces, emotions, objects, speech-to-text, and OCR from videos. | **Azure AI Video Indexer** | `https://api.videoindexer.ai/<location>/Accounts/<account-id>/Videos/<video-id>/Index` | [📝 View Examples](./Analyze_Video.md) |
| 7 | **🤖 Vision-Enabled Generative AI** | Combines vision input (image or video) with generative models (e.g., GPT-4o, Florence, or Phi-3-Vision) for multimodal AI. | **Azure AI Foundry / Azure OpenAI (GPT-4o Vision)** | `https://<your-hub>.openai.azure.com/openai/deployments/<deployment-name>/chat/completions?api-version=2024-06-01` | [📝 View Examples](./Vision_Enabled_Generative_AI.md) |


## 🧩 AI-102 Exam Key Points

### Service Categories:
- **🔍 Azure AI Vision & Face APIs** → Core image analysis and face recognition
- **🎯 Custom Vision** → Train custom classification and object detection models
- **🎬 Video Indexer** → Media analysis with Azure Media Services integration
- **🤖 Multimodal AI** → Azure AI Foundry with vision-enabled models (GPT-4o, Florence)

### 🎯 Study Strategy

1. **📖 Complete the learning path** to understand all vision capabilities
2. **🛠️ Practice with hands-on labs** using real Azure resources
3. **💡 Review capability examples** and understand API differences
4. **🔄 Test different scenarios** for each service type
5. **📝 Focus on integration patterns** between services

## 📋 Quick Reference

### Key Services to Master:
- **🔧 Azure AI Vision** - Image analysis, OCR, object detection
- **👤 Azure AI Face** - Face detection, recognition, verification
- **🎯 Azure Custom Vision** - Custom image classification and object detection
- **🎬 Azure Video Indexer** - Video content analysis and insights
- **🤖 Azure OpenAI GPT-4o** - Vision-enabled generative AI

### 📘 Microsoft Learn References

- 📖 [Analyze images with Azure AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/call-analyze-image)
- 📄 [Read text in images with Azure AI Vision OCR](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/how-to/call-read-api)
- 👤 [Use the Azure Face service](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/identity-client-library?tabs=windows%2Cvisual-studio&pivots=foundry-portal)
- 🎯 [Custom Vision service overview](https://learn.microsoft.com/en-us/azure/ai-services/custom-vision-service/overview)
- 🎬 [Azure Video Indexer overview](https://learn.microsoft.com/en-us/azure/azure-video-indexer/video-indexer-overview)

---

**📌 Remember:** Computer vision solutions require understanding both the technical implementation and appropriate service selection for different use cases!
- [Azure OpenAI GPT-4o with Vision](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/use-your-data?tabs=ai-search%2Ccopilot)