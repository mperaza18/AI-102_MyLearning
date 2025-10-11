# 🔑 Key Phrase Extraction with Azure AI Language

> **Comprehensive code examples for extracting key phrases and concepts using Azure AI Language services**

## 📋 Overview

Azure AI Language's Key Phrase Extraction API identifies the main talking points and important concepts in text. This service is useful for content summarization, topic modeling, and understanding document themes without reading the entire content.

## 🔧 Key Features

- **🎯 Main Topic Identification** - Extract central themes and concepts
- **📝 Content Summarization** - Identify key points for quick overview
- **🏷️ Automatic Tagging** - Generate tags for content categorization
- **🌐 Multi-language Support** - Support for 10+ languages
- **⚡ Real-time Processing** - Fast extraction suitable for real-time applications
- **📊 Confidence Scoring** - Quality assessment for extracted phrases
- **🔄 Batch Processing** - Efficient processing of multiple documents

## 🐍 Python Examples

### Basic Key Phrase Extraction

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def extract_key_phrases_basic():
    """Basic key phrase extraction from text"""
    
    documents = [
        "Dr. Smith has a very modern medical office, and she has great staff. "
        "The technology used is state-of-the-art and the patient care is excellent.",
        
        "Microsoft Azure provides cloud computing services including virtual machines, "
        "storage, databases, and artificial intelligence capabilities for businesses.",
        
        "The restaurant had amazing food with authentic Italian flavors. "
        "The service was outstanding and the atmosphere was perfect for a romantic dinner.",
        
        "Climate change is affecting global weather patterns, leading to more extreme storms, "
        "rising sea levels, and changes in agricultural productivity worldwide."
    ]
    
    response = client.extract_key_phrases(documents, language="en")
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1}:")
            print(f"Text: {documents[idx]}")
            print("Key phrases found:")
            
            for phrase in doc.key_phrases:
                print(f"  - {phrase}")
            print("-" * 60)
        else:
            print(f"Document {idx + 1} has an error: {doc.error}")

# Run basic key phrase extraction
extract_key_phrases_basic()
```

### Advanced Key Phrase Analysis

```python
def analyze_key_phrases_advanced():
    """Advanced analysis with phrase categorization and frequency"""
    
    documents = [
        """
        Artificial intelligence and machine learning are transforming the healthcare industry. 
        Medical professionals are using AI-powered diagnostic tools to improve patient outcomes. 
        Deep learning algorithms can analyze medical images with remarkable accuracy, 
        helping doctors detect diseases earlier than traditional methods.
        """,
        
        """
        Sustainable energy solutions are crucial for environmental protection. 
        Solar panels and wind turbines are becoming more efficient and cost-effective. 
        Battery technology improvements are enabling better energy storage systems, 
        supporting the transition to renewable energy sources.
        """,
        
        """
        Digital transformation is reshaping business operations across industries. 
        Cloud computing platforms provide scalable infrastructure for modern applications. 
        Data analytics and business intelligence tools help companies make informed decisions 
        based on real-time insights and predictive modeling.
        """
    ]
    
    response = client.extract_key_phrases(documents)
    
    # Analyze phrases across all documents
    all_phrases = []
    phrase_frequency = {}
    
    for idx, doc in enumerate(response):
        if not doc.is_error:
            print(f"Document {idx + 1} - Key Phrases:")
            
            # Categorize phrases by length/complexity
            short_phrases = []
            medium_phrases = []
            long_phrases = []
            
            for phrase in doc.key_phrases:
                all_phrases.append(phrase.lower())
                phrase_frequency[phrase.lower()] = phrase_frequency.get(phrase.lower(), 0) + 1
                
                word_count = len(phrase.split())
                if word_count <= 2:
                    short_phrases.append(phrase)
                elif word_count <= 4:
                    medium_phrases.append(phrase)
                else:
                    long_phrases.append(phrase)
            
            print(f"  Short phrases (1-2 words): {short_phrases}")
            print(f"  Medium phrases (3-4 words): {medium_phrases}")
            print(f"  Long phrases (5+ words): {long_phrases}")
            print()
    
    # Find most common phrases across documents
    common_phrases = sorted(phrase_frequency.items(), key=lambda x: x[1], reverse=True)
    print("Most common phrases across all documents:")
    for phrase, count in common_phrases[:10]:
        if count > 1:
            print(f"  '{phrase}' - appears {count} times")

# Run advanced analysis
analyze_key_phrases_advanced()
```

### Domain-Specific Key Phrase Extraction

```python
def extract_domain_specific_phrases():
    """Extract key phrases from domain-specific content"""
    
    domains = {
        "Technology": [
            "Our new machine learning platform uses advanced neural networks and "
            "deep learning algorithms to process big data and provide real-time analytics. "
            "The cloud-based infrastructure ensures scalability and high availability.",
            
            "Blockchain technology and cryptocurrency are revolutionizing financial services. "
            "Smart contracts and decentralized applications are enabling new business models "
            "with enhanced security and transparency."
        ],
        
        "Healthcare": [
            "The patient underwent a comprehensive medical examination including blood tests, "
            "X-rays, and MRI scans. The physician prescribed antibiotics and recommended "
            "physical therapy for rehabilitation.",
            
            "Clinical trials for the new cancer treatment showed promising results. "
            "The immunotherapy drug demonstrated significant improvement in patient survival rates "
            "with minimal side effects compared to traditional chemotherapy."
        ],
        
        "Finance": [
            "The quarterly financial report shows strong revenue growth and improved profit margins. "
            "Investment portfolio diversification and risk management strategies have yielded "
            "positive returns despite market volatility.",
            
            "Central bank monetary policy decisions affect interest rates and inflation. "
            "Economic indicators suggest sustainable growth with controlled unemployment rates "
            "and stable consumer price index."
        ]
    }
    
    for domain, texts in domains.items():
        print(f"=== {domain.upper()} DOMAIN ===")
        
        response = client.extract_key_phrases(texts)
        
        domain_phrases = set()
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"\nDocument {idx + 1} key phrases:")
                for phrase in doc.key_phrases:
                    print(f"  - {phrase}")
                    domain_phrases.add(phrase.lower())
        
        print(f"\nUnique phrases in {domain} domain: {len(domain_phrases)}")
        print("-" * 50)

# Run domain-specific extraction
extract_domain_specific_phrases()
```

### Batch Processing with Error Handling

```python
def batch_key_phrase_extraction():
    """Process multiple documents with proper error handling"""
    
    documents = [
        {"id": "1", "language": "en", "text": "Microsoft Azure provides comprehensive cloud services."},
        {"id": "2", "language": "es", "text": "La inteligencia artificial está transformando las industrias."},
        {"id": "3", "language": "en", "text": ""},  # Empty document - will cause error
        {"id": "4", "language": "fr", "text": "L'innovation technologique améliore notre qualité de vie."},
        {"id": "5", "language": "en", "text": "x" * 5200}  # Too long - will cause error
    ]
    
    try:
        response = client.extract_key_phrases(documents)
        
        successful_extractions = 0
        total_phrases = 0
        
        for doc in response:
            if not doc.is_error:
                successful_extractions += 1
                phrases = doc.key_phrases
                total_phrases += len(phrases)
                
                print(f"Document ID: {doc.id}")
                print(f"Key phrases ({len(phrases)} found):")
                for phrase in phrases:
                    print(f"  - {phrase}")
                print()
            else:
                print(f"Document ID: {doc.id} - Error: {doc.error.code}")
                print(f"  Message: {doc.error.message}")
                print()
        
        print(f"Summary:")
        print(f"  Successful extractions: {successful_extractions}/{len(documents)}")
        print(f"  Total key phrases extracted: {total_phrases}")
        print(f"  Average phrases per document: {total_phrases/successful_extractions:.1f}")
        
    except Exception as e:
        print(f"Batch processing failed: {str(e)}")

# Run batch processing
batch_key_phrase_extraction()
```

### Content Tagging and Categorization

```python
def content_tagging_system():
    """Use key phrases for automatic content tagging"""
    
    articles = [
        {
            "title": "Future of Electric Vehicles",
            "content": "Electric vehicles are becoming mainstream with improvements in battery technology, "
                      "charging infrastructure, and government incentives. Tesla, BMW, and other manufacturers "
                      "are investing heavily in electric car production and autonomous driving features."
        },
        {
            "title": "Healthy Cooking Tips",
            "content": "Nutritious meal preparation involves fresh ingredients, balanced portions, and "
                      "proper cooking techniques. Organic vegetables, lean proteins, and whole grains "
                      "provide essential nutrients for a healthy lifestyle and weight management."
        },
        {
            "title": "Remote Work Productivity",
            "content": "Working from home requires effective time management, communication tools, and "
                      "a dedicated workspace. Video conferencing, project management software, and "
                      "collaboration platforms enable distributed teams to maintain productivity."
        }
    ]
    
    # Define category keywords for classification
    categories = {
        "Technology": ["technology", "software", "digital", "automation", "artificial intelligence", 
                      "machine learning", "data", "cloud", "platform", "system"],
        "Automotive": ["vehicle", "car", "electric", "battery", "driving", "automotive", 
                      "transportation", "manufacturing"],
        "Health": ["health", "nutrition", "medical", "wellness", "diet", "exercise", 
                  "lifestyle", "organic", "protein"],
        "Business": ["business", "management", "productivity", "work", "team", "organization", 
                    "strategy", "operations", "remote work"]
    }
    
    for article in articles:
        print(f"Article: {article['title']}")
        
        # Extract key phrases from content
        response = client.extract_key_phrases([article['content']])
        
        if response[0].is_error:
            print(f"Error processing article: {response[0].error}")
            continue
        
        key_phrases = [phrase.lower() for phrase in response[0].key_phrases]
        print(f"Key phrases: {', '.join(response[0].key_phrases)}")
        
        # Categorize based on key phrases
        article_categories = []
        category_scores = {}
        
        for category, keywords in categories.items():
            score = 0
            matched_keywords = []
            
            for phrase in key_phrases:
                for keyword in keywords:
                    if keyword in phrase:
                        score += 1
                        matched_keywords.append(keyword)
            
            if score > 0:
                category_scores[category] = score
                article_categories.append(category)
        
        # Sort categories by relevance
        sorted_categories = sorted(category_scores.items(), key=lambda x: x[1], reverse=True)
        
        print("Suggested categories:")
        for category, score in sorted_categories:
            print(f"  - {category} (relevance: {score})")
        
        print("-" * 60)

# Run content tagging
content_tagging_system()
```

## 🔷 C# Examples

### Basic Key Phrase Extraction

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
        
        KeyPhraseExtractionExample(client);
        BatchKeyPhraseExtraction(client);
    }
    
    static void KeyPhraseExtractionExample(TextAnalyticsClient client)
    {
        var response = client.ExtractKeyPhrases(@"Dr. Smith has a very modern medical office, and she has great staff.");

        Console.WriteLine("Key phrases:");
        foreach (string keyphrase in response.Value)
        {
            Console.WriteLine($"\t{keyphrase}");
        }
    }
    
    static void BatchKeyPhraseExtraction(TextAnalyticsClient client)
    {
        var documents = new List<string>
        {
            "Microsoft Azure provides comprehensive cloud computing services.",
            "The restaurant had amazing food with authentic Italian flavors.",
            "Climate change is affecting global weather patterns worldwide."
        };

        ExtractKeyPhrasesResultCollection keyPhrasesPerDocuments = client.ExtractKeyPhrasesBatch(documents);

        foreach (ExtractKeyPhrasesResult keyPhrasesInDocument in keyPhrasesPerDocuments)
        {
            if (!keyPhrasesInDocument.HasError)
            {
                Console.WriteLine($"Document contains {keyPhrasesInDocument.KeyPhrases.Count} key phrases:");
                foreach (string keyPhrase in keyPhrasesInDocument.KeyPhrases)
                {
                    Console.WriteLine($"  - {keyPhrase}");
                }
            }
            else
            {
                Console.WriteLine($"Error: {keyPhrasesInDocument.Error.ErrorCode} - {keyPhrasesInDocument.Error.Message}");
            }
            Console.WriteLine();
        }
    }
}
```

### Advanced Analysis in C#

```csharp
static void AnalyzeKeyPhrasesAdvanced(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Artificial intelligence and machine learning are transforming healthcare.")
        {
            Language = "en",
        },
        new TextDocumentInput("2", "La inteligencia artificial está revolucionando la medicina.")
        {
            Language = "es",
        }
    };

    var options = new ExtractKeyPhrasesOptions()
    {
        ModelVersion = "latest"
    };

    ExtractKeyPhrasesResultCollection results = client.ExtractKeyPhrasesBatch(documents, options);

    var allPhrases = new Dictionary<string, int>();

    foreach (ExtractKeyPhrasesResult result in results)
    {
        Console.WriteLine($"Document ID: {result.Id}");
        
        if (!result.HasError)
        {
            Console.WriteLine($"  Found {result.KeyPhrases.Count} key phrases:");
            
            foreach (string phrase in result.KeyPhrases)
            {
                Console.WriteLine($"    - {phrase}");
                
                // Count phrase frequency
                string lowerPhrase = phrase.ToLower();
                if (allPhrases.ContainsKey(lowerPhrase))
                    allPhrases[lowerPhrase]++;
                else
                    allPhrases[lowerPhrase] = 1;
            }
        }
        else
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        Console.WriteLine();
    }

    // Display phrase statistics
    Console.WriteLine("Phrase frequency analysis:");
    foreach (var kvp in allPhrases.OrderByDescending(x => x.Value))
    {
        Console.WriteLine($"  '{kvp.Key}' appears {kvp.Value} time(s)");
    }
}
```

## 🌐 REST API Examples

### Basic Key Phrase Extraction

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Dr. Smith has a very modern medical office, and she has great staff."
        }
      ]
    }
  }'
```

### Multi-language Key Phrase Extraction

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/:analyze-text?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Microsoft Azure provides comprehensive cloud computing services."
        },
        {
          "id": "2",
          "language": "es", 
          "text": "La inteligencia artificial está transformando las industrias."
        },
        {
          "id": "3",
          "language": "fr",
          "text": "L innovation technologique améliore notre qualité de vie."
        }
      ]
    }
  }'
```

### Python REST Implementation

```python
import requests
import json

def key_phrase_extraction_rest():
    """Key phrase extraction using REST API"""
    
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/language/:analyze-text?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "kind": "KeyPhraseExtraction",
        "parameters": {
            "modelVersion": "latest"
        },
        "analysisInput": {
            "documents": [
                {
                    "id": "1",
                    "language": "en",
                    "text": "Dr. Smith has a very modern medical office, and she has great staff. "
                           "The technology used is state-of-the-art and the patient care is excellent."
                }
            ]
        }
    }
    
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 200:
        result = response.json()
        
        for document in result["results"]["documents"]:
            print(f"Document ID: {document['id']}")
            print("Key phrases:")
            
            for phrase in document['keyPhrases']:
                print(f"  - {phrase}")
    else:
        print(f"Error: {response.status_code} - {response.text}")

# Run REST example
key_phrase_extraction_rest()
```

## 📊 Response Format

### Sample JSON Response

```json
{
  "kind": "KeyPhraseExtractionResults",
  "results": {
    "documents": [
      {
        "id": "1",
        "keyPhrases": [
          "Dr. Smith",
          "modern medical office",
          "great staff",
          "state-of-the-art technology",
          "excellent patient care"
        ],
        "warnings": []
      }
    ],
    "errors": [],
    "modelVersion": "2023-04-01"
  }
}
```

## 🎯 Use Cases and Applications

### Content Summarization

```python
def content_summarization():
    """Use key phrases to create content summaries"""
    
    long_articles = [
        """
        The global shift towards renewable energy is accelerating as governments and businesses 
        recognize the urgent need to address climate change. Solar power installations have 
        increased dramatically, with photovoltaic technology becoming more efficient and affordable. 
        Wind energy projects are expanding both onshore and offshore, taking advantage of 
        technological improvements in turbine design. Energy storage solutions, particularly 
        battery systems, are crucial for managing the intermittent nature of renewable sources. 
        Government policies and financial incentives are driving investment in clean energy 
        infrastructure. The transition from fossil fuels requires significant changes in energy 
        systems, grid modernization, and workforce development in the renewable energy sector.
        """,
        
        """
        Artificial intelligence is revolutionizing healthcare delivery through innovative applications 
        in medical imaging, drug discovery, and patient care. Machine learning algorithms can 
        analyze medical scans with accuracy matching or exceeding human radiologists. AI-powered 
        diagnostic tools help doctors identify diseases earlier and more accurately. Natural language 
        processing systems extract valuable insights from electronic health records and medical 
        literature. Robotic surgery systems provide enhanced precision and minimally invasive 
        procedures. Personalized medicine approaches use AI to tailor treatments based on individual 
        patient characteristics and genetic profiles. However, challenges remain in data privacy, 
        regulatory approval, and ensuring AI systems are transparent and explainable to healthcare 
        professionals.
        """
    ]
    
    for idx, article in enumerate(long_articles, 1):
        print(f"Article {idx} Summary:")
        print("-" * 30)
        
        response = client.extract_key_phrases([article])
        
        if not response[0].is_error:
            key_phrases = response[0].key_phrases
            
            # Group phrases by importance (length and specificity)
            important_phrases = [p for p in key_phrases if len(p.split()) >= 2]
            single_words = [p for p in key_phrases if len(p.split()) == 1]
            
            print("Main Topics:")
            for phrase in important_phrases[:5]:  # Top 5 important phrases
                print(f"  • {phrase}")
            
            print("\nKey Terms:")
            print(f"  {', '.join(single_words[:8])}")  # Top 8 single terms
            
            print(f"\nTotal key concepts identified: {len(key_phrases)}")
        else:
            print(f"Error: {response[0].error}")
        
        print("\n" + "="*50 + "\n")

# Run content summarization
content_summarization()
```

### SEO Keyword Extraction

```python
def seo_keyword_extraction():
    """Extract SEO keywords from web content"""
    
    web_content = [
        """
        Digital marketing strategies for small businesses include search engine optimization, 
        social media marketing, content creation, and email campaigns. SEO best practices 
        involve keyword research, on-page optimization, link building, and technical SEO. 
        Social media platforms like Facebook, Instagram, and LinkedIn provide targeted 
        advertising opportunities. Content marketing through blogs, videos, and infographics 
        helps establish thought leadership and drive organic traffic. Pay-per-click advertising 
        on Google Ads and social media can generate immediate results with proper campaign 
        management and budget optimization.
        """,
        
        """
        E-commerce website design requires user-friendly navigation, mobile responsiveness, 
        and secure payment processing. Product catalog management includes high-quality images, 
        detailed descriptions, and inventory tracking. Customer experience optimization involves 
        fast loading times, easy checkout processes, and effective customer support. Shopping 
        cart abandonment can be reduced through email retargeting, guest checkout options, 
        and transparent shipping costs. Conversion rate optimization uses A/B testing, 
        analytics tools, and user feedback to improve sales performance.
        """
    ]
    
    for idx, content in enumerate(web_content, 1):
        print(f"SEO Keywords for Content {idx}:")
        print("-" * 40)
        
        response = client.extract_key_phrases([content])
        
        if not response[0].is_error:
            phrases = response[0].key_phrases
            
            # Categorize keywords by type
            short_tail = [p for p in phrases if len(p.split()) <= 2]
            long_tail = [p for p in phrases if len(p.split()) > 2]
            
            print("Short-tail Keywords (1-2 words):")
            for phrase in short_tail[:10]:
                print(f"  • {phrase}")
            
            print("\nLong-tail Keywords (3+ words):")
            for phrase in long_tail[:8]:
                print(f"  • {phrase}")
            
            # Suggest focus keywords
            print("\nSuggested Focus Keywords:")
            focus_keywords = [p for p in phrases if 
                            len(p.split()) >= 2 and 
                            any(word in p.lower() for word in 
                                ['strategy', 'optimization', 'marketing', 'business', 'design'])][:5]
            for phrase in focus_keywords:
                print(f"  ★ {phrase}")
        
        print("\n" + "="*50 + "\n")

# Run SEO keyword extraction
seo_keyword_extraction()
```

## 🔧 Best Practices

### Phrase Quality Assessment

```python
def assess_phrase_quality():
    """Assess and filter key phrases based on quality criteria"""
    
    text = """
    The innovative artificial intelligence platform leverages advanced machine learning algorithms 
    and deep neural networks to provide comprehensive data analytics solutions. Our cutting-edge 
    technology stack includes natural language processing, computer vision, and predictive modeling 
    capabilities that enable businesses to extract actionable insights from big data repositories.
    """
    
    response = client.extract_key_phrases([text])
    
    if not response[0].is_error:
        phrases = response[0].key_phrases
        
        print("Quality Assessment of Extracted Phrases:")
        print("-" * 45)
        
        high_quality = []
        medium_quality = []
        low_quality = []
        
        for phrase in phrases:
            words = phrase.split()
            word_count = len(words)
            
            # Quality criteria
            has_technical_terms = any(word.lower() in [
                'artificial', 'intelligence', 'machine', 'learning', 'analytics', 
                'technology', 'data', 'algorithm', 'platform'
            ] for word in words)
            
            has_stopwords = any(word.lower() in [
                'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to'
            ] for word in words)
            
            is_meaningful_length = 2 <= word_count <= 5
            
            # Categorize by quality
            if has_technical_terms and is_meaningful_length and not has_stopwords:
                high_quality.append(phrase)
            elif is_meaningful_length and (has_technical_terms or not has_stopwords):
                medium_quality.append(phrase)
            else:
                low_quality.append(phrase)
        
        print("High Quality Phrases (Technical & Meaningful):")
        for phrase in high_quality:
            print(f"  ✓ {phrase}")
        
        print("\nMedium Quality Phrases:")
        for phrase in medium_quality:
            print(f"  ~ {phrase}")
        
        print("\nLow Quality Phrases:")
        for phrase in low_quality:
            print(f"  ✗ {phrase}")
        
        print(f"\nQuality Distribution:")
        print(f"  High: {len(high_quality)} ({len(high_quality)/len(phrases)*100:.1f}%)")
        print(f"  Medium: {len(medium_quality)} ({len(medium_quality)/len(phrases)*100:.1f}%)")
        print(f"  Low: {len(low_quality)} ({len(low_quality)/len(phrases)*100:.1f}%)")

# Run quality assessment
assess_phrase_quality()
```

### Performance Optimization

```python
def optimized_phrase_extraction():
    """Optimize key phrase extraction for large volumes"""
    
    # Simulate large document processing
    documents = [f"Document {i}: Technology innovation drives business transformation and competitive advantage." 
                for i in range(25)]  # 25 documents
    
    # Process in batches (API limit is typically 10 documents per request)
    batch_size = 10
    all_phrases = []
    
    print(f"Processing {len(documents)} documents in batches of {batch_size}")
    
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i + batch_size]
        print(f"Processing batch {i//batch_size + 1}...")
        
        try:
            response = client.extract_key_phrases(batch)
            
            batch_phrases = []
            for doc in response:
                if not doc.is_error:
                    batch_phrases.extend(doc.key_phrases)
                else:
                    print(f"  Error in document: {doc.error}")
            
            all_phrases.extend(batch_phrases)
            print(f"  Extracted {len(batch_phrases)} phrases from batch")
            
        except Exception as e:
            print(f"  Batch failed: {str(e)}")
    
    # Deduplicate and analyze
    unique_phrases = list(set(phrase.lower() for phrase in all_phrases))
    
    print(f"\nResults Summary:")
    print(f"  Total phrases extracted: {len(all_phrases)}")
    print(f"  Unique phrases: {len(unique_phrases)}")
    print(f"  Duplicate rate: {(len(all_phrases) - len(unique_phrases))/len(all_phrases)*100:.1f}%")

# Run optimized processing
optimized_phrase_extraction()
```

## 📚 Additional Resources

- **[Azure AI Language Key Phrases Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/key-phrase-extraction/)**
- **[Supported Languages](https://docs.microsoft.com/azure/cognitive-services/language-service/key-phrase-extraction/language-support)**
- **[REST API Reference](https://docs.microsoft.com/rest/api/language/text-analytics/key-phrases)**
- **[Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)**
- **[Pricing Information](https://azure.microsoft.com/pricing/details/cognitive-services/language-service/)**

## 🎯 AI-102 Exam Tips

- Understand that key phrase extraction identifies main concepts, not just frequent words
- Know the difference between key phrases and entity recognition
- Practice with different text types (technical, casual, formal)
- Understand multi-language support and language detection
- Know batch processing limits and best practices
- Practice integrating key phrases with other text analytics features