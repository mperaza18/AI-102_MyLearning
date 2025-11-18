# 🔷 Named Entity Recognition REST API Examples

> **REST API examples for Named Entity Recognition using Azure AI Language services**

## 🛠️ Setup and Authentication

### Authentication Methods

```bash
# Set environment variables
export LANGUAGE_ENDPOINT="https://your-resource.cognitiveservices.azure.com/"
export LANGUAGE_KEY="your-api-key"
```

### Basic Headers

```bash
# Standard headers for all requests
HEADERS=(-H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" -H "Content-Type: application/json")
```

## 🔧 Basic Examples

### Simple Entity Recognition

```bash
# Basic NER request
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Microsoft was founded by Bill Gates and Paul Allen in 1975. The company is headquartered in Redmond, Washington."
        }
      ]
    }
  }' | jq '.'
```

### Entity Recognition with Categories

```bash
# Request specific entity categories
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "John Smith works at Microsoft in Seattle. He can be reached at john.smith@email.com or +1-555-123-4567."
        }
      ]
    }
  }' | jq '.results.documents[0].entities[] | {text: .text, category: .category, confidenceScore: .confidenceScore}'
```

## 🔧 Advanced Examples

### Batch Processing Multiple Documents

```bash
# Process multiple documents
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Apple Inc. was founded by Steve Jobs in Cupertino, California."
        },
        {
          "id": "2",
          "language": "en", 
          "text": "Google was established in 1998 by Larry Page and Sergey Brin at Stanford University."
        },
        {
          "id": "3",
          "language": "es",
          "text": "Barcelona es una ciudad en España conocida por la Sagrada Familia."
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, entities: [.entities[] | {text: .text, category: .category}]}'
```

### Multi-Language Support

```bash
# Analyze entities in different languages
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "EntityRecognition",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "en",
          "language": "en",
          "text": "Microsoft Corporation is located in Redmond, Washington."
        },
        {
          "id": "fr",
          "language": "fr",
          "text": "Microsoft Corporation est située à Redmond, Washington."
        },
        {
          "id": "de",
          "language": "de",
          "text": "Microsoft Corporation befindet sich in Redmond, Washington."
        }
      ]
    }
  }' | jq '.results.documents[] | {language: .id, entities: [.entities[] | select(.category == "Location" or .category == "Organization")]}'
```

### Entity Linking (Knowledge Base)

```bash
# Entity linking to knowledge base
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Microsoft was founded by Bill Gates. The company is now led by Satya Nadella."
        }
      ]
    }
  }' | jq '.results.documents[0].entities[] | {name: .name, wikipediaId: .id, url: .url, matches: [.matches[] | {text: .text, confidenceScore: .confidenceScore}]}'
```

## 🚀 PowerShell Examples

### Basic NER with PowerShell

```powershell
# PowerShell script for NER
$endpoint = $env:LANGUAGE_ENDPOINT
$key = $env:LANGUAGE_KEY

$headers = @{
    'Ocp-Apim-Subscription-Key' = $key
    'Content-Type' = 'application/json'
}

$body = @{
    kind = "EntityRecognition"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = @(
            @{
                id = "1"
                language = "en"
                text = "Amazon Web Services (AWS) is a subsidiary of Amazon.com Inc., headquartered in Seattle, Washington."
            }
        )
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Display entities
$response.results.documents[0].entities | ForEach-Object {
    Write-Host "Entity: $($_.text) - Category: $($_.category) - Confidence: $($_.confidenceScore)"
}
```

### Batch Processing with PowerShell

```powershell
# Process customer feedback for entity extraction
$customerFeedback = @(
    "I called Apple support about my iPhone 13 Pro issue.",
    "Amazon Prime delivery was fast to New York City.",
    "Tesla Model S is manufactured in Fremont, California."
)

$documents = @()
for ($i = 0; $i -lt $customerFeedback.Length; $i++) {
    $documents += @{
        id = ($i + 1).ToString()
        language = "en"
        text = $customerFeedback[$i]
    }
}

$body = @{
    kind = "EntityRecognition"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = $documents
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Analyze results
foreach ($doc in $response.results.documents) {
    Write-Host "Document $($doc.id):"
    $doc.entities | Group-Object category | ForEach-Object {
        Write-Host "  $($_.Name): $($_.Group.text -join ', ')"
    }
}
```

## 🐍 Python Requests Examples

### Basic NER with Python

```python
import requests
import json
import os

endpoint = os.getenv('LANGUAGE_ENDPOINT')
key = os.getenv('LANGUAGE_KEY')

def analyze_entities(text, language='en'):
    url = f"{endpoint}language/:analyze-text?api-version=2023-04-01"
    
    headers = {
        'Ocp-Apim-Subscription-Key': key,
        'Content-Type': 'application/json'
    }
    
    body = {
        "kind": "EntityRecognition",
        "parameters": {
            "modelVersion": "latest"
        },
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": language,
                    "text": text
                }
            ]
        }
    }
    
    response = requests.post(url, headers=headers, json=body)
    
    if response.status_code == 200:
        result = response.json()
        return result['results']['documents'][0]['entities']
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return None

# Example usage
text = "Apple Inc. is planning to open a new store in Times Square, New York City."
entities = analyze_entities(text)

if entities:
    for entity in entities:
        print(f"Entity: {entity['text']}")
        print(f"Category: {entity['category']}")
        print(f"Confidence: {entity['confidenceScore']:.2f}")
        print("---")
```

### Entity Analysis Pipeline

```python
import requests
import json
from typing import List, Dict

class EntityAnalyzer:
    def __init__(self, endpoint: str, key: str):
        self.endpoint = endpoint
        self.key = key
        self.headers = {
            'Ocp-Apim-Subscription-Key': key,
            'Content-Type': 'application/json'
        }
    
    def analyze_batch(self, texts: List[str], language: str = 'en') -> Dict:
        url = f"{self.endpoint}language/:analyze-text?api-version=2023-04-01"
        
        documents = []
        for i, text in enumerate(texts):
            documents.append({
                "id": str(i + 1),
                "language": language,
                "text": text
            })
        
        body = {
            "kind": "EntityRecognition",
            "parameters": {
                "modelVersion": "latest"
            },
            "analysisInput": {
                "documents": documents
            }
        }
        
        response = requests.post(url, headers=self.headers, json=body)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"API Error: {response.status_code} - {response.text}")
    
    def extract_entities_by_category(self, texts: List[str], categories: List[str] = None) -> Dict:
        result = self.analyze_batch(texts)
        
        extracted_entities = {}
        for doc in result['results']['documents']:
            doc_id = doc['id']
            entities = doc['entities']
            
            if categories:
                entities = [e for e in entities if e['category'] in categories]
            
            extracted_entities[doc_id] = entities
        
        return extracted_entities

# Usage example
analyzer = EntityAnalyzer(endpoint, key)

sample_texts = [
    "Microsoft CEO Satya Nadella announced new AI features at the Seattle headquarters.",
    "Google's parent company Alphabet reported Q3 earnings in Mountain View, California.",
    "Amazon is hiring software engineers for their AWS division in Dublin, Ireland."
]

# Extract only organizations and locations
entities_by_category = analyzer.extract_entities_by_category(
    sample_texts, 
    categories=['Organization', 'Location', 'Person']
)

for doc_id, entities in entities_by_category.items():
    print(f"Document {doc_id}:")
    for category in ['Organization', 'Person', 'Location']:
        category_entities = [e['text'] for e in entities if e['category'] == category]
        if category_entities:
            print(f"  {category}: {', '.join(category_entities)}")
    print()
```

## 📊 Entity Statistics and Analysis

### Entity Frequency Analysis

```bash
# Create a script to analyze entity frequency
cat > analyze_entities.sh << 'EOF'
#!/bin/bash

ENDPOINT=$LANGUAGE_ENDPOINT
KEY=$LANGUAGE_KEY

# Sample news articles
TEXTS=(
    "Apple Inc. announced new iPhone models at their Cupertino headquarters."
    "Microsoft Azure cloud services are expanding to new regions including Tokyo and London."
    "Tesla CEO Elon Musk visited the Berlin Gigafactory last Tuesday."
    "Amazon Web Services reported strong growth in Q3 2023 earnings."
)

echo "Entity Frequency Analysis"
echo "========================"

for i in "${!TEXTS[@]}"; do
    echo "Analyzing text $((i+1)): ${TEXTS[i]}"
    
    curl -s -X POST "${ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
      -H "Ocp-Apim-Subscription-Key: ${KEY}" \
      -H "Content-Type: application/json" \
      -d "{
        \"kind\": \"EntityRecognition\",
        \"parameters\": {
          \"modelVersion\": \"latest\"
        },
        \"analysisInput\": {
          \"documents\": [
            {
              \"id\": \"1\",
              \"language\": \"en\",
              \"text\": \"${TEXTS[i]}\"
            }
          ]
        }
      }" | jq -r '.results.documents[0].entities[] | "\(.category): \(.text) (confidence: \(.confidenceScore))"'
    
    echo "---"
done
EOF

chmod +x analyze_entities.sh
./analyze_entities.sh
```

### Entity Confidence Filtering

```bash
# Filter entities by confidence score
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "John works at Microsoft in Seattle. He might visit Google in California next week."
        }
      ]
    }
  }' | jq '.results.documents[0].entities[] | select(.confidenceScore > 0.8) | {text: .text, category: .category, confidence: .confidenceScore}'
```

## 🔍 Custom Entity Categories

### Organization and Location Focus

```bash
# Focus on business entities
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "IBM and Oracle are competing in the enterprise software market. IBM is headquartered in Armonk, New York, while Oracle is based in Austin, Texas."
        }
      ]
    }
  }' | jq '.results.documents[0].entities[] | select(.category == "Organization" or .category == "Location") | {entity: .text, type: .category, confidence: .confidenceScore}'
```

### Person and Contact Information

```bash
# Extract personal information
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Contact Sarah Johnson at sarah.johnson@company.com or call +1-555-0123. She works in the New York office."
        }
      ]
    }
  }' | jq '.results.documents[0].entities[] | select(.category == "Person" or .category == "Email" or .category == "PhoneNumber") | {text: .text, category: .category}'
```

## 📚 Additional Resources

- [Entity Recognition API Reference](https://docs.microsoft.com/rest/api/language/text-analysis-runtime/analyze-text)
- [Supported Entity Categories](https://docs.microsoft.com/azure/cognitive-services/language-service/named-entity-recognition/concepts/named-entity-categories)
- [Language Support](https://docs.microsoft.com/azure/cognitive-services/language-service/named-entity-recognition/language-support)