# ❓ Question Answering with Azure AI Language

> **Comprehensive code examples for building conversational AI and knowledge base solutions using Azure AI Language Question Answering**

## 📋 Overview

Azure AI Language Question Answering service enables you to build conversational AI applications that can answer user questions from your data sources. It supports both knowledge base projects and direct text querying without pre-built knowledge bases.

## 🔧 Key Features

- **📚 Knowledge Base Creation** - Build from documents, websites, and structured Q&A
- **🤖 Conversational AI** - Natural language understanding for user queries
- **🔄 Follow-up Questions** - Support for multi-turn conversations
- **📝 Direct Text Querying** - Answer questions from provided text without training
- **🌐 Multi-language Support** - Support for multiple languages
- **🔍 Confidence Scoring** - Confidence levels for answer quality
- **📊 Active Learning** - Improve models based on user interactions
- **🚀 Real-time Responses** - Fast query processing for production applications

## 🐍 Python Examples

### Basic Question Answering from Knowledge Base

```python
from azure.ai.language.questionanswering import QuestionAnsweringClient
from azure.core.credentials import AzureKeyCredential
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = QuestionAnsweringClient(endpoint, credential)

# Knowledge base configuration
knowledge_base_project = "FAQ-Project"
deployment_name = "production"

def basic_question_answering():
    """Basic Q&A from a deployed knowledge base"""
    
    questions = [
        "How much battery life do I have left?",
        "How do I reset my device?",
        "What is the warranty period?",
        "How to contact customer support?"
    ]
    
    for question in questions:
        print(f"Q: {question}")
        
        try:
            output = client.get_answers(
                question=question,
                project_name=knowledge_base_project,
                deployment_name=deployment_name
            )
            
            if output.answers:
                best_answer = output.answers[0]
                print(f"A: {best_answer.answer}")
                print(f"Confidence: {best_answer.confidence:.2f}")
                print(f"Source: {best_answer.source}")
                print("-" * 50)
            else:
                print("A: No answer found")
                print("-" * 50)
                
        except Exception as e:
            print(f"Error: {str(e)}")
            print("-" * 50)

# Run basic Q&A
basic_question_answering()
```

### Question Answering from Text (No Knowledge Base Required)

```python
from azure.ai.language.questionanswering import models as qna

def question_answering_from_text():
    """Answer questions directly from provided text"""
    
    # Sample knowledge content
    knowledge_content = [
        "Power and charging: It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state. "
        "It can take longer if you're using your Surface for power-intensive activities like gaming or video streaming while you're charging it.",
        
        "You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges. "
        "The USB port on the power supply is only for charging, not for data transfer.",
        
        "If you want to use a USB device, plug it into the USB port on your Surface. "
        "The Surface Pro 4 has one USB 3.0 port located on the left side of the device.",
        
        "Surface Pro 4 comes with Windows 10 Pro pre-installed. "
        "You can upgrade to Windows 11 if your device meets the system requirements."
    ]
    
    questions = [
        "How long does it take to charge?",
        "Can I charge other devices?",
        "Where is the USB port located?",
        "What operating system comes with Surface Pro 4?",
        "Can I install Linux?" # This should return low confidence or no answer
    ]
    
    for question in questions:
        print(f"Q: {question}")
        
        try:
            input_data = qna.AnswersFromTextOptions(
                question=question,
                text_documents=knowledge_content
            )
            
            output = client.get_answers_from_text(input_data)
            
            if output.answers:
                # Filter answers by confidence threshold
                high_confidence_answers = [
                    answer for answer in output.answers 
                    if answer.confidence > 0.5
                ]
                
                if high_confidence_answers:
                    best_answer = high_confidence_answers[0]
                    print(f"A: {best_answer.answer}")
                    print(f"Confidence: {best_answer.confidence:.2f}")
                else:
                    print("A: No confident answer found")
                    if output.answers:
                        print(f"Best attempt (low confidence): {output.answers[0].answer}")
                        print(f"Confidence: {output.answers[0].confidence:.2f}")
            else:
                print("A: No answer found")
                
        except Exception as e:
            print(f"Error: {str(e)}")
            
        print("-" * 60)

# Run text-based Q&A
question_answering_from_text()
```

### Multi-turn Conversations with Follow-up Questions

```python
def multi_turn_conversation():
    """Handle follow-up questions in conversations"""
    
    # First question
    question1 = "How long should my Surface battery last?"
    
    output1 = client.get_answers(
        question=question1,
        project_name=knowledge_base_project,
        deployment_name=deployment_name
    )
    
    if output1.answers:
        answer1 = output1.answers[0]
        print(f"Q1: {question1}")
        print(f"A1: {answer1.answer}")
        print(f"QnA ID: {answer1.qna_id}")
        print()
        
        # Follow-up question using context from previous answer
        question2 = "How long should charging take?"
        
        output2 = client.get_answers(
            question=question2,
            answer_context=qna.KnowledgeBaseAnswerContext(
                previous_qna_id=answer1.qna_id
            ),
            project_name=knowledge_base_project,
            deployment_name=deployment_name
        )
        
        if output2.answers:
            answer2 = output2.answers[0]
            print(f"Q2: {question2}")
            print(f"A2: {answer2.answer}")
            print(f"Confidence: {answer2.confidence:.2f}")
        else:
            print("No follow-up answer found")

# Run multi-turn conversation
multi_turn_conversation()
```

### Advanced Q&A with Filtering and Customization

```python
def advanced_question_answering():
    """Advanced Q&A with metadata filtering and customization"""
    
    questions_with_filters = [
        {
            "question": "What are the system requirements?",
            "filters": {
                "metadata": {
                    "category": "technical",
                    "product": "surface"
                }
            }
        },
        {
            "question": "What is the return policy?",
            "filters": {
                "metadata": {
                    "category": "policy"
                }
            }
        }
    ]
    
    for item in questions_with_filters:
        question = item["question"]
        print(f"Q: {question}")
        
        try:
            output = client.get_answers(
                question=question,
                project_name=knowledge_base_project,
                deployment_name=deployment_name,
                top=3,  # Get top 3 answers
                confidence_threshold=0.3,  # Minimum confidence
                answer_context=qna.KnowledgeBaseAnswerContext(
                    previous_qna_id=None
                ),
                ranker_type="Default",
                filters=qna.QueryFilters(
                    metadata_filter=qna.MetadataFilter(
                        metadata=item["filters"]["metadata"]
                    )
                ) if "filters" in item else None
            )
            
            if output.answers:
                print(f"Found {len(output.answers)} answer(s):")
                for idx, answer in enumerate(output.answers, 1):
                    print(f"  {idx}. {answer.answer}")
                    print(f"     Confidence: {answer.confidence:.2f}")
                    print(f"     Source: {answer.source}")
                    if answer.metadata:
                        print(f"     Metadata: {answer.metadata}")
                    print()
            else:
                print("No answers found with the specified filters")
                
        except Exception as e:
            print(f"Error: {str(e)}")
            
        print("-" * 60)

# Run advanced Q&A
advanced_question_answering()
```

### Async Question Answering for High Performance

```python
import asyncio
from azure.ai.language.questionanswering.aio import QuestionAnsweringClient

async def async_question_answering():
    """High-performance async question answering"""
    
    credential = AzureKeyCredential(key)
    
    async with QuestionAnsweringClient(endpoint, credential) as async_client:
        
        questions = [
            "How do I reset my password?",
            "What are the supported file formats?",
            "How to backup my data?",
            "What is the maximum file size?",
            "How to share files with others?"
        ]
        
        # Process questions concurrently
        tasks = []
        for question in questions:
            task = async_client.get_answers(
                question=question,
                project_name=knowledge_base_project,
                deployment_name=deployment_name
            )
            tasks.append((question, task))
        
        # Wait for all results
        for question, task in tasks:
            try:
                output = await task
                if output.answers:
                    answer = output.answers[0]
                    print(f"Q: {question}")
                    print(f"A: {answer.answer}")
                    print(f"Confidence: {answer.confidence:.2f}")
                    print("-" * 40)
                else:
                    print(f"Q: {question}")
                    print("A: No answer found")
                    print("-" * 40)
            except Exception as e:
                print(f"Q: {question}")
                print(f"Error: {str(e)}")
                print("-" * 40)

# Run async Q&A
asyncio.run(async_question_answering())
```

## 🔷 C# Examples

### Basic Question Answering

```csharp
using Azure;
using Azure.AI.Language.QuestionAnswering;
using System;
using System.Threading.Tasks;

class Program
{
    static string endpoint = Environment.GetEnvironmentVariable("LANGUAGE_ENDPOINT");
    static string key = Environment.GetEnvironmentVariable("LANGUAGE_KEY");
    static string projectName = "FAQ-Project";
    static string deploymentName = "production";
    
    static async Task Main(string[] args)
    {
        var credential = new AzureKeyCredential(key);
        var client = new QuestionAnsweringClient(new Uri(endpoint), credential);
        
        await BasicQuestionAnswering(client);
        await QuestionAnsweringFromText(client);
    }
    
    static async Task BasicQuestionAnswering(QuestionAnsweringClient client)
    {
        string question = "How much battery life do I have left?";
        
        try
        {
            Response<AnswersResult> response = await client.GetAnswersAsync(
                question,
                projectName,
                deploymentName);
            
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine($"Q: {question}");
                Console.WriteLine($"A: {answer.Answer}");
                Console.WriteLine($"Confidence: {answer.Confidence:F2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
                break; // Show only the best answer
            }
        }
        catch (RequestFailedException ex)
        {
            Console.WriteLine($"Error: {ex.ErrorCode} - {ex.Message}");
        }
    }
    
    static async Task QuestionAnsweringFromText(QuestionAnsweringClient client)
    {
        string question = "How long does it take to charge?";
        
        var options = new AnswersFromTextOptions(
            question,
            new string[]
            {
                "Power and charging. It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state.",
                "You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges."
            });
        
        try
        {
            Response<AnswersFromTextResult> response = await client.GetAnswersFromTextAsync(options);
            
            foreach (TextAnswer answer in response.Value.Answers)
            {
                if (answer.Confidence > 0.5)
                {
                    Console.WriteLine($"Q: {question}");
                    Console.WriteLine($"A: {answer.Answer}");
                    Console.WriteLine($"Confidence: {answer.Confidence:F2}");
                    Console.WriteLine();
                    break;
                }
            }
        }
        catch (RequestFailedException ex)
        {
            Console.WriteLine($"Error: {ex.ErrorCode} - {ex.Message}");
        }
    }
}
```

### Multi-turn Conversation in C#

```csharp
static async Task MultiTurnConversation(QuestionAnsweringClient client)
{
    // First question
    string question1 = "How long should my Surface battery last?";
    
    var response1 = await client.GetAnswersAsync(question1, projectName, deploymentName);
    
    if (response1.Value.Answers.Count > 0)
    {
        var answer1 = response1.Value.Answers[0];
        Console.WriteLine($"Q1: {question1}");
        Console.WriteLine($"A1: {answer1.Answer}");
        Console.WriteLine($"QnA ID: {answer1.QnaId}");
        Console.WriteLine();
        
        // Follow-up question
        string question2 = "How long should charging take?";
        
        var options = new AnswersOptions
        {
            AnswerContext = new KnowledgeBaseAnswerContext
            {
                PreviousQnaId = answer1.QnaId
            }
        };
        
        var response2 = await client.GetAnswersAsync(question2, projectName, deploymentName, options);
        
        if (response2.Value.Answers.Count > 0)
        {
            var answer2 = response2.Value.Answers[0];
            Console.WriteLine($"Q2: {question2}");
            Console.WriteLine($"A2: {answer2.Answer}");
            Console.WriteLine($"Confidence: {answer2.Confidence:F2}");
        }
    }
}
```

## 🌐 REST API Examples

### Basic Knowledge Base Query

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:query-knowledgebases?projectName=<project-name>&api-version=2021-10-01&deploymentName=production" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "top": 3,
    "question": "How much battery life do I have left?",
    "includeUnstructuredSources": true,
    "confidenceScoreThreshold": 0.3
  }'
```

### Question Answering from Text

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:query-text?api-version=2021-10-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "How long does it take to charge?",
    "records": [
      {
        "id": "1",
        "text": "Power and charging. It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state."
      },
      {
        "id": "2", 
        "text": "You can use the USB port on your Surface Pro 4 power supply to charge other devices, like a phone, while your Surface charges."
      }
    ],
    "language": "en",
    "stringIndexType": "TextElements_v8"
  }'
```

### Multi-turn Conversation

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:query-knowledgebases?projectName=<project-name>&api-version=2021-10-01&deploymentName=production" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "top": 1,
    "question": "How long should charging take?",
    "answerContext": {
      "previousQnaId": 27
    },
    "includeUnstructuredSources": true
  }'
```

### Python REST Implementation

```python
import requests
import json

def question_answering_rest():
    """Question Answering using REST API"""
    
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    project_name = "FAQ-Project"
    deployment_name = "production"
    
    # Knowledge base query
    kb_url = f"{endpoint}/language/:query-knowledgebases"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    params = {
        "projectName": project_name,
        "api-version": "2021-10-01",
        "deploymentName": deployment_name
    }
    
    payload = {
        "top": 3,
        "question": "How much battery life do I have left?",
        "includeUnstructuredSources": True,
        "confidenceScoreThreshold": 0.3
    }
    
    response = requests.post(kb_url, params=params, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        if result["answers"]:
            for idx, answer in enumerate(result["answers"], 1):
                print(f"Answer {idx}:")
                print(f"  Text: {answer['answer']}")
                print(f"  Confidence: {answer['confidence']:.2f}")
                print(f"  Source: {answer.get('source', 'N/A')}")
                if answer.get("metadata"):
                    print(f"  Metadata: {answer['metadata']}")
                print()
        else:
            print("No answers found")
    else:
        print(f"Error: {response.status_code} - {response.text}")

def text_query_rest():
    """Query text directly using REST API"""
    
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/language/:query-text?api-version=2021-10-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "question": "How long does it take to charge?",
        "records": [
            {
                "id": "1",
                "text": "Power and charging. It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state."
            },
            {
                "id": "2",
                "text": "You can use the USB port on your Surface Pro 4 power supply to charge other devices."
            }
        ],
        "language": "en",
        "stringIndexType": "TextElements_v8"
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        if result["answers"]:
            best_answer = result["answers"][0]
            print(f"Answer: {best_answer['answer']}")
            print(f"Confidence: {best_answer['confidence']:.2f}")
            print(f"Span: {best_answer['answerSpan']['text']}")
        else:
            print("No answers found")
    else:
        print(f"Error: {response.status_code} - {response.text}")

# Run REST examples
question_answering_rest()
text_query_rest()
```

## 📊 Response Format

### Knowledge Base Response

```json
{
  "answers": [
    {
      "questions": [
        "How much battery life do I have left?"
      ],
      "answer": "Your Surface battery should last approximately 9 hours of typical use. You can check your current battery level in the system tray.",
      "confidence": 0.87,
      "id": 42,
      "source": "surface-faq.pdf",
      "metadata": {
        "category": "hardware",
        "product": "surface"
      },
      "dialog": {
        "isContextOnly": false,
        "prompts": [
          {
            "displayOrder": 1,
            "qnaId": 43,
            "displayText": "How to extend battery life?"
          }
        ]
      }
    }
  ]
}
```

### Text Query Response

```json
{
  "answers": [
    {
      "answer": "It takes two to four hours to charge the Surface Pro 4 battery fully from an empty state.",
      "confidence": 0.94,
      "answerSpan": {
        "text": "two to four hours",
        "confidenceScore": 0.96,
        "offset": 11,
        "length": 17
      }
    }
  ]
}
```

## 🎯 Use Cases and Patterns

### FAQ Chatbot

```python
def faq_chatbot():
    """Simple FAQ chatbot implementation"""
    
    print("FAQ Chatbot - Type 'quit' to exit")
    print("-" * 40)
    
    conversation_history = []
    
    while True:
        question = input("You: ").strip()
        
        if question.lower() in ['quit', 'exit', 'bye']:
            print("Goodbye!")
            break
            
        if not question:
            continue
            
        try:
            # Use conversation context if available
            answer_context = None
            if conversation_history:
                # Use the most recent QnA ID for context
                answer_context = qna.KnowledgeBaseAnswerContext(
                    previous_qna_id=conversation_history[-1].get('qna_id')
                )
            
            output = client.get_answers(
                question=question,
                project_name=knowledge_base_project,
                deployment_name=deployment_name,
                answer_context=answer_context,
                confidence_threshold=0.3
            )
            
            if output.answers and output.answers[0].confidence > 0.5:
                answer = output.answers[0]
                print(f"Bot: {answer.answer}")
                
                # Store conversation history
                conversation_history.append({
                    'question': question,
                    'answer': answer.answer,
                    'qna_id': answer.qna_id,
                    'confidence': answer.confidence
                })
                
                # Show follow-up prompts if available
                if hasattr(answer, 'dialog') and answer.dialog and answer.dialog.prompts:
                    print("\nSuggested follow-ups:")
                    for prompt in answer.dialog.prompts:
                        print(f"  - {prompt.display_text}")
                    print()
                        
            else:
                print("Bot: I'm sorry, I don't have a good answer for that question.")
                print("Bot: Could you try rephrasing your question?")
                
        except Exception as e:
            print(f"Bot: Sorry, I encountered an error: {str(e)}")
            
        print("-" * 40)

# Run the chatbot
faq_chatbot()
```

### Document Q&A System

```python
def document_qa_system():
    """Query multiple documents for answers"""
    
    # Sample documents (in practice, these would be loaded from files)
    documents = {
        "user_manual": """
        The device should be charged for 2-4 hours initially. 
        To reset the device, hold the power button for 10 seconds.
        The warranty period is 2 years from date of purchase.
        """,
        "troubleshooting": """
        If the device won't turn on, check the battery level.
        For network issues, restart your router and try again.
        Contact support at support@company.com for hardware issues.
        """,
        "specifications": """
        Memory: 8GB RAM, 256GB Storage
        Display: 13.3 inch, 2880x1920 resolution
        Battery: Up to 9 hours typical use
        Weight: 1.7 pounds
        """
    }
    
    questions = [
        "How long should I charge the device initially?",
        "What should I do if the device won't turn on?",
        "How much memory does the device have?",
        "What is the warranty period?",
        "How do I contact support?"
    ]
    
    for question in questions:
        print(f"Q: {question}")
        
        # Combine all documents for comprehensive search
        combined_text = list(documents.values())
        
        input_data = qna.AnswersFromTextOptions(
            question=question,
            text_documents=combined_text
        )
        
        output = client.get_answers_from_text(input_data)
        
        if output.answers and output.answers[0].confidence > 0.6:
            answer = output.answers[0]
            print(f"A: {answer.answer}")
            print(f"Confidence: {answer.confidence:.2f}")
        else:
            print("A: Answer not found in available documents")
            
        print("-" * 50)

# Run document Q&A
document_qa_system()
```

## 🔧 Best Practices

### Error Handling and Fallbacks

```python
def robust_qa_with_fallbacks(question):
    """Implement robust Q&A with fallback strategies"""
    
    try:
        # Try knowledge base first
        output = client.get_answers(
            question=question,
            project_name=knowledge_base_project,
            deployment_name=deployment_name,
            confidence_threshold=0.7
        )
        
        if output.answers and output.answers[0].confidence > 0.7:
            return {
                'answer': output.answers[0].answer,
                'confidence': output.answers[0].confidence,
                'source': 'knowledge_base'
            }
        
        # Fallback to predefined FAQ
        fallback_answers = {
            'contact': "You can contact our support team at support@company.com or call 1-800-SUPPORT.",
            'hours': "Our support hours are Monday-Friday 9AM-5PM EST.",
            'return': "Our return policy allows returns within 30 days of purchase."
        }
        
        question_lower = question.lower()
        for key, answer in fallback_answers.items():
            if key in question_lower:
                return {
                    'answer': answer,
                    'confidence': 0.5,
                    'source': 'fallback'
                }
        
        # Final fallback
        return {
            'answer': "I'm sorry, I couldn't find an answer to your question. Please contact our support team for assistance.",
            'confidence': 0.1,
            'source': 'default'
        }
        
    except Exception as e:
        return {
            'answer': f"I encountered an error while processing your question: {str(e)}",
            'confidence': 0.0,
            'source': 'error'
        }

# Test robust Q&A
test_questions = [
    "How do I reset my password?",
    "What are your business hours?",
    "This is a completely random question with no answer"
]

for q in test_questions:
    result = robust_qa_with_fallbacks(q)
    print(f"Q: {q}")
    print(f"A: {result['answer']}")
    print(f"Source: {result['source']} (Confidence: {result['confidence']:.2f})")
    print("-" * 50)
```

### Performance Optimization

```python
def optimized_batch_qa():
    """Optimize Q&A for batch processing"""
    
    questions = [
        "How do I reset my device?",
        "What is the warranty period?",
        "How to contact support?",
        "What are the system requirements?",
        "How to backup my data?"
    ]
    
    # Use async processing for better performance
    async def process_questions_async():
        credential = AzureKeyCredential(key)
        
        async with QuestionAnsweringClient(endpoint, credential) as async_client:
            tasks = []
            
            for question in questions:
                task = async_client.get_answers(
                    question=question,
                    project_name=knowledge_base_project,
                    deployment_name=deployment_name,
                    top=1,  # Limit to best answer only
                    confidence_threshold=0.3
                )
                tasks.append((question, task))
            
            results = []
            for question, task in tasks:
                try:
                    output = await task
                    if output.answers:
                        results.append({
                            'question': question,
                            'answer': output.answers[0].answer,
                            'confidence': output.answers[0].confidence
                        })
                    else:
                        results.append({
                            'question': question,
                            'answer': 'No answer found',
                            'confidence': 0.0
                        })
                except Exception as e:
                    results.append({
                        'question': question,
                        'answer': f'Error: {str(e)}',
                        'confidence': 0.0
                    })
            
            return results
    
    # Run batch processing
    import asyncio
    results = asyncio.run(process_questions_async())
    
    for result in results:
        print(f"Q: {result['question']}")
        print(f"A: {result['answer']}")
        print(f"Confidence: {result['confidence']:.2f}")
        print("-" * 40)

# Run optimized batch Q&A
optimized_batch_qa()
```

## 📚 Additional Resources

- **[Azure AI Language Q&A Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/question-answering/)**
- **[Custom Question Answering](https://docs.microsoft.com/azure/cognitive-services/language-service/question-answering/how-to/create-test-deploy/)**
- **[REST API Reference](https://docs.microsoft.com/rest/api/language/question-answering/)**
- **[Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-language-questionanswering/)**
- **[Pricing Information](https://azure.microsoft.com/pricing/details/cognitive-services/language-service/)**

## 🎯 AI-102 Exam Tips

- Understand the difference between knowledge base projects and direct text querying
- Practice with multi-turn conversations and context management
- Know how to implement confidence thresholds and fallback strategies
- Understand metadata filtering and custom training
- Practice with both synchronous and asynchronous processing
- Know the limitations and best practices for production deployments