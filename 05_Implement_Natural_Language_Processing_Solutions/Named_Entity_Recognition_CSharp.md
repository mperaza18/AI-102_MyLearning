# 🔷 Named Entity Recognition C# SDK Examples

> **C# SDK examples for Named Entity Recognition using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```xml
<PackageReference Include="Azure.AI.TextAnalytics" Version="5.3.0" />
<PackageReference Include="Azure.Identity" Version="1.10.3" />
<PackageReference Include="Microsoft.Extensions.Configuration" Version="7.0.0" />
<PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="7.0.0" />
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
        RecognizeEntitiesBasic(client);
        RecognizeEntitiesBatch(client);
        RecognizeLinkedEntities(client);
        await RecognizeEntitiesAsync(client);
        AnalyzeBusinessDocument(client);
    }
}
```

## 🔧 Basic Examples

### Simple Entity Recognition

```csharp
static void RecognizeEntitiesBasic(TextAnalyticsClient client)
{
    string document = "Microsoft was founded by Bill Gates and Paul Allen in 1975. The company is headquartered in Redmond, Washington.";

    try
    {
        Response<CategorizedEntityCollection> response = client.RecognizeEntities(document);
        CategorizedEntityCollection entitiesResult = response.Value;

        Console.WriteLine($"Recognized {entitiesResult.Count} entities:");
        foreach (CategorizedEntity entity in entitiesResult)
        {
            Console.WriteLine($"  Entity: {entity.Text}");
            Console.WriteLine($"  Category: {entity.Category}");
            Console.WriteLine($"  Subcategory: {entity.SubCategory}");
            Console.WriteLine($"  Confidence Score: {entity.ConfidenceScore:0.00}");
            Console.WriteLine($"  Offset: {entity.Offset}, Length: {entity.Length}");
            Console.WriteLine();
        }
    }
    catch (RequestFailedException exception)
    {
        Console.WriteLine($"Error Code: {exception.ErrorCode}");
        Console.WriteLine($"Message: {exception.Message}");
    }
}
```

### Batch Entity Recognition

```csharp
static void RecognizeEntitiesBatch(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "Apple Inc. is an American multinational technology company headquartered in Cupertino, California.",
        "Google was founded by Larry Page and Sergey Brin while they were Ph.D. students at Stanford University in California.",
        "Amazon.com, Inc. is an American multinational technology company based in Seattle, Washington that focuses on e-commerce."
    };

    RecognizeEntitiesResultCollection results = client.RecognizeEntitiesBatch(documents);

    int i = 0;
    foreach (RecognizeEntitiesResult result in results)
    {
        Console.WriteLine($"Document {i + 1}:");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            // Group entities by category
            var entitiesByCategory = result.Entities
                .GroupBy(e => e.Category)
                .ToDictionary(g => g.Key, g => g.ToList());

            foreach (var category in entitiesByCategory)
            {
                Console.WriteLine($"  {category.Key} entities:");
                foreach (var entity in category.Value)
                {
                    Console.WriteLine($"    - {entity.Text} (confidence: {entity.ConfidenceScore:0.00})");
                }
            }
        }
        Console.WriteLine();
        i++;
    }
}
```

## 🔧 Advanced Examples

### Multi-Language Entity Recognition

```csharp
static void RecognizeEntitiesMultiLanguage(TextAnalyticsClient client)
{
    var documents = new List<TextDocumentInput>()
    {
        new TextDocumentInput("1", "Microsoft Corporation is headquartered in Redmond, Washington.")
        {
             Language = "en",
        },
        new TextDocumentInput("2", "Microsoft Corporation est basée à Redmond, Washington.")
        {
             Language = "fr",
        },
        new TextDocumentInput("3", "Microsoft Corporation tiene su sede en Redmond, Washington.")
        {
             Language = "es",
        },
        new TextDocumentInput("4", "Microsoft Corporation hat ihren Hauptsitz in Redmond, Washington.")
        {
             Language = "de",
        }
    };

    RecognizeEntitiesResultCollection results = client.RecognizeEntitiesBatch(documents);

    foreach (RecognizeEntitiesResult result in results)
    {
        Console.WriteLine($"Document ID: {result.Id}");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            Console.WriteLine($"  Recognized {result.Entities.Count} entities:");
            
            // Focus on Organization and Location entities
            var relevantEntities = result.Entities
                .Where(e => e.Category == EntityCategory.Organization || 
                           e.Category == EntityCategory.Location)
                .ToList();
            
            foreach (CategorizedEntity entity in relevantEntities)
            {
                Console.WriteLine($"    {entity.Category}: {entity.Text} (confidence: {entity.ConfidenceScore:0.00})");
            }
        }
        Console.WriteLine();
    }
}
```

### Entity Linking (Knowledge Base Integration)

```csharp
static void RecognizeLinkedEntities(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "Microsoft was founded by Bill Gates and Paul Allen. Steve Jobs founded Apple Inc.",
        "Albert Einstein developed the theory of relativity. Isaac Newton formulated the laws of motion.",
        "The Eiffel Tower is located in Paris, France. The Statue of Liberty is in New York."
    };

    RecognizeLinkedEntitiesResultCollection results = client.RecognizeLinkedEntitiesBatch(documents);

    int docIndex = 1;
    foreach (RecognizeLinkedEntitiesResult result in results)
    {
        Console.WriteLine($"Document {docIndex}:");
        
        if (result.HasError)
        {
            Console.WriteLine($"  Error: {result.Error.ErrorCode} - {result.Error.Message}");
        }
        else
        {
            Console.WriteLine($"  Recognized {result.Entities.Count} linked entities:");
            
            foreach (LinkedEntity linkedEntity in result.Entities)
            {
                Console.WriteLine($"    Entity: {linkedEntity.Name}");
                Console.WriteLine($"    Language: {linkedEntity.Language}");
                Console.WriteLine($"    Data Source: {linkedEntity.DataSource}");
                Console.WriteLine($"    URL: {linkedEntity.Url}");
                Console.WriteLine($"    Entity ID: {linkedEntity.DataSourceEntityId}");
                
                Console.WriteLine("    Matches:");
                foreach (LinkedEntityMatch match in linkedEntity.Matches)
                {
                    Console.WriteLine($"      Text: '{match.Text}'");
                    Console.WriteLine($"      Confidence: {match.ConfidenceScore:0.00}");
                    Console.WriteLine($"      Offset: {match.Offset}, Length: {match.Length}");
                }
                Console.WriteLine();
            }
        }
        Console.WriteLine();
        docIndex++;
    }
}
```

## 🔧 Specialized Use Cases

### Business Document Analysis

```csharp
public class BusinessEntityAnalyzer
{
    private readonly TextAnalyticsClient _client;
    
    public BusinessEntityAnalyzer(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public BusinessEntityResult AnalyzeBusinessDocument(string document)
    {
        try
        {
            Response<CategorizedEntityCollection> response = _client.RecognizeEntities(document);
            CategorizedEntityCollection entities = response.Value;
            
            var result = new BusinessEntityResult();
            
            foreach (CategorizedEntity entity in entities)
            {
                switch (entity.Category)
                {
                    case EntityCategory.Organization:
                        result.Organizations.Add(new EntityInfo 
                        { 
                            Text = entity.Text, 
                            Confidence = entity.ConfidenceScore,
                            Offset = entity.Offset,
                            Length = entity.Length
                        });
                        break;
                        
                    case EntityCategory.Person:
                        result.People.Add(new EntityInfo 
                        { 
                            Text = entity.Text, 
                            Confidence = entity.ConfidenceScore,
                            Offset = entity.Offset,
                            Length = entity.Length
                        });
                        break;
                        
                    case EntityCategory.Location:
                        result.Locations.Add(new EntityInfo 
                        { 
                            Text = entity.Text, 
                            Confidence = entity.ConfidenceScore,
                            Offset = entity.Offset,
                            Length = entity.Length
                        });
                        break;
                        
                    case EntityCategory.DateTime:
                        result.Dates.Add(new EntityInfo 
                        { 
                            Text = entity.Text, 
                            Confidence = entity.ConfidenceScore,
                            Offset = entity.Offset,
                            Length = entity.Length
                        });
                        break;
                        
                    case EntityCategory.Quantity:
                        if (entity.SubCategory == "Currency")
                        {
                            result.FinancialInfo.Add(new EntityInfo 
                            { 
                                Text = entity.Text, 
                                Confidence = entity.ConfidenceScore,
                                Offset = entity.Offset,
                                Length = entity.Length
                            });
                        }
                        break;
                }
            }
            
            return result;
        }
        catch (RequestFailedException ex)
        {
            throw new InvalidOperationException($"Entity recognition failed: {ex.Message}", ex);
        }
    }
    
    public ContractAnalysisResult AnalyzeContract(string contractText)
    {
        var businessEntities = AnalyzeBusinessDocument(contractText);
        
        return new ContractAnalysisResult
        {
            Parties = businessEntities.Organizations.Concat(businessEntities.People).ToList(),
            Locations = businessEntities.Locations,
            ImportantDates = businessEntities.Dates,
            FinancialTerms = businessEntities.FinancialInfo,
            TotalEntitiesFound = businessEntities.Organizations.Count + 
                               businessEntities.People.Count + 
                               businessEntities.Locations.Count + 
                               businessEntities.Dates.Count + 
                               businessEntities.FinancialInfo.Count
        };
    }
}

// Supporting classes
public class EntityInfo
{
    public string Text { get; set; }
    public double Confidence { get; set; }
    public int Offset { get; set; }
    public int Length { get; set; }
}

public class BusinessEntityResult
{
    public List<EntityInfo> Organizations { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> People { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> Locations { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> Dates { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> FinancialInfo { get; set; } = new List<EntityInfo>();
}

public class ContractAnalysisResult
{
    public List<EntityInfo> Parties { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> Locations { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> ImportantDates { get; set; } = new List<EntityInfo>();
    public List<EntityInfo> FinancialTerms { get; set; } = new List<EntityInfo>();
    public int TotalEntitiesFound { get; set; }
}

// Usage example
static void AnalyzeBusinessDocument(TextAnalyticsClient client)
{
    var analyzer = new BusinessEntityAnalyzer(client);
    
    string contractText = @"
        This Software License Agreement is entered into on January 15, 2024, between 
        TechCorp Inc., a corporation organized under the laws of Delaware, with offices 
        at 123 Innovation Drive, San Francisco, CA 94105, and Client Solutions LLC, 
        with principal offices at 456 Business Ave, New York, NY 10001. 
        
        The total contract value is $150,000 payable over 12 months. For questions, 
        contact John Smith at john.smith@techcorp.com or call +1-555-123-4567.
    ";
    
    var contractAnalysis = analyzer.AnalyzeContract(contractText);
    
    Console.WriteLine("CONTRACT ANALYSIS RESULTS");
    Console.WriteLine("========================");
    Console.WriteLine($"Total entities found: {contractAnalysis.TotalEntitiesFound}");
    Console.WriteLine();
    
    if (contractAnalysis.Parties.Any())
    {
        Console.WriteLine("Contract Parties:");
        foreach (var party in contractAnalysis.Parties)
        {
            Console.WriteLine($"  • {party.Text} (confidence: {party.Confidence:0.00})");
        }
        Console.WriteLine();
    }
    
    if (contractAnalysis.Locations.Any())
    {
        Console.WriteLine("Locations:");
        foreach (var location in contractAnalysis.Locations)
        {
            Console.WriteLine($"  • {location.Text} (confidence: {location.Confidence:0.00})");
        }
        Console.WriteLine();
    }
    
    if (contractAnalysis.ImportantDates.Any())
    {
        Console.WriteLine("Important Dates:");
        foreach (var date in contractAnalysis.ImportantDates)
        {
            Console.WriteLine($"  • {date.Text} (confidence: {date.Confidence:0.00})");
        }
        Console.WriteLine();
    }
    
    if (contractAnalysis.FinancialTerms.Any())
    {
        Console.WriteLine("Financial Terms:");
        foreach (var term in contractAnalysis.FinancialTerms)
        {
            Console.WriteLine($"  • {term.Text} (confidence: {term.Confidence:0.00})");
        }
    }
}
```

### Customer Feedback Entity Extraction

```csharp
public class CustomerFeedbackAnalyzer
{
    private readonly TextAnalyticsClient _client;
    
    public CustomerFeedbackAnalyzer(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public FeedbackAnalysisResult AnalyzeFeedbackBatch(List<string> feedbackList)
    {
        var result = new FeedbackAnalysisResult();
        
        try
        {
            RecognizeEntitiesResultCollection response = _client.RecognizeEntitiesBatch(feedbackList);
            
            int feedbackIndex = 0;
            foreach (RecognizeEntitiesResult doc in response)
            {
                if (!doc.HasError)
                {
                    Console.WriteLine($"Feedback {feedbackIndex + 1}: '{feedbackList[feedbackIndex].Substring(0, Math.Min(60, feedbackList[feedbackIndex].Length))}...'");
                    
                    foreach (CategorizedEntity entity in doc.Entities)
                    {
                        if (entity.ConfidenceScore > 0.7) // Filter by confidence
                        {
                            switch (entity.Category)
                            {
                                case EntityCategory.Organization:
                                    result.CompaniesMentioned.Add(entity.Text);
                                    break;
                                case EntityCategory.Person:
                                    result.PeopleMentioned.Add(entity.Text);
                                    break;
                                case EntityCategory.Location:
                                    result.LocationsMentioned.Add(entity.Text);
                                    break;
                                case EntityCategory.Product:
                                    result.ProductsMentioned.Add(entity.Text);
                                    break;
                            }
                        }
                    }
                    
                    Console.WriteLine($"  Entities found: {doc.Entities.Count}");
                }
                else
                {
                    Console.WriteLine($"Feedback {feedbackIndex + 1} - Error: {doc.Error.Message}");
                }
                
                feedbackIndex++;
            }
            
            // Remove duplicates and convert to final result
            result.CompaniesMentioned = result.CompaniesMentioned.Distinct().ToHashSet();
            result.PeopleMentioned = result.PeopleMentioned.Distinct().ToHashSet();
            result.LocationsMentioned = result.LocationsMentioned.Distinct().ToHashSet();
            result.ProductsMentioned = result.ProductsMentioned.Distinct().ToHashSet();
            
            return result;
        }
        catch (RequestFailedException ex)
        {
            throw new InvalidOperationException($"Feedback analysis failed: {ex.Message}", ex);
        }
    }
}

public class FeedbackAnalysisResult
{
    public HashSet<string> CompaniesMentioned { get; set; } = new HashSet<string>();
    public HashSet<string> PeopleMentioned { get; set; } = new HashSet<string>();
    public HashSet<string> LocationsMentioned { get; set; } = new HashSet<string>();
    public HashSet<string> ProductsMentioned { get; set; } = new HashSet<string>();
}

// Usage example
static void AnalyzeCustomerFeedback(TextAnalyticsClient client)
{
    var analyzer = new CustomerFeedbackAnalyzer(client);
    
    var customerFeedback = new List<string>
    {
        "I contacted Apple support about my iPhone 13 Pro battery issue. The representative Sarah was very helpful.",
        "Amazon Prime delivery to Seattle was delayed. I called customer service and spoke with Mike Johnson.",
        "Tesla Model S charging issue at the Fremont service center. The technician John fixed it quickly.",
        "Microsoft Teams integration with our CRM system works great. Our IT director Jane Smith is impressed."
    };
    
    var analysisResults = analyzer.AnalyzeFeedbackBatch(customerFeedback);
    
    Console.WriteLine("\n" + new string('=', 50));
    Console.WriteLine("CUSTOMER FEEDBACK ANALYSIS SUMMARY");
    Console.WriteLine(new string('=', 50));
    
    if (analysisResults.CompaniesMentioned.Any())
    {
        Console.WriteLine("\nCompanies Mentioned:");
        foreach (var company in analysisResults.CompaniesMentioned)
        {
            Console.WriteLine($"  • {company}");
        }
    }
    
    if (analysisResults.PeopleMentioned.Any())
    {
        Console.WriteLine("\nPeople Mentioned:");
        foreach (var person in analysisResults.PeopleMentioned)
        {
            Console.WriteLine($"  • {person}");
        }
    }
    
    if (analysisResults.LocationsMentioned.Any())
    {
        Console.WriteLine("\nLocations Mentioned:");
        foreach (var location in analysisResults.LocationsMentioned)
        {
            Console.WriteLine($"  • {location}");
        }
    }
    
    if (analysisResults.ProductsMentioned.Any())
    {
        Console.WriteLine("\nProducts Mentioned:");
        foreach (var product in analysisResults.ProductsMentioned)
        {
            Console.WriteLine($"  • {product}");
        }
    }
}
```

## 🚀 Async Operations

### Asynchronous Entity Recognition

```csharp
static async Task RecognizeEntitiesAsync(TextAnalyticsClient client)
{
    var documents = new List<string>
    {
        "Google CEO Sundar Pichai announced new AI features at Google I/O in Mountain View.",
        "Apple's Tim Cook presented the latest iPhone at the Steve Jobs Theater in Cupertino.",
        "Microsoft's Satya Nadella discussed Azure services at the Seattle headquarters."
    };

    try
    {
        Response<RecognizeEntitiesResultCollection> response = 
            await client.RecognizeEntitiesBatchAsync(documents);
        
        RecognizeEntitiesResultCollection results = response.Value;
        
        int docIndex = 0;
        foreach (RecognizeEntitiesResult result in results)
        {
            Console.WriteLine($"Document {docIndex + 1}:");
            
            if (!result.HasError)
            {
                var entitiesByCategory = result.Entities
                    .Where(e => e.ConfidenceScore > 0.8)
                    .GroupBy(e => e.Category)
                    .ToDictionary(g => g.Key, g => g.ToList());

                foreach (var categoryGroup in entitiesByCategory)
                {
                    Console.WriteLine($"  {categoryGroup.Key}:");
                    foreach (var entity in categoryGroup.Value)
                    {
                        Console.WriteLine($"    - {entity.Text} (confidence: {entity.ConfidenceScore:0.00})");
                    }
                }
            }
            else
            {
                Console.WriteLine($"  Error: {result.Error.Message}");
            }
            
            docIndex++;
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
    for (int i = 0; i < 50; i++)
    {
        documents.Add($"Company {i} is located in City {i % 10} and was founded by Person {i}.");
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
    
    // Show some statistics
    var allEntities = allResults
        .Where(r => r.Success)
        .SelectMany(r => r.Entities)
        .ToList();
    
    var entityCounts = allEntities
        .GroupBy(e => e.Category)
        .ToDictionary(g => g.Key, g => g.Count());
    
    Console.WriteLine("\nEntity Statistics:");
    foreach (var entityType in entityCounts)
    {
        Console.WriteLine($"  {entityType.Key}: {entityType.Value}");
    }
}

static async Task<List<EntityRecognitionResult>> ProcessBatchAsync(TextAnalyticsClient client, List<string> batch)
{
    try
    {
        var response = await client.RecognizeEntitiesBatchAsync(batch);
        var results = new List<EntityRecognitionResult>();
        
        for (int i = 0; i < response.Value.Count; i++)
        {
            var doc = response.Value[i];
            if (!doc.HasError)
            {
                results.Add(new EntityRecognitionResult
                {
                    Success = true,
                    Entities = doc.Entities.ToList()
                });
            }
            else
            {
                results.Add(new EntityRecognitionResult
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
        return batch.Select(_ => new EntityRecognitionResult
        {
            Success = false,
            ErrorMessage = ex.Message
        }).ToList();
    }
}

public class EntityRecognitionResult
{
    public bool Success { get; set; }
    public List<CategorizedEntity> Entities { get; set; } = new List<CategorizedEntity>();
    public string ErrorMessage { get; set; }
}
```

## 📊 Data Analysis Integration

### Entity Statistics with LINQ

```csharp
static void EntityAnalysisWithLinq(TextAnalyticsClient client)
{
    var newsArticles = new List<string>
    {
        "Apple Inc. announced record quarterly earnings. CEO Tim Cook praised the team's performance.",
        "Microsoft Azure cloud services expanded to three new regions including Tokyo, Japan.",
        "Tesla CEO Elon Musk visited the Berlin Gigafactory to oversee Model Y production.",
        "Amazon Web Services reported strong growth in artificial intelligence and machine learning services.",
        "Google's parent company Alphabet posted solid financial results for Q3 2023."
    };

    RecognizeEntitiesResultCollection results = client.RecognizeEntitiesBatch(newsArticles);

    // Extract all entities
    var allEntities = results
        .Where(doc => !doc.HasError)
        .SelectMany((doc, docIndex) => doc.Entities.Select(entity => new
        {
            DocumentIndex = docIndex,
            Text = entity.Text,
            Category = entity.Category.ToString(),
            Subcategory = entity.SubCategory,
            ConfidenceScore = entity.ConfidenceScore,
            Offset = entity.Offset,
            Length = entity.Length
        }))
        .ToList();

    Console.WriteLine("ENTITY ANALYSIS REPORT");
    Console.WriteLine("=" * 50);

    // 1. Entity category distribution
    Console.WriteLine("\n1. Entity Categories:");
    var categoryStats = allEntities
        .GroupBy(e => e.Category)
        .OrderByDescending(g => g.Count())
        .ToDictionary(g => g.Key, g => g.Count());

    foreach (var category in categoryStats)
    {
        Console.WriteLine($"  {category.Key}: {category.Value}");
    }

    // 2. Most frequent entities
    Console.WriteLine("\n2. Most Frequent Entities:");
    var entityFrequency = allEntities
        .GroupBy(e => e.Text)
        .OrderByDescending(g => g.Count())
        .Take(10)
        .ToDictionary(g => g.Key, g => g.Count());

    foreach (var entity in entityFrequency)
    {
        Console.WriteLine($"  {entity.Key}: {entity.Value} occurrences");
    }

    // 3. Average confidence by category
    Console.WriteLine("\n3. Average Confidence by Category:");
    var avgConfidence = allEntities
        .GroupBy(e => e.Category)
        .OrderByDescending(g => g.Average(e => e.ConfidenceScore))
        .ToDictionary(g => g.Key, g => g.Average(e => e.ConfidenceScore));

    foreach (var category in avgConfidence)
    {
        Console.WriteLine($"  {category.Key}: {category.Value:0.000}");
    }

    // 4. High-confidence entities (>0.8)
    Console.WriteLine("\n4. High-Confidence Entities (>0.8):");
    var highConfidenceEntities = allEntities
        .Where(e => e.ConfidenceScore > 0.8)
        .GroupBy(e => e.Text)
        .OrderByDescending(g => g.Count())
        .Take(10)
        .ToDictionary(g => g.Key, g => g.Count());

    foreach (var entity in highConfidenceEntities)
    {
        Console.WriteLine($"  {entity.Key}: {entity.Value} occurrences");
    }

    // 5. Entity distribution per document
    Console.WriteLine("\n5. Entity Distribution by Document:");
    var docStats = allEntities
        .GroupBy(e => e.DocumentIndex)
        .Select(g => new
        {
            DocumentIndex = g.Key,
            EntityCount = g.Count(),
            Categories = g.GroupBy(e => e.Category).ToDictionary(cg => cg.Key, cg => cg.Count())
        })
        .ToList();

    foreach (var docStat in docStats)
    {
        Console.WriteLine($"  Document {docStat.DocumentIndex + 1}: {docStat.EntityCount} entities");
        foreach (var category in docStat.Categories)
        {
            Console.WriteLine($"    {category.Key}: {category.Value}");
        }
    }
}
```

### Entity Network Analysis

```csharp
using System.Text.Json;

public class EntityNetwork
{
    public Dictionary<string, int> Nodes { get; set; } = new Dictionary<string, int>();
    public Dictionary<string, int> Edges { get; set; } = new Dictionary<string, int>();
    
    public void AddCoOccurrence(string entity1, string entity2)
    {
        // Add nodes
        if (!Nodes.ContainsKey(entity1)) Nodes[entity1] = 0;
        if (!Nodes.ContainsKey(entity2)) Nodes[entity2] = 0;
        
        Nodes[entity1]++;
        Nodes[entity2]++;
        
        // Add edge (ensure consistent ordering)
        var edge = string.Compare(entity1, entity2) < 0 ? $"{entity1}--{entity2}" : $"{entity2}--{entity1}";
        
        if (!Edges.ContainsKey(edge)) Edges[edge] = 0;
        Edges[edge]++;
    }
    
    public void PrintAnalysis()
    {
        Console.WriteLine("ENTITY NETWORK ANALYSIS");
        Console.WriteLine("=" * 30);
        Console.WriteLine($"Number of entities: {Nodes.Count}");
        Console.WriteLine($"Number of relationships: {Edges.Count}");
        
        // Most connected entities
        Console.WriteLine("\nMost Connected Entities:");
        var topEntities = Nodes
            .OrderByDescending(n => n.Value)
            .Take(5)
            .ToList();
        
        foreach (var entity in topEntities)
        {
            Console.WriteLine($"  {entity.Key}: {entity.Value} connections");
        }
        
        // Strongest relationships
        Console.WriteLine("\nStrongest Relationships:");
        var topEdges = Edges
            .OrderByDescending(e => e.Value)
            .Take(5)
            .ToList();
        
        foreach (var edge in topEdges)
        {
            Console.WriteLine($"  {edge.Key}: {edge.Value} co-occurrences");
        }
    }
}

static void CreateEntityNetwork(TextAnalyticsClient client)
{
    var businessDocuments = new List<string>
    {
        "Microsoft and Apple are competing in the personal computing market.",
        "Google and Amazon are major players in cloud computing services.",
        "Tesla and Apple are both innovative technology companies based in California.",
        "Microsoft Azure competes with Amazon Web Services and Google Cloud Platform.",
        "Apple and Google collaborate on some mobile technologies while competing in others."
    };

    RecognizeEntitiesResultCollection results = client.RecognizeEntitiesBatch(businessDocuments);
    
    var network = new EntityNetwork();
    
    foreach (RecognizeEntitiesResult doc in results)
    {
        if (!doc.HasError)
        {
            // Get high-confidence organization entities from this document
            var docEntities = doc.Entities
                .Where(e => e.Category == EntityCategory.Organization && e.ConfidenceScore > 0.7)
                .Select(e => e.Text)
                .ToList();
            
            // Create co-occurrence relationships
            for (int i = 0; i < docEntities.Count; i++)
            {
                for (int j = i + 1; j < docEntities.Count; j++)
                {
                    network.AddCoOccurrence(docEntities[i], docEntities[j]);
                }
            }
        }
    }
    
    network.PrintAnalysis();
}
```

## 🔍 Error Handling and Best Practices

### Robust Error Handling

```csharp
static void RobustEntityRecognition(TextAnalyticsClient client, List<string> documents)
{
    try
    {
        var options = new TextAnalyticsRequestOptions
        {
            IncludeStatistics = true,
            ModelVersion = "latest"
        };
        
        RecognizeEntitiesResultCollection response = client.RecognizeEntitiesBatch(documents, options: options);
        
        Console.WriteLine($"Batch Statistics:");
        Console.WriteLine($"  Document Count: {response.Statistics.DocumentCount}");
        Console.WriteLine($"  Valid Document Count: {response.Statistics.ValidDocumentCount}");
        Console.WriteLine($"  Invalid Document Count: {response.Statistics.InvalidDocumentCount}");
        Console.WriteLine($"  Transaction Count: {response.Statistics.TransactionCount}");
        Console.WriteLine();
        
        int docIndex = 0;
        foreach (RecognizeEntitiesResult doc in response)
        {
            Console.WriteLine($"Document {docIndex + 1}:");
            
            if (!doc.HasError)
            {
                Console.WriteLine($"  Statistics: {doc.Statistics.CharacterCount} characters, {doc.Statistics.TransactionCount} transactions");
                Console.WriteLine($"  Entities found: {doc.Entities.Count}");
                
                foreach (var entity in doc.Entities.Take(3)) // Show first 3 entities
                {
                    Console.WriteLine($"    {entity.Text} ({entity.Category}) - confidence: {entity.ConfidenceScore:0.00}");
                }
            }
            else
            {
                Console.WriteLine($"  Error: {doc.Error.ErrorCode} - {doc.Error.Message}");
                
                // Handle specific error cases
                switch (doc.Error.ErrorCode)
                {
                    case "InvalidDocumentBatch":
                        Console.WriteLine("    Suggestion: Check document formatting and size limits");
                        break;
                    case "UnsupportedLanguageCode":
                        Console.WriteLine("    Suggestion: Verify language code is supported");
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
    "Microsoft Corporation is a technology company.",
    "", // Empty string - will cause error
    "Apple Inc. develops consumer electronics.",
    new string('x', 5200), // Too long - will cause error
    "Google LLC focuses on internet services."
};

RobustEntityRecognition(client, testDocs);
```

## 📚 Additional Resources

- [Azure AI Language .NET SDK Documentation](https://docs.microsoft.com/dotnet/api/azure.ai.textanalytics/)
- [Named Entity Recognition Samples](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/textanalytics/Azure.AI.TextAnalytics)