# 🛡️ Content Safety with Azure AI Content Safety

> **Comprehensive code examples for detecting harmful content using Azure AI Content Safety services**

## 📋 Overview

Azure AI Content Safety is a specialized service that detects harmful content across multiple categories including hate speech, violence, self-harm, and sexual content. It provides confidence scores and severity levels to help applications make informed decisions about content moderation.

## 🔧 Key Features

- **🚫 Hate Speech Detection** - Identify content targeting individuals or groups based on attributes
- **⚔️ Violence Detection** - Detect descriptions or depictions of violence against people or animals  
- **🔞 Sexual Content Detection** - Identify sexual or adult content inappropriate for general audiences
- **⚠️ Self-Harm Detection** - Detect content promoting or describing self-harm activities
- **📊 Severity Levels** - Four-level severity scale (Safe, Low, Medium, High)
- **🎯 Confidence Scores** - Reliability scores for each detection
- **🌐 Multi-language Support** - Support for multiple languages
- **⚡ Real-time Analysis** - Fast content analysis for real-time applications

## 🐍 Python Examples

### Basic Content Safety Analysis

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
from azure.core.credentials import AzureKeyCredential
from azure.core.exceptions import HttpResponseError
import os

# Authentication
endpoint = os.environ["CONTENT_SAFETY_ENDPOINT"]
key = os.environ["CONTENT_SAFETY_KEY"]
credential = AzureKeyCredential(key)
client = ContentSafetyClient(endpoint, credential)

def analyze_text_content_basic():
    """Basic content safety analysis for text"""
    
    test_texts = [
        "This is a normal, safe message about technology.",
        "I hate all people from that country, they should all disappear.",
        "Here's a detailed description of how to harm yourself...",
        "This contains explicit sexual content that's inappropriate...",
        "I'm going to hurt someone very badly with these weapons."
    ]
    
    for idx, text in enumerate(test_texts):
        try:
            # Analyze text for harmful content
            request = AnalyzeTextOptions(
                text=text,
                categories=[
                    TextCategory.HATE,
                    TextCategory.SELF_HARM, 
                    TextCategory.SEXUAL,
                    TextCategory.VIOLENCE
                ],
                output_type="FourSeverityLevels"
            )
            
            response = client.analyze_text(request)
            
            print(f"Text {idx + 1}: '{text[:50]}...'")
            print("Content Safety Analysis:")
            
            for category_result in response.categories_analysis:
                category_name = category_result.category.value
                severity = category_result.severity
                
                print(f"  {category_name}: Severity {severity} (Score: {category_result.score:.3f})")
            
            # Overall assessment
            max_severity = max([result.severity for result in response.categories_analysis])
            if max_severity >= 6:  # High severity (6-7)
                print("  ⚠️ HIGH RISK: Content should be blocked")
            elif max_severity >= 4:  # Medium severity (4-5)
                print("  ⚡ MEDIUM RISK: Content needs review")
            elif max_severity >= 2:  # Low severity (2-3)
                print("  ⚠️ LOW RISK: Content may need attention")
            else:
                print("  ✅ SAFE: Content appears safe")
            
            print("-" * 60)
            
        except HttpResponseError as e:
            print(f"Error analyzing text {idx + 1}: {e.error.code} - {e.error.message}")
        except Exception as e:
            print(f"Unexpected error for text {idx + 1}: {str(e)}")

# Run basic content analysis
analyze_text_content_basic()
```

### Advanced Content Moderation with Custom Thresholds

```python
def advanced_content_moderation():
    """Advanced content moderation with custom severity thresholds"""
    
    # Define custom thresholds for different categories
    moderation_config = {
        "hate": {"block_threshold": 4, "review_threshold": 2},
        "violence": {"block_threshold": 6, "review_threshold": 4}, 
        "sexual": {"block_threshold": 4, "review_threshold": 2},
        "self_harm": {"block_threshold": 2, "review_threshold": 0}  # Very strict for self-harm
    }
    
    content_samples = [
        {
            "id": "social_post_1",
            "text": "Just posted a great photo from my vacation! The beach was amazing.",
            "context": "Social media post"
        },
        {
            "id": "comment_1", 
            "text": "I disagree with your political views and think you're completely wrong about this issue.",
            "context": "Comment thread"
        },
        {
            "id": "message_1",
            "text": "I really don't like people from that region, they're all the same and cause problems.",
            "context": "Private message"
        },
        {
            "id": "review_1",
            "text": "This movie has some intense fight scenes with realistic violence and blood.",
            "context": "Movie review"
        }
    ]
    
    moderation_results = []
    
    for content in content_samples:
        try:
            request = AnalyzeTextOptions(
                text=content["text"],
                categories=[TextCategory.HATE, TextCategory.SELF_HARM, TextCategory.SEXUAL, TextCategory.VIOLENCE],
                output_type="FourSeverityLevels"
            )
            
            response = client.analyze_text(request)
            
            # Analyze results against custom thresholds
            content_decision = {
                "content_id": content["id"],
                "context": content["context"],
                "text_preview": content["text"][:100] + "..." if len(content["text"]) > 100 else content["text"],
                "category_results": {},
                "overall_action": "APPROVE",
                "reasons": []
            }
            
            for category_result in response.categories_analysis:
                category_name = category_result.category.value.lower()
                severity = category_result.severity
                score = category_result.score
                
                content_decision["category_results"][category_name] = {
                    "severity": severity,
                    "score": score
                }
                
                # Check against thresholds
                if category_name in moderation_config:
                    thresholds = moderation_config[category_name]
                    
                    if severity >= thresholds["block_threshold"]:
                        content_decision["overall_action"] = "BLOCK"
                        content_decision["reasons"].append(f"High {category_name} content (severity {severity})")
                    elif severity >= thresholds["review_threshold"] and content_decision["overall_action"] != "BLOCK":
                        content_decision["overall_action"] = "REVIEW"
                        content_decision["reasons"].append(f"Moderate {category_name} content (severity {severity})")
            
            moderation_results.append(content_decision)
            
            # Display results
            print(f"Content ID: {content_decision['content_id']}")
            print(f"Context: {content_decision['context']}")
            print(f"Text: {content_decision['text_preview']}")
            print(f"Decision: {content_decision['overall_action']}")
            
            if content_decision['reasons']:
                print(f"Reasons: {'; '.join(content_decision['reasons'])}")
            
            print("Category Analysis:")
            for category, result in content_decision["category_results"].items():
                print(f"  {category.title()}: Severity {result['severity']} (Score: {result['score']:.3f})")
            
            print("-" * 80)
            
        except Exception as e:
            print(f"Error analyzing content {content['id']}: {str(e)}")
    
    # Summary statistics
    total_content = len(moderation_results)
    approved = sum(1 for result in moderation_results if result["overall_action"] == "APPROVE")
    blocked = sum(1 for result in moderation_results if result["overall_action"] == "BLOCK") 
    review = sum(1 for result in moderation_results if result["overall_action"] == "REVIEW")
    
    print(f"\nMODERATION SUMMARY:")
    print(f"Total content analyzed: {total_content}")
    print(f"Approved: {approved} ({approved/total_content*100:.1f}%)")
    print(f"Needs review: {review} ({review/total_content*100:.1f}%)")  
    print(f"Blocked: {blocked} ({blocked/total_content*100:.1f}%)")

advanced_content_moderation()
```

### Batch Content Analysis

```python
def batch_content_analysis(content_list):
    """Efficiently analyze multiple pieces of content"""
    
    results = {
        "total_analyzed": 0,
        "safe_content": 0,
        "flagged_content": 0,
        "errors": 0,
        "category_stats": {
            "hate": 0,
            "violence": 0,
            "sexual": 0,
            "self_harm": 0
        },
        "detailed_results": []
    }
    
    for idx, content_text in enumerate(content_list):
        try:
            request = AnalyzeTextOptions(
                text=content_text,
                categories=[TextCategory.HATE, TextCategory.SELF_HARM, TextCategory.SEXUAL, TextCategory.VIOLENCE],
                output_type="FourSeverityLevels"
            )
            
            response = client.analyze_text(request)
            
            # Process results
            content_analysis = {
                "index": idx,
                "text_preview": content_text[:50] + "..." if len(content_text) > 50 else content_text,
                "is_safe": True,
                "flagged_categories": [],
                "max_severity": 0
            }
            
            for category_result in response.categories_analysis:
                category_name = category_result.category.value.lower()
                severity = category_result.severity
                
                content_analysis["max_severity"] = max(content_analysis["max_severity"], severity)
                
                # Flag content with medium or high severity (4+)
                if severity >= 4:
                    content_analysis["is_safe"] = False
                    content_analysis["flagged_categories"].append({
                        "category": category_name,
                        "severity": severity,
                        "score": category_result.score
                    })
                    results["category_stats"][category_name] += 1
            
            # Update counters
            results["total_analyzed"] += 1
            if content_analysis["is_safe"]:
                results["safe_content"] += 1
            else:
                results["flagged_content"] += 1
            
            results["detailed_results"].append(content_analysis)
            
            # Progress indicator
            if (idx + 1) % 10 == 0:
                print(f"Processed {idx + 1}/{len(content_list)} items...")
                
        except Exception as e:
            results["errors"] += 1
            print(f"Error processing item {idx}: {str(e)}")
    
    # Display summary
    print(f"\nBATCH ANALYSIS COMPLETE")
    print(f"=" * 50)
    print(f"Total items: {results['total_analyzed']}")
    print(f"Safe content: {results['safe_content']} ({results['safe_content']/results['total_analyzed']*100:.1f}%)")
    print(f"Flagged content: {results['flagged_content']} ({results['flagged_content']/results['total_analyzed']*100:.1f}%)")
    print(f"Processing errors: {results['errors']}")
    
    print(f"\nFLAGGED CATEGORIES:")
    for category, count in results["category_stats"].items():
        if count > 0:
            print(f"  {category.title()}: {count} items")
    
    # Show flagged items
    flagged_items = [item for item in results["detailed_results"] if not item["is_safe"]]
    if flagged_items:
        print(f"\nFLAGGED CONTENT DETAILS:")
        for item in flagged_items[:5]:  # Show first 5 flagged items
            print(f"  Item {item['index']}: {item['text_preview']}")
            for flag in item["flagged_categories"]:
                print(f"    - {flag['category'].title()}: Severity {flag['severity']} (Score: {flag['score']:.3f})")
    
    return results

# Example usage with sample content
sample_content = [
    "Great job on the presentation! Really well done.",
    "I completely disagree with this approach, but I respect your opinion.",
    "This product is terrible and I want my money back.",
    "The weather is nice today, perfect for a walk.",
    "I can't stand people who think differently than me, they're all idiots.",
    "Looking forward to the weekend and some relaxation time."
]

batch_results = batch_content_analysis(sample_content)
```

### Real-time Content Filtering

```python
import time
from datetime import datetime

class ContentModerator:
    """Real-time content moderation class"""
    
    def __init__(self, client, config=None):
        self.client = client
        self.config = config or {
            "auto_block_threshold": 6,  # Auto-block high severity
            "review_threshold": 4,      # Flag for review
            "log_all_analysis": True
        }
        self.analysis_log = []
    
    def moderate_content(self, content, user_id=None, context=None):
        """Moderate a single piece of content in real-time"""
        
        start_time = time.time()
        
        try:
            request = AnalyzeTextOptions(
                text=content,
                categories=[TextCategory.HATE, TextCategory.SELF_HARM, TextCategory.SEXUAL, TextCategory.VIOLENCE],
                output_type="FourSeverityLevels"
            )
            
            response = self.client.analyze_text(request)
            analysis_time = time.time() - start_time
            
            # Process results
            moderation_result = {
                "timestamp": datetime.now().isoformat(),
                "user_id": user_id,
                "context": context,
                "content_length": len(content),
                "analysis_time_ms": round(analysis_time * 1000, 2),
                "decision": "APPROVE",
                "confidence": "HIGH",
                "flagged_categories": [],
                "max_severity": 0,
                "requires_human_review": False
            }
            
            # Analyze each category
            for category_result in response.categories_analysis:
                category_name = category_result.category.value.lower()
                severity = category_result.severity
                score = category_result.score
                
                moderation_result["max_severity"] = max(moderation_result["max_severity"], severity)
                
                if severity >= self.config["auto_block_threshold"]:
                    moderation_result["decision"] = "BLOCK"
                    moderation_result["confidence"] = "HIGH"
                    moderation_result["flagged_categories"].append({
                        "category": category_name,
                        "severity": severity,
                        "score": score,
                        "reason": "High severity content"
                    })
                elif severity >= self.config["review_threshold"]:
                    if moderation_result["decision"] != "BLOCK":
                        moderation_result["decision"] = "REVIEW"
                        moderation_result["requires_human_review"] = True
                    moderation_result["flagged_categories"].append({
                        "category": category_name,
                        "severity": severity,
                        "score": score,
                        "reason": "Moderate severity content"
                    })
            
            # Adjust confidence based on score consistency
            if moderation_result["max_severity"] >= 4:
                avg_score = sum(result.score for result in response.categories_analysis) / len(response.categories_analysis)
                if avg_score < 0.7:
                    moderation_result["confidence"] = "MEDIUM"
                elif avg_score < 0.5:
                    moderation_result["confidence"] = "LOW"
            
            # Log analysis if configured
            if self.config["log_all_analysis"]:
                self.analysis_log.append(moderation_result)
            
            return moderation_result
            
        except Exception as e:
            error_result = {
                "timestamp": datetime.now().isoformat(),
                "user_id": user_id,
                "context": context,
                "decision": "ERROR",
                "error": str(e),
                "analysis_time_ms": round((time.time() - start_time) * 1000, 2)
            }
            
            if self.config["log_all_analysis"]:
                self.analysis_log.append(error_result)
            
            return error_result
    
    def get_moderation_stats(self):
        """Get statistics from recent moderation activities"""
        
        if not self.analysis_log:
            return {"message": "No moderation data available"}
        
        total_items = len(self.analysis_log)
        decisions = {}
        avg_analysis_time = 0
        error_count = 0
        
        for log_entry in self.analysis_log:
            decision = log_entry.get("decision", "UNKNOWN")
            decisions[decision] = decisions.get(decision, 0) + 1
            
            if "analysis_time_ms" in log_entry:
                avg_analysis_time += log_entry["analysis_time_ms"]
            
            if decision == "ERROR":
                error_count += 1
        
        avg_analysis_time = avg_analysis_time / max(total_items - error_count, 1)
        
        return {
            "total_analyzed": total_items,
            "decisions": decisions,
            "average_analysis_time_ms": round(avg_analysis_time, 2),
            "error_rate": round(error_count / total_items * 100, 2) if total_items > 0 else 0
        }

# Example usage
moderator = ContentModerator(client)

# Test real-time moderation
test_messages = [
    ("user123", "Just wanted to say thanks for all your help!", "chat"),
    ("user456", "I hate everyone in this stupid group!", "chat"),
    ("user789", "Check out this cool article I found about AI", "forum_post"),
    ("user101", "People like you make me want to hurt myself", "private_message")
]

print("REAL-TIME CONTENT MODERATION")
print("=" * 50)

for user_id, content, context in test_messages:
    result = moderator.moderate_content(content, user_id=user_id, context=context)
    
    print(f"\nUser: {user_id}")
    print(f"Content: {content}")
    print(f"Decision: {result['decision']} ({result.get('confidence', 'N/A')} confidence)")
    print(f"Analysis time: {result.get('analysis_time_ms', 0)}ms")
    
    if result['flagged_categories']:
        print("Flagged for:")
        for flag in result['flagged_categories']:
            print(f"  - {flag['category'].title()}: Severity {flag['severity']}")

# Show statistics
print(f"\nMODERATION STATISTICS:")
stats = moderator.get_moderation_stats()
for key, value in stats.items():
    print(f"  {key.replace('_', ' ').title()}: {value}")
```

## 🔷 C# Examples

### Basic Content Safety Analysis

```csharp
using Azure.AI.ContentSafety;
using Azure.AI.ContentSafety.Models;
using Azure.Core;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static string endpoint = Environment.GetEnvironmentVariable("CONTENT_SAFETY_ENDPOINT");
    static string key = Environment.GetEnvironmentVariable("CONTENT_SAFETY_KEY");
    
    static async Task Main(string[] args)
    {
        var credential = new AzureKeyCredential(key);
        var client = new ContentSafetyClient(new Uri(endpoint), credential);
        
        await AnalyzeTextContent(client);
        await BatchContentAnalysis(client);
    }
    
    static async Task AnalyzeTextContent(ContentSafetyClient client)
    {
        var testTexts = new List<string>
        {
            "This is a completely normal and safe message.",
            "I really dislike people from that country, they cause all the problems.",
            "Here's how you can harm yourself using common household items...",
            "This message contains explicit adult content that's inappropriate..."
        };

        foreach (var (text, index) in testTexts.Select((text, index) => (text, index)))
        {
            try
            {
                var request = new AnalyzeTextOptions(text)
                {
                    Categories = { TextCategory.Hate, TextCategory.SelfHarm, TextCategory.Sexual, TextCategory.Violence },
                    OutputType = AnalyzeTextOutputType.FourSeverityLevels
                };

                var response = await client.AnalyzeTextAsync(request);

                Console.WriteLine($"Text {index + 1}: {text.Substring(0, Math.Min(text.Length, 50))}...");
                Console.WriteLine("Content Safety Analysis:");

                int maxSeverity = 0;
                foreach (var categoryAnalysis in response.Value.CategoriesAnalysis)
                {
                    var categoryName = categoryAnalysis.Category.ToString();
                    var severity = categoryAnalysis.Severity;
                    var score = categoryAnalysis.Score;

                    maxSeverity = Math.Max(maxSeverity, severity ?? 0);
                    
                    Console.WriteLine($"  {categoryName}: Severity {severity} (Score: {score:F3})");
                }

                // Overall assessment
                string riskLevel = maxSeverity switch
                {
                    >= 6 => "⚠️ HIGH RISK: Content should be blocked",
                    >= 4 => "⚡ MEDIUM RISK: Content needs review", 
                    >= 2 => "⚠️ LOW RISK: Content may need attention",
                    _ => "✅ SAFE: Content appears safe"
                };
                
                Console.WriteLine($"  {riskLevel}");
                Console.WriteLine(new string('-', 60));
            }
            catch (RequestFailedException ex)
            {
                Console.WriteLine($"Error analyzing text {index + 1}: {ex.ErrorCode} - {ex.Message}");
            }
        }
    }
    
    static async Task BatchContentAnalysis(ContentSafetyClient client)
    {
        var contentItems = new List<(string Id, string Text, string Context)>
        {
            ("post_1", "Great weather today! Perfect for a picnic.", "Social media post"),
            ("comment_1", "I disagree with this policy decision completely.", "Forum comment"),
            ("message_1", "Those people are all the same and I can't stand them.", "Chat message"),
            ("review_1", "This movie has intense violence but great storytelling.", "Product review")
        };

        var moderationResults = new List<object>();
        
        foreach (var item in contentItems)
        {
            try
            {
                var request = new AnalyzeTextOptions(item.Text)
                {
                    Categories = { TextCategory.Hate, TextCategory.SelfHarm, TextCategory.Sexual, TextCategory.Violence },
                    OutputType = AnalyzeTextOutputType.FourSeverityLevels
                };

                var response = await client.AnalyzeTextAsync(request);
                
                var result = new
                {
                    ContentId = item.Id,
                    Context = item.Context,
                    TextPreview = item.Text.Length > 50 ? item.Text.Substring(0, 50) + "..." : item.Text,
                    Decision = "APPROVE",
                    FlaggedCategories = new List<object>(),
                    MaxSeverity = 0
                };

                foreach (var categoryAnalysis in response.Value.CategoriesAnalysis)
                {
                    var severity = categoryAnalysis.Severity ?? 0;
                    if (severity > result.MaxSeverity)
                    {
                        result = result with { MaxSeverity = severity };
                    }

                    if (severity >= 4) // Medium to High severity
                    {
                        result.FlaggedCategories.Add(new
                        {
                            Category = categoryAnalysis.Category.ToString(),
                            Severity = severity,
                            Score = categoryAnalysis.Score
                        });
                        
                        result = result with { Decision = severity >= 6 ? "BLOCK" : "REVIEW" };
                    }
                }

                moderationResults.Add(result);
                
                Console.WriteLine($"Content ID: {result.ContentId}");
                Console.WriteLine($"Context: {result.Context}");
                Console.WriteLine($"Decision: {result.Decision}");
                Console.WriteLine($"Max Severity: {result.MaxSeverity}");
                
                if (result.FlaggedCategories.Count > 0)
                {
                    Console.WriteLine("Flagged Categories:");
                    foreach (dynamic flag in result.FlaggedCategories)
                    {
                        Console.WriteLine($"  - {flag.Category}: Severity {flag.Severity}");
                    }
                }
                Console.WriteLine(new string('-', 50));
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error processing {item.Id}: {ex.Message}");
            }
        }
        
        // Summary
        var totalItems = moderationResults.Count;
        var approved = moderationResults.Count(r => ((dynamic)r).Decision == "APPROVE");
        var blocked = moderationResults.Count(r => ((dynamic)r).Decision == "BLOCK");
        var review = moderationResults.Count(r => ((dynamic)r).Decision == "REVIEW");
        
        Console.WriteLine($"\nMODERATION SUMMARY:");
        Console.WriteLine($"Total analyzed: {totalItems}");
        Console.WriteLine($"Approved: {approved} ({(double)approved / totalItems * 100:F1}%)");
        Console.WriteLine($"Needs review: {review} ({(double)review / totalItems * 100:F1}%)");
        Console.WriteLine($"Blocked: {blocked} ({(double)blocked / totalItems * 100:F1}%)");
    }
}
```

### Advanced Content Moderation Pipeline

```csharp
public class ContentModerationPipeline
{
    private readonly ContentSafetyClient _client;
    private readonly ModerationConfig _config;
    private readonly List<ModerationResult> _moderationLog;

    public ContentModerationPipeline(ContentSafetyClient client, ModerationConfig config = null)
    {
        _client = client;
        _config = config ?? new ModerationConfig();
        _moderationLog = new List<ModerationResult>();
    }

    public async Task<ModerationResult> ModerateContentAsync(string content, string userId = null, string context = null)
    {
        var startTime = DateTime.UtcNow;
        
        try
        {
            var request = new AnalyzeTextOptions(content)
            {
                Categories = { TextCategory.Hate, TextCategory.SelfHarm, TextCategory.Sexual, TextCategory.Violence },
                OutputType = AnalyzeTextOutputType.FourSeverityLevels
            };

            var response = await _client.AnalyzeTextAsync(request);
            var analysisTime = DateTime.UtcNow - startTime;

            var result = new ModerationResult
            {
                Timestamp = DateTime.UtcNow,
                UserId = userId,
                Context = context,
                ContentLength = content.Length,
                AnalysisTimeMs = (int)analysisTime.TotalMilliseconds,
                Decision = ModerationDecision.Approve,
                Confidence = ConfidenceLevel.High,
                FlaggedCategories = new List<FlaggedCategory>(),
                MaxSeverity = 0
            };

            foreach (var categoryAnalysis in response.Value.CategoriesAnalysis)
            {
                var severity = categoryAnalysis.Severity ?? 0;
                result.MaxSeverity = Math.Max(result.MaxSeverity, severity);

                if (severity >= _config.AutoBlockThreshold)
                {
                    result.Decision = ModerationDecision.Block;
                    result.FlaggedCategories.Add(new FlaggedCategory
                    {
                        Category = categoryAnalysis.Category.ToString(),
                        Severity = severity,
                        Score = categoryAnalysis.Score ?? 0,
                        Reason = "High severity content"
                    });
                }
                else if (severity >= _config.ReviewThreshold && result.Decision != ModerationDecision.Block)
                {
                    result.Decision = ModerationDecision.Review;
                    result.RequiresHumanReview = true;
                    result.FlaggedCategories.Add(new FlaggedCategory
                    {
                        Category = categoryAnalysis.Category.ToString(),
                        Severity = severity,
                        Score = categoryAnalysis.Score ?? 0,
                        Reason = "Moderate severity content"
                    });
                }
            }

            _moderationLog.Add(result);
            return result;
        }
        catch (Exception ex)
        {
            var errorResult = new ModerationResult
            {
                Timestamp = DateTime.UtcNow,
                UserId = userId,
                Context = context,
                Decision = ModerationDecision.Error,
                ErrorMessage = ex.Message,
                AnalysisTimeMs = (int)(DateTime.UtcNow - startTime).TotalMilliseconds
            };

            _moderationLog.Add(errorResult);
            return errorResult;
        }
    }

    public ModerationStats GetModerationStats()
    {
        if (!_moderationLog.Any())
            return new ModerationStats { Message = "No moderation data available" };

        var totalItems = _moderationLog.Count;
        var decisions = _moderationLog.GroupBy(r => r.Decision)
                                    .ToDictionary(g => g.Key.ToString(), g => g.Count());
        
        var successfulAnalyses = _moderationLog.Where(r => r.Decision != ModerationDecision.Error);
        var avgAnalysisTime = successfulAnalyses.Any() ? 
                             successfulAnalyses.Average(r => r.AnalysisTimeMs) : 0;
        
        var errorCount = _moderationLog.Count(r => r.Decision == ModerationDecision.Error);

        return new ModerationStats
        {
            TotalAnalyzed = totalItems,
            Decisions = decisions,
            AverageAnalysisTimeMs = Math.Round(avgAnalysisTime, 2),
            ErrorRate = totalItems > 0 ? Math.Round((double)errorCount / totalItems * 100, 2) : 0
        };
    }
}

// Supporting classes
public class ModerationConfig
{
    public int AutoBlockThreshold { get; set; } = 6;
    public int ReviewThreshold { get; set; } = 4;
    public bool LogAllAnalysis { get; set; } = true;
}

public class ModerationResult
{
    public DateTime Timestamp { get; set; }
    public string UserId { get; set; }
    public string Context { get; set; }
    public int ContentLength { get; set; }
    public int AnalysisTimeMs { get; set; }
    public ModerationDecision Decision { get; set; }
    public ConfidenceLevel Confidence { get; set; }
    public List<FlaggedCategory> FlaggedCategories { get; set; }
    public int MaxSeverity { get; set; }
    public bool RequiresHumanReview { get; set; }
    public string ErrorMessage { get; set; }
}

public class FlaggedCategory
{
    public string Category { get; set; }
    public int Severity { get; set; }
    public double Score { get; set; }
    public string Reason { get; set; }
}

public enum ModerationDecision
{
    Approve,
    Review, 
    Block,
    Error
}

public enum ConfidenceLevel
{
    Low,
    Medium,
    High
}

public class ModerationStats
{
    public string Message { get; set; }
    public int TotalAnalyzed { get; set; }
    public Dictionary<string, int> Decisions { get; set; }
    public double AverageAnalysisTimeMs { get; set; }
    public double ErrorRate { get; set; }
}
```

## 🌐 REST API Examples

### Basic Content Safety Analysis

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/contentsafety/text:analyze?api-version=2024-09-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "I hate all people from that country, they should disappear.",
    "categories": ["Hate", "SelfHarm", "Sexual", "Violence"],
    "outputType": "FourSeverityLevels"
  }'
```

### Content Safety with Custom Thresholds

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/contentsafety/text:analyze?api-version=2024-09-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "This movie has some intense fight scenes with realistic violence.",
    "categories": ["Violence"],
    "outputType": "FourSeverityLevels"
  }'
```

### Python REST Implementation

```python
import requests
import json

def content_safety_rest():
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    url = f"{endpoint}/contentsafety/text:analyze?api-version=2024-09-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    test_content = [
        "This is a normal, safe message about technology.",
        "I hate all people from that region, they're terrible.",
        "Here's how to hurt yourself using these methods...",
        "This contains explicit adult content..."
    ]
    
    for idx, text in enumerate(test_content):
        payload = {
            "text": text,
            "categories": ["Hate", "SelfHarm", "Sexual", "Violence"],
            "outputType": "FourSeverityLevels"
        }
        
        response = requests.post(url, json=payload, headers=headers)
        
        if response.status_code == 200:
            result = response.json()
            
            print(f"Text {idx + 1}: {text[:50]}...")
            print("Content Safety Analysis:")
            
            max_severity = 0
            for category in result.get("categoriesAnalysis", []):
                category_name = category["category"]
                severity = category["severity"]
                score = category["score"]
                
                max_severity = max(max_severity, severity)
                print(f"  {category_name}: Severity {severity} (Score: {score:.3f})")
            
            # Risk assessment
            if max_severity >= 6:
                risk_level = "⚠️ HIGH RISK: Content should be blocked"
            elif max_severity >= 4:
                risk_level = "⚡ MEDIUM RISK: Content needs review"
            elif max_severity >= 2:
                risk_level = "⚠️ LOW RISK: Content may need attention"
            else:
                risk_level = "✅ SAFE: Content appears safe"
            
            print(f"  {risk_level}")
            print("-" * 60)
        else:
            print(f"Error analyzing text {idx + 1}: {response.status_code} - {response.text}")

content_safety_rest()
```

### Advanced REST Batch Processing

```python
def batch_content_safety_rest(content_list):
    """Process multiple content items using REST API"""
    
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    url = f"{endpoint}/contentsafety/text:analyze?api-version=2024-09-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    results = []
    
    for idx, content_item in enumerate(content_list):
        try:
            payload = {
                "text": content_item["text"],
                "categories": ["Hate", "SelfHarm", "Sexual", "Violence"],
                "outputType": "FourSeverityLevels"
            }
            
            response = requests.post(url, json=payload, headers=headers)
            
            if response.status_code == 200:
                analysis_result = response.json()
                
                # Process the results
                moderation_decision = {
                    "content_id": content_item.get("id", f"item_{idx}"),
                    "original_text": content_item["text"],
                    "decision": "APPROVE",
                    "flagged_categories": [],
                    "max_severity": 0,
                    "analysis_successful": True
                }
                
                for category in analysis_result.get("categoriesAnalysis", []):
                    severity = category["severity"]
                    moderation_decision["max_severity"] = max(moderation_decision["max_severity"], severity)
                    
                    if severity >= 6:  # High severity
                        moderation_decision["decision"] = "BLOCK"
                        moderation_decision["flagged_categories"].append({
                            "category": category["category"],
                            "severity": severity,
                            "score": category["score"],
                            "action": "block"
                        })
                    elif severity >= 4:  # Medium severity
                        if moderation_decision["decision"] != "BLOCK":
                            moderation_decision["decision"] = "REVIEW"
                        moderation_decision["flagged_categories"].append({
                            "category": category["category"],
                            "severity": severity,
                            "score": category["score"],
                            "action": "review"
                        })
                
                results.append(moderation_decision)
                
                # Progress indicator
                if (idx + 1) % 5 == 0:
                    print(f"Processed {idx + 1}/{len(content_list)} items...")
                    
            else:
                # Handle API errors
                error_result = {
                    "content_id": content_item.get("id", f"item_{idx}"),
                    "decision": "ERROR",
                    "error_code": response.status_code,
                    "error_message": response.text,
                    "analysis_successful": False
                }
                results.append(error_result)
                
        except Exception as e:
            # Handle network or other errors
            error_result = {
                "content_id": content_item.get("id", f"item_{idx}"),
                "decision": "ERROR", 
                "error_message": str(e),
                "analysis_successful": False
            }
            results.append(error_result)
    
    # Generate summary statistics
    successful_analyses = [r for r in results if r.get("analysis_successful", False)]
    total_items = len(results)
    
    if successful_analyses:
        approved = sum(1 for r in successful_analyses if r["decision"] == "APPROVE")
        blocked = sum(1 for r in successful_analyses if r["decision"] == "BLOCK")
        review = sum(1 for r in successful_analyses if r["decision"] == "REVIEW")
        errors = total_items - len(successful_analyses)
        
        print(f"\nBATCH PROCESSING SUMMARY:")
        print(f"Total items: {total_items}")
        print(f"Successful analyses: {len(successful_analyses)}")
        print(f"Approved: {approved} ({approved/len(successful_analyses)*100:.1f}%)")
        print(f"Needs review: {review} ({review/len(successful_analyses)*100:.1f}%)")
        print(f"Blocked: {blocked} ({blocked/len(successful_analyses)*100:.1f}%)")
        print(f"Errors: {errors}")
        
        # Show some flagged content examples
        flagged_content = [r for r in successful_analyses if r["decision"] in ["BLOCK", "REVIEW"]]
        if flagged_content:
            print(f"\nFLAGGED CONTENT EXAMPLES (showing first 3):")
            for item in flagged_content[:3]:
                print(f"  ID: {item['content_id']} - Decision: {item['decision']}")
                print(f"  Text: {item['original_text'][:100]}...")
                for flag in item["flagged_categories"]:
                    print(f"    - {flag['category']}: Severity {flag['severity']}")
                print()
    
    return results

# Example usage
sample_content = [
    {"id": "post_1", "text": "Great job everyone, keep up the excellent work!"},
    {"id": "comment_1", "text": "I strongly disagree with this opinion but respect your right to have it."},
    {"id": "message_1", "text": "I can't stand people from that country, they're all troublemakers."},
    {"id": "review_1", "text": "This movie contains graphic violence and disturbing scenes."},
    {"id": "chat_1", "text": "Sometimes I feel like ending it all, life is too hard."}
]

batch_results = batch_content_safety_rest(sample_content)
```

## 📊 Response Format

### Sample JSON Response

```json
{
  "categoriesAnalysis": [
    {
      "category": "Hate",
      "severity": 6,
      "score": 0.8945
    },
    {
      "category": "Violence", 
      "severity": 2,
      "score": 0.1234
    },
    {
      "category": "Sexual",
      "severity": 0,
      "score": 0.0123
    },
    {
      "category": "SelfHarm",
      "severity": 0,
      "score": 0.0089
    }
  ]
}
```

## 🎯 Content Categories and Severity Levels

### Content Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **Hate** | Content targeting individuals/groups | Racism, discrimination, slurs |
| **Violence** | Descriptions of violence | Physical harm, weapons, threats |
| **Sexual** | Adult/sexual content | Explicit descriptions, inappropriate content |
| **SelfHarm** | Self-injury content | Suicide ideation, self-harm instructions |

### Severity Scale (0-7)

| Level | Description | Recommended Action |
|-------|-------------|-------------------|
| **0-1** | Safe content | ✅ Allow |
| **2-3** | Low severity | ⚠️ Monitor or flag |
| **4-5** | Medium severity | 📋 Review required |
| **6-7** | High severity | 🚫 Block content |

## 🔧 Best Practices

### Content Moderation Workflow

```python
class ContentModerationWorkflow:
    """Complete content moderation workflow implementation"""
    
    def __init__(self, client, config=None):
        self.client = client
        self.config = config or self._get_default_config()
        self.moderation_queue = []
        self.human_review_queue = []
    
    def _get_default_config(self):
        return {
            "severity_thresholds": {
                "hate": {"block": 4, "review": 2},
                "violence": {"block": 6, "review": 4},
                "sexual": {"block": 4, "review": 2}, 
                "self_harm": {"block": 2, "review": 0}
            },
            "auto_actions": {
                "high_confidence_block": True,
                "escalate_to_human": True,
                "log_all_decisions": True
            }
        }
    
    async def process_content(self, content_item):
        """Process a single content item through the moderation pipeline"""
        
        # Step 1: Content Safety Analysis
        safety_result = await self._analyze_content_safety(content_item)
        
        # Step 2: Apply business rules
        moderation_decision = self._apply_moderation_rules(safety_result)
        
        # Step 3: Handle decision
        await self._handle_moderation_decision(content_item, moderation_decision)
        
        return moderation_decision
    
    async def _analyze_content_safety(self, content_item):
        """Analyze content using Azure Content Safety"""
        
        request = AnalyzeTextOptions(
            text=content_item["text"],
            categories=[TextCategory.HATE, TextCategory.SELF_HARM, TextCategory.SEXUAL, TextCategory.VIOLENCE],
            output_type="FourSeverityLevels"
        )
        
        response = self.client.analyze_text(request)
        
        return {
            "content_id": content_item.get("id"),
            "analysis_result": response,
            "timestamp": time.time()
        }
    
    def _apply_moderation_rules(self, safety_result):
        """Apply business-specific moderation rules"""
        
        decision = {
            "action": "APPROVE",
            "confidence": "HIGH",
            "reasons": [],
            "requires_human_review": False,
            "escalation_priority": "LOW"
        }
        
        for category_result in safety_result["analysis_result"].categories_analysis:
            category_name = category_result.category.value.lower()
            severity = category_result.severity
            score = category_result.score
            
            if category_name in self.config["severity_thresholds"]:
                thresholds = self.config["severity_thresholds"][category_name]
                
                if severity >= thresholds["block"]:
                    decision["action"] = "BLOCK"
                    decision["reasons"].append(f"High {category_name} content (severity {severity})")
                    
                    if category_name == "self_harm":
                        decision["escalation_priority"] = "URGENT"
                    elif score > 0.8:
                        decision["escalation_priority"] = "HIGH"
                        
                elif severity >= thresholds["review"]:
                    if decision["action"] != "BLOCK":
                        decision["action"] = "REVIEW"
                        decision["requires_human_review"] = True
                    decision["reasons"].append(f"Moderate {category_name} content (severity {severity})")
        
        # Adjust confidence based on score consistency
        avg_score = sum(r.score for r in safety_result["analysis_result"].categories_analysis) / 4
        if avg_score < 0.5:
            decision["confidence"] = "MEDIUM" if decision["action"] != "APPROVE" else "HIGH"
        elif avg_score < 0.3:
            decision["confidence"] = "LOW"
        
        return decision
    
    async def _handle_moderation_decision(self, content_item, decision):
        """Handle the moderation decision appropriately"""
        
        if decision["action"] == "BLOCK":
            await self._block_content(content_item, decision)
            
        elif decision["action"] == "REVIEW":
            await self._queue_for_human_review(content_item, decision)
            
        elif decision["action"] == "APPROVE":
            await self._approve_content(content_item, decision)
        
        # Log decision if configured
        if self.config["auto_actions"]["log_all_decisions"]:
            self._log_moderation_decision(content_item, decision)
    
    async def _block_content(self, content_item, decision):
        """Handle blocked content"""
        print(f"🚫 BLOCKED: Content {content_item.get('id', 'unknown')}")
        print(f"   Reasons: {'; '.join(decision['reasons'])}")
        
        # Could integrate with content management system here
        # await content_management_system.block_content(content_item["id"])
    
    async def _queue_for_human_review(self, content_item, decision):
        """Queue content for human review"""
        review_item = {
            "content": content_item,
            "decision": decision,
            "queued_at": time.time(),
            "priority": decision["escalation_priority"]
        }
        
        self.human_review_queue.append(review_item)
        print(f"📋 QUEUED FOR REVIEW: Content {content_item.get('id', 'unknown')}")
        print(f"   Priority: {decision['escalation_priority']}")
    
    async def _approve_content(self, content_item, decision):
        """Handle approved content"""
        print(f"✅ APPROVED: Content {content_item.get('id', 'unknown')}")
        
        # Could integrate with publishing system here
        # await publishing_system.publish_content(content_item)
    
    def _log_moderation_decision(self, content_item, decision):
        """Log moderation decision for audit purposes"""
        log_entry = {
            "timestamp": time.time(),
            "content_id": content_item.get("id"),
            "decision": decision,
            "content_preview": content_item["text"][:100]
        }
        self.moderation_queue.append(log_entry)

# Example usage
workflow = ContentModerationWorkflow(client)

test_content = [
    {"id": "msg_001", "text": "Thanks for sharing this helpful information!"},
    {"id": "msg_002", "text": "I hate everyone from that stupid country!"},
    {"id": "msg_003", "text": "Sometimes I think about ending my life..."}
]

for content in test_content:
    decision = asyncio.run(workflow.process_content(content))
    print(f"Final decision: {decision['action']}\n")
```

### Performance Optimization

```python
import asyncio
import aiohttp
from concurrent.futures import ThreadPoolExecutor

class OptimizedContentModerator:
    """High-performance content moderation with batching and caching"""
    
    def __init__(self, client, max_concurrent=10):
        self.client = client
        self.max_concurrent = max_concurrent
        self.analysis_cache = {}
        self.rate_limit_semaphore = asyncio.Semaphore(max_concurrent)
    
    async def moderate_content_batch(self, content_list, use_cache=True):
        """Process multiple content items concurrently"""
        
        tasks = []
        for content_item in content_list:
            task = self._moderate_single_item(content_item, use_cache)
            tasks.append(task)
        
        # Process all items concurrently with rate limiting
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Separate successful results from errors
        successful_results = []
        errors = []
        
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                errors.append({
                    "content_id": content_list[i].get("id", f"item_{i}"),
                    "error": str(result)
                })
            else:
                successful_results.append(result)
        
        return {
            "successful": successful_results,
            "errors": errors,
            "total_processed": len(content_list),
            "success_rate": len(successful_results) / len(content_list) * 100
        }
    
    async def _moderate_single_item(self, content_item, use_cache):
        """Moderate a single content item with caching and rate limiting"""
        
        async with self.rate_limit_semaphore:
            content_hash = hash(content_item["text"])
            
            # Check cache first
            if use_cache and content_hash in self.analysis_cache:
                cached_result = self.analysis_cache[content_hash]
                return {
                    "content_id": content_item.get("id"),
                    "decision": cached_result["decision"],
                    "cached": True,
                    "analysis_time_ms": 0
                }
            
            # Perform analysis
            start_time = time.time()
            
            request = AnalyzeTextOptions(
                text=content_item["text"],
                categories=[TextCategory.HATE, TextCategory.SELF_HARM, TextCategory.SEXUAL, TextCategory.VIOLENCE],
                output_type="FourSeverityLevels"
            )
            
            response = self.client.analyze_text(request)
            analysis_time = (time.time() - start_time) * 1000
            
            # Process results
            decision = self._make_moderation_decision(response)
            
            # Cache results
            if use_cache:
                self.analysis_cache[content_hash] = {
                    "decision": decision,
                    "cached_at": time.time()
                }
            
            return {
                "content_id": content_item.get("id"),
                "decision": decision,
                "cached": False,
                "analysis_time_ms": analysis_time
            }
    
    def _make_moderation_decision(self, analysis_response):
        """Make moderation decision based on analysis results"""
        
        max_severity = 0
        flagged_categories = []
        
        for category_result in analysis_response.categories_analysis:
            severity = category_result.severity
            max_severity = max(max_severity, severity)
            
            if severity >= 4:  # Medium to high severity
                flagged_categories.append({
                    "category": category_result.category.value,
                    "severity": severity,
                    "score": category_result.score
                })
        
        # Make decision based on max severity
        if max_severity >= 6:
            action = "BLOCK"
        elif max_severity >= 4:
            action = "REVIEW" 
        else:
            action = "APPROVE"
        
        return {
            "action": action,
            "max_severity": max_severity,
            "flagged_categories": flagged_categories
        }
    
    def get_cache_stats(self):
        """Get cache performance statistics"""
        return {
            "cache_size": len(self.analysis_cache),
            "cache_entries": list(self.analysis_cache.keys())[:10]  # First 10 for debugging
        }
    
    def clear_cache(self, older_than_seconds=3600):
        """Clear old cache entries"""
        current_time = time.time()
        expired_keys = [
            key for key, value in self.analysis_cache.items()
            if current_time - value["cached_at"] > older_than_seconds
        ]
        
        for key in expired_keys:
            del self.analysis_cache[key]
        
        return len(expired_keys)

# Example usage for high-volume content moderation
optimized_moderator = OptimizedContentModerator(client, max_concurrent=20)

# Simulate large batch of content
large_content_batch = [
    {"id": f"content_{i}", "text": f"This is test message number {i} with various content."} 
    for i in range(100)
]

# Add some problematic content
large_content_batch.extend([
    {"id": "problematic_1", "text": "I hate all people from that country!"},
    {"id": "problematic_2", "text": "Here's how to harm yourself..."},
    {"id": "problematic_3", "text": "Explicit sexual content here..."}
])

# Process the batch
batch_results = asyncio.run(optimized_moderator.moderate_content_batch(large_content_batch))

print(f"Batch Processing Results:")
print(f"Total processed: {batch_results['total_processed']}")
print(f"Success rate: {batch_results['success_rate']:.1f}%")
print(f"Successful analyses: {len(batch_results['successful'])}")
print(f"Errors: {len(batch_results['errors'])}")

# Show cache stats
cache_stats = optimized_moderator.get_cache_stats()
print(f"Cache size: {cache_stats['cache_size']}")
```

## 📚 Additional Resources

- **[What is Azure AI Content Safety?](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview)**
- **[Content Safety Concepts](https://learn.microsoft.com/en-us/azure/ai-services/content-safety/concepts/harm-categories)**
- **[REST API Reference](https://learn.microsoft.com/en-us/rest/api/contentsafety/)**
- **[SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-contentsafety/)**
- **[Responsible AI Guidelines](https://docs.microsoft.com/azure/ai-services/responsible-use-of-ai-overview)**

## 🎯 AI-102 Exam Tips

- Understand the four main content categories (Hate, Violence, Sexual, SelfHarm)
- Know the severity scale (0-7) and appropriate response thresholds
- Practice with confidence score interpretation and decision making
- Understand when to use human review vs automated decisions
- Know the API limits and best practices for batch processing
- Practice with different moderation workflows and escalation procedures