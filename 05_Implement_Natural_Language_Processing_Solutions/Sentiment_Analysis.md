# 😊 Sentiment Analysis with Azure AI Language

> **Comprehensive code examples for sentiment analysis and opinion mining using Azure AI Language services**

## 📋 Overview

Azure AI Language's Sentiment Analysis API analyzes text to determine overall sentiment (positive, negative, neutral) with confidence scores. The service also supports opinion mining to extract granular insights about aspects and opinions in the text.

## 🔧 Key Features

- **📊 Document-level Sentiment** - Overall sentiment classification with confidence scores
- **📝 Sentence-level Analysis** - Sentiment analysis for individual sentences
- **🎯 Opinion Mining** - Extract targets (what people are talking about) and assessments (opinions about targets)
- **📈 Confidence Scores** - Numerical confidence for positive, negative, and neutral classifications
- **🌐 Multi-language Support** - Support for 10+ languages including English, Spanish, French, German
- **⚡ Real-time Processing** - Fast analysis suitable for real-time applications

## 🐍 Python Examples

### Basic Sentiment Analysis

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def analyze_basic_sentiment():
    documents = [
        "I had the best day of my life. I decided to go sky-diving and it made me appreciate my whole life so much more.",
        "This was a terrible experience. The service was awful and the food was cold.",
        "The weather is okay today."
    ]
    
    response = client.analyze_sentiment(documents, language="en")
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            print(f"  Text: {documents[idx]}")
            print(f"  Sentiment: {doc.sentiment}")
            print(f"  Confidence Scores:")
            print(f"    Positive: {doc.confidence_scores.positive:.2f}")
            print(f"    Neutral: {doc.confidence_scores.neutral:.2f}")
            print(f"    Negative: {doc.confidence_scores.negative:.2f}")
            print()
        else:
            print(f"Document {idx + 1} has an error: {doc.error}")

# Run the analysis
analyze_basic_sentiment()
```

### Advanced Sentiment Analysis with Opinion Mining

```python
def analyze_sentiment_with_opinion_mining():
    documents = [
        "The food and service were unacceptable. The concierge was nice, however.",
        "The hotel room was clean but the bed was uncomfortable. The staff was very helpful."
    ]
    
    response = client.analyze_sentiment(
        documents, 
        show_opinion_mining=True,
        language="en"
    )
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            print(f"  Overall sentiment: {doc.sentiment}")
            print(f"  Confidence scores: Positive={doc.confidence_scores.positive:.2f}, "
                  f"Neutral={doc.confidence_scores.neutral:.2f}, "
                  f"Negative={doc.confidence_scores.negative:.2f}")
            print()
            
            # Analyze sentence-level sentiment and opinions
            for sentence_idx, sentence in enumerate(doc.sentences):
                print(f"  Sentence {sentence_idx + 1}: '{sentence.text}'")
                print(f"    Sentiment: {sentence.sentiment}")
                print(f"    Confidence: Pos={sentence.confidence_scores.positive:.2f}, "
                      f"Neut={sentence.confidence_scores.neutral:.2f}, "
                      f"Neg={sentence.confidence_scores.negative:.2f}")
                
                # Opinion mining results
                if sentence.mined_opinions:
                    for opinion in sentence.mined_opinions:
                        target = opinion.target
                        print(f"    Target: '{target.text}' (Sentiment: {target.sentiment}, "
                              f"Confidence: {target.confidence_scores.positive:.2f})")
                        
                        for assessment in opinion.assessments:
                            print(f"      Assessment: '{assessment.text}' (Sentiment: {assessment.sentiment}, "
                                  f"Confidence: {assessment.confidence_scores.positive:.2f})")
                print()

# Run opinion mining analysis
analyze_sentiment_with_opinion_mining()
```

### Batch Processing for Large Datasets

```python
def batch_sentiment_analysis():
    # Process multiple documents efficiently
    documents = [
        {"id": "1", "language": "en", "text": "Great product! Highly recommended."},
        {"id": "2", "language": "en", "text": "Poor quality. Would not buy again."},
        {"id": "3", "language": "es", "text": "Me encanta este producto. Excelente calidad."},
        {"id": "4", "language": "fr", "text": "Service client décevant mais produit correct."}
    ]
    
    response = client.analyze_sentiment(documents)
    
    for doc in response:
        if not doc.is_error:
            print(f"Document ID: {doc.id}")
            print(f"  Sentiment: {doc.sentiment}")
            print(f"  Confidence: {doc.confidence_scores.positive:.2f} (pos), "
                  f"{doc.confidence_scores.negative:.2f} (neg)")
        else:
            print(f"Document ID: {doc.id} - Error: {doc.error}")
    print()

# Run batch processing
batch_sentiment_analysis()
```

## 🔷 C# Examples

### Basic Sentiment Analysis

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
        
        AnalyzeSentiment(client);
        SentimentAnalysisWithOpinionMining(client);
    }
    
    static void AnalyzeSentiment(TextAnalyticsClient client)
    {
        string document = "I had the best day of my life. I decided to go sky-diving and it made me appreciate my whole life so much more.";

        try
        {
            Response<DocumentSentiment> response = client.AnalyzeSentiment(document);
            DocumentSentiment docSentiment = response.Value;

            Console.WriteLine($"Document sentiment is {docSentiment.Sentiment} with:");
            Console.WriteLine($"  Positive confidence score: {docSentiment.ConfidenceScores.Positive:0.00}");
            Console.WriteLine($"  Neutral confidence score: {docSentiment.ConfidenceScores.Neutral:0.00}");
            Console.WriteLine($"  Negative confidence score: {docSentiment.ConfidenceScores.Negative:0.00}");
        }
        catch (RequestFailedException exception)
        {
            Console.WriteLine($"Error Code: {exception.ErrorCode}");
            Console.WriteLine($"Message: {exception.Message}");
        }
    }
    
    static void SentimentAnalysisWithOpinionMining(TextAnalyticsClient client)
    {
        var documents = new List<string>
        {
            "The food and service were unacceptable. The concierge was nice, however."
        };

        var options = new AnalyzeSentimentOptions()
        {
            IncludeOpinionMining = true
        };

        AnalyzeSentimentResultCollection reviews = client.AnalyzeSentimentBatch(documents, options);

        foreach (AnalyzeSentimentResult review in reviews)
        {
            Console.WriteLine($"Document sentiment: {review.DocumentSentiment.Sentiment}\n");
            Console.WriteLine($"\tPositive score: {review.DocumentSentiment.ConfidenceScores.Positive:0.00}");
            Console.WriteLine($"\tNegative score: {review.DocumentSentiment.ConfidenceScores.Negative:0.00}");
            Console.WriteLine($"\tNeutral score: {review.DocumentSentiment.ConfidenceScores.Neutral:0.00}\n");
            
            foreach (SentenceSentiment sentence in review.DocumentSentiment.Sentences)
            {
                Console.WriteLine($"\tText: \"{sentence.Text}\"");
                Console.WriteLine($"\tSentence sentiment: {sentence.Sentiment}");
                Console.WriteLine($"\tSentence positive score: {sentence.ConfidenceScores.Positive:0.00}");
                Console.WriteLine($"\tSentence negative score: {sentence.ConfidenceScores.Negative:0.00}");
                Console.WriteLine($"\tSentence neutral score: {sentence.ConfidenceScores.Neutral:0.00}\n");

                foreach (SentenceOpinion sentenceOpinion in sentence.Opinions)
                {
                    Console.WriteLine($"\tTarget: {sentenceOpinion.Target.Text}, Value: {sentenceOpinion.Target.Sentiment}");
                    Console.WriteLine($"\tTarget positive score: {sentenceOpinion.Target.ConfidenceScores.Positive:0.00}");
                    Console.WriteLine($"\tTarget negative score: {sentenceOpinion.Target.ConfidenceScores.Negative:0.00}");
                    
                    foreach (AssessmentSentiment assessment in sentenceOpinion.Assessments)
                    {
                        Console.WriteLine($"\t\tRelated Assessment: {assessment.Text}, Value: {assessment.Sentiment}");
                        Console.WriteLine($"\t\tRelated Assessment positive score: {assessment.ConfidenceScores.Positive:0.00}");
                        Console.WriteLine($"\t\tRelated Assessment negative score: {assessment.ConfidenceScores.Negative:0.00}");
                    }
                }
            }
            Console.WriteLine($"\n");
        }
    }
}
```

## 🌐 REST API Examples

### Basic Sentiment Analysis

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
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
          "text": "I had the best day of my life. This was an amazing experience!"
        },
        {
          "id": "2", 
          "language": "en",
          "text": "This was terrible. I am very disappointed."
        }
      ]
    }
  }'
```

### Sentiment Analysis with Opinion Mining

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
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

### Python using requests library

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
                    "text": "The food and service were unacceptable. The concierge was nice, however."
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

## 📊 Response Format

### Sample JSON Response

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
            "length": 51,
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
          }
        ]
      }
    ]
  }
}
```

## 🎯 Use Cases

### Customer Feedback Analysis

```python
def analyze_customer_feedback():
    """Analyze customer reviews for a product or service"""
    
    feedback = [
        "Great product! Fast shipping and excellent customer service.",
        "The quality is poor and it broke after one week. Very disappointed.",
        "Average product. Nothing special but does the job.",
        "Love the design but the price is too high for what you get.",
        "Outstanding quality and value. Will definitely buy again!"
    ]
    
    response = client.analyze_sentiment(feedback, show_opinion_mining=True)
    
    positive_count = negative_count = neutral_count = 0
    
    for doc in response:
        if doc.sentiment == "positive":
            positive_count += 1
        elif doc.sentiment == "negative":
            negative_count += 1
        else:
            neutral_count += 1
            
        print(f"Review: '{doc.sentences[0].text[:50]}...'")
        print(f"Sentiment: {doc.sentiment} (Confidence: {max(doc.confidence_scores.positive, doc.confidence_scores.negative, doc.confidence_scores.neutral):.2f})")
        print()
    
    print(f"Summary: {positive_count} positive, {negative_count} negative, {neutral_count} neutral reviews")

analyze_customer_feedback()
```

### Social Media Monitoring

```python
def social_media_sentiment():
    """Monitor social media posts for brand sentiment"""
    
    posts = [
        "Just tried @YourBrand's new product. Amazing quality! #love",
        "@YourBrand customer service is terrible. Still waiting for response.",
        "Neutral opinion about @YourBrand. It's okay I guess.",
        "@YourBrand has the best coffee in town! Highly recommended."
    ]
    
    response = client.analyze_sentiment(posts)
    
    brand_sentiment = {"positive": 0, "negative": 0, "neutral": 0}
    
    for idx, doc in enumerate(response):
        brand_sentiment[doc.sentiment] += 1
        print(f"Post {idx + 1}: {doc.sentiment} (Score: {getattr(doc.confidence_scores, doc.sentiment):.2f})")
    
    print(f"\nBrand Sentiment Overview:")
    for sentiment, count in brand_sentiment.items():
        print(f"  {sentiment.title()}: {count} posts")

social_media_sentiment()
```

## 🔧 Best Practices

### Error Handling

```python
def robust_sentiment_analysis(documents):
    """Implement proper error handling for sentiment analysis"""
    
    try:
        response = client.analyze_sentiment(documents)
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"Document {idx + 1}: {doc.sentiment}")
            else:
                print(f"Document {idx + 1} failed: {doc.error.code} - {doc.error.message}")
                
    except Exception as e:
        print(f"API call failed: {str(e)}")
        # Implement retry logic or fallback here

# Test with mixed valid/invalid content
test_docs = [
    "This is a great product!",
    "",  # Empty string - will cause error
    "Bad experience with customer service.",
    "x" * 5200  # Too long - will cause error
]

robust_sentiment_analysis(test_docs)
```

### Performance Optimization

```python
def optimized_sentiment_analysis():
    """Optimize sentiment analysis for better performance"""
    
    # Batch processing - more efficient than individual calls
    documents = [f"Review {i}: This product is {'great' if i % 2 == 0 else 'terrible'}!" 
                for i in range(10)]
    
    # Process in batches of 10 (API limit)
    batch_size = 10
    
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i + batch_size]
        
        response = client.analyze_sentiment(
            batch,
            show_opinion_mining=False,  # Disable if not needed for better performance
            model_version="latest"
        )
        
        for doc in response:
            if not doc.is_error:
                print(f"Sentiment: {doc.sentiment}")

optimized_sentiment_analysis()
```

## 📚 Additional Resources

- **[Azure AI Language Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/sentiment-opinion-mining/)**
- **[REST API Reference](https://docs.microsoft.com/rest/api/language/text-analytics/sentiment)**
- **[Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)**
- **[.NET SDK Documentation](https://docs.microsoft.com/dotnet/api/azure.ai.textanalytics/)**
- **[Pricing Information](https://azure.microsoft.com/pricing/details/cognitive-services/language-service/)**

## 🎯 AI-102 Exam Tips

- Understand the difference between document-level and sentence-level sentiment
- Know how to enable and interpret opinion mining results
- Practice with different confidence score thresholds
- Understand multi-language support capabilities
- Know the API limits and batch processing best practices