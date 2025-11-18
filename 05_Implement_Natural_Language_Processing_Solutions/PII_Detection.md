# 🔒 PII Detection & Redaction with Azure AI Language

> **Comprehensive code examples for identifying and redacting personally identifiable information using Azure AI Language services**

## 📋 Overview

Azure AI Language's PII Detection & Redaction API identifies and redacts personally identifiable information (PII) in text documents. This service helps protect privacy by detecting sensitive information like names, addresses, phone numbers, email addresses, and more.

## 🔧 Key Features

- **👤 Personal Information Detection** - Names, addresses, phone numbers, emails
- **💳 Financial Information** - Credit card numbers, bank account details
- **🆔 Identity Documents** - SSNs, passport numbers, driver's licenses
- **🔄 Text Redaction** - Replace PII with placeholder tokens or asterisks
- **🎯 Confidence Scores** - Reliability scores for each detected PII entity
- **🌐 Multi-language Support** - Support for multiple languages
- **⚙️ Customizable Domains** - Healthcare (PHI), finance, and general domains
- **📊 Entity Categories** - Detailed classification of PII types

## 🐍 Python Examples

### Basic PII Detection

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def detect_pii_basic():
    documents = [
        "My name is John Smith and my email is john.smith@contoso.com. "
        "My phone number is (555) 123-4567 and I live at 123 Main St, Seattle, WA 98101.",
        "Patient Sarah Johnson (DOB: 03/15/1985) has SSN 123-45-6789. "
        "Contact her at sarah.j@email.com or call (206) 555-0199."
    ]
    
    response = client.recognize_pii_entities(documents, language="en")
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            print(f"Original text: {documents[idx]}")
            print(f"Redacted text: {doc.redacted_text}")
            print("PII Entities found:")
            
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

# Run basic PII detection
detect_pii_basic()
```

### Advanced PII Detection with Domain Specification

```python
def detect_pii_with_domain():
    """Detect PII with specific domain (PHI for healthcare)"""
    
    healthcare_document = """
    Patient: Emily Davis
    DOB: 12/08/1990
    SSN: 987-65-4321
    Insurance ID: ABC123456789
    Phone: (425) 555-0123
    Email: emily.davis@outlook.com
    Address: 456 Pine Street, Bellevue, WA 98004
    Emergency Contact: Michael Davis (spouse) - (425) 555-0124
    
    Medical Record Number: MR-2024-001234
    Provider: Dr. Amanda Rodriguez, MD
    Date of Service: January 15, 2024
    """
    
    # Use PHI domain for healthcare documents
    response = client.recognize_pii_entities(
        [healthcare_document], 
        language="en",
        domain_filter="phi"  # Protected Health Information
    )
    
    doc = response[0]
    if not doc.is_error:
        print("Healthcare Document PII Analysis:")
        print(f"Original length: {len(healthcare_document)} characters")
        print(f"Redacted text:\n{doc.redacted_text}")
        print(f"\nDetected {len(doc.entities)} PII entities:")
        
        # Group by category
        categories = {}
        for entity in doc.entities:
            category = entity.category
            if category not in categories:
                categories[category] = []
            categories[category].append(entity)
        
        for category, entities in categories.items():
            print(f"\n{category}:")
            for entity in entities:
                subcategory = f" ({entity.subcategory})" if entity.subcategory else ""
                print(f"  - {entity.text}{subcategory} [Confidence: {entity.confidence_score:.2f}]")

detect_pii_with_domain()
```

### Custom PII Categories Selection

```python
def detect_specific_pii_categories():
    """Detect only specific PII categories"""
    
    document = """
    Contact Information:
    - John Doe: john.doe@company.com, (555) 987-6543
    - Jane Smith: jane.smith@company.com, (555) 123-4567
    - Credit Card: 4532 1234 5678 9012
    - Bank Account: 123456789 (Routing: 021000021)
    - Social Security: 123-45-6789
    """
    
    # Specify which categories to detect
    categories_to_include = [
        "Email",
        "PhoneNumber", 
        "CreditCardNumber",
        "USSocialSecurityNumber"
    ]
    
    response = client.recognize_pii_entities(
        [document],
        language="en",
        categories_filter=categories_to_include
    )
    
    doc = response[0]
    if not doc.is_error:
        print("Filtered PII Detection (Email, Phone, Credit Card, SSN only):")
        print(f"Redacted text: {doc.redacted_text}")
        
        print(f"\nDetected entities:")
        for entity in doc.entities:
            print(f"  - Category: {entity.category}")
            print(f"    Text: {entity.text}")
            print(f"    Confidence: {entity.confidence_score:.2f}")
            print()

detect_specific_pii_categories()
```

### Batch Processing with Error Handling

```python
def batch_pii_detection():
    """Process multiple documents with error handling"""
    
    documents = [
        {
            "id": "1",
            "language": "en", 
            "text": "Contact John at john@email.com or call (555) 123-4567."
        },
        {
            "id": "2",
            "language": "en",
            "text": ""  # Empty document - will cause error
        },
        {
            "id": "3", 
            "language": "en",
            "text": "SSN: 123-45-6789, Credit Card: 4532-1234-5678-9012"
        }
    ]
    
    try:
        response = client.recognize_pii_entities(documents)
        
        for doc in response:
            print(f"Document ID: {doc.id}")
            
            if not doc.is_error:
                print(f"  Found {len(doc.entities)} PII entities")
                print(f"  Redacted text: {doc.redacted_text}")
                
                for entity in doc.entities:
                    print(f"    - {entity.category}: {entity.text}")
            else:
                print(f"  Error: {doc.error.code} - {doc.error.message}")
            print()
            
    except Exception as e:
        print(f"API call failed: {str(e)}")

batch_pii_detection()
```

## 🔷 C# Examples

### Basic PII Detection

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
        
        RecognizePiiEntities(client);
        RecognizePiiEntitiesWithDomain(client);
    }
    
    static void RecognizePiiEntities(TextAnalyticsClient client)
    {
        string document = "My name is John Smith and my email is john.smith@contoso.com. " +
                         "My phone number is (555) 123-4567 and I live at 123 Main St, Seattle, WA.";

        try
        {
            Response<PiiEntityCollection> response = client.RecognizePiiEntities(document);
            PiiEntityCollection piiEntities = response.Value;

            Console.WriteLine($"Redacted Text: {piiEntities.RedactedText}");
            Console.WriteLine($"");
            Console.WriteLine($"Recognized {piiEntities.Count} PII entities:");
            
            foreach (PiiEntity entity in piiEntities)
            {
                Console.WriteLine($"  Text: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine($"  Confidence Score: {entity.ConfidenceScore:F2}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine();
            }
        }
        catch (RequestFailedException exception)
        {
            Console.WriteLine($"Error Code: {exception.ErrorCode}");
            Console.WriteLine($"Message: {exception.Message}");
        }
    }
    
    static void RecognizePiiEntitiesWithDomain(TextAnalyticsClient client)
    {
        var documents = new List<string>()
        {
            "Patient: Sarah Johnson. DOB: 03/15/1985. SSN: 123-45-6789. " +
            "Contact: sarah.j@email.com or (206) 555-0199."
        };

        var options = new RecognizePiiEntitiesOptions()
        {
            DomainFilter = PiiEntityDomain.ProtectedHealthInformation
        };

        RecognizePiiEntitiesResultCollection results = client.RecognizePiiEntitiesBatch(documents, options);

        foreach (RecognizePiiEntitiesResult result in results)
        {
            Console.WriteLine($"Document ID: {result.Id}");
            
            if (result.HasError)
            {
                Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
            }
            else
            {
                Console.WriteLine($"  Redacted text: {result.Entities.RedactedText}");
                Console.WriteLine($"  Found {result.Entities.Count} PII entities:");
                
                foreach (PiiEntity entity in result.Entities)
                {
                    Console.WriteLine($"    - {entity.Category}: {entity.Text}");
                }
            }
            Console.WriteLine();
        }
    }
}
```

### Advanced PII Processing

```csharp
static void AdvancedPiiProcessing(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Contact info: john@email.com, (555) 123-4567")
        {
             Language = "en",
        },
        new TextDocumentInput("2", "Información personal: juan@correo.com, (555) 987-6543")
        {
             Language = "es",
        }
    };

    var options = new RecognizePiiEntitiesOptions()
    {
        CategoriesFilter = { 
            PiiEntityCategory.Email, 
            PiiEntityCategory.PhoneNumber 
        }
    };

    RecognizePiiEntitiesResultCollection results = client.RecognizePiiEntitiesBatch(documents, options);

    foreach (RecognizePiiEntitiesResult result in results)
    {
        Console.WriteLine($"Document ID: {result.Id}");
        
        if (!result.HasError)
        {
            Console.WriteLine($"Language: {documents.FirstOrDefault(d => d.Id == result.Id)?.Language}");
            Console.WriteLine($"Original text length: {documents.FirstOrDefault(d => d.Id == result.Id)?.Text.Length}");
            Console.WriteLine($"Redacted text: {result.Entities.RedactedText}");
            
            var entitiesByCategory = result.Entities.GroupBy(e => e.Category);
            foreach (var categoryGroup in entitiesByCategory)
            {
                Console.WriteLine($"  {categoryGroup.Key}:");
                foreach (var entity in categoryGroup)
                {
                    Console.WriteLine($"    - {entity.Text} (Confidence: {entity.ConfidenceScore:F2})");
                }
            }
        }
        Console.WriteLine();
    }
}
```

## 🌐 REST API Examples

### Basic PII Detection

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "PiiEntityRecognition",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "My name is John Smith and my email is john.smith@contoso.com."
        }
      ]
    }
  }'
```

### PII Detection with Domain Filter

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "PiiEntityRecognition",
    "parameters": {
      "modelVersion": "latest",
      "domain": "phi"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Patient Sarah Johnson, DOB: 03/15/1985, SSN: 123-45-6789."
        }
      ]
    }
  }'
```

### PII Detection with Category Filter

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "PiiEntityRecognition",
    "parameters": {
      "modelVersion": "latest",
      "piiCategories": ["Email", "PhoneNumber", "CreditCardNumber"]
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Contact: john@email.com, (555) 123-4567. Credit Card: 4532-1234-5678-9012."
        }
      ]
    }
  }'
```

### Python REST Implementation

```python
import requests
import json

def pii_detection_rest():
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/language/:analyze-text?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "kind": "PiiEntityRecognition",
        "parameters": {
            "modelVersion": "latest",
            "domain": "phi",
            "piiCategories": ["Email", "PhoneNumber", "USSocialSecurityNumber"]
        },
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": "en",
                    "text": "Patient John Smith, SSN: 123-45-6789, Email: john@hospital.com, Phone: (555) 123-4567."
                }
            ]
        }
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        for document in result["results"]["documents"]:
            print(f"Document ID: {document['id']}")
            print(f"Redacted Text: {document['redactedText']}")
            print("PII Entities:")
            
            for entity in document['entities']:
                print(f"  - Text: {entity['text']}")
                print(f"    Category: {entity['category']}")
                print(f"    Confidence: {entity['confidenceScore']:.2f}")
                print(f"    Offset: {entity['offset']}")
                print()
    else:
        print(f"Error: {response.status_code} - {response.text}")

pii_detection_rest()
```

## 📊 Response Format

### Sample JSON Response

```json
{
  "kind": "PiiEntityRecognitionResults",
  "results": {
    "documents": [
      {
        "redactedText": "My name is ****** and my email is ********************. My phone number is ************** and I live at ***************************.",
        "id": "1",
        "entities": [
          {
            "text": "John Smith",
            "category": "Person",
            "offset": 11,
            "length": 10,
            "confidenceScore": 0.99
          },
          {
            "text": "john.smith@contoso.com",
            "category": "Email",
            "offset": 37,
            "length": 22,
            "confidenceScore": 0.8
          },
          {
            "text": "(555) 123-4567",
            "category": "PhoneNumber",
            "offset": 79,
            "length": 14,
            "confidenceScore": 0.8
          },
          {
            "text": "123 Main St, Seattle, WA 98101",
            "category": "Address",
            "offset": 108,
            "length": 31,
            "confidenceScore": 0.95
          }
        ]
      }
    ]
  }
}
```

## 🎯 PII Entity Categories

### Personal Information

| Category | Description | Examples |
|----------|-------------|----------|
| **Person** | Person names | "John Smith", "Dr. Johnson" |
| **Email** | Email addresses | "john@email.com", "user@domain.org" |
| **PhoneNumber** | Phone numbers | "(555) 123-4567", "+1-206-555-0199" |
| **Address** | Physical addresses | "123 Main St, Seattle, WA" |
| **Age** | Person's age | "25 years old", "age 30" |

### Identity Documents

| Category | Description | Examples |
|----------|-------------|----------|
| **USSocialSecurityNumber** | US SSNs | "123-45-6789" |
| **USDriversLicenseNumber** | US driver's licenses | "D123456789" |
| **USPassportNumber** | US passport numbers | "123456789" |
| **ABARoutingNumber** | Bank routing numbers | "021000021" |
| **SWIFTCode** | International bank codes | "CHASUS33" |

### Financial Information

| Category | Description | Examples |
|----------|-------------|----------|
| **CreditCardNumber** | Credit card numbers | "4532-1234-5678-9012" |
| **InternationalBankingAccountNumber** | IBAN numbers | "GB82 WEST 1234 5698 7654 32" |
| **USBankAccountNumber** | US bank accounts | "123456789" |

### Healthcare (PHI Domain)

| Category | Description | Examples |
|----------|-------------|----------|
| **Date** | Medical dates | "01/15/2024" |
| **Name** | Patient names | "John Doe" |
| **MedicalRecordNumber** | Medical record IDs | "MR-123456" |
| **HealthInsuranceNumber** | Insurance IDs | "ABC123456789" |

## 🔧 Best Practices

### Confidence Threshold Implementation

```python
def filter_pii_by_confidence(documents, min_confidence=0.8):
    """Filter PII entities by confidence score"""
    
    response = client.recognize_pii_entities(documents)
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            high_confidence_pii = [
                entity for entity in doc.entities 
                if entity.confidence_score >= min_confidence
            ]
            
            print(f"Document {idx + 1} - High confidence PII entities:")
            for entity in high_confidence_pii:
                print(f"  {entity.text} ({entity.category}) - {entity.confidence_score:.2f}")
            
            # Create custom redaction based on filtered entities
            redacted_text = documents[idx]
            for entity in sorted(high_confidence_pii, key=lambda x: x.offset, reverse=True):
                start = entity.offset
                end = entity.offset + entity.length
                redacted_text = redacted_text[:start] + "*" * entity.length + redacted_text[end:]
            
            print(f"  Custom redacted text: {redacted_text}")
            print()

# Test confidence-based filtering
test_docs = ["Contact John Smith at john@email.com or (555) 123-4567"]
filter_pii_by_confidence(test_docs, min_confidence=0.9)
```

### Custom Redaction Patterns

```python
def custom_redaction_patterns():
    """Implement custom redaction patterns"""
    
    document = "Contact John Smith (SSN: 123-45-6789) at john@email.com"
    
    response = client.recognize_pii_entities([document])
    doc = response[0]
    
    if not doc.is_error:
        custom_redacted = document
        
        # Define custom redaction patterns per category
        redaction_patterns = {
            "Person": "[NAME REDACTED]",
            "USSocialSecurityNumber": "[SSN REDACTED]",
            "Email": "[EMAIL REDACTED]",
            "PhoneNumber": "[PHONE REDACTED]"
        }
        
        # Apply custom redaction (process in reverse order to maintain offsets)
        for entity in sorted(doc.entities, key=lambda x: x.offset, reverse=True):
            pattern = redaction_patterns.get(entity.category, "[PII REDACTED]")
            start = entity.offset
            end = entity.offset + entity.length
            custom_redacted = custom_redacted[:start] + pattern + custom_redacted[end:]
        
        print("Original text:", document)
        print("Default redaction:", doc.redacted_text)
        print("Custom redaction:", custom_redacted)

custom_redaction_patterns()
```

### Bulk Document Processing

```python
def bulk_pii_processing(file_paths):
    """Process multiple files for PII detection"""
    
    results = {}
    
    for file_path in file_paths:
        try:
            with open(file_path, 'r', encoding='utf-8') as file:
                content = file.read()
            
            # Split large documents into chunks (5120 character limit)
            chunk_size = 5000
            chunks = [content[i:i+chunk_size] for i in range(0, len(content), chunk_size)]
            
            file_pii_entities = []
            
            for chunk_idx, chunk in enumerate(chunks):
                response = client.recognize_pii_entities([chunk])
                
                if not response[0].is_error:
                    for entity in response[0].entities:
                        # Adjust offset for chunk position
                        adjusted_entity = {
                            'text': entity.text,
                            'category': entity.category,
                            'confidence': entity.confidence_score,
                            'offset': entity.offset + (chunk_idx * chunk_size)
                        }
                        file_pii_entities.append(adjusted_entity)
            
            results[file_path] = {
                'total_entities': len(file_pii_entities),
                'entities': file_pii_entities
            }
            
            print(f"Processed {file_path}: {len(file_pii_entities)} PII entities found")
            
        except Exception as e:
            print(f"Error processing {file_path}: {str(e)}")
            results[file_path] = {'error': str(e)}
    
    return results

# Example usage (assuming you have text files to process)
# file_list = ['document1.txt', 'document2.txt', 'document3.txt']
# results = bulk_pii_processing(file_list)
```

## 📚 Additional Resources

- **[What is PII detection in Azure AI Language?](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/overview)**
- **[PII Entity Categories](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/concepts/entity-categories)**
- **[REST API Reference](https://learn.microsoft.com/rest/api/language/text-analysis-runtime/)**
- **[Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)**
- **[Privacy and Compliance Guidelines](https://docs.microsoft.com/azure/cognitive-services/language-service/personally-identifiable-information/concepts/privacy)**

## 🎯 AI-102 Exam Tips

- Understand the different PII categories and when to use domain filters
- Know the difference between general PII detection and PHI (healthcare) domain
- Practice with confidence score interpretation and filtering
- Understand custom redaction patterns and use cases
- Know the text length limits and batch processing capabilities
- Practice with multi-language PII detection scenarios