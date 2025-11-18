# 🐍 Sentiment Analysis Python SDK Examples

> **Python SDK examples for sentiment analysis using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```bash
pip install azure-ai-textanalytics azure-identity
```

### Basic Authentication

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Using API key
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

# Using Azure Identity (recommended for production)
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)
```

## 🔧 Basic Examples

### Simple Sentiment Analysis

```python
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

analyze_basic_sentiment()
```

### Sentiment Analysis with Opinion Mining

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

analyze_sentiment_with_opinion_mining()
```

## 🔧 Advanced Examples

### Batch Processing with TextDocumentInput

```python
def batch_sentiment_analysis():
    from azure.ai.textanalytics import TextDocumentInput
    
    # Using TextDocumentInput for more control
    documents = [
        TextDocumentInput(id="1", text="Great product! Highly recommended.", language="en"),
        TextDocumentInput(id="2", text="Poor quality. Would not buy again.", language="en"),
        TextDocumentInput(id="3", text="Me encanta este producto. Excelente calidad.", language="es"),
        TextDocumentInput(id="4", text="Service client décevant mais produit correct.", language="fr")
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

batch_sentiment_analysis()
```

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

## 🔧 Error Handling and Best Practices

### Robust Error Handling

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

### Confidence Threshold Filtering

```python
def filter_by_confidence(documents, min_confidence=0.8):
    """Filter results by confidence threshold"""
    
    response = client.analyze_sentiment(documents)
    
    high_confidence_results = []
    
    for doc in response:
        if not doc.is_error:
            max_confidence = max(
                doc.confidence_scores.positive,
                doc.confidence_scores.negative,
                doc.confidence_scores.neutral
            )
            
            if max_confidence >= min_confidence:
                high_confidence_results.append({
                    'sentiment': doc.sentiment,
                    'confidence': max_confidence,
                    'text': documents[response.index(doc)][:50] + "..."
                })
    
    print(f"High confidence results ({min_confidence} threshold):")
    for result in high_confidence_results:
        print(f"  {result['sentiment']}: {result['confidence']:.2f} - {result['text']}")

# Test confidence filtering
test_docs = ["I absolutely love this!", "It's okay I guess", "This is completely terrible!"]
filter_by_confidence(test_docs, min_confidence=0.9)
```

## 🚀 Async Operations

### Asynchronous Sentiment Analysis

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient

async def async_sentiment_analysis():
    """Perform sentiment analysis asynchronously"""
    
    credential = AzureKeyCredential(key)
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        documents = [
            "I love this new feature!",
            "The service was disappointing.",
            "It works as expected."
        ]
        
        response = await client.analyze_sentiment(documents)
        
        for doc in response:
            if not doc.is_error:
                print(f"Sentiment: {doc.sentiment}")
                print(f"Confidence: {doc.confidence_scores.positive:.2f} (pos)")
            else:
                print(f"Error: {doc.error}")

# Run async function
asyncio.run(async_sentiment_analysis())
```

### Concurrent Processing

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient

async def process_multiple_batches():
    """Process multiple batches concurrently"""
    
    credential = AzureKeyCredential(key)
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        # Create multiple batches
        batch1 = ["Great service!", "Poor quality product."]
        batch2 = ["Amazing experience!", "Very disappointed."]
        batch3 = ["It's okay.", "Excellent value for money."]
        
        # Process batches concurrently
        tasks = [
            client.analyze_sentiment(batch1),
            client.analyze_sentiment(batch2), 
            client.analyze_sentiment(batch3)
        ]
        
        results = await asyncio.gather(*tasks)
        
        for batch_idx, batch_result in enumerate(results):
            print(f"Batch {batch_idx + 1} results:")
            for doc in batch_result:
                if not doc.is_error:
                    print(f"  {doc.sentiment}")

asyncio.run(process_multiple_batches())
```

## 📊 Data Analysis Integration

### Integration with pandas

```python
import pandas as pd

def sentiment_analysis_to_dataframe(texts):
    """Convert sentiment analysis results to pandas DataFrame"""
    
    response = client.analyze_sentiment(texts)
    
    data = []
    for idx, doc in enumerate(response):
        if not doc.is_error:
            data.append({
                'text': texts[idx],
                'sentiment': doc.sentiment,
                'positive_score': doc.confidence_scores.positive,
                'negative_score': doc.confidence_scores.negative,
                'neutral_score': doc.confidence_scores.neutral
            })
    
    df = pd.DataFrame(data)
    return df

# Example usage
reviews = [
    "Great product, highly recommend!",
    "Terrible quality, waste of money.",
    "It's an okay product, nothing special.",
    "Excellent customer service and fast delivery!"
]

df = sentiment_analysis_to_dataframe(reviews)
print(df)

# Basic analysis
print(f"\nSentiment distribution:")
print(df['sentiment'].value_counts())
print(f"\nAverage positive score: {df['positive_score'].mean():.2f}")
```

### Visualization with matplotlib

```python
import matplotlib.pyplot as plt

def visualize_sentiment_results(texts):
    """Visualize sentiment analysis results"""
    
    response = client.analyze_sentiment(texts)
    
    sentiments = []
    confidence_scores = []
    
    for doc in response:
        if not doc.is_error:
            sentiments.append(doc.sentiment)
            max_score = max(
                doc.confidence_scores.positive,
                doc.confidence_scores.negative,
                doc.confidence_scores.neutral
            )
            confidence_scores.append(max_score)
    
    # Create visualization
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
    
    # Sentiment distribution
    sentiment_counts = pd.Series(sentiments).value_counts()
    ax1.pie(sentiment_counts.values, labels=sentiment_counts.index, autopct='%1.1f%%')
    ax1.set_title('Sentiment Distribution')
    
    # Confidence scores
    ax2.hist(confidence_scores, bins=10, alpha=0.7)
    ax2.set_xlabel('Confidence Score')
    ax2.set_ylabel('Frequency')
    ax2.set_title('Confidence Score Distribution')
    
    plt.tight_layout()
    plt.show()

# Example usage
sample_reviews = [
    "Amazing product!",
    "Terrible experience.",
    "It's okay.",
    "Love it!",
    "Not good at all.",
    "Pretty decent.",
    "Excellent quality!",
    "Disappointing results."
]

visualize_sentiment_results(sample_reviews)
```

## 📚 Additional Resources

- [Azure AI Language Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)
- [Sentiment Analysis Samples](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics)