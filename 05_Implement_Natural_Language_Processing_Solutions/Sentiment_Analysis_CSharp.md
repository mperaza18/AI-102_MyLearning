# 🔷 Sentiment Analysis C# SDK Examples

> **C# SDK examples for sentiment analysis using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```xml
<PackageReference Include="Azure.AI.TextAnalytics" Version="5.3.0" />
<PackageReference Include="Azure.Identity" Version="1.10.3" />
```

### Basic Authentication

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using Azure.Identity;
using System;

class Program
{
    // Using API key
    static string endpoint = Environment.GetEnvironmentVariable("LANGUAGE_ENDPOINT");
    static string key = Environment.GetEnvironmentVariable("LANGUAGE_KEY");
    
    static void Main(string[] args)
    {
        // Method 1: Using API Key
        var credential = new AzureKeyCredential(key);
        var client = new TextAnalyticsClient(new Uri(endpoint), credential);
        
        // Method 2: Using Azure Identity (recommended for production)
        var identityCredential = new DefaultAzureCredential();
        var identityClient = new TextAnalyticsClient(new Uri(endpoint), identityCredential);
        
        AnalyzeSentiment(client);
        SentimentAnalysisWithOpinionMining(client);
        BatchSentimentAnalysis(client);
    }
}
```

## 🔧 Basic Examples

### Simple Sentiment Analysis

```csharp
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
        Console.WriteLine();
    }
    catch (RequestFailedException exception)
    {
        Console.WriteLine($"Error Code: {exception.ErrorCode}");
        Console.WriteLine($"Message: {exception.Message}");
    }
}
```

### Sentiment Analysis with Opinion Mining

```csharp
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
        Console.WriteLine($"Document sentiment: {review.DocumentSentiment.Sentiment}");
        Console.WriteLine($"Positive score: {review.DocumentSentiment.ConfidenceScores.Positive:0.00}");
        Console.WriteLine($"Negative score: {review.DocumentSentiment.ConfidenceScores.Negative:0.00}");
        Console.WriteLine($"Neutral score: {review.DocumentSentiment.ConfidenceScores.Neutral:0.00}");
        Console.WriteLine();
        
        foreach (SentenceSentiment sentence in review.DocumentSentiment.Sentences)
        {
            Console.WriteLine($"Text: \"{sentence.Text}\"");
            Console.WriteLine($"Sentence sentiment: {sentence.Sentiment}");
            Console.WriteLine($"Positive score: {sentence.ConfidenceScores.Positive:0.00}");
            Console.WriteLine($"Negative score: {sentence.ConfidenceScores.Negative:0.00}");
            Console.WriteLine($"Neutral score: {sentence.ConfidenceScores.Neutral:0.00}");
            Console.WriteLine();

            foreach (SentenceOpinion sentenceOpinion in sentence.Opinions)
            {
                Console.WriteLine($"Target: {sentenceOpinion.Target.Text}");
                Console.WriteLine($"Target sentiment: {sentenceOpinion.Target.Sentiment}");
                Console.WriteLine($"Target positive score: {sentenceOpinion.Target.ConfidenceScores.Positive:0.00}");
                Console.WriteLine($"Target negative score: {sentenceOpinion.Target.ConfidenceScores.Negative:0.00}");
                
                foreach (AssessmentSentiment assessment in sentenceOpinion.Assessments)
                {
                    Console.WriteLine($"Assessment: {assessment.Text}");
                    Console.WriteLine($"Assessment sentiment: {assessment.Sentiment}");
                    Console.WriteLine($"Assessment positive score: {assessment.ConfidenceScores.Positive:0.00}");
                    Console.WriteLine($"Assessment negative score: {assessment.ConfidenceScores.Negative:0.00}");
                }
            }
        }
        Console.WriteLine();
    }
}
```

## 🔧 Advanced Examples

### Batch Processing with TextDocumentInput

```csharp
static void BatchSentimentAnalysis(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Great product! Highly recommended.")
        {
             Language = "en",
        },
        new TextDocumentInput("2", "Poor quality. Would not buy again.")
        {
             Language = "en",
        },
        new TextDocumentInput("3", "Me encanta este producto. Excelente calidad.")
        {
             Language = "es",
        },
        new TextDocumentInput("4", "Service client décevant mais produit correct.")
        {
             Language = "fr",
        }
    };

    AnalyzeSentimentResultCollection results = client.AnalyzeSentimentBatch(documents);

    foreach (AnalyzeSentimentResult result in results)
    {
        Console.WriteLine($"Document ID: {result.Id}");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            Console.WriteLine($"  Sentiment: {result.DocumentSentiment.Sentiment}");
            Console.WriteLine($"  Positive: {result.DocumentSentiment.ConfidenceScores.Positive:0.00}");
            Console.WriteLine($"  Negative: {result.DocumentSentiment.ConfidenceScores.Negative:0.00}");
            Console.WriteLine($"  Neutral: {result.DocumentSentiment.ConfidenceScores.Neutral:0.00}");
        }
        Console.WriteLine();
    }
}
```

### Customer Feedback Analysis

```csharp
static void AnalyzeCustomerFeedback(TextAnalyticsClient client)
{
    var feedback = new List<string>
    {
        "Great product! Fast shipping and excellent customer service.",
        "The quality is poor and it broke after one week. Very disappointed.",
        "Average product. Nothing special but does the job.",
        "Love the design but the price is too high for what you get.",
        "Outstanding quality and value. Will definitely buy again!"
    };

    var options = new AnalyzeSentimentOptions()
    {
        IncludeOpinionMining = true
    };

    AnalyzeSentimentResultCollection results = client.AnalyzeSentimentBatch(feedback, options);

    int positiveCount = 0, negativeCount = 0, neutralCount = 0;

    foreach (AnalyzeSentimentResult result in results)
    {
        if (!result.HasError)
        {
            switch (result.DocumentSentiment.Sentiment)
            {
                case TextSentiment.Positive:
                    positiveCount++;
                    break;
                case TextSentiment.Negative:
                    negativeCount++;
                    break;
                case TextSentiment.Neutral:
                    neutralCount++;
                    break;
            }

            var maxConfidence = Math.Max(
                Math.Max(result.DocumentSentiment.ConfidenceScores.Positive,
                        result.DocumentSentiment.ConfidenceScores.Negative),
                result.DocumentSentiment.ConfidenceScores.Neutral);

            Console.WriteLine($"Review: '{feedback[results.ToList().IndexOf(result)][..Math.Min(50, feedback[results.ToList().IndexOf(result)].Length)]}...'");
            Console.WriteLine($"Sentiment: {result.DocumentSentiment.Sentiment} (Confidence: {maxConfidence:0.00})");
            Console.WriteLine();
        }
    }

    Console.WriteLine($"Summary: {positiveCount} positive, {negativeCount} negative, {neutralCount} neutral reviews");
}
```

### Social Media Monitoring

```csharp
static void SocialMediaSentiment(TextAnalyticsClient client)
{
    var posts = new List<string>
    {
        "Just tried @YourBrand's new product. Amazing quality! #love",
        "@YourBrand customer service is terrible. Still waiting for response.",
        "Neutral opinion about @YourBrand. It's okay I guess.",
        "@YourBrand has the best coffee in town! Highly recommended."
    };

    AnalyzeSentimentResultCollection results = client.AnalyzeSentimentBatch(posts);

    var brandSentiment = new Dictionary<string, int>
    {
        ["Positive"] = 0,
        ["Negative"] = 0,
        ["Neutral"] = 0
    };

    int postNumber = 1;
    foreach (AnalyzeSentimentResult result in results)
    {
        if (!result.HasError)
        {
            brandSentiment[result.DocumentSentiment.Sentiment.ToString()]++;
            
            var sentimentScore = result.DocumentSentiment.Sentiment switch
            {
                TextSentiment.Positive => result.DocumentSentiment.ConfidenceScores.Positive,
                TextSentiment.Negative => result.DocumentSentiment.ConfidenceScores.Negative,
                TextSentiment.Neutral => result.DocumentSentiment.ConfidenceScores.Neutral,
                _ => 0.0
            };

            Console.WriteLine($"Post {postNumber}: {result.DocumentSentiment.Sentiment} (Score: {sentimentScore:0.00})");
            postNumber++;
        }
    }

    Console.WriteLine($"\nBrand Sentiment Overview:");
    foreach (var sentiment in brandSentiment)
    {
        Console.WriteLine($"  {sentiment.Key}: {sentiment.Value} posts");
    }
}
```

## 🔧 Error Handling and Best Practices

### Robust Error Handling

```csharp
static void RobustSentimentAnalysis(TextAnalyticsClient client, List<string> documents)
{
    try
    {
        AnalyzeSentimentResultCollection response = client.AnalyzeSentimentBatch(documents);
        
        int docIndex = 0;
        foreach (AnalyzeSentimentResult doc in response)
        {
            docIndex++;
            if (!doc.HasError)
            {
                Console.WriteLine($"Document {docIndex}: {doc.DocumentSentiment.Sentiment}");
            }
            else
            {
                Console.WriteLine($"Document {docIndex} failed: {doc.Error.ErrorCode} - {doc.Error.Message}");
            }
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine($"API call failed: {ex.Message}");
        Console.WriteLine($"Error code: {ex.ErrorCode}");
        // Implement retry logic or fallback here
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected error: {ex.Message}");
    }
}

// Test with mixed valid/invalid content
var testDocs = new List<string>
{
    "This is a great product!",
    "", // Empty string - will cause error
    "Bad experience with customer service.",
    new string('x', 5200) // Too long - will cause error
};

RobustSentimentAnalysis(client, testDocs);
```

### Performance Optimization

```csharp
static void OptimizedSentimentAnalysis(TextAnalyticsClient client)
{
    // Generate sample documents
    var documents = new List<string>();
    for (int i = 0; i < 20; i++)
    {
        documents.Add($"Review {i}: This product is {(i % 2 == 0 ? "great" : "terrible")}!");
    }

    // Process in batches of 10 (API limit)
    const int batchSize = 10;
    
    for (int i = 0; i < documents.Count; i += batchSize)
    {
        var batch = documents.Skip(i).Take(batchSize).ToList();
        
        var options = new AnalyzeSentimentOptions()
        {
            IncludeOpinionMining = false, // Disable if not needed for better performance
            ModelVersion = "latest"
        };
        
        AnalyzeSentimentResultCollection response = client.AnalyzeSentimentBatch(batch, options);
        
        foreach (AnalyzeSentimentResult doc in response)
        {
            if (!doc.HasError)
            {
                Console.WriteLine($"Sentiment: {doc.DocumentSentiment.Sentiment}");
            }
        }
    }
}
```

### Confidence Threshold Filtering

```csharp
static void FilterByConfidence(TextAnalyticsClient client, List<string> documents, double minConfidence = 0.8)
{
    AnalyzeSentimentResultCollection response = client.AnalyzeSentimentBatch(documents);
    
    var highConfidenceResults = new List<(string sentiment, double confidence, string text)>();
    
    int docIndex = 0;
    foreach (AnalyzeSentimentResult doc in response)
    {
        if (!doc.HasError)
        {
            double maxConfidence = Math.Max(
                Math.Max(doc.DocumentSentiment.ConfidenceScores.Positive,
                        doc.DocumentSentiment.ConfidenceScores.Negative),
                doc.DocumentSentiment.ConfidenceScores.Neutral);
            
            if (maxConfidence >= minConfidence)
            {
                string textPreview = documents[docIndex].Length > 50 
                    ? documents[docIndex][..50] + "..."
                    : documents[docIndex];
                    
                highConfidenceResults.Add((doc.DocumentSentiment.Sentiment.ToString(), maxConfidence, textPreview));
            }
        }
        docIndex++;
    }
    
    Console.WriteLine($"High confidence results ({minConfidence} threshold):");
    foreach (var result in highConfidenceResults)
    {
        Console.WriteLine($"  {result.sentiment}: {result.confidence:0.00} - {result.text}");
    }
}

// Test confidence filtering
var testDocs = new List<string>
{
    "I absolutely love this!",
    "It's okay I guess", 
    "This is completely terrible!"
};

FilterByConfidence(client, testDocs, 0.9);
```

## 🚀 Async Operations

### Asynchronous Sentiment Analysis

```csharp
using System.Threading.Tasks;

static async Task AsyncSentimentAnalysis(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "I love this new feature!",
        "The service was disappointing.",
        "It works as expected."
    };

    try
    {
        Response<AnalyzeSentimentResultCollection> response = 
            await client.AnalyzeSentimentBatchAsync(documents);
        
        foreach (AnalyzeSentimentResult doc in response.Value)
        {
            if (!doc.HasError)
            {
                Console.WriteLine($"Sentiment: {doc.DocumentSentiment.Sentiment}");
                Console.WriteLine($"Confidence: {doc.DocumentSentiment.ConfidenceScores.Positive:0.00} (pos)");
            }
            else
            {
                Console.WriteLine($"Error: {doc.Error.Message}");
            }
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine($"Request failed: {ex.Message}");
    }
}

// Usage
await AsyncSentimentAnalysis(client);
```

### Concurrent Processing

```csharp
static async Task ProcessMultipleBatches(TextAnalyticsClient client)
{
    // Create multiple batches
    var batch1 = new List<string> { "Great service!", "Poor quality product." };
    var batch2 = new List<string> { "Amazing experience!", "Very disappointed." };
    var batch3 = new List<string> { "It's okay.", "Excellent value for money." };

    // Process batches concurrently
    var tasks = new List<Task<Response<AnalyzeSentimentResultCollection>>>
    {
        client.AnalyzeSentimentBatchAsync(batch1),
        client.AnalyzeSentimentBatchAsync(batch2),
        client.AnalyzeSentimentBatchAsync(batch3)
    };

    Response<AnalyzeSentimentResultCollection>[] results = await Task.WhenAll(tasks);

    for (int batchIndex = 0; batchIndex < results.Length; batchIndex++)
    {
        Console.WriteLine($"Batch {batchIndex + 1} results:");
        foreach (AnalyzeSentimentResult doc in results[batchIndex].Value)
        {
            if (!doc.HasError)
            {
                Console.WriteLine($"  {doc.DocumentSentiment.Sentiment}");
            }
        }
    }
}

await ProcessMultipleBatches(client);
```

## 📊 Data Analysis Integration

### LINQ Integration

```csharp
using System.Linq;

static void SentimentAnalysisWithLinq(TextAnalyticsClient client)
{
    var reviews = new List<string>
    {
        "Great product, highly recommend!",
        "Terrible quality, waste of money.",
        "It's an okay product, nothing special.",
        "Excellent customer service and fast delivery!"
    };

    AnalyzeSentimentResultCollection results = client.AnalyzeSentimentBatch(reviews);

    var sentimentData = results
        .Where(r => !r.HasError)
        .Select((r, index) => new
        {
            Text = reviews[index],
            Sentiment = r.DocumentSentiment.Sentiment.ToString(),
            PositiveScore = r.DocumentSentiment.ConfidenceScores.Positive,
            NegativeScore = r.DocumentSentiment.ConfidenceScores.Negative,
            NeutralScore = r.DocumentSentiment.ConfidenceScores.Neutral
        })
        .ToList();

    // Analysis using LINQ
    var sentimentCounts = sentimentData
        .GroupBy(s => s.Sentiment)
        .ToDictionary(g => g.Key, g => g.Count());

    var averagePositiveScore = sentimentData.Average(s => s.PositiveScore);

    Console.WriteLine("Sentiment Distribution:");
    foreach (var sentiment in sentimentCounts)
    {
        Console.WriteLine($"  {sentiment.Key}: {sentiment.Value}");
    }
    
    Console.WriteLine($"Average Positive Score: {averagePositiveScore:0.00}");
}
```

### Custom Result Class

```csharp
public class SentimentResult
{
    public string Text { get; set; }
    public string Sentiment { get; set; }
    public double PositiveScore { get; set; }
    public double NegativeScore { get; set; }
    public double NeutralScore { get; set; }
    public bool HasError { get; set; }
    public string ErrorMessage { get; set; }
}

static List<SentimentResult> AnalyzeSentimentToCustomClass(TextAnalyticsClient client, List<string> texts)
{
    AnalyzeSentimentResultCollection results = client.AnalyzeSentimentBatch(texts);
    
    var sentimentResults = new List<SentimentResult>();
    
    int textIndex = 0;
    foreach (AnalyzeSentimentResult result in results)
    {
        var sentimentResult = new SentimentResult
        {
            Text = texts[textIndex]
        };
        
        if (!result.HasError)
        {
            sentimentResult.Sentiment = result.DocumentSentiment.Sentiment.ToString();
            sentimentResult.PositiveScore = result.DocumentSentiment.ConfidenceScores.Positive;
            sentimentResult.NegativeScore = result.DocumentSentiment.ConfidenceScores.Negative;
            sentimentResult.NeutralScore = result.DocumentSentiment.ConfidenceScores.Neutral;
            sentimentResult.HasError = false;
        }
        else
        {
            sentimentResult.HasError = true;
            sentimentResult.ErrorMessage = result.Error.Message;
        }
        
        sentimentResults.Add(sentimentResult);
        textIndex++;
    }
    
    return sentimentResults;
}

// Usage
var testTexts = new List<string>
{
    "Amazing product!",
    "Terrible experience.",
    "It's okay."
};

var results = AnalyzeSentimentToCustomClass(client, testTexts);
foreach (var result in results)
{
    if (!result.HasError)
    {
        Console.WriteLine($"{result.Sentiment}: {result.PositiveScore:0.00} (positive)");
    }
    else
    {
        Console.WriteLine($"Error: {result.ErrorMessage}");
    }
}
```

## 📚 Additional Resources

- [Azure AI Language .NET SDK Documentation](https://docs.microsoft.com/dotnet/api/azure.ai.textanalytics/)
- [Sentiment Analysis Samples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics)