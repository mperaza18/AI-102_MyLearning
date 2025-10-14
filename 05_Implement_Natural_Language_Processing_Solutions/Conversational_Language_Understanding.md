# 🤖 Conversational Language Understanding (CLU) with Azure AI Language

> **Comprehensive code examples for building intent recognition and entity extraction models using Azure AI Language CLU services**

## 📋 Overview

Azure AI Language's Conversational Language Understanding (CLU) enables you to build custom natural language understanding models to predict user intents and extract entities from conversational text. CLU is designed for building chatbots, voice assistants, and other conversational AI applications.

## 🔧 Key Features

- **🎯 Intent Recognition** - Classify user utterances into predefined intents with confidence scores
- **🏷️ Entity Extraction** - Extract structured information from user inputs (names, dates, locations, etc.)
- **🌐 Multilingual Support** - Support for multiple languages with single model deployment
- **🔄 Real-time Predictions** - Fast inference suitable for interactive applications
- **📊 Confidence Scoring** - Numerical confidence scores for both intents and entities
- **🛠️ Custom Training** - Train models on your specific domain and use cases

## 🐍 Python Examples

### Basic Intent Recognition and Entity Extraction

```python
import os
from azure.core.credentials import AzureKeyCredential
from azure.ai.language.conversations import ConversationAnalysisClient

# Authentication
clu_endpoint = os.environ["AZURE_CONVERSATIONS_ENDPOINT"]
clu_key = os.environ["AZURE_CONVERSATIONS_KEY"]
project_name = os.environ["AZURE_CONVERSATIONS_PROJECT_NAME"]
deployment_name = os.environ["AZURE_CONVERSATIONS_DEPLOYMENT_NAME"]

def analyze_conversation_basic():
    """Basic example of intent recognition and entity extraction"""
    
    client = ConversationAnalysisClient(clu_endpoint, AzureKeyCredential(clu_key))
    
    with client:
        query = "Send an email to Carol about tomorrow's demo"
        
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": project_name,
                    "deploymentName": deployment_name,
                    "verbose": True
                }
            }
        )
        
        # Display results
        print(f"Query: {result['result']['query']}")
        print(f"Project kind: {result['result']['prediction']['projectKind']}\n")
        
        # Top intent
        print(f"Top intent: {result['result']['prediction']['topIntent']}")
        
        # All intents with confidence scores
        print("\nAll intents:")
        for intent in result['result']['prediction']['intents']:
            print(f"  {intent['category']}: {intent['confidenceScore']:.2f}")
        
        # Entities
        print("\nEntities:")
        for entity in result['result']['prediction']['entities']:
            print(f"  Category: {entity['category']}")
            print(f"  Text: {entity['text']}")
            print(f"  Confidence: {entity['confidenceScore']:.2f}")
            print(f"  Offset: {entity['offset']}, Length: {entity['length']}")
            
            # Handle resolutions if present
            if "resolutions" in entity:
                print("  Resolutions:")
                for resolution in entity["resolutions"]:
                    print(f"    Kind: {resolution['resolutionKind']}")
                    print(f"    Value: {resolution['value']}")
            
            # Handle extra information if present
            if "extraInformation" in entity:
                print("  Extra information:")
                for extra in entity["extraInformation"]:
                    print(f"    Kind: {extra['extraInformationKind']}")
                    if extra["extraInformationKind"] == "ListKey":
                        print(f"    Key: {extra['key']}")
                    elif extra["extraInformationKind"] == "EntitySubtype":
                        print(f"    Value: {extra['value']}")
            print()

# Run the analysis
analyze_conversation_basic()
```

### Advanced CLU with Multiple Queries

```python
def analyze_multiple_queries():
    """Analyze multiple queries in batch for comparison"""
    
    client = ConversationAnalysisClient(clu_endpoint, AzureKeyCredential(clu_key))
    
    test_queries = [
        "Book a flight to New York for next Friday",
        "What's the weather like in Seattle?",
        "Cancel my appointment for tomorrow",
        "Show me emails from John",
        "Turn on the lights in the living room"
    ]
    
    with client:
        for i, query in enumerate(test_queries, 1):
            print(f"\n--- Query {i}: {query} ---")
            
            result = client.analyze_conversation(
                task={
                    "kind": "Conversation",
                    "analysisInput": {
                        "conversationItem": {
                            "participantId": str(i),
                            "id": str(i),
                            "modality": "text",
                            "language": "en",
                            "text": query
                        },
                        "isLoggingEnabled": False
                    },
                    "parameters": {
                        "projectName": project_name,
                        "deploymentName": deployment_name,
                        "verbose": True
                    }
                }
            )
            
            prediction = result['result']['prediction']
            
            # Show top intent and confidence
            top_intent = prediction['topIntent']
            top_confidence = next(
                intent['confidenceScore'] 
                for intent in prediction['intents'] 
                if intent['category'] == top_intent
            )
            
            print(f"Intent: {top_intent} (confidence: {top_confidence:.2f})")
            
            # Show entities if any
            if prediction['entities']:
                print("Entities:")
                for entity in prediction['entities']:
                    print(f"  - {entity['category']}: '{entity['text']}' ({entity['confidenceScore']:.2f})")
            else:
                print("No entities detected")

# Run multiple query analysis
analyze_multiple_queries()
```

### CLU with Error Handling and Logging

```python
import logging
from azure.core.exceptions import HttpResponseError

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def analyze_conversation_with_error_handling():
    """CLU analysis with comprehensive error handling"""
    
    try:
        client = ConversationAnalysisClient(clu_endpoint, AzureKeyCredential(clu_key))
        
        query = "Find me a restaurant near downtown"
        logger.info(f"Analyzing query: {query}")
        
        with client:
            result = client.analyze_conversation(
                task={
                    "kind": "Conversation",
                    "analysisInput": {
                        "conversationItem": {
                            "participantId": "user1",
                            "id": "conv1",
                            "modality": "text",
                            "language": "en",
                            "text": query
                        },
                        "isLoggingEnabled": True
                    },
                    "parameters": {
                        "projectName": project_name,
                        "deploymentName": deployment_name,
                        "verbose": True,
                        "stringIndexType": "TextElement_V8"
                    }
                }
            )
            
            prediction = result['result']['prediction']
            
            # Check confidence threshold
            top_intent = prediction['topIntent']
            top_confidence = max(
                intent['confidenceScore'] 
                for intent in prediction['intents']
            )
            
            if top_confidence < 0.5:
                logger.warning(f"Low confidence intent prediction: {top_intent} ({top_confidence:.2f})")
                print("⚠️ The model is not confident about this prediction.")
            else:
                logger.info(f"High confidence prediction: {top_intent} ({top_confidence:.2f})")
            
            print(f"Query: {query}")
            print(f"Predicted Intent: {top_intent}")
            print(f"Confidence Score: {top_confidence:.2f}")
            
            # Process entities with validation
            entities = prediction.get('entities', [])
            if entities:
                print(f"\nExtracted {len(entities)} entities:")
                for entity in entities:
                    entity_text = entity['text']
                    entity_confidence = entity['confidenceScore']
                    
                    if entity_confidence < 0.7:
                        logger.warning(f"Low confidence entity: {entity_text} ({entity_confidence:.2f})")
                    
                    print(f"  - {entity['category']}: '{entity_text}' (confidence: {entity_confidence:.2f})")
                    
    except HttpResponseError as e:
        logger.error(f"HTTP error occurred: {e.status_code} - {e.message}")
        print(f"❌ API Error: {e.message}")
        
    except Exception as e:
        logger.error(f"Unexpected error: {str(e)}")
        print(f"❌ Error: {str(e)}")

# Run with error handling
analyze_conversation_with_error_handling()
```

## 🌐 REST API Examples

### Basic Intent Recognition Request

```bash
curl -X POST \
  "https://<your-resource>.cognitiveservices.azure.com/language/:analyze-conversations?api-version=2023-04-01" \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{
    "kind": "Conversation",
    "analysisInput": {
      "conversationItem": {
        "id": "1",
        "participantId": "1",
        "text": "Book a table for 2 people at 7 PM"
      }
    },
    "parameters": {
      "projectName": "RestaurantBot",
      "deploymentName": "production",
      "stringIndexType": "TextElement_V8"
    }
  }'
```

### Response Format

```json
{
  "kind": "ConversationResult",
  "result": {
    "query": "Book a table for 2 people at 7 PM",
    "prediction": {
      "topIntent": "BookTable",
      "projectKind": "Conversation",
      "intents": [
        {
          "category": "BookTable",
          "confidenceScore": 0.95
        },
        {
          "category": "GetInfo",
          "confidenceScore": 0.03
        },
        {
          "category": "None",
          "confidenceScore": 0.02
        }
      ],
      "entities": [
        {
          "category": "PartySize",
          "text": "2 people",
          "offset": 17,
          "length": 8,
          "confidenceScore": 0.92
        },
        {
          "category": "Time",
          "text": "7 PM",
          "offset": 29,
          "length": 4,
          "confidenceScore": 0.88,
          "resolutions": [
            {
              "resolutionKind": "DateTimeResolution",
              "dateTimeSubKind": "Time",
              "timex": "T19",
              "value": "19:00:00"
            }
          ]
        }
      ]
    }
  }
}
```

### Multilingual Query Example

```bash
curl -X POST \
  "https://<your-resource>.cognitiveservices.azure.com/language/:analyze-conversations?api-version=2023-04-01" \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{
    "kind": "Conversation",
    "analysisInput": {
      "conversationItem": {
        "id": "1",
        "participantId": "1",
        "text": "Réserver une table pour deux personnes",
        "language": "fr"
      }
    },
    "parameters": {
      "projectName": "MultilingualBot",
      "deploymentName": "production",
      "stringIndexType": "TextElement_V8"
    }
  }'
```

## 💻 C# Examples

### Basic CLU Client Setup

```csharp
using Azure.AI.Language.Conversations;
using Azure.Core;
using System;
using System.Threading.Tasks;

public class CLUExample
{
    private readonly string endpoint = Environment.GetEnvironmentVariable("AZURE_CONVERSATIONS_ENDPOINT");
    private readonly string key = Environment.GetEnvironmentVariable("AZURE_CONVERSATIONS_KEY");
    private readonly string projectName = Environment.GetEnvironmentVariable("AZURE_CONVERSATIONS_PROJECT_NAME");
    private readonly string deploymentName = Environment.GetEnvironmentVariable("AZURE_CONVERSATIONS_DEPLOYMENT_NAME");

    public async Task AnalyzeConversationAsync()
    {
        var client = new ConversationAnalysisClient(new Uri(endpoint), new AzureKeyCredential(key));
        
        var requestData = RequestContent.Create(new
        {
            kind = "Conversation",
            analysisInput = new
            {
                conversationItem = new
                {
                    participantId = "1",
                    id = "1",
                    modality = "text",
                    language = "en",
                    text = "I want to book a flight to Paris"
                }
            },
            parameters = new
            {
                projectName = projectName,
                deploymentName = deploymentName,
                verbose = true
            }
        });

        Response response = await client.AnalyzeConversationAsync(requestData);
        
        // Parse response
        using JsonDocument result = JsonDocument.Parse(response.Content.ToMemory());
        JsonElement prediction = result.RootElement
            .GetProperty("result")
            .GetProperty("prediction");
            
        string topIntent = prediction.GetProperty("topIntent").GetString();
        Console.WriteLine($"Top Intent: {topIntent}");
        
        // Display entities
        if (prediction.TryGetProperty("entities", out JsonElement entities))
        {
            foreach (JsonElement entity in entities.EnumerateArray())
            {
                string category = entity.GetProperty("category").GetString();
                string text = entity.GetProperty("text").GetString();
                double confidence = entity.GetProperty("confidenceScore").GetDouble();
                
                Console.WriteLine($"Entity: {category} = '{text}' (confidence: {confidence:F2})");
            }
        }
    }
}
```

### Integration with Bot Framework

```csharp
using Microsoft.Bot.Builder.AI.Language.Conversations;
using Microsoft.Bot.Builder;
using Microsoft.Extensions.Configuration;

public class FlightBookingRecognizer : IRecognizer
{
    private readonly CluRecognizer _recognizer;

    public FlightBookingRecognizer(IConfiguration configuration)
    {
        var cluIsConfigured = !string.IsNullOrEmpty(configuration["CluProjectName"]) &&
                             !string.IsNullOrEmpty(configuration["CluDeploymentName"]) &&
                             !string.IsNullOrEmpty(configuration["CluAPIKey"]) &&
                             !string.IsNullOrEmpty(configuration["CluAPIHostName"]);
        
        if (cluIsConfigured)
        {
            var cluApplication = new CluApplication(
                configuration["CluProjectName"],
                configuration["CluDeploymentName"],
                configuration["CluAPIKey"],
                "https://" + configuration["CluAPIHostName"]);
                
            var recognizerOptions = new CluOptions(cluApplication)
            {
                Language = "en"
            };

            _recognizer = new CluRecognizer(recognizerOptions);
        }
    }

    public async Task<RecognizerResult> RecognizeAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        return await _recognizer.RecognizeAsync(turnContext, cancellationToken);
    }

    public async Task<T> RecognizeAsync<T>(ITurnContext turnContext, CancellationToken cancellationToken) where T : IRecognizerConvert, new()
    {
        return await _recognizer.RecognizeAsync<T>(turnContext, cancellationToken);
    }
}

// Usage in MainDialog
public async Task<DialogTurnResult> ActStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    var cluResult = await _cluRecognizer.RecognizeAsync<FlightBooking>(stepContext.Context, cancellationToken);

    switch (cluResult.TopIntent().intent)
    {
        case FlightBooking.Intent.BookFlight:
            var bookingDetails = new BookingDetails()
            {
                Destination = cluResult.Entities.toCity,
                Origin = cluResult.Entities.fromCity,
                TravelDate = cluResult.Entities.flightDate,
            };
            return await stepContext.BeginDialogAsync(nameof(BookingDialog), bookingDetails, cancellationToken);

        case FlightBooking.Intent.GetWeather:
            var getWeatherMessageText = "TODO: get weather flow here";
            var getWeatherMessage = MessageFactory.Text(getWeatherMessageText, getWeatherMessageText, InputHints.IgnoringInput);
            await stepContext.Context.SendActivityAsync(getWeatherMessage, cancellationToken);
            break;

        default:
            var didntUnderstandMessageText = $"Sorry, I didn't get that. Please try asking in a different way (intent was {cluResult.TopIntent().intent})";
            var didntUnderstandMessage = MessageFactory.Text(didntUnderstandMessageText, didntUnderstandMessageText, InputHints.IgnoringInput);
            await stepContext.Context.SendActivityAsync(didntUnderstandMessage, cancellationToken);
            break;
    }

    return await stepContext.NextAsync(null, cancellationToken);
}
```

## 📊 Model Training and Management

### Project Import via REST API

```bash
# Import a CLU project
curl -X POST \
  "https://<your-resource>.cognitiveservices.azure.com/language/authoring/analyze-conversations/projects/EmailApp/:import?api-version=2023-04-01" \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{
    "projectFileVersion": "2023-04-01",
    "stringIndexType": "Utf16CodeUnit",
    "metadata": {
      "projectKind": "Conversation",
      "settings": {
        "confidenceThreshold": 0.7
      },
      "projectName": "EmailApp",
      "multilingual": true,
      "description": "Email management bot",
      "language": "en-us"
    },
    "assets": {
      "projectKind": "Conversation",
      "intents": [
        {"category": "ReadEmail"},
        {"category": "DeleteEmail"},
        {"category": "SendEmail"}
      ],
      "entities": [
        {"category": "Sender"},
        {"category": "Recipient"},
        {"category": "Subject"}
      ],
      "utterances": [
        {
          "text": "Show me emails from John",
          "dataset": "Train",
          "intent": "ReadEmail",
          "entities": [
            {
              "category": "Sender",
              "offset": 17,
              "length": 4
            }
          ]
        }
      ]
    }
  }'
```

### Training a Model

```bash
# Start training job
curl -X POST \
  "https://<your-resource>.cognitiveservices.azure.com/language/authoring/analyze-conversations/projects/EmailApp/:train?api-version=2023-04-01" \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{
    "modelLabel": "EmailModel_v1",
    "trainingMode": "standard",
    "trainingConfigVersion": "2023-04-01",
    "evaluationOptions": {
      "kind": "percentage",
      "testingSplitPercentage": 20,
      "trainingSplitPercentage": 80
    }
  }'
```

### Deploying a Model

```bash
# Deploy trained model
curl -X PUT \
  "https://<your-resource>.cognitiveservices.azure.com/language/authoring/analyze-conversations/projects/EmailApp/deployments/production?api-version=2023-04-01" \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{
    "trainedModelLabel": "EmailModel_v1"
  }'
```

## 🔍 Best Practices

### 1. Intent Design

```python
# Good intent naming - specific and actionable
intents = [
    "BookFlight",
    "CancelBooking", 
    "CheckFlightStatus",
    "ChangeFlightDate"
]

# Avoid - too generic
# intents = ["Travel", "Help", "Information"]
```

### 2. Entity Extraction Patterns

```python
def handle_date_entities(entities):
    """Process date entities with proper validation"""
    
    for entity in entities:
        if entity['category'] == 'DateTime':
            # Check for resolution information
            if 'resolutions' in entity:
                for resolution in entity['resolutions']:
                    if resolution['resolutionKind'] == 'DateTimeResolution':
                        # Use structured datetime value
                        parsed_date = resolution['value']
                        print(f"Parsed date: {parsed_date}")
                    else:
                        # Fall back to original text
                        raw_text = entity['text']
                        print(f"Raw date text: {raw_text}")
```

### 3. Confidence Threshold Management

```python
def evaluate_prediction_confidence(prediction, intent_threshold=0.7, entity_threshold=0.6):
    """Evaluate if predictions meet confidence requirements"""
    
    top_intent = prediction['topIntent']
    top_confidence = max(intent['confidenceScore'] for intent in prediction['intents'])
    
    if top_confidence < intent_threshold:
        return {
            'action': 'clarify',
            'message': 'I\'m not sure what you\'re asking. Could you rephrase?'
        }
    
    # Check entity confidence
    low_confidence_entities = [
        entity for entity in prediction.get('entities', [])
        if entity['confidenceScore'] < entity_threshold
    ]
    
    if low_confidence_entities:
        return {
            'action': 'confirm',
            'message': f'I understood you want to {top_intent}, but could you clarify some details?'
        }
    
    return {
        'action': 'proceed',
        'intent': top_intent,
        'entities': prediction.get('entities', [])
    }
```

## 🚀 Advanced Use Cases

### Orchestration Workflow

```python
def analyze_with_orchestration():
    """Handle orchestration between multiple CLU projects"""
    
    client = ConversationAnalysisClient(clu_endpoint, AzureKeyCredential(clu_key))
    
    query = "Reserve a table for 2 at the Italian restaurant"
    
    with client:
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": "OrchestrationProject",  # Orchestration project
                    "deploymentName": "production",
                    "verbose": True
                }
            }
        )
        
        prediction = result['result']['prediction']
        top_intent = prediction['topIntent']
        
        # Handle orchestration results
        if prediction['projectKind'] == 'Orchestration':
            top_intent_object = prediction['intents'][top_intent]
            target_project_kind = top_intent_object['targetProjectKind']
            
            if target_project_kind == 'Conversation':
                # Route to specific CLU project
                clu_response = top_intent_object['result']['prediction']
                print(f"Routed to CLU - Intent: {clu_response['topIntent']}")
                
            elif target_project_kind == 'QuestionAnswering':
                # Route to QnA project
                qna_response = top_intent_object['result']
                print(f"Routed to QnA - Answer: {qna_response['answers'][0]['answer']}")
                
            elif target_project_kind == 'Luis':
                # Route to LUIS project
                luis_response = top_intent_object['result']['prediction']
                print(f"Routed to LUIS - Intent: {luis_response['topIntent']}")

# Run orchestration example
analyze_with_orchestration()
```

## 📚 Related Resources

- **🔗 [CLU REST API Documentation](https://docs.microsoft.com/rest/api/language/2023-04-01/conversational-analysis-runtime)**
- **🐍 [Python SDK Reference](https://docs.microsoft.com/python/api/azure-ai-language-conversations/)**
- **💻 [.NET SDK Reference](https://docs.microsoft.com/dotnet/api/azure.ai.language.conversations/)**
- **🤖 [Bot Framework Integration](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-clu)**
- **🏗️ [CLU Best Practices](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/best-practices)**

## 🎯 AI-102 Exam Focus Areas

- ✅ **Intent Recognition** - Understanding how to classify user intentions
- ✅ **Entity Extraction** - Extracting structured data from natural language
- ✅ **Model Training** - Creating and training custom CLU models
- ✅ **Deployment** - Deploying models and managing versions
- ✅ **Integration** - Connecting CLU with Bot Framework and other services
- ✅ **Multilingual Support** - Building models that work across languages
- ✅ **Orchestration** - Routing queries to appropriate services