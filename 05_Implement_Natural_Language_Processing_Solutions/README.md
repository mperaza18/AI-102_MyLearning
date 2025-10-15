# 🧠 Natural Language Processing Solutions

## 📚 Overview

This section covers Natural Language Processing (NLP) capabilities available in Azure AI Services, aligned with the **AI-102 exam objectives (15–25% of exam content)**. Learn to implement text analysis, language understanding, translation, and speech processing solutions.

## 🧠 Azure AI Language Capabilities

| **Capability** | **Description** | **Service Used** | **REST Endpoint** | **Code Examples** |
|---|---|---|---|---|
| **Sentiment Analysis** 😊 | Analyze text sentiment with confidence scores and opinion mining | Azure AI Language | `/language/:analyze-text?api-version=2022-05-01` | [📄 View Examples](./Sentiment_Analysis.md) |
| **Named Entity Recognition** 🏷️ | Extract and classify entities (people, places, organizations) | Azure AI Language | `/language/:analyze-text?api-version=2022-05-01` | [📄 View Examples](./Named_Entity_Recognition.md) |
| **Key Phrase Extraction** 🔑 | Identify main concepts and important phrases in text | Azure AI Language | `/language/:analyze-text?api-version=2022-05-01` | [📄 View Examples](./Key_Phrase_Extraction.md) |
| **Language Detection** 🌐 | Detect the language of input text with confidence scores | Azure AI Language | `/language/:analyze-text?api-version=2022-05-01` | [📄 View Examples](./Language_Detection.md) |
| **Text Summarization** 📝 | Generate extractive and abstractive summaries of documents | Azure AI Language | `/language/analyze-text/jobs?api-version=2022-05-01` | [📄 View Examples](./Text_Summarization.md) |
| **Question Answering** ❓ | Build conversational AI with custom or prebuilt knowledge bases | Azure AI Language | `/language/:query-knowledgebases?api-version=2021-10-01` | [📄 View Examples](./Question_Answering.md) |
| **PII Detection & Redaction** 🔒 | Identify and redact personally identifiable information | Azure AI Language | `/language/:analyze-text?api-version=2022-05-01` | [📄 View Examples](./PII_Detection.md) |
| **Healthcare Text Analytics** 🏥 | Extract medical entities, relations, and assertions | Azure AI Language | `/language/analyze-text/jobs?api-version=2022-05-01` | [📄 View Examples](./Healthcare_Analytics.md) |
| **Content Safety** 🛡️ | Detect harmful content categories with severity levels | Azure AI Content Safety | `/contentsafety/text:analyze?api-version=2024-09-01` | [📄 View Examples](./Content_Safety.md) |
| **Build a Conversational Language Understanding (CLU) Model** 🤖 | Detects intents and entities in user input for chatbots | Azure AI Language – CLU | `/language/:analyze-conversations?api-version=2023-04-01` | [📄 View Examples](./Conversational_Language_Understanding.md) |

### 🔗 Quick Links
- **📋 [Azure AI Language Overview](https://docs.microsoft.com/azure/cognitive-services/language-service/)**
- **🚀 [Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)**
- **⚙️ [.NET SDK Documentation](https://docs.microsoft.com/dotnet/api/azure.ai.textanalytics/)**
- **🌐 [REST API Reference](https://docs.microsoft.com/rest/api/language/)**

> Note
> - Prebuilt text analysis features (Sentiment, NER, Key Phrases, Language Detection, PII) use the unified endpoint: `https://<endpoint>/language/:analyze-text?api-version=2022-05-01`.
> - Long-running operations like Document Summarization and Text Analytics for Health use the jobs endpoint: `https://<endpoint>/language/analyze-text/jobs?api-version=2022-05-01`.
> - Custom Question Answering runtime uses: `https://<endpoint>/language/:query-knowledgebases?api-version=2021-10-01`.
> - Azure AI Content Safety is a separate service and uses: `https://<endpoint>/contentsafety/text:analyze?api-version=2024-09-01`.
> - For the latest GA API versions, see the Azure AI Language REST reference. Some quickstarts may show newer preview versions.

## 🎯 AI-102 Exam Coverage

## 🗺️ Learning Path

### 🟢 Beginner Level
1. **Text Analysis Fundamentals** - Start with basic sentiment analysis and entity extraction
2. **Language Detection** - Implement multi-language text processing
3. **Simple Translation** - Build basic translation workflows

### 🟡 Intermediate Level
4. **Question Answering Systems** - Create knowledge base solutions
5. **Custom Text Classification** - Train domain-specific models
6. **Speech Integration** - Add voice capabilities to applications

### 🔴 Advanced Level
7. **Conversational AI (CLU)** - Build intelligent chatbots
8. **Custom Entity Recognition** - Extract specialized business entities
9. **Multimodal AI Applications** - Combine speech, text, and AI generation

## 📋 Exam Preparation Checklist

### Core NLP Concepts
- [ ] **Text Analytics** - Sentiment, key phrases, entities, language detection
- [ ] **Custom Models** - Training, deployment, and management
- [ ] **Language Understanding** - Intent recognition and entity extraction
- [ ] **Translation Services** - Text and speech translation workflows
- [ ] **Speech Processing** - STT, TTS, and speech translation

### Implementation Skills
- [ ] **REST API Integration** - Direct service calls and authentication
- [ ] **SDK Usage** - Python and C# client libraries
- [ ] **Model Training** - Custom classification and NER models
- [ ] **Real-time Processing** - Streaming and batch operations
- [ ] **Error Handling** - Retry logic and fallback strategies

### Azure-Specific Knowledge
- [ ] **Service Configuration** - Endpoints, keys, and regions
- [ ] **Pricing Models** - Understanding cost optimization
- [ ] **Security** - Managed identity and key management
- [ ] **Monitoring** - Logging and performance metrics
- [ ] **Integration** - Combining multiple AI services

## 🛠️ Quick Start Commands

```bash
# Install required packages
pip install azure-ai-textanalytics azure-cognitiveservices-language-luis azure-ai-translation-text

# Set environment variables
export AZURE_LANGUAGE_ENDPOINT="https://your-resource.cognitiveservices.azure.com/"
export AZURE_LANGUAGE_KEY="your-api-key"
export AZURE_TRANSLATOR_KEY="your-translator-key"
export AZURE_SPEECH_KEY="your-speech-key"
export AZURE_SPEECH_REGION="your-region"
```

## 📊 Service Comparison

| Service | Primary Use Case | Training Required | Real-time Support | Multi-language |
|---------|------------------|-------------------|-------------------|----------------|
| **Text Analytics** | General text analysis | ❌ No | ✅ Yes | ✅ Yes |
| **Custom Classification** | Domain-specific categorization | ✅ Yes | ✅ Yes | ✅ Yes |
| **Custom NER** | Specialized entity extraction | ✅ Yes | ✅ Yes | ✅ Yes |
| **CLU** | Conversational understanding | ✅ Yes | ✅ Yes | ✅ Yes |
| **Question Answering** | Knowledge base queries | ✅ Yes | ✅ Yes | ✅ Yes |
| **Translator** | Text/document translation | ❌ No | ✅ Yes | ✅ Yes |
| **Speech Services** | Voice processing | ❌/✅ Optional | ✅ Yes | ✅ Yes |

## 🎓 Additional Resources

- [Azure AI Language Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/)
- [Azure AI Translator Documentation](https://docs.microsoft.com/azure/cognitive-services/translator/)
- [Azure AI Speech Documentation](https://docs.microsoft.com/azure/cognitive-services/speech-service/)
- [AI-102 Exam Study Guide](https://docs.microsoft.com/learn/certifications/exams/ai-102)
- [Azure AI Services Pricing](https://azure.microsoft.com/pricing/details/cognitive-services/)