# 🌐 Language Detection

## 📋 Overview

Language Detection is a core Azure AI Language service that automatically identifies the language of input text with confidence scores. This capability is essential for multilingual applications, content routing, and preprocessing for other language services.

### ✨ Key Features
- 🎯 **Automatic Detection**: Identify language from text without prior knowledge
- 📊 **Confidence Scoring**: Get reliability scores for detection results
- 🌍 **120+ Languages**: Support for major world languages and regional variants
- ⚡ **Batch Processing**: Analyze multiple documents in a single request
- 🔄 **Real-time Processing**: Fast detection suitable for interactive applications
- 📝 **Script Detection**: Identify writing systems (Latin, Cyrillic, Arabic, etc.)

### 🎯 Common Use Cases
- **Content Routing**: Direct content to appropriate language-specific systems
- **Multilingual Chatbots**: Automatically switch to user's preferred language
- **Content Moderation**: Apply language-specific filtering rules
- **Analytics**: Understand audience language demographics
- **Translation Preprocessing**: Detect source language before translation

---

## 🐍 Python Implementation

### Basic Language Detection

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Initialize client
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def detect_language_basic():
    """Basic language detection example"""
    documents = [
        "Hello, world!",
        "Bonjour le monde!",
        "¡Hola mundo!",
        "Hallo Welt!",
        "こんにちは世界！",
        "Привет мир!"
    ]
    
    try:
        result = client.detect_language(documents)
        
        for idx, doc in enumerate(result):
            if not doc.is_error:
                detected_language = doc.primary_language
                print(f"Document {idx + 1}: '{documents[idx]}'")
                print(f"  Language: {detected_language.name} ({detected_language.iso6391_name})")
                print(f"  Confidence: {detected_language.confidence_score:.3f}")
                print("---")
            else:
                print(f"Document {idx + 1}: Error - {doc.error.message}")
                
    except Exception as e:
        print(f"Error: {e}")

# Run basic detection
detect_language_basic()
```

### Advanced Language Detection with Batch Processing

```python
def detect_language_batch():
    """Batch language detection with detailed analysis"""
    
    # Mixed language documents
    documents = [
        {
            "id": "1",
            "text": "The quick brown fox jumps over the lazy dog."
        },
        {
            "id": "2", 
            "text": "Le renard brun et rapide saute par-dessus le chien paresseux."
        },
        {
            "id": "3",
            "text": "Der schnelle braune Fuchs springt über den faulen Hund."
        },
        {
            "id": "4",
            "text": "Il veloce volpe marrone salta sopra il cane pigro."
        },
        {
            "id": "5",
            "text": "这是一个测试文档。"  # Chinese
        }
    ]
    
    try:
        # Extract text for detection
        texts = [doc["text"] for doc in documents]
        result = client.detect_language(texts)
        
        print("📊 Batch Language Detection Results")
        print("=" * 50)
        
        for idx, (doc, detection) in enumerate(zip(documents, result)):
            if not detection.is_error:
                language = detection.primary_language
                
                print(f"Document ID: {doc['id']}")
                print(f"Text: {doc['text'][:60]}...")
                print(f"Language: {language.name}")
                print(f"ISO Code: {language.iso6391_name}")
                print(f"Confidence: {language.confidence_score:.3f}")
                
                # Confidence level interpretation
                if language.confidence_score >= 0.9:
                    confidence_level = "Very High"
                elif language.confidence_score >= 0.7:
                    confidence_level = "High" 
                elif language.confidence_score >= 0.5:
                    confidence_level = "Medium"
                else:
                    confidence_level = "Low"
                    
                print(f"Confidence Level: {confidence_level}")
                print("---")
            else:
                print(f"Document {doc['id']}: Error - {detection.error.message}")
                
    except Exception as e:
        print(f"Error in batch detection: {e}")

# Run batch detection
detect_language_batch()
```

### Content Router Implementation

```python
import json
from typing import Dict, List

class ContentLanguageRouter:
    """Route content based on detected language"""
    
    def __init__(self, client: TextAnalyticsClient):
        self.client = client
        self.language_routes = {
            "en": "english_queue",
            "es": "spanish_queue", 
            "fr": "french_queue",
            "de": "german_queue",
            "zh": "chinese_queue",
            "ja": "japanese_queue"
        }
        
    def route_content(self, content_items: List[Dict]) -> Dict:
        """Route content items based on detected language"""
        
        # Extract text for detection
        texts = [item.get("text", "") for item in content_items]
        
        try:
            detection_results = self.client.detect_language(texts)
            routing_results = {
                "successful_routes": [],
                "failed_routes": [],
                "unhandled_languages": []
            }
            
            for item, detection in zip(content_items, detection_results):
                if not detection.is_error:
                    language = detection.primary_language
                    confidence = language.confidence_score
                    lang_code = language.iso6391_name
                    
                    # Only route if confidence is high enough
                    if confidence >= 0.7:
                        if lang_code in self.language_routes:
                            queue = self.language_routes[lang_code]
                            routing_results["successful_routes"].append({
                                "content_id": item.get("id"),
                                "detected_language": language.name,
                                "confidence": confidence,
                                "route": queue,
                                "text_preview": item["text"][:100]
                            })
                        else:
                            routing_results["unhandled_languages"].append({
                                "content_id": item.get("id"),
                                "detected_language": language.name,
                                "confidence": confidence,
                                "text_preview": item["text"][:100]
                            })
                    else:
                        routing_results["failed_routes"].append({
                            "content_id": item.get("id"),
                            "reason": "Low confidence",
                            "confidence": confidence,
                            "detected_language": language.name
                        })
                else:
                    routing_results["failed_routes"].append({
                        "content_id": item.get("id"),
                        "reason": detection.error.message
                    })
                    
            return routing_results
            
        except Exception as e:
            return {"error": str(e)}

# Example usage
def demo_content_router():
    """Demonstrate content routing"""
    router = ContentLanguageRouter(client)
    
    content_items = [
        {"id": "msg_001", "text": "Hello, I need help with my account"},
        {"id": "msg_002", "text": "Hola, necesito ayuda con mi cuenta"},
        {"id": "msg_003", "text": "Bonjour, j'ai besoin d'aide avec mon compte"},
        {"id": "msg_004", "text": "こんにちは、アカウントのサポートが必要です"},
        {"id": "msg_005", "text": "x y z"}  # Ambiguous text
    ]
    
    results = router.route_content(content_items)
    
    print("🚦 Content Routing Results")
    print("=" * 40)
    print(json.dumps(results, indent=2))

# Run content router demo
demo_content_router()
```

### Performance Monitoring

```python
import time
from statistics import mean, stdev

def performance_benchmark():
    """Benchmark language detection performance"""
    
    test_texts = [
        "This is a test document in English.",
        "Ceci est un document de test en français.", 
        "Dies ist ein Testdokument auf Deutsch.",
        "Questo è un documento di prova in italiano.",
        "这是一个中文测试文档。"
    ] * 20  # Repeat for larger batch
    
    print("🔬 Performance Benchmark")
    print("=" * 30)
    
    # Single document timing
    single_times = []
    for text in test_texts[:5]:
        start_time = time.time()
        result = client.detect_language([text])
        end_time = time.time()
        single_times.append(end_time - start_time)
    
    print(f"Single Document Detection:")
    print(f"  Average: {mean(single_times):.3f}s")
    print(f"  Std Dev: {stdev(single_times):.3f}s")
    
    # Batch timing
    batch_start = time.time()
    batch_result = client.detect_language(test_texts)
    batch_end = time.time()
    batch_time = batch_end - batch_start
    
    print(f"\nBatch Detection ({len(test_texts)} documents):")
    print(f"  Total Time: {batch_time:.3f}s")
    print(f"  Per Document: {batch_time/len(test_texts):.3f}s")
    print(f"  Throughput: {len(test_texts)/batch_time:.1f} docs/sec")

# Run performance benchmark
performance_benchmark()
```

---

## 🔷 C# Implementation

### Basic Language Detection

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class LanguageDetectionService
{
    private readonly TextAnalyticsClient _client;
    
    public LanguageDetectionService(string endpoint, string key)
    {
        var credential = new AzureKeyCredential(key);
        _client = new TextAnalyticsClient(new Uri(endpoint), credential);
    }
    
    public async Task DetectLanguageBasicAsync()
    {
        var documents = new List<string>
        {
            "Hello, world!",
            "Bonjour le monde!",
            "¡Hola mundo!",
            "Hallo Welt!",
            "こんにちは世界！",
            "Привет мир!"
        };

        try
        {
            var response = await _client.DetectLanguageBatchAsync(documents);
            
            for (int i = 0; i < documents.Count; i++)
            {
                var result = response.Value[i];
                if (!result.HasError)
                {
                    var language = result.PrimaryLanguage;
                    Console.WriteLine($"Document {i + 1}: '{documents[i]}'");
                    Console.WriteLine($"  Language: {language.Name} ({language.Iso6391Name})");
                    Console.WriteLine($"  Confidence: {language.ConfidenceScore:F3}");
                    Console.WriteLine("---");
                }
                else
                {
                    Console.WriteLine($"Document {i + 1}: Error - {result.Error.Message}");
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }
}
```

### Advanced Batch Processing

```csharp
public class ContentLanguageRouter
{
    private readonly TextAnalyticsClient _client;
    private readonly Dictionary<string, string> _languageRoutes;
    
    public ContentLanguageRouter(TextAnalyticsClient client)
    {
        _client = client;
        _languageRoutes = new Dictionary<string, string>
        {
            ["en"] = "english_queue",
            ["es"] = "spanish_queue",
            ["fr"] = "french_queue", 
            ["de"] = "german_queue",
            ["zh"] = "chinese_queue",
            ["ja"] = "japanese_queue"
        };
    }
    
    public async Task<RoutingResult> RouteContentAsync(List<ContentItem> contentItems)
    {
        var texts = contentItems.Select(item => item.Text).ToList();
        var result = new RoutingResult();
        
        try
        {
            var response = await _client.DetectLanguageBatchAsync(texts);
            
            for (int i = 0; i < contentItems.Count; i++)
            {
                var detection = response.Value[i];
                var item = contentItems[i];
                
                if (!detection.HasError)
                {
                    var language = detection.PrimaryLanguage;
                    var confidence = language.ConfidenceScore;
                    
                    if (confidence >= 0.7)
                    {
                        if (_languageRoutes.ContainsKey(language.Iso6391Name))
                        {
                            result.SuccessfulRoutes.Add(new RouteInfo
                            {
                                ContentId = item.Id,
                                DetectedLanguage = language.Name,
                                Confidence = confidence,
                                Route = _languageRoutes[language.Iso6391Name],
                                TextPreview = item.Text.Substring(0, Math.Min(100, item.Text.Length))
                            });
                        }
                        else
                        {
                            result.UnhandledLanguages.Add(new UnhandledLanguage
                            {
                                ContentId = item.Id,
                                DetectedLanguage = language.Name,
                                Confidence = confidence
                            });
                        }
                    }
                    else
                    {
                        result.FailedRoutes.Add(new FailedRoute
                        {
                            ContentId = item.Id,
                            Reason = "Low confidence",
                            Confidence = confidence
                        });
                    }
                }
                else
                {
                    result.FailedRoutes.Add(new FailedRoute
                    {
                        ContentId = item.Id,
                        Reason = detection.Error.Message
                    });
                }
            }
        }
        catch (Exception ex)
        {
            result.Error = ex.Message;
        }
        
        return result;
    }
}

// Supporting classes
public class ContentItem
{
    public string Id { get; set; }
    public string Text { get; set; }
}

public class RoutingResult
{
    public List<RouteInfo> SuccessfulRoutes { get; set; } = new();
    public List<FailedRoute> FailedRoutes { get; set; } = new();
    public List<UnhandledLanguage> UnhandledLanguages { get; set; } = new();
    public string Error { get; set; }
}

public class RouteInfo
{
    public string ContentId { get; set; }
    public string DetectedLanguage { get; set; }
    public double Confidence { get; set; }
    public string Route { get; set; }
    public string TextPreview { get; set; }
}
```

---

## 🌐 REST API Implementation

### Basic Language Detection

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "text": "Hello, world!"
        },
        {
          "id": "2", 
          "text": "Bonjour le monde!"
        },
        {
          "id": "3",
          "text": "¡Hola mundo!"
        }
      ]
    }
  }'
```

### Python REST Implementation

```python
import requests
import json
import os

class LanguageDetectionREST:
    """REST API implementation for language detection"""
    
    def __init__(self):
        self.endpoint = os.environ["LANGUAGE_ENDPOINT"]
        self.key = os.environ["LANGUAGE_KEY"]
        self.api_version = "2022-05-01"
        
    def detect_language(self, documents):
        """Detect language using REST API"""
        
    url = f"{self.endpoint}/language/:analyze-text?api-version={self.api_version}"
        headers = {
            "Ocp-Apim-Subscription-Key": self.key,
            "Content-Type": "application/json"
        }
        
        # Format documents
        if isinstance(documents, list) and isinstance(documents[0], str):
            # Convert string list to document format
            formatted_docs = [
                {"id": str(i+1), "text": text} 
                for i, text in enumerate(documents)
            ]
        else:
            formatted_docs = documents
            
        payload = {
            "kind": "LanguageDetection",
            "parameters": {
                "modelVersion": "latest"
            },
            "analysisInput": {
                "documents": formatted_docs
            }
        }
        
        try:
            response = requests.post(url, headers=headers, json=payload)
            response.raise_for_status()
            return response.json()
            
        except requests.exceptions.RequestException as e:
            return {"error": str(e)}

# Example usage
def demo_rest_detection():
    """Demonstrate REST API language detection"""
    detector = LanguageDetectionREST()
    
    test_documents = [
        "The weather today is beautiful.",
        "El clima hoy está hermoso.",
        "Le temps aujourd'hui est magnifique.",
        "Das Wetter heute ist schön."
    ]
    
    result = detector.detect_language(test_documents)
    
    if "error" not in result:
        print("🌐 REST API Language Detection Results")
        print("=" * 45)
        
        for doc_result in result["results"]["documents"]:
            doc_id = doc_result["id"]
            detected_language = doc_result["detectedLanguage"]
            
            print(f"Document {doc_id}:")
            print(f"  Text: {test_documents[int(doc_id)-1]}")
            print(f"  Language: {detected_language['name']}")
            print(f"  ISO Code: {detected_language['iso6391Name']}")
            print(f"  Confidence: {detected_language['confidenceScore']:.3f}")
            print("---")
    else:
        print(f"Error: {result['error']}")

# Run REST demo
demo_rest_detection()
```

---

## 📊 Response Format

### Successful Response

```json
{
  "kind": "LanguageDetectionResults",
  "results": {
    "documents": [
      {
        "id": "1",
        "detectedLanguage": {
          "name": "English",
          "iso6391Name": "en",
          "confidenceScore": 0.99
        },
        "warnings": []
      },
      {
        "id": "2", 
        "detectedLanguage": {
          "name": "French",
          "iso6391Name": "fr", 
          "confidenceScore": 0.95
        },
        "warnings": []
      }
    ],
    "errors": [],
    "modelVersion": "2022-10-01"
  }
}
```

### Error Response

```json
{
  "kind": "LanguageDetectionResults",
  "results": {
    "documents": [],
    "errors": [
      {
        "id": "1",
        "error": {
          "code": "InvalidDocument",
          "message": "Document text is empty."
        }
      }
    ],
    "modelVersion": "2022-10-01"
  }
}
```

---

## 🎯 Real-World Use Cases

### 1. Multilingual Customer Support System

```python
class MultilingualSupportRouter:
    """Route customer inquiries to language-specific agents"""
    
    def __init__(self, client):
        self.client = client
        self.agent_assignments = {
            "en": ["agent_001", "agent_002", "agent_003"],
            "es": ["agent_004", "agent_005"], 
            "fr": ["agent_006"],
            "de": ["agent_007"]
        }
        
    def route_inquiry(self, customer_inquiry):
        """Route customer inquiry to appropriate agent"""
        
        result = self.client.detect_language([customer_inquiry["message"]])
        detection = result[0]
        
        if not detection.is_error:
            language = detection.primary_language
            confidence = language.confidence_score
            lang_code = language.iso6391_name
            
            if confidence >= 0.8 and lang_code in self.agent_assignments:
                # Find available agent
                available_agents = self.agent_assignments[lang_code]
                assigned_agent = available_agents[0]  # Simplified assignment
                
                return {
                    "status": "routed",
                    "customer_id": customer_inquiry["customer_id"],
                    "detected_language": language.name,
                    "confidence": confidence,
                    "assigned_agent": assigned_agent,
                    "queue": f"{lang_code}_support"
                }
            else:
                # Route to default English support
                return {
                    "status": "default_route",
                    "customer_id": customer_inquiry["customer_id"],
                    "detected_language": language.name,
                    "confidence": confidence,
                    "assigned_agent": "agent_001",
                    "queue": "default_support",
                    "note": "Low confidence or unsupported language"
                }
        else:
            return {
                "status": "error",
                "customer_id": customer_inquiry["customer_id"],
                "error": detection.error.message
            }

# Example usage
support_router = MultilingualSupportRouter(client)

inquiries = [
    {"customer_id": "cust_001", "message": "I need help with my order"},
    {"customer_id": "cust_002", "message": "Necesito ayuda con mi pedido"},
    {"customer_id": "cust_003", "message": "J'ai besoin d'aide avec ma commande"}
]

for inquiry in inquiries:
    routing_result = support_router.route_inquiry(inquiry)
    print(f"Customer {routing_result['customer_id']}: {routing_result['status']}")
    print(f"  Language: {routing_result.get('detected_language', 'Unknown')}")
    print(f"  Agent: {routing_result.get('assigned_agent', 'None')}")
    print("---")
```

### 2. Content Analytics Dashboard

```python
class ContentLanguageAnalytics:
    """Analyze content language distribution"""
    
    def __init__(self, client):
        self.client = client
        
    def analyze_content_distribution(self, content_samples):
        """Analyze language distribution in content"""
        
        texts = [sample["text"] for sample in content_samples]
        results = self.client.detect_language(texts)
        
        language_stats = {}
        total_content = 0
        high_confidence_count = 0
        
        for sample, detection in zip(content_samples, results):
            if not detection.is_error:
                language = detection.primary_language
                lang_name = language.name
                confidence = language.confidence_score
                
                # Track language statistics
                if lang_name not in language_stats:
                    language_stats[lang_name] = {
                        "count": 0,
                        "total_confidence": 0,
                        "content_samples": []
                    }
                
                language_stats[lang_name]["count"] += 1
                language_stats[lang_name]["total_confidence"] += confidence
                language_stats[lang_name]["content_samples"].append({
                    "id": sample["id"],
                    "confidence": confidence,
                    "preview": sample["text"][:100]
                })
                
                total_content += 1
                if confidence >= 0.9:
                    high_confidence_count += 1
        
        # Calculate analytics
        analytics = {
            "total_content_analyzed": total_content,
            "high_confidence_detections": high_confidence_count,
            "high_confidence_percentage": (high_confidence_count / total_content * 100) if total_content > 0 else 0,
            "language_distribution": {}
        }
        
        for lang, stats in language_stats.items():
            avg_confidence = stats["total_confidence"] / stats["count"]
            percentage = (stats["count"] / total_content * 100) if total_content > 0 else 0
            
            analytics["language_distribution"][lang] = {
                "count": stats["count"],
                "percentage": round(percentage, 2),
                "average_confidence": round(avg_confidence, 3),
                "samples": stats["content_samples"][:3]  # Top 3 samples
            }
        
        return analytics

# Example usage
analytics = ContentLanguageAnalytics(client)

sample_content = [
    {"id": "post_001", "text": "Great product! Highly recommend."},
    {"id": "post_002", "text": "Excelente producto! Lo recomiendo mucho."},
    {"id": "post_003", "text": "Produit excellent! Je le recommande vivement."},
    {"id": "post_004", "text": "Amazing service and fast delivery!"},
    {"id": "post_005", "text": "Servicio increíble y entrega rápida!"}
]

distribution = analytics.analyze_content_distribution(sample_content)
print("📊 Content Language Distribution Analysis")
print("=" * 50)
print(json.dumps(distribution, indent=2))
```

---

## ✅ Best Practices

### 1. Error Handling and Resilience

```python
from azure.core.exceptions import HttpResponseError
import time
import logging

class RobustLanguageDetector:
    """Language detector with comprehensive error handling"""
    
    def __init__(self, client):
        self.client = client
        self.logger = logging.getLogger(__name__)
        
    def detect_with_retry(self, documents, max_retries=3, base_delay=1):
        """Detect language with retry logic"""
        
        for attempt in range(max_retries):
            try:
                result = self.client.detect_language(documents)
                return self._process_results(documents, result)
                
            except HttpResponseError as e:
                if e.status_code == 429:  # Rate limiting
                    delay = base_delay * (2 ** attempt)
                    self.logger.warning(f"Rate limited, retrying in {delay}s...")
                    time.sleep(delay)
                    continue
                elif e.status_code >= 500:  # Server errors
                    if attempt < max_retries - 1:
                        delay = base_delay * (2 ** attempt)
                        self.logger.warning(f"Server error, retrying in {delay}s...")
                        time.sleep(delay)
                        continue
                    else:
                        self.logger.error(f"Server error after {max_retries} attempts: {e}")
                        raise
                else:
                    self.logger.error(f"Client error: {e}")
                    raise
                    
            except Exception as e:
                self.logger.error(f"Unexpected error: {e}")
                if attempt < max_retries - 1:
                    time.sleep(base_delay)
                    continue
                else:
                    raise
        
        raise Exception(f"Failed after {max_retries} attempts")
    
    def _process_results(self, documents, results):
        """Process detection results with error handling"""
        processed_results = []
        
        for idx, (doc, result) in enumerate(zip(documents, results)):
            if result.is_error:
                self.logger.warning(f"Document {idx} error: {result.error.message}")
                processed_results.append({
                    "document_index": idx,
                    "text": doc,
                    "error": result.error.message,
                    "language": None,
                    "confidence": 0.0
                })
            else:
                language = result.primary_language
                processed_results.append({
                    "document_index": idx,
                    "text": doc,
                    "error": None,
                    "language": language.name,
                    "iso_code": language.iso6391_name,
                    "confidence": language.confidence_score
                })
        
        return processed_results

# Example usage with error handling
detector = RobustLanguageDetector(client)

test_docs = [
    "Hello world",
    "",  # Empty document - will cause error
    "Bonjour monde", 
    "x" * 10000  # Very long document - might cause error
]

try:
    results = detector.detect_with_retry(test_docs)
    for result in results:
        if result["error"]:
            print(f"Error in document {result['document_index']}: {result['error']}")
        else:
            print(f"Document {result['document_index']}: {result['language']} ({result['confidence']:.3f})")
except Exception as e:
    print(f"Detection failed: {e}")
```

### 2. Performance Optimization

```python
class OptimizedLanguageDetector:
    """Optimized language detector for high-throughput scenarios"""
    
    def __init__(self, client):
        self.client = client
        self.batch_size = 10  # Optimal batch size for Text Analytics
        self.cache = {}  # Simple text-based cache
        
    def detect_language_optimized(self, documents):
        """Optimized detection with caching and batching"""
        
        # Separate cached and non-cached documents
        cached_results = []
        to_detect = []
        doc_mapping = []
        
        for idx, doc in enumerate(documents):
            doc_hash = hash(doc.strip().lower())
            if doc_hash in self.cache:
                cached_results.append((idx, self.cache[doc_hash]))
            else:
                to_detect.append(doc)
                doc_mapping.append((idx, doc_hash))
        
        print(f"📈 Cache hit rate: {len(cached_results)}/{len(documents)} ({len(cached_results)/len(documents)*100:.1f}%)")
        
        # Process non-cached documents in batches
        detection_results = []
        for i in range(0, len(to_detect), self.batch_size):
            batch = to_detect[i:i + self.batch_size]
            batch_results = self.client.detect_language(batch)
            detection_results.extend(batch_results)
        
        # Cache new results
        for (idx, doc_hash), result in zip(doc_mapping, detection_results):
            if not result.is_error:
                language_info = {
                    "language": result.primary_language.name,
                    "iso_code": result.primary_language.iso6391_name,
                    "confidence": result.primary_language.confidence_score
                }
                self.cache[doc_hash] = language_info
        
        # Combine all results
        all_results = [None] * len(documents)
        
        # Add cached results
        for idx, result in cached_results:
            all_results[idx] = result
            
        # Add detection results
        detection_idx = 0
        for idx, _ in doc_mapping:
            result = detection_results[detection_idx]
            if not result.is_error:
                language = result.primary_language
                all_results[idx] = {
                    "language": language.name,
                    "iso_code": language.iso6391_name,
                    "confidence": language.confidence_score
                }
            else:
                all_results[idx] = {"error": result.error.message}
            detection_idx += 1
        
        return all_results

# Performance comparison
def performance_comparison():
    """Compare optimized vs standard detection"""
    
    # Generate test documents with duplicates
    base_texts = [
        "Hello world", "Bonjour monde", "Hola mundo",
        "Guten Tag Welt", "Ciao mondo"
    ]
    test_documents = base_texts * 20  # Create duplicates for cache testing
    
    import random
    random.shuffle(test_documents)
    
    # Standard detection
    start_time = time.time()
    standard_results = client.detect_language(test_documents)
    standard_time = time.time() - start_time
    
    # Optimized detection
    optimizer = OptimizedLanguageDetector(client)
    start_time = time.time()
    optimized_results = optimizer.detect_language_optimized(test_documents)
    optimized_time = time.time() - start_time
    
    print(f"\n⚡ Performance Comparison")
    print(f"Documents processed: {len(test_documents)}")
    print(f"Standard detection: {standard_time:.3f}s")
    print(f"Optimized detection: {optimized_time:.3f}s")
    print(f"Improvement: {((standard_time - optimized_time) / standard_time * 100):.1f}%")

# Run performance comparison
performance_comparison()
```

---

## 🎯 AI-102 Exam Tips

### Key Concepts to Remember

1. **Confidence Thresholds**
   - Use confidence scores to determine reliability
   - Typical thresholds: >0.9 (high), >0.7 (medium), <0.5 (low)

2. **Batch Processing Benefits**
   - More efficient than individual requests
   - Better throughput for large datasets
   - Cost-effective for bulk operations

3. **Error Handling**
   - Always check for `is_error` in responses
   - Implement retry logic for transient failures
   - Handle rate limiting with exponential backoff

4. **Language Codes**
   - Understand ISO 639-1 codes (en, es, fr, etc.)
   - Know major language families and scripts
   - Be aware of regional variants

5. **Integration Patterns**
   - Pre-processing for other language services
   - Content routing in multilingual applications
   - Analytics and reporting use cases

### Common Exam Scenarios

- **Multilingual chatbot routing**
- **Content classification pipelines**
- **Customer support automation**
- **Social media analytics**
- **Document processing workflows**

---

## 📚 Additional Resources

- **📖 [Language Detection Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/language-detection/overview)**
- **🔧 [Python SDK Reference](https://docs.microsoft.com/python/api/azure-ai-textanalytics/azure.ai.textanalytics.textanalyticsclient.detect_language)**
- **🌐 [REST API Reference](https://docs.microsoft.com/rest/api/language/text-analysis-runtime/analyze-text)**
- **💡 [Best Practices Guide](https://docs.microsoft.com/azure/cognitive-services/language-service/concepts/best-practices)**
- **🎓 [AI-102 Study Guide](https://docs.microsoft.com/learn/certifications/exams/ai-102)**

---

*This guide provides comprehensive coverage of Azure AI Language Detection capabilities for the AI-102 certification exam.*