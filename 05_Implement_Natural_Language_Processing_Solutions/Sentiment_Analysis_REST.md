# 🌐 Sentiment Analysis REST API Examples

> **REST API examples for sentiment analysis using Azure AI Language services**

## 📋 Basic REST API Call

### Simple Sentiment Analysis

```bash
curl -i -X POST https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01 \
-H "Content-Type: application/json" \
-H "Ocp-Apim-Subscription-Key: <your-key>" \
-d '{
    "kind": "SentimentAnalysis",
    "parameters": {
        "modelVersion": "latest"
    },
    "analysisInput": {
        "documents": [
            {
                "id": "1",
                "language": "en",
                "text": "I had the best day of my life! This was an amazing experience."
            },
            {
                "id": "2",
                "language": "en", 
                "text": "This was terrible. I am very disappointed with the service."
            }
        ]
    }
}'
```

### Sentiment Analysis with Opinion Mining

```bash
curl -i -X POST https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01 \
-H "Content-Type: application/json" \
-H "Ocp-Apim-Subscription-Key: <your-key>" \
-d '{
    "kind": "SentimentAnalysis",
    "parameters": {
        "modelVersion": "latest",
        "opinionMining": true
    },
    "analysisInput": {
        "documents": [
            {
                "id": "1",
                "language": "en",
                "text": "The food and service were unacceptable. The concierge was nice, however."
            }
        ]
    }
}'
```

## 🔧 Advanced REST Examples

### Multi-language Sentiment Analysis

```bash
curl -i -X POST https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01 \
-H "Content-Type: application/json" \
-H "Ocp-Apim-Subscription-Key: <your-key>" \
-d '{
    "kind": "SentimentAnalysis",
    "parameters": {
        "modelVersion": "latest"
    },
    "analysisInput": {
        "documents": [
            {
                "id": "1",
                "language": "en",
                "text": "Great product! Highly recommended."
            },
            {
                "id": "2",
                "language": "es",
                "text": "Me encanta este producto. Excelente calidad."
            },
            {
                "id": "3",
                "language": "fr",
                "text": "Service client décevant mais produit correct."
            }
        ]
    }
}'
```

### Using PowerShell

```powershell
$endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
$key = "<your-key>"
$uri = "$endpoint/language/:analyze-text?api-version=2022-05-01"

$headers = @{
    "Ocp-Apim-Subscription-Key" = $key
    "Content-Type" = "application/json"
}

$body = @{
    kind = "SentimentAnalysis"
    parameters = @{
        modelVersion = "latest"
        opinionMining = $true
    }
    analysisInput = @{
        documents = @(
            @{
                id = "1"
                language = "en"
                text = "I love this new feature! It works perfectly."
            }
        )
    }
} | ConvertTo-Json -Depth 4

$response = Invoke-RestMethod -Uri $uri -Method Post -Headers $headers -Body $body
$response | ConvertTo-Json -Depth 5
```

### Using Python requests

```python
import requests
import json

def sentiment_analysis_rest():
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/language/:analyze-text?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "kind": "SentimentAnalysis",
        "parameters": {
            "modelVersion": "latest",
            "opinionMining": True
        },
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": "en",
                    "text": "The food was great but the service was slow."
                }
            ]
        }
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        for document in result["results"]["documents"]:
            print(f"Document ID: {document['id']}")
            print(f"Sentiment: {document['sentiment']}")
            print(f"Confidence Scores: {document['confidenceScores']}")
            
            for sentence in document['sentences']:
                print(f"  Sentence: {sentence['text']}")
                print(f"  Sentiment: {sentence['sentiment']}")
                
                if 'opinions' in sentence:
                    for opinion in sentence['opinions']:
                        print(f"    Target: {opinion['target']['text']}")
                        for assessment in opinion['assessments']:
                            print(f"      Assessment: {assessment['text']}")
    else:
        print(f"Error: {response.status_code} - {response.text}")

sentiment_analysis_rest()
```

## 📊 Sample Response

```json
{
  "kind": "SentimentAnalysisResults",
  "results": {
    "documents": [
      {
        "id": "1",
        "sentiment": "mixed",
        "confidenceScores": {
          "positive": 0.47,
          "neutral": 0.0,
          "negative": 0.52
        },
        "sentences": [
          {
            "sentiment": "negative",
            "confidenceScores": {
              "positive": 0.0,
              "neutral": 0.0,
              "negative": 0.99
            },
            "offset": 0,
            "length": 39,
            "text": "The food and service were unacceptable.",
            "opinions": [
              {
                "target": {
                  "sentiment": "negative",
                  "confidenceScores": {
                    "positive": 0.0,
                    "negative": 1.0
                  },
                  "offset": 4,
                  "length": 4,
                  "text": "food"
                },
                "assessments": [
                  {
                    "sentiment": "negative",
                    "confidenceScores": {
                      "positive": 0.0,
                      "negative": 1.0
                    },
                    "offset": 25,
                    "length": 12,
                    "text": "unacceptable",
                    "isNegated": false
                  }
                ]
              }
            ]
          },
          {
            "sentiment": "positive",
            "confidenceScores": {
              "positive": 0.94,
              "neutral": 0.01,
              "negative": 0.05
            },
            "offset": 40,
            "length": 32,
            "text": "The concierge was nice, however.",
            "opinions": [
              {
                "target": {
                  "sentiment": "positive",
                  "confidenceScores": {
                    "positive": 1.0,
                    "negative": 0.0
                  },
                  "offset": 44,
                  "length": 9,
                  "text": "concierge"
                },
                "assessments": [
                  {
                    "sentiment": "positive",
                    "confidenceScores": {
                      "positive": 1.0,
                      "negative": 0.0
                    },
                    "offset": 58,
                    "length": 4,
                    "text": "nice",
                    "isNegated": false
                  }
                ]
              }
            ]
          }
        ]
      }
    ],
    "errors": [],
    "modelVersion": "2022-11-01"
  }
}
```

## 🔧 Error Handling

### Common HTTP Status Codes

```bash
# 400 Bad Request - Invalid request format
curl -i -X POST https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01 \
-H "Content-Type: application/json" \
-H "Ocp-Apim-Subscription-Key: <your-key>" \
-d '{
    "kind": "SentimentAnalysis",
    "analysisInput": {
        "documents": [
            {
                "id": "1",
                "text": ""
            }
        ]
    }
}'
```

### Error Response Format

```json
{
  "error": {
    "code": "InvalidDocument",
    "message": "Document text is empty.",
    "target": "documents[0]"
  }
}
```

## 📚 Additional Resources

- [Azure AI Language REST API Reference](https://learn.microsoft.com/rest/api/language/)
- [Sentiment Analysis API Documentation](https://learn.microsoft.com/azure/ai-services/language-service/sentiment-opinion-mining/)
- [API Rate Limits and Quotas](https://learn.microsoft.com/azure/ai-services/language-service/concepts/data-limits)