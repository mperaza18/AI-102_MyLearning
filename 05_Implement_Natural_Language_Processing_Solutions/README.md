# 🧠 Natural Language Processing Solutions

## 📚 Overview

This section covers Natural Language Processing (NLP) capabilities available in Azure AI Services, aligned with the **AI-102 exam objectives (15–25% of exam content)**. Learn to implement text analysis, language understanding, translation, and speech processing solutions.

## 🎯 Azure NLP Capabilities

| # | 🔧 Capability | 📝 Description | 🏷️ Azure Service | 🌐 Example Endpoint | 📖 Implementation Guide |
|---|---|---|---|---|---|
| **1** | **Analyze Text with Azure AI Language** | Performs sentiment analysis, key phrase extraction, named entity recognition, and language detection | Azure AI Language | `https://<your-resource>.cognitiveservices.azure.com/language/:analyze-text?api-version=2023-04-15` | [📄 Text Analysis Guide](./Analyze_Text_with_Azure_AI_Language.md) |
| **2** | **Create Question Answering Solutions** | Build knowledge bases that answer user questions from documents or websites | Azure AI Language – Question Answering | `https://<your-resource>.cognitiveservices.azure.com/language/:query-knowledgebases?projectName=<project-name>&api-version=2023-04-15` | [📄 Q&A Solutions Guide](./Create_Question_Answering_Solutions.md) |
| **3** | **Build Conversational Language Understanding (CLU)** | Detects intents and entities in user input for chatbots and conversational AI | Azure AI Language – CLU | `https://<your-resource>.cognitiveservices.azure.com/language/:analyze-conversations?projectName=<project-name>&deploymentName=production&api-version=2023-04-15` | [📄 CLU Model Guide](./Build_Conversational_Language_Understanding_Model.md) |
| **4** | **Create Custom Text Classification** | Custom model for text classification trained in Azure AI Studio | Azure AI Language – Custom Text Classification | `https://<your-resource>.cognitiveservices.azure.com/language/:analyze-text/projects/<project-name>/deployments/<deployment-name>?api-version=2023-04-15` | [📄 Text Classification Guide](./Create_Custom_Text_Classification.md) |
| **5** | **Custom Named Entity Recognition** | Custom model to extract entities (e.g., invoice numbers, patient IDs) | Azure AI Language – Custom NER | `https://<your-resource>.cognitiveservices.azure.com/language/:analyze-text/projects/<project-name>/deployments/<deployment-name>?api-version=2023-04-15` | [📄 Custom NER Guide](./Custom_Named_Entity_Recognition.md) |
| **6** | **Translate Text with Azure AI Translator** | Detect and translate text across 100+ languages with high accuracy | Azure AI Translator | `https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=es` | [📄 Text Translation Guide](./Translate_Text_with_Azure_AI_Translator.md) |
| **7** | **Create Speech-Enabled Apps** | Integrate voice capabilities into applications with speech-to-text and text-to-speech | Azure AI Speech | `wss://<region>.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1` | [📄 Speech Apps Guide](./Create_Speech_Enabled_Apps.md) |
| **8** | **Translate Speech** | Converts spoken input into translated text and audio in real-time | Azure AI Speech Translation | `wss://<region>.stt.speech.microsoft.com/speech/translation/cognitiveservices/v1` | [📄 Speech Translation Guide](./Translate_Speech.md) |
| **9** | **Audio-Enabled Generative AI Applications** | Create applications where users can talk with an LLM using voice input/output | Azure OpenAI + Azure Speech | `https://<your-azure-openai-resource>.openai.azure.com/openai/deployments/<model>/chat/completions?api-version=2024-02-15-preview` | [📄 Audio AI Guide](./Develop_Audio_Enabled_Generative_AI_Application.md) |

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