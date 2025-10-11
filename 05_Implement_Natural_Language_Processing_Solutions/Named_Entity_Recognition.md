# 🏷️ Named Entity Recognition with Azure AI Language

> **Comprehensive code examples for entity extraction and classification using Azure AI Language services**

## 📋 Overview

Azure AI Language's Named Entity Recognition (NER) API identifies and classifies entities in text such as people, places, organizations, dates, and more. It provides both general entity recognition and specialized healthcare entity recognition.

## 🔧 Key Features

- **👥 Person Recognition** - Names of people with confidence scores
- **🏢 Organization Detection** - Company names, institutions, government bodies
- **📍 Location Identification** - Cities, countries, landmarks, addresses
- **📅 DateTime Extraction** - Dates, times, durations, age references
- **💰 Quantity Recognition** - Numbers, percentages, measurements
- **🌐 Multi-language Support** - Support for 10+ languages
- **🏥 Healthcare Entities** - Medical terms, dosages, procedures (specialized endpoint)
- **🔗 Entity Linking** - Links entities to knowledge bases like Wikipedia

## 🐍 Python Examples

### Basic Named Entity Recognition

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def recognize_entities_basic():
    documents = [
        "I had a wonderful trip to Seattle last week and visited the Space Needle. "
        "I met with Dr. Smith at Microsoft headquarters on January 15th, 2024.",
        "Apple Inc. announced their new iPhone model will be released in September. "
        "The CEO Tim Cook will present at the Cupertino campus."
    ]
    
    response = client.recognize_entities(documents, language="en")
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            print(f"Text: {documents[idx]}")
            print("Entities found:")
            
            for entity in doc.entities:
                print(f"  - Text: '{entity.text}'")
                print(f"    Category: {entity.category}")
                print(f"    Subcategory: {entity.subcategory}")
                print(f"    Confidence: {entity.confidence_score:.2f}")
                print(f"    Offset: {entity.offset}")
                print(f"    Length: {entity.length}")
                print()
        else:
            print(f"Document {idx + 1} has an error: {doc.error}")

# Run basic entity recognition
recognize_entities_basic()
```

### Advanced Entity Recognition with Categories

```python
def recognize_entities_by_category():
    """Organize entities by their categories"""
    
    document = """
    Microsoft Corporation was founded by Bill Gates and Paul Allen in Albuquerque, New Mexico, on April 4, 1975.
    The company is now headquartered in Redmond, Washington. In 2021, Microsoft's revenue was $168 billion.
    Satya Nadella has been the CEO since February 4, 2014.
    """
    
    response = client.recognize_entities([document])
    
    if response[0].is_error:
        print(f"Error: {response[0].error}")
        return
    
    # Group entities by category
    entities_by_category = {}
    
    for entity in response[0].entities:
        category = entity.category
        if category not in entities_by_category:
            entities_by_category[category] = []
        
        entities_by_category[category].append({
            'text': entity.text,
            'subcategory': entity.subcategory,
            'confidence': entity.confidence_score
        })
    
    # Display entities organized by category
    for category, entities in entities_by_category.items():
        print(f"\n{category.upper()} ENTITIES:")
        for entity in entities:
            subcategory = f" ({entity['subcategory']})" if entity['subcategory'] else ""
            print(f"  - {entity['text']}{subcategory} [Confidence: {entity['confidence']:.2f}]")

recognize_entities_by_category()
```

### Healthcare Entity Recognition

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient

async def healthcare_entity_recognition():
    """Extract healthcare-specific entities"""
    
    credential = AzureKeyCredential(key)
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        documents = [
            "Patient John Smith was prescribed 50mg of ibuprofen twice daily for chronic pain. "
            "He has a history of diabetes mellitus and hypertension. Next appointment scheduled for March 15th.",
            "The patient underwent appendectomy last month. Post-operative recovery was normal. "
            "Current medications include 10mg lisinopril daily and 500mg metformin twice daily."
        ]
        
        # Start healthcare analysis
        operation = await client.begin_analyze_healthcare_entities(documents)
        
        # Wait for completion
        result = await operation.result()
        
        async for page in result:
            for idx, doc_result in enumerate(page):
                if not doc_result.is_error:
                    print(f"Healthcare Entities in Document {idx + 1}:")
                    
                    # Group by entity category
                    categories = {}
                    for entity in doc_result.entities:
                        cat = entity.category
                        if cat not in categories:
                            categories[cat] = []
                        categories[cat].append(entity)
                    
                    for category, entities in categories.items():
                        print(f"\n  {category}:")
                        for entity in entities:
                            print(f"    - {entity.text}")
                            if entity.normalized_text:
                                print(f"      Normalized: {entity.normalized_text}")
                            print(f"      Confidence: {entity.confidence_score:.2f}")
                    
                    # Entity relations
                    if doc_result.entity_relations:
                        print(f"\n  Entity Relations:")
                        for relation in doc_result.entity_relations:
                            print(f"    Relation Type: {relation.relation_type}")
                            for role in relation.roles:
                                print(f"      {role.name}: {role.entity.text}")
                    print("-" * 50)

# Run healthcare entity recognition
asyncio.run(healthcare_entity_recognition())
```

### Entity Linking

```python
def recognize_linked_entities():
    """Link entities to external knowledge bases"""
    
    documents = [
        "Microsoft was founded by Bill Gates and Paul Allen. "
        "The company is headquartered in Seattle, Washington.",
        "Apple Inc. is based in Cupertino, California. "
        "Steve Jobs co-founded the company with Steve Wozniak."
    ]
    
    response = client.recognize_linked_entities(documents)
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            
            for entity in doc.entities:
                print(f"\n  Entity: {entity.name}")
                print(f"  Data Source: {entity.data_source}")
                print(f"  URL: {entity.url}")
                print(f"  Language: {entity.language}")
                print(f"  Data Source Entity ID: {entity.data_source_entity_id}")
                
                print("  Matches:")
                for match in entity.matches:
                    print(f"    - Text: '{match.text}'")
                    print(f"      Confidence: {match.confidence_score:.2f}")
                    print(f"      Offset: {match.offset}")

recognize_linked_entities()
```

## 🔷 C# Examples

### Basic Named Entity Recognition

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using System;
using System.Collections.Generic;

class Program
{
    static string endpoint = Environment.GetEnvironmentVariable("LANGUAGE_ENDPOINT");
    static string key = Environment.GetEnvironmentVariable("LANGUAGE_KEY");
    
    static void Main(string[] args)
    {
        var credential = new AzureKeyCredential(key);
        var client = new TextAnalyticsClient(new Uri(endpoint), credential);
        
        EntityRecognitionExample(client);
        EntityLinkingExample(client);
    }
    
    static void EntityRecognitionExample(TextAnalyticsClient client)
    {
        var response = client.RecognizeEntities("I had a wonderful trip to Seattle last week and visited the Space Needle.");
        
        Console.WriteLine("Named Entities:");
        foreach (var entity in response.Value)
        {
            Console.WriteLine($"\tText: {entity.Text}");
            Console.WriteLine($"\tCategory: {entity.Category}");
            Console.WriteLine($"\tSub-Category: {entity.SubCategory}");
            Console.WriteLine($"\tConfidence Score: {entity.ConfidenceScore:F2}");
            Console.WriteLine($"\tLength: {entity.Length}");
            Console.WriteLine($"\tOffset: {entity.Offset}\n");
        }
    }
    
    static void EntityLinkingExample(TextAnalyticsClient client)
    {
        var response = client.RecognizeLinkedEntities(
            "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, " +
            "to develop and sell BASIC interpreters for the Altair 8800");

        Console.WriteLine("Linked Entities:");
        foreach (LinkedEntity entity in response.Value)
        {
            Console.WriteLine($"\tName: {entity.Name}");
            Console.WriteLine($"\tData Source: {entity.DataSource}");
            Console.WriteLine($"\tURL: {entity.Url}");
            Console.WriteLine($"\tEntity Id in Data Source: {entity.DataSourceEntityId}");
            
            foreach (LinkedEntityMatch match in entity.Matches)
            {
                Console.WriteLine($"\t\tText: {match.Text}");
                Console.WriteLine($"\t\tConfidence Score: {match.ConfidenceScore:F2}");
                Console.WriteLine($"\t\tOffset: {match.Offset}");
                Console.WriteLine($"\t\tLength: {match.Length}\n");
            }
        }
    }
}
```

### Batch Processing with Error Handling

```csharp
static void BatchEntityRecognition(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Microsoft was founded by Bill Gates.")
        {
             Language = "en",
        },
        new TextDocumentInput("2", "Apple Inc. fue fundada por Steve Jobs.")
        {
             Language = "es",
        },
        new TextDocumentInput("3", "")
        {
             Language = "en",
        }
    };

    RecognizeEntitiesResultCollection entitiesPerDocuments = client.RecognizeEntitiesBatch(documents);

    foreach (RecognizeEntitiesResult entitiesInDocument in entitiesPerDocuments)
    {
        Console.WriteLine($"Document ID: {entitiesInDocument.Id}");
        
        if (entitiesInDocument.HasError)
        {
            Console.WriteLine($"  Error: {entitiesInDocument.Error.ErrorCode} - {entitiesInDocument.Error.Message}");
        }
        else
        {
            Console.WriteLine($"  Recognized {entitiesInDocument.Entities.Count} entities:");
            foreach (CategorizedEntity entity in entitiesInDocument.Entities)
            {
                Console.WriteLine($"    - {entity.Text} ({entity.Category})");
            }
        }
        Console.WriteLine();
    }
}
```

## 🌐 REST API Examples

### Basic Entity Recognition

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "EntityRecognition",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "I had a wonderful trip to Seattle last week and visited the Space Needle."
        }
      ]
    }
  }'
```

### Entity Linking

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "EntityLinking",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Microsoft was founded by Bill Gates and Paul Allen."
        }
      ]
    }
  }'
```

### Healthcare Entity Recognition

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/analyze-text/jobs?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "Healthcare Entity Recognition Job",
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Patient was prescribed 100mg ibuprofen twice daily."
        }
      ]
    },
    "tasks": [
      {
        "kind": "Healthcare",
        "taskName": "Healthcare Analysis"
      }
    ]
  }'
```

### Python REST Implementation

```python
import requests
import json
import time

def entity_recognition_rest():
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/language/:analyze-text?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "kind": "EntityRecognition",
        "parameters": {
            "modelVersion": "latest"
        },
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": "en",
                    "text": "I had a wonderful trip to Seattle last week and visited the Space Needle."
                }
            ]
        }
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        for document in result["results"]["documents"]:
            print(f"Document ID: {document['id']}")
            print("Entities:")
            
            for entity in document['entities']:
                print(f"  - Text: {entity['text']}")
                print(f"    Category: {entity['category']}")
                print(f"    Confidence: {entity['confidenceScore']:.2f}")
                print(f"    Offset: {entity['offset']}")
                print()
    else:
        print(f"Error: {response.status_code} - {response.text}")

def healthcare_entities_rest():
    """Healthcare entity recognition using REST API"""
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    # Start healthcare analysis job
    url = f"{endpoint}/language/analyze-text/jobs?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "displayName": "Healthcare NER Job",
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": "en",
                    "text": "Patient was prescribed 100mg ibuprofen twice daily for pain management."
                }
            ]
        },
        "tasks": [
            {
                "kind": "Healthcare",
                "taskName": "Healthcare Analysis"
            }
        ]
    }
    
    # Submit job
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 202:
        # Get job location
        operation_location = response.headers.get('operation-location')
        print(f"Job submitted. Checking status...")
        
        # Poll for completion
        while True:
            status_response = requests.get(operation_location, headers={
                "Ocp-Apim-Subscription-Key": key
            })
            
            if status_response.status_code == 200:
                status_result = status_response.json()
                
                if status_result['status'] == 'succeeded':
                    # Extract healthcare entities
                    for task in status_result['tasks']['items']:
                        if task['kind'] == 'HealthcareLROResults':
                            for document in task['results']['documents']:
                                print(f"\nHealthcare entities in document {document['id']}:")
                                for entity in document['entities']:
                                    print(f"  - {entity['text']} ({entity['category']})")
                                    print(f"    Confidence: {entity['confidenceScore']:.2f}")
                    break
                elif status_result['status'] == 'failed':
                    print("Job failed")
                    break
                else:
                    print("Job still running...")
                    time.sleep(2)
            else:
                print(f"Error checking status: {status_response.status_code}")
                break
    else:
        print(f"Error submitting job: {response.status_code} - {response.text}")

# Run REST examples
entity_recognition_rest()
healthcare_entities_rest()
```

## 📊 Response Format

### Entity Recognition Response

```json
{
  "kind": "EntityRecognitionResults",
  "results": {
    "documents": [
      {
        "id": "1",
        "entities": [
          {
            "text": "Seattle",
            "category": "Location",
            "subcategory": "GPE",
            "offset": 32,
            "length": 7,
            "confidenceScore": 0.99
          },
          {
            "text": "last week",
            "category": "DateTime",
            "subcategory": "DateRange",
            "offset": 40,
            "length": 9,
            "confidenceScore": 0.8
          },
          {
            "text": "Space Needle",
            "category": "Location",
            "subcategory": "Structural",
            "offset": 67,
            "length": 12,
            "confidenceScore": 0.96
          }
        ]
      }
    ]
  }
}
```

### Healthcare Entities Response

```json
{
  "jobId": "healthcare-job-123",
  "lastUpdateDateTime": "2024-01-15T10:30:00Z",
  "createdDateTime": "2024-01-15T10:29:45Z",
  "expirationDateTime": "2024-01-16T10:29:45Z",
  "status": "succeeded",
  "tasks": {
    "completed": 1,
    "failed": 0,
    "inProgress": 0,
    "total": 1,
    "items": [
      {
        "kind": "HealthcareLROResults",
        "results": {
          "documents": [
            {
              "id": "1",
              "entities": [
                {
                  "offset": 26,
                  "length": 5,
                  "text": "100mg",
                  "category": "Dosage",
                  "confidenceScore": 0.99
                },
                {
                  "offset": 32,
                  "length": 9,
                  "text": "ibuprofen",
                  "category": "MedicationName",
                  "confidenceScore": 0.97,
                  "normalizedText": "Ibuprofen"
                },
                {
                  "offset": 42,
                  "length": 11,
                  "text": "twice daily",
                  "category": "Frequency",
                  "confidenceScore": 0.95
                }
              ],
              "relations": [
                {
                  "relationType": "DosageOfMedication",
                  "roles": [
                    {
                      "entity": {
                        "offset": 26,
                        "length": 5,
                        "text": "100mg"
                      },
                      "name": "Dosage"
                    },
                    {
                      "entity": {
                        "offset": 32,
                        "length": 9,
                        "text": "ibuprofen"
                      },
                      "name": "Medication"
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

## 🎯 Entity Categories

### General Entity Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Person** | People's names | "John Smith", "Dr. Johnson" |
| **Location** | Geographic locations | "Seattle", "United States", "Space Needle" |
| **Organization** | Companies, institutions | "Microsoft", "Harvard University" |
| **DateTime** | Dates, times, ranges | "January 15th", "last week", "2:30 PM" |
| **Quantity** | Numbers, measurements | "50 miles", "3 hours", "25%" |
| **PersonType** | Job titles, roles | "CEO", "teacher", "customer" |
| **Event** | Named events | "World Cup", "Christmas" |
| **Product** | Commercial products | "iPhone", "Windows 10" |
| **Skill** | Skills and abilities | "programming", "leadership" |

### Healthcare Entity Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **MedicationName** | Drug names | "ibuprofen", "acetaminophen" |
| **Dosage** | Medication dosage | "100mg", "2 tablets" |
| **Frequency** | Dosage frequency | "twice daily", "every 4 hours" |
| **ConditionName** | Medical conditions | "diabetes", "hypertension" |
| **SymptomOrSign** | Medical symptoms | "fever", "headache" |
| **TreatmentName** | Medical treatments | "surgery", "physical therapy" |
| **BodyStructure** | Body parts | "heart", "liver" |
| **MedicationClass** | Drug classifications | "antibiotic", "painkiller" |

## 🔧 Best Practices

### Confidence Score Filtering

```python
def filter_entities_by_confidence(documents, min_confidence=0.8):
    """Filter entities based on confidence scores"""
    
    response = client.recognize_entities(documents)
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            high_confidence_entities = [
                entity for entity in doc.entities 
                if entity.confidence_score >= min_confidence
            ]
            
            print(f"Document {idx + 1} - High confidence entities:")
            for entity in high_confidence_entities:
                print(f"  {entity.text} ({entity.category}) - {entity.confidence_score:.2f}")

# Test confidence filtering
test_docs = ["Microsoft was founded by Bill Gates in Seattle."]
filter_entities_by_confidence(test_docs, min_confidence=0.9)
```

### Entity Deduplication

```python
def deduplicate_entities(documents):
    """Remove duplicate entities across documents"""
    
    response = client.recognize_entities(documents)
    all_entities = {}
    
    for doc in response:
        if not doc.is_error:
            for entity in doc.entities:
                key = (entity.text.lower(), entity.category)
                if key not in all_entities or entity.confidence_score > all_entities[key]['confidence']:
                    all_entities[key] = {
                        'text': entity.text,
                        'category': entity.category,
                        'confidence': entity.confidence_score
                    }
    
    print("Unique entities found:")
    for entity_info in all_entities.values():
        print(f"  {entity_info['text']} ({entity_info['category']}) - {entity_info['confidence']:.2f}")

# Test deduplication
duplicate_docs = [
    "Microsoft was founded by Bill Gates.",
    "Bill Gates co-founded Microsoft Corporation.",
    "The company Microsoft is based in Seattle."
]
deduplicate_entities(duplicate_docs)
```

## 📚 Additional Resources

- **[Azure AI Language NER Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/named-entity-recognition/)**
- **[Healthcare NER Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/text-analytics-for-health/)**
- **[Entity Categories Reference](https://docs.microsoft.com/azure/cognitive-services/language-service/named-entity-recognition/concepts/named-entity-categories)**
- **[REST API Reference](https://docs.microsoft.com/rest/api/language/text-analytics/named-entity-recognition)**
- **[Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)**

## 🎯 AI-102 Exam Tips

- Understand the difference between general and healthcare entity recognition
- Know the main entity categories and their subcategories
- Practice with entity linking and confidence score interpretation
- Understand when to use synchronous vs asynchronous processing
- Know the limits for text length and batch sizes
- Practice with multi-language entity recognition