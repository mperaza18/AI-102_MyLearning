# 🔷 Key Phrase Extraction C# SDK Examples

> **C# SDK examples for Key Phrase Extraction using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```xml
<PackageReference Include="Azure.AI.TextAnalytics" Version="5.3.0" />
<PackageReference Include="Azure.Identity" Version="1.10.3" />
<PackageReference Include="Microsoft.Extensions.Configuration" Version="7.0.0" />
<PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="7.0.0" />
<PackageReference Include="System.Linq.Async" Version="6.0.1" />
```

### Basic Authentication Setup

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using Azure.Identity;
using Microsoft.Extensions.Configuration;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

class Program
{
    private static string endpoint;
    private static string key;
    
    static async Task Main(string[] args)
    {
        // Load configuration
        var config = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", optional: true)
            .AddEnvironmentVariables()
            .Build();
        
        endpoint = config["LANGUAGE_ENDPOINT"] ?? Environment.GetEnvironmentVariable("LANGUAGE_ENDPOINT");
        key = config["LANGUAGE_KEY"] ?? Environment.GetEnvironmentVariable("LANGUAGE_KEY");
        
        // Method 1: Using API Key
        var credential = new AzureKeyCredential(key);
        var client = new TextAnalyticsClient(new Uri(endpoint), credential);
        
        // Method 2: Using Azure Identity (recommended for production)
        var identityCredential = new DefaultAzureCredential();
        var identityClient = new TextAnalyticsClient(new Uri(endpoint), identityCredential);
        
        await RunExamples(client);
    }
    
    static async Task RunExamples(TextAnalyticsClient client)
    {
        ExtractKeyPhrasesBasic(client);
        ExtractKeyPhrasesBatch(client);
        ExtractKeyPhrasesMultiLanguage(client);
        await ExtractKeyPhrasesAsync(client);
        AnalyzeContentThemes(client);
        AnalyzeCustomerFeedback(client);
    }
}
```

## 🔧 Basic Examples

### Simple Key Phrase Extraction

```csharp
static void ExtractKeyPhrasesBasic(TextAnalyticsClient client)
{
    string document = "Microsoft Azure is a comprehensive cloud computing platform that offers a wide range of services including virtual machines, databases, artificial intelligence tools, and analytics services. It helps businesses scale their operations efficiently and reduce infrastructure costs through flexible pricing models.";

    try
    {
        Response<KeyPhraseCollection> response = client.ExtractKeyPhrases(document);
        KeyPhraseCollection keyPhrases = response.Value;

        Console.WriteLine($"Extracted {keyPhrases.Count} key phrases:");
        foreach (string keyPhrase in keyPhrases)
        {
            Console.WriteLine($"  • {keyPhrase}");
        }
        Console.WriteLine();
    }
    catch (RequestFailedException exception)
    {
        Console.WriteLine($"Error Code: {exception.ErrorCode}");
        Console.WriteLine($"Message: {exception.Message}");
    }
}
```

### Batch Key Phrase Extraction

```csharp
static void ExtractKeyPhrasesBatch(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "Our quarterly results demonstrate strong performance in cloud services and artificial intelligence solutions.",
        "Digital transformation initiatives are driving innovation and operational efficiency across multiple business units.",
        "Customer experience improvements through data analytics and machine learning have increased satisfaction scores.",
        "Supply chain optimization using predictive analytics has reduced costs and improved delivery performance."
    };

    ExtractKeyPhrasesResultCollection results = client.ExtractKeyPhrasesBatch(documents);

    int i = 0;
    foreach (ExtractKeyPhrasesResult result in results)
    {
        Console.WriteLine($"Document {i + 1}:");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            Console.WriteLine($"  Key phrases ({result.KeyPhrases.Count} found):");
            foreach (string keyPhrase in result.KeyPhrases)
            {
                Console.WriteLine($"    • {keyPhrase}");
            }
            
            // Show warnings if any
            if (result.Warnings.Any())
            {
                Console.WriteLine("  Warnings:");
                foreach (var warning in result.Warnings)
                {
                    Console.WriteLine($"    - {warning.Code}: {warning.Message}");
                }
            }
        }
        Console.WriteLine();
        i++;
    }
}
```

## 🔧 Advanced Examples

### Multi-Language Key Phrase Extraction

```csharp
static void ExtractKeyPhrasesMultiLanguage(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Cloud computing and artificial intelligence are transforming business operations.")
        {
             Language = "en",
        },
        new TextDocumentInput("2", "La computación en la nube y la inteligencia artificial están transformando las operaciones comerciales.")
        {
             Language = "es",
        },
        new TextDocumentInput("3", "Le cloud computing et l'intelligence artificielle transforment les opérations commerciales.")
        {
             Language = "fr",
        },
        new TextDocumentInput("4", "Cloud Computing und künstliche Intelligenz transformieren Geschäftsabläufe.")
        {
             Language = "de",
        }
    };

    ExtractKeyPhrasesResultCollection results = client.ExtractKeyPhrasesBatch(documents);

    var languageResults = new Dictionary<string, List<string>>();

    foreach (ExtractKeyPhrasesResult result in results)
    {
        var documentInput = documents.First(d => d.Id == result.Id);
        string language = documentInput.Language;
        
        Console.WriteLine($"Document ID: {result.Id} (Language: {language})");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            if (!languageResults.ContainsKey(language))
                languageResults[language] = new List<string>();
            
            languageResults[language].AddRange(result.KeyPhrases);
            
            Console.WriteLine($"  Key phrases: {string.Join(", ", result.KeyPhrases)}");
        }
        Console.WriteLine();
    }

    // Analyze common themes across languages
    Console.WriteLine("CROSS-LANGUAGE ANALYSIS:");
    foreach (var langResult in languageResults)
    {
        Console.WriteLine($"\n{langResult.Key.ToUpper()} key phrases:");
        var phraseFreq = langResult.Value
            .GroupBy(p => p)
            .OrderByDescending(g => g.Count())
            .Take(5);
        
        foreach (var group in phraseFreq)
        {
            Console.WriteLine($"  • {group.Key} ({group.Count()})");
        }
    }
}
```

## 🔧 Specialized Use Cases

### Content Theme Analysis

```csharp
public class ContentThemeAnalyzer
{
    private readonly TextAnalyticsClient _client;
    
    public ContentThemeAnalyzer(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public ThemeAnalysisResult AnalyzeContentThemes(List<string> documents, List<string> documentLabels = null)
    {
        try
        {
            ExtractKeyPhrasesResultCollection results = _client.ExtractKeyPhrasesBatch(documents);
            
            var allPhrases = new List<string>();
            var docPhrases = new Dictionary<string, List<string>>();
            
            for (int i = 0; i < results.Count; i++)
            {
                var result = results[i];
                string docLabel = documentLabels?[i] ?? $"Document {i + 1}";
                
                if (!result.HasError)
                {
                    var phrases = result.KeyPhrases.ToList();
                    docPhrases[docLabel] = phrases;
                    allPhrases.AddRange(phrases);
                }
                else
                {
                    Console.WriteLine($"{docLabel} - Error: {result.Error.Message}");
                }
            }
            
            // Calculate statistics
            var phraseFrequency = allPhrases
                .GroupBy(p => p)
                .ToDictionary(g => g.Key, g => g.Count());
            
            // Find common themes (phrases appearing in multiple documents)
            var phraseDistribution = new Dictionary<string, int>();
            foreach (var docPhrase in docPhrases.Values)
            {
                foreach (var phrase in docPhrase.Distinct())
                {
                    phraseDistribution[phrase] = phraseDistribution.GetValueOrDefault(phrase, 0) + 1;
                }
            }
            
            var commonThemes = phraseDistribution
                .Where(p => p.Value > 1)
                .OrderByDescending(p => p.Value)
                .ToDictionary(p => p.Key, p => p.Value);
            
            // Categorize phrases by length
            var uniquePhrases = allPhrases.Distinct().ToList();
            var phraseCategories = new
            {
                ShortPhrases = uniquePhrases.Where(p => p.Split(' ').Length <= 2).ToList(),
                MediumPhrases = uniquePhrases.Where(p => p.Split(' ').Length >= 3 && p.Split(' ').Length <= 4).ToList(),
                LongPhrases = uniquePhrases.Where(p => p.Split(' ').Length > 4).ToList()
            };
            
            return new ThemeAnalysisResult
            {
                TotalDocuments = documents.Count,
                SuccessfulAnalyses = docPhrases.Count,
                TotalKeyPhrases = allPhrases.Count,
                UniquePhrases = uniquePhrases.Count,
                PhraseFrequency = phraseFrequency,
                DocumentPhrases = docPhrases,
                CommonThemes = commonThemes,
                ShortPhrases = phraseCategories.ShortPhrases,
                MediumPhrases = phraseCategories.MediumPhrases,
                LongPhrases = phraseCategories.LongPhrases
            };
        }
        catch (RequestFailedException ex)
        {
            throw new InvalidOperationException($"Theme analysis failed: {ex.Message}", ex);
        }
    }
    
    public void PrintThemeReport(ThemeAnalysisResult results)
    {
        Console.WriteLine("CONTENT THEME ANALYSIS REPORT");
        Console.WriteLine(new string('=', 50));
        Console.WriteLine($"Total Documents: {results.TotalDocuments}");
        Console.WriteLine($"Successful Analyses: {results.SuccessfulAnalyses}");
        Console.WriteLine($"Total Key Phrases: {results.TotalKeyPhrases}");
        Console.WriteLine($"Unique Phrases: {results.UniquePhrases}");
        
        Console.WriteLine("\n📊 Most Frequent Key Phrases:");
        var topPhrases = results.PhraseFrequency
            .OrderByDescending(p => p.Value)
            .Take(10);
        
        foreach (var phrase in topPhrases)
        {
            Console.WriteLine($"  • {phrase.Key}: {phrase.Value}");
        }
        
        Console.WriteLine("\n🔗 Common Themes Across Documents:");
        if (results.CommonThemes.Any())
        {
            foreach (var theme in results.CommonThemes.Take(10))
            {
                Console.WriteLine($"  • {theme.Key} (in {theme.Value} documents)");
            }
        }
        else
        {
            Console.WriteLine("  No common themes found across multiple documents");
        }
        
        Console.WriteLine("\n📏 Phrase Categories by Length:");
        Console.WriteLine($"  Short phrases (1-2 words): {results.ShortPhrases.Count}");
        Console.WriteLine($"  Medium phrases (3-4 words): {results.MediumPhrases.Count}");
        Console.WriteLine($"  Long phrases (5+ words): {results.LongPhrases.Count}");
        
        Console.WriteLine("\n📄 Document-Specific Phrases:");
        foreach (var doc in results.DocumentPhrases)
        {
            Console.WriteLine($"  {doc.Key}: {doc.Value.Count} key phrases");
            Console.WriteLine($"    Top 3: {string.Join(", ", doc.Value.Take(3))}");
        }
    }
}

// Supporting classes
public class ThemeAnalysisResult
{
    public int TotalDocuments { get; set; }
    public int SuccessfulAnalyses { get; set; }
    public int TotalKeyPhrases { get; set; }
    public int UniquePhrases { get; set; }
    public Dictionary<string, int> PhraseFrequency { get; set; } = new Dictionary<string, int>();
    public Dictionary<string, List<string>> DocumentPhrases { get; set; } = new Dictionary<string, List<string>>();
    public Dictionary<string, int> CommonThemes { get; set; } = new Dictionary<string, int>();
    public List<string> ShortPhrases { get; set; } = new List<string>();
    public List<string> MediumPhrases { get; set; } = new List<string>();
    public List<string> LongPhrases { get; set; } = new List<string>();
}

// Usage example
static void AnalyzeContentThemes(TextAnalyticsClient client)
{
    var analyzer = new ContentThemeAnalyzer(client);
    
    var technologyArticles = new List<string>
    {
        "Artificial intelligence and machine learning technologies are revolutionizing data analysis and business intelligence.",
        "Cloud computing platforms provide scalable infrastructure for modern applications and data storage solutions.",
        "Cybersecurity frameworks protect digital assets through advanced threat detection and prevention systems.",
        "Digital transformation strategies leverage emerging technologies to improve operational efficiency and customer experience.",
        "Internet of Things devices generate massive datasets that require advanced analytics and real-time processing capabilities."
    };
    
    var articleLabels = new List<string>
    {
        "AI/ML Article", "Cloud Computing", "Cybersecurity", "Digital Transformation", "IoT Analytics"
    };
    
    var themeAnalysis = analyzer.AnalyzeContentThemes(technologyArticles, articleLabels);
    analyzer.PrintThemeReport(themeAnalysis);
}
```

### Customer Feedback Analysis

```csharp
public class CustomerFeedbackAnalyzer
{
    private readonly TextAnalyticsClient _client;
    
    public CustomerFeedbackAnalyzer(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public FeedbackAnalysisResult AnalyzeFeedbackThemes(List<string> feedbackTexts)
    {
        try
        {
            ExtractKeyPhrasesResultCollection response = _client.ExtractKeyPhrasesBatch(feedbackTexts);
            
            // Define categories for feedback analysis
            var categories = new Dictionary<string, List<string>>
            {
                ["product_features"] = new List<string>(),
                ["service_quality"] = new List<string>(),
                ["user_experience"] = new List<string>(),
                ["technical_issues"] = new List<string>(),
                ["pricing_value"] = new List<string>(),
                ["performance"] = new List<string>(),
                ["support"] = new List<string>()
            };
            
            // Keywords for categorization
            var categoryKeywords = new Dictionary<string, List<string>>
            {
                ["product_features"] = new List<string> { "product", "feature", "functionality", "capability", "tool", "option" },
                ["service_quality"] = new List<string> { "service", "quality", "support", "help", "assistance", "care" },
                ["user_experience"] = new List<string> { "experience", "interface", "usability", "design", "navigation", "user" },
                ["technical_issues"] = new List<string> { "bug", "error", "issue", "problem", "crash", "failure", "technical" },
                ["pricing_value"] = new List<string> { "price", "cost", "value", "expensive", "affordable", "pricing", "budget" },
                ["performance"] = new List<string> { "speed", "fast", "slow", "performance", "efficiency", "response", "latency" },
                ["support"] = new List<string> { "support", "help desk", "customer service", "documentation", "training" }
            };
            
            var allFeedbackPhrases = new List<string>();
            
            for (int i = 0; i < response.Count; i++)
            {
                var result = response[i];
                
                if (!result.HasError)
                {
                    allFeedbackPhrases.AddRange(result.KeyPhrases);
                    
                    // Categorize phrases
                    foreach (string phrase in result.KeyPhrases)
                    {
                        string phraseLower = phrase.ToLower();
                        bool categorized = false;
                        
                        foreach (var category in categoryKeywords)
                        {
                            if (category.Value.Any(keyword => phraseLower.Contains(keyword)))
                            {
                                categories[category.Key].Add(phrase);
                                categorized = true;
                                break;
                            }
                        }
                    }
                }
                else
                {
                    Console.WriteLine($"Feedback {i + 1} - Error: {result.Error.Message}");
                }
            }
            
            // Calculate category statistics
            var categoryStats = new Dictionary<string, CategoryStatistics>();
            foreach (var category in categories)
            {
                if (category.Value.Any())
                {
                    var phraseGroups = category.Value
                        .GroupBy(p => p)
                        .OrderByDescending(g => g.Count())
                        .Take(5)
                        .ToDictionary(g => g.Key, g => g.Count());
                    
                    categoryStats[category.Key] = new CategoryStatistics
                    {
                        Count = category.Value.Count,
                        UniquePhrases = category.Value.Distinct().Count(),
                        TopPhrases = phraseGroups
                    };
                }
            }
            
            var overallTopPhrases = allFeedbackPhrases
                .GroupBy(p => p)
                .OrderByDescending(g => g.Count())
                .Take(10)
                .ToDictionary(g => g.Key, g => g.Count());
            
            return new FeedbackAnalysisResult
            {
                TotalFeedbackCount = feedbackTexts.Count,
                TotalKeyPhrases = allFeedbackPhrases.Count,
                Categories = categoryStats,
                OverallTopPhrases = overallTopPhrases
            };
        }
        catch (RequestFailedException ex)
        {
            throw new InvalidOperationException($"Feedback analysis failed: {ex.Message}", ex);
        }
    }
    
    public void PrintFeedbackReport(FeedbackAnalysisResult results)
    {
        Console.WriteLine("CUSTOMER FEEDBACK ANALYSIS");
        Console.WriteLine(new string('=', 40));
        Console.WriteLine($"Total Feedback Items: {results.TotalFeedbackCount}");
        Console.WriteLine($"Total Key Phrases: {results.TotalKeyPhrases}");
        
        Console.WriteLine("\n🎯 Overall Top Key Phrases:");
        foreach (var phrase in results.OverallTopPhrases)
        {
            Console.WriteLine($"  • {phrase.Key}: {phrase.Value}");
        }
        
        Console.WriteLine("\n📋 Category Analysis:");
        foreach (var category in results.Categories)
        {
            Console.WriteLine($"\n  {category.Key.Replace("_", " ").ToUpper()}:");
            Console.WriteLine($"    Phrase Count: {category.Value.Count}");
            Console.WriteLine($"    Unique Phrases: {category.Value.UniquePhrases}");
            Console.WriteLine("    Top Phrases:");
            
            foreach (var phrase in category.Value.TopPhrases)
            {
                Console.WriteLine($"      • {phrase.Key}: {phrase.Value}");
            }
        }
    }
}

// Supporting classes
public class CategoryStatistics
{
    public int Count { get; set; }
    public int UniquePhrases { get; set; }
    public Dictionary<string, int> TopPhrases { get; set; } = new Dictionary<string, int>();
}

public class FeedbackAnalysisResult
{
    public int TotalFeedbackCount { get; set; }
    public int TotalKeyPhrases { get; set; }
    public Dictionary<string, CategoryStatistics> Categories { get; set; } = new Dictionary<string, CategoryStatistics>();
    public Dictionary<string, int> OverallTopPhrases { get; set; } = new Dictionary<string, int>();
}

// Usage example
static void AnalyzeCustomerFeedback(TextAnalyticsClient client)
{
    var analyzer = new CustomerFeedbackAnalyzer(client);
    
    var customerFeedback = new List<string>
    {
        "The product quality is excellent and the user interface is very intuitive. Customer support response time could be improved.",
        "Great value for money and performance is outstanding. Some advanced features are hard to find in the interface.",
        "Technical issues with the mobile app but the web version works perfectly. Documentation is comprehensive.",
        "Pricing is competitive and the service quality exceeds expectations. Would like more training materials.",
        "Fast performance and reliable service. The support team is knowledgeable and helpful with technical problems."
    };
    
    var feedbackResults = analyzer.AnalyzeFeedbackThemes(customerFeedback);
    analyzer.PrintFeedbackReport(feedbackResults);
}
```

## 🚀 Async Operations

### Asynchronous Key Phrase Extraction

```csharp
static async Task ExtractKeyPhrasesAsync(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "Cloud computing platforms enable digital transformation and scalable business solutions.",
        "Artificial intelligence applications revolutionize data analysis and decision-making processes.",
        "Cybersecurity measures protect sensitive information and maintain data privacy compliance."
    };

    try
    {
        Response<ExtractKeyPhrasesResultCollection> response = 
            await client.ExtractKeyPhrasesBatchAsync(documents);
        
        ExtractKeyPhrasesResultCollection results = response.Value;
        
        for (int i = 0; i < results.Count; i++)
        {
            var result = results[i];
            Console.WriteLine($"Document {i + 1}:");
            
            if (!result.HasError)
            {
                Console.WriteLine($"  Key phrases ({result.KeyPhrases.Count} found):");
                foreach (string keyPhrase in result.KeyPhrases)
                {
                    Console.WriteLine($"    • {keyPhrase}");
                }
            }
            else
            {
                Console.WriteLine($"  Error: {result.Error.Message}");
            }
            Console.WriteLine();
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine($"Request failed: {ex.Message}");
    }
}
```

### Concurrent Processing with Tasks

```csharp
static async Task ProcessLargeDatasetConcurrent(TextAnalyticsClient client)
{
    // Generate sample documents
    var documents = new List<string>();
    for (int i = 0; i < 30; i++)
    {
        documents.Add($"Technology company {i} focuses on cloud computing and artificial intelligence solutions.");
    }
    
    // Split into batches of 10 (API limit)
    const int batchSize = 10;
    var batches = new List<List<string>>();
    
    for (int i = 0; i < documents.Count; i += batchSize)
    {
        batches.Add(documents.Skip(i).Take(batchSize).ToList());
    }
    
    var startTime = DateTime.Now;
    
    // Process batches concurrently
    var tasks = batches.Select(batch => ProcessBatchAsync(client, batch)).ToArray();
    var results = await Task.WhenAll(tasks);
    
    var endTime = DateTime.Now;
    
    // Flatten results
    var allResults = results.SelectMany(r => r).ToList();
    
    Console.WriteLine($"Processed {documents.Count} documents in {(endTime - startTime).TotalSeconds:F2} seconds");
    Console.WriteLine($"Used {batches.Count} batches processed concurrently");
    Console.WriteLine($"Successfully processed: {allResults.Count(r => r.Success)} documents");
    
    // Analyze results
    var allKeyPhrases = allResults
        .Where(r => r.Success)
        .SelectMany(r => r.KeyPhrases)
        .ToList();
    
    Console.WriteLine($"Total key phrases extracted: {allKeyPhrases.Count}");
    Console.WriteLine($"Unique key phrases: {allKeyPhrases.Distinct().Count()}");
    
    // Show top phrases
    var phraseFreq = allKeyPhrases
        .GroupBy(p => p)
        .OrderByDescending(g => g.Count())
        .Take(10)
        .ToDictionary(g => g.Key, g => g.Count());
    
    Console.WriteLine("\nTop 10 Key Phrases:");
    foreach (var phrase in phraseFreq)
    {
        Console.WriteLine($"  • {phrase.Key}: {phrase.Value}");
    }
}

static async Task<List<KeyPhraseExtractionResult>> ProcessBatchAsync(TextAnalyticsClient client, List<string> batch)
{
    try
    {
        var response = await client.ExtractKeyPhrasesBatchAsync(batch);
        var results = new List<KeyPhraseExtractionResult>();
        
        foreach (var doc in response.Value)
        {
            if (!doc.HasError)
            {
                results.Add(new KeyPhraseExtractionResult
                {
                    Success = true,
                    KeyPhrases = doc.KeyPhrases.ToList()
                });
            }
            else
            {
                results.Add(new KeyPhraseExtractionResult
                {
                    Success = false,
                    ErrorMessage = doc.Error.Message
                });
            }
        }
        
        return results;
    }
    catch (Exception ex)
    {
        return batch.Select(_ => new KeyPhraseExtractionResult
        {
            Success = false,
            ErrorMessage = ex.Message
        }).ToList();
    }
}

public class KeyPhraseExtractionResult
{
    public bool Success { get; set; }
    public List<string> KeyPhrases { get; set; } = new List<string>();
    public string ErrorMessage { get; set; }
}
```

## 📊 Data Analysis Integration

### LINQ-based Analysis

```csharp
public class KeyPhraseAnalytics
{
    private readonly TextAnalyticsClient _client;
    
    public KeyPhraseAnalytics(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public AnalyticsReport GenerateAnalyticsReport(List<string> documents, List<string> documentCategories = null)
    {
        try
        {
            ExtractKeyPhrasesResultCollection response = _client.ExtractKeyPhrasesBatch(documents);
            
            // Extract all key phrases with metadata
            var allData = response
                .Where(doc => !doc.HasError)
                .SelectMany((doc, docIndex) => doc.KeyPhrases.Select(phrase => new KeyPhraseData
                {
                    DocumentIndex = docIndex,
                    Phrase = phrase,
                    WordCount = phrase.Split(' ').Length,
                    CharacterLength = phrase.Length,
                    Category = documentCategories?[docIndex] ?? "General",
                    HasNumbers = phrase.Any(char.IsDigit),
                    IsTechnical = IsTechnicalPhrase(phrase)
                }))
                .ToList();
            
            // Generate comprehensive analytics
            var report = new AnalyticsReport
            {
                TotalDocuments = documents.Count,
                TotalKeyPhrases = allData.Count,
                UniqueKeyPhrases = allData.Select(d => d.Phrase).Distinct().Count(),
                
                // Category analysis
                CategoryDistribution = allData
                    .GroupBy(d => d.Category)
                    .ToDictionary(g => g.Key, g => g.Count()),
                
                // Phrase frequency analysis
                PhraseFrequency = allData
                    .GroupBy(d => d.Phrase)
                    .OrderByDescending(g => g.Count())
                    .Take(15)
                    .ToDictionary(g => g.Key, g => g.Count()),
                
                // Length analysis
                LengthDistribution = allData
                    .GroupBy(d => d.WordCount)
                    .OrderBy(g => g.Key)
                    .ToDictionary(g => g.Key, g => g.Count()),
                
                // Average statistics
                AverageCharacterLength = allData.Average(d => d.CharacterLength),
                AverageWordCount = allData.Average(d => d.WordCount),
                
                // Technical vs non-technical
                TechnicalPhraseCount = allData.Count(d => d.IsTechnical),
                NonTechnicalPhraseCount = allData.Count(d => !d.IsTechnical),
                
                // Phrases with numbers
                PhrasesWithNumbers = allData.Count(d => d.HasNumbers),
                
                // Document-specific analysis
                DocumentAnalysis = allData
                    .GroupBy(d => d.DocumentIndex)
                    .Select(g => new DocumentAnalysisData
                    {
                        DocumentIndex = g.Key,
                        KeyPhraseCount = g.Count(),
                        UniqueKeyPhrases = g.Select(d => d.Phrase).Distinct().Count(),
                        AverageWordCount = g.Average(d => d.WordCount),
                        Category = g.First().Category
                    })
                    .ToList()
            };
            
            return report;
        }
        catch (RequestFailedException ex)
        {
            throw new InvalidOperationException($"Analytics generation failed: {ex.Message}", ex);
        }
    }
    
    private bool IsTechnicalPhrase(string phrase)
    {
        var technicalTerms = new[] { "technology", "software", "system", "platform", "algorithm", 
                                   "data", "analytics", "intelligence", "computing", "digital" };
        return technicalTerms.Any(term => phrase.ToLower().Contains(term));
    }
    
    public void PrintAnalyticsReport(AnalyticsReport report)
    {
        Console.WriteLine("KEY PHRASE ANALYTICS REPORT");
        Console.WriteLine(new string('=', 45));
        Console.WriteLine($"Total Documents: {report.TotalDocuments}");
        Console.WriteLine($"Total Key Phrases: {report.TotalKeyPhrases}");
        Console.WriteLine($"Unique Key Phrases: {report.UniqueKeyPhrases}");
        Console.WriteLine($"Average Character Length: {report.AverageCharacterLength:F2}");
        Console.WriteLine($"Average Word Count: {report.AverageWordCount:F2}");
        
        Console.WriteLine("\n📊 Category Distribution:");
        foreach (var category in report.CategoryDistribution.OrderByDescending(c => c.Value))
        {
            Console.WriteLine($"  {category.Key}: {category.Value} phrases");
        }
        
        Console.WriteLine("\n🔤 Top Key Phrases:");
        foreach (var phrase in report.PhraseFrequency.Take(10))
        {
            Console.WriteLine($"  • {phrase.Key}: {phrase.Value} occurrences");
        }
        
        Console.WriteLine("\n📏 Word Count Distribution:");
        foreach (var length in report.LengthDistribution)
        {
            Console.WriteLine($"  {length.Key} words: {length.Value} phrases");
        }
        
        Console.WriteLine($"\n🔧 Technical Analysis:");
        Console.WriteLine($"  Technical phrases: {report.TechnicalPhraseCount}");
        Console.WriteLine($"  Non-technical phrases: {report.NonTechnicalPhraseCount}");
        Console.WriteLine($"  Phrases with numbers: {report.PhrasesWithNumbers}");
        
        Console.WriteLine("\n📄 Document Analysis:");
        foreach (var doc in report.DocumentAnalysis)
        {
            Console.WriteLine($"  Document {doc.DocumentIndex + 1} ({doc.Category}):");
            Console.WriteLine($"    Key phrases: {doc.KeyPhraseCount}");
            Console.WriteLine($"    Unique phrases: {doc.UniqueKeyPhrases}");
            Console.WriteLine($"    Avg words per phrase: {doc.AverageWordCount:F2}");
        }
    }
}

// Supporting classes
public class KeyPhraseData
{
    public int DocumentIndex { get; set; }
    public string Phrase { get; set; }
    public int WordCount { get; set; }
    public int CharacterLength { get; set; }
    public string Category { get; set; }
    public bool HasNumbers { get; set; }
    public bool IsTechnical { get; set; }
}

public class DocumentAnalysisData
{
    public int DocumentIndex { get; set; }
    public int KeyPhraseCount { get; set; }
    public int UniqueKeyPhrases { get; set; }
    public double AverageWordCount { get; set; }
    public string Category { get; set; }
}

public class AnalyticsReport
{
    public int TotalDocuments { get; set; }
    public int TotalKeyPhrases { get; set; }
    public int UniqueKeyPhrases { get; set; }
    public Dictionary<string, int> CategoryDistribution { get; set; } = new Dictionary<string, int>();
    public Dictionary<string, int> PhraseFrequency { get; set; } = new Dictionary<string, int>();
    public Dictionary<int, int> LengthDistribution { get; set; } = new Dictionary<int, int>();
    public double AverageCharacterLength { get; set; }
    public double AverageWordCount { get; set; }
    public int TechnicalPhraseCount { get; set; }
    public int NonTechnicalPhraseCount { get; set; }
    public int PhrasesWithNumbers { get; set; }
    public List<DocumentAnalysisData> DocumentAnalysis { get; set; } = new List<DocumentAnalysisData>();
}

// Usage example for analytics
var analytics = new KeyPhraseAnalytics(client);

var businessDocuments = new List<string>
{
    "Digital transformation initiatives drive operational efficiency and customer satisfaction improvements.",
    "Financial technology solutions enhance payment processing and regulatory compliance capabilities.",
    "Healthcare analytics platforms improve patient outcomes through predictive modeling and data insights.",
    "Supply chain automation reduces costs and improves inventory management across global operations."
};

var documentCategories = new List<string> { "Business", "FinTech", "Healthcare", "Logistics" };

var analyticsReport = analytics.GenerateAnalyticsReport(businessDocuments, documentCategories);
analytics.PrintAnalyticsReport(analyticsReport);
```

## 🔍 Error Handling and Best Practices

### Robust Error Handling

```csharp
static void RobustKeyPhraseExtraction(TextAnalyticsClient client, List<string> documents)
{
    try
    {
        var options = new TextAnalyticsRequestOptions
        {
            IncludeStatistics = true,
            ModelVersion = "latest"
        };
        
        ExtractKeyPhrasesResultCollection response = client.ExtractKeyPhrasesBatch(documents, options: options);
        
        Console.WriteLine("Batch Statistics:");
        Console.WriteLine($"  Document Count: {response.Statistics.DocumentCount}");
        Console.WriteLine($"  Valid Document Count: {response.Statistics.ValidDocumentCount}");
        Console.WriteLine($"  Invalid Document Count: {response.Statistics.InvalidDocumentCount}");
        Console.WriteLine($"  Transaction Count: {response.Statistics.TransactionCount}");
        Console.WriteLine();
        
        int docIndex = 0;
        foreach (ExtractKeyPhrasesResult doc in response)
        {
            Console.WriteLine($"Document {docIndex + 1}:");
            
            if (!doc.HasError)
            {
                Console.WriteLine($"  Statistics: {doc.Statistics.CharacterCount} characters, {doc.Statistics.TransactionCount} transactions");
                Console.WriteLine($"  Key phrases found: {doc.KeyPhrases.Count}");
                
                foreach (string keyPhrase in doc.KeyPhrases.Take(5)) // Show first 5 phrases
                {
                    Console.WriteLine($"    • {keyPhrase}");
                }
                
                if (doc.KeyPhrases.Count > 5)
                {
                    Console.WriteLine($"    ... and {doc.KeyPhrases.Count - 5} more");
                }
            }
            else
            {
                Console.WriteLine($"  Error: {doc.Error.ErrorCode} - {doc.Error.Message}");
                
                // Handle specific error cases
                switch (doc.Error.ErrorCode)
                {
                    case "InvalidDocumentBatch":
                        Console.WriteLine("    Suggestion: Check document formatting and batch size limits");
                        break;
                    case "UnsupportedLanguageCode":
                        Console.WriteLine("    Suggestion: Verify language code is supported");
                        break;
                    case "InvalidDocument":
                        Console.WriteLine("    Suggestion: Check document content and length");
                        break;
                    default:
                        Console.WriteLine("    Suggestion: Check API documentation for error details");
                        break;
                }
            }
            Console.WriteLine();
            docIndex++;
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine($"API call failed: {ex.Message}");
        Console.WriteLine($"Error code: {ex.ErrorCode}");
        Console.WriteLine($"Status: {ex.Status}");
        
        // Implement retry logic or fallback here
        if (ex.Status == 429) // Too Many Requests
        {
            Console.WriteLine("Rate limit exceeded. Consider implementing exponential backoff retry.");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected error: {ex.Message}");
    }
}

// Test with mixed valid/invalid content
var testDocs = new List<string>
{
    "Cloud computing enables scalable business solutions.",
    "", // Empty string - will cause error
    "Machine learning algorithms improve data analysis.",
    new string('x', 5200), // Too long - will cause error
    "Digital transformation drives innovation."
};

RobustKeyPhraseExtraction(client, testDocs);
```

## 📚 Additional Resources

- [Azure AI Language .NET SDK Documentation](https://docs.microsoft.com/dotnet/api/azure.ai.textanalytics/)
- [Key Phrase Extraction Samples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics)