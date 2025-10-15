# 📝 Text Summarization

## 📋 Overview

Text Summarization is an advanced Azure AI Language service that automatically generates concise summaries of long documents. It supports both extractive summarization (selecting key sentences) and abstractive summarization (generating new summary text), making it ideal for document processing, content analysis, and information extraction.

### ✨ Key Features
- 📄 **Extractive Summarization**: Extract key sentences from original text
- 🧠 **Abstractive Summarization**: Generate new coherent summary text
- 📊 **Sentence Ranking**: Importance scoring for selected sentences
- 🎯 **Customizable Length**: Control summary length and sentence count
- 📱 **Multi-format Support**: Handle various document types and structures
- ⚡ **Batch Processing**: Summarize multiple documents efficiently

### 🎯 Common Use Cases
- **Document Analysis**: Quickly understand lengthy reports and papers
- **News Aggregation**: Create brief summaries of news articles
- **Meeting Notes**: Summarize meeting transcripts and recordings
- **Research Papers**: Extract key findings and conclusions
- **Email Processing**: Generate brief overviews of long email threads
- **Content Curation**: Create digestible content for social media

---

## 🐍 Python Implementation

### Basic Text Summarization

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os
import json

# Initialize client
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

def extractive_summarization_basic():
    """Basic extractive summarization example"""
    
    document = """
    The rise of artificial intelligence has transformed industries across the globe. 
    Machine learning algorithms now power recommendation systems, autonomous vehicles, 
    and medical diagnostic tools. Companies are investing billions in AI research and 
    development to gain competitive advantages. However, the rapid advancement also 
    raises concerns about job displacement and ethical implications. Experts argue that 
    proper regulation and retraining programs are essential to harness AI's benefits 
    while mitigating risks. The future workforce will need to adapt to collaborate 
    with intelligent systems rather than compete against them.
    """
    
    try:
        # Start text analysis operation
        poller = client.begin_analyze_actions(
            documents=[document],
            actions=[
                client.ExtractSummaryAction(
                    max_sentence_count=3,
                    order_by="Rank"
                )
            ]
        )
        
        # Wait for completion
        result = poller.result()
        
        for doc_result in result:
            for action_result in doc_result:
                if action_result.kind == "ExtractiveSummarization":
                    print("📄 Extractive Summary:")
                    print("=" * 30)
                    
                    for sentence in action_result.sentences:
                        print(f"Rank: {sentence.rank_score:.3f}")
                        print(f"Text: {sentence.text}")
                        print(f"Offset: {sentence.offset}")
                        print("---")
                        
    except Exception as e:
        print(f"Error: {e}")

# Run basic summarization
extractive_summarization_basic()
```

### Advanced Document Summarization

```python
def analyze_document_with_summarization(document_text, summary_type="both"):
    """Comprehensive document analysis with summarization"""
    
    try:
        actions = []
        
        # Add extractive summarization
        if summary_type in ["extractive", "both"]:
            actions.append(
                client.ExtractSummaryAction(
                    max_sentence_count=5,
                    order_by="Rank"
                )
            )
        
        # Add abstractive summarization  
        if summary_type in ["abstractive", "both"]:
            actions.append(
                client.AbstractSummaryAction(
                    sentence_count=3
                )
            )
        
        # Start analysis
        poller = client.begin_analyze_actions(
            documents=[document_text],
            actions=actions
        )
        
        result = poller.result()
        
        analysis_results = {
            "original_length": len(document_text),
            "word_count": len(document_text.split()),
            "extractive_summary": None,
            "abstractive_summary": None
        }
        
        for doc_result in result:
            for action_result in doc_result:
                if action_result.kind == "ExtractiveSummarization":
                    extractive_sentences = []
                    for sentence in action_result.sentences:
                        extractive_sentences.append({
                            "text": sentence.text,
                            "rank_score": sentence.rank_score,
                            "offset": sentence.offset,
                            "length": sentence.length
                        })
                    
                    analysis_results["extractive_summary"] = {
                        "sentences": extractive_sentences,
                        "summary_text": " ".join([s["text"] for s in extractive_sentences]),
                        "compression_ratio": len(" ".join([s["text"] for s in extractive_sentences])) / len(document_text)
                    }
                
                elif action_result.kind == "AbstractiveSummarization":
                    abstractive_summaries = []
                    for summary in action_result.summaries:
                        abstractive_summaries.append({
                            "text": summary.text,
                            "contexts": [context.offset for context in summary.contexts]
                        })
                    
                    analysis_results["abstractive_summary"] = {
                        "summaries": abstractive_summaries,
                        "summary_text": abstractive_summaries[0]["text"] if abstractive_summaries else "",
                        "compression_ratio": len(abstractive_summaries[0]["text"]) / len(document_text) if abstractive_summaries else 0
                    }
        
        return analysis_results
        
    except Exception as e:
        return {"error": str(e)}

# Example with a longer document
def demo_comprehensive_analysis():
    """Demonstrate comprehensive document analysis"""
    
    long_document = """
    Climate change represents one of the most pressing challenges of our time, with far-reaching 
    implications for global ecosystems, human societies, and economic systems. The scientific 
    consensus overwhelmingly supports the conclusion that human activities, particularly the 
    emission of greenhouse gases from fossil fuel combustion, are the primary drivers of 
    contemporary climate change.
    
    Temperature records from around the world show a clear warming trend over the past century, 
    with the most rapid warming occurring in recent decades. The Intergovernmental Panel on 
    Climate Change (IPCC) reports that global average temperatures have risen by approximately 
    1.1 degrees Celsius since pre-industrial times. This seemingly small increase has triggered 
    significant changes in weather patterns, sea levels, and ecosystem dynamics.
    
    The impacts of climate change are already visible across multiple domains. Arctic ice sheets 
    and glaciers are retreating at unprecedented rates, contributing to rising sea levels that 
    threaten coastal communities worldwide. Extreme weather events, including hurricanes, 
    droughts, floods, and heatwaves, have become more frequent and intense. These changes pose 
    risks to agricultural productivity, water resources, and human health.
    
    Addressing climate change requires coordinated global action involving multiple strategies. 
    Mitigation efforts focus on reducing greenhouse gas emissions through renewable energy 
    adoption, energy efficiency improvements, and carbon pricing mechanisms. Adaptation measures 
    help communities prepare for and respond to climate impacts through infrastructure upgrades, 
    disaster preparedness, and ecosystem restoration. International cooperation through agreements 
    like the Paris Climate Accord provides frameworks for collective action.
    
    The transition to a low-carbon economy presents both challenges and opportunities. While it 
    requires significant investments and economic restructuring, it also creates new industries, 
    jobs, and innovation opportunities. Successful climate action depends on engagement from 
    governments, businesses, and individuals working together toward sustainable solutions.
    """
    
    print("🔍 Comprehensive Document Analysis")
    print("=" * 50)
    
    results = analyze_document_with_summarization(long_document, "both")
    
    if "error" not in results:
        print(f"📊 Document Statistics:")
        print(f"  Original Length: {results['original_length']} characters")
        print(f"  Word Count: {results['word_count']} words")
        print()
        
        if results["extractive_summary"]:
            ext_summary = results["extractive_summary"]
            print(f"📄 Extractive Summary ({len(ext_summary['sentences'])} sentences):")
            print(f"  Compression Ratio: {ext_summary['compression_ratio']:.1%}")
            print(f"  Summary: {ext_summary['summary_text']}")
            print()
        
        if results["abstractive_summary"]:
            abs_summary = results["abstractive_summary"]
            print(f"🧠 Abstractive Summary:")
            print(f"  Compression Ratio: {abs_summary['compression_ratio']:.1%}")
            print(f"  Summary: {abs_summary['summary_text']}")
        
    else:
        print(f"Error: {results['error']}")

# Run comprehensive demo
demo_comprehensive_analysis()
```

### Batch Document Processing

```python
class DocumentSummarizer:
    """Batch document summarization with advanced features"""
    
    def __init__(self, client):
        self.client = client
        
    def summarize_documents_batch(self, documents, summary_config=None):
        """Batch summarize multiple documents"""
        
        if summary_config is None:
            summary_config = {
                "extractive": {
                    "max_sentence_count": 3,
                    "order_by": "Rank"
                },
                "abstractive": {
                    "sentence_count": 2
                }
            }
        
        try:
            actions = [
                self.client.ExtractSummaryAction(**summary_config["extractive"]),
                self.client.AbstractSummaryAction(**summary_config["abstractive"])
            ]
            
            # Start batch analysis
            poller = self.client.begin_analyze_actions(
                documents=documents,
                actions=actions
            )
            
            results = poller.result()
            
            processed_results = []
            
            for doc_idx, doc_result in enumerate(results):
                doc_summary = {
                    "document_index": doc_idx,
                    "original_text": documents[doc_idx][:200] + "..." if len(documents[doc_idx]) > 200 else documents[doc_idx],
                    "extractive_summary": None,
                    "abstractive_summary": None,
                    "errors": []
                }
                
                for action_result in doc_result:
                    if hasattr(action_result, 'is_error') and action_result.is_error:
                        doc_summary["errors"].append(action_result.error.message)
                        continue
                        
                    if action_result.kind == "ExtractiveSummarization":
                        sentences = [
                            {
                                "text": sentence.text,
                                "rank": sentence.rank_score
                            }
                            for sentence in action_result.sentences
                        ]
                        doc_summary["extractive_summary"] = {
                            "sentences": sentences,
                            "summary_text": " ".join([s["text"] for s in sentences])
                        }
                    
                    elif action_result.kind == "AbstractiveSummarization":
                        summaries = [summary.text for summary in action_result.summaries]
                        doc_summary["abstractive_summary"] = {
                            "summaries": summaries,
                            "primary_summary": summaries[0] if summaries else ""
                        }
                
                processed_results.append(doc_summary)
            
            return processed_results
            
        except Exception as e:
            return {"error": str(e)}

# Example usage
def demo_batch_summarization():
    """Demonstrate batch document summarization"""
    
    news_articles = [
        """
        Technology companies are racing to develop more efficient artificial intelligence systems. 
        Recent breakthroughs in neural network architectures have led to significant improvements 
        in processing speed and accuracy. Major tech firms are investing heavily in specialized 
        AI hardware to support these advanced models. The competition is driving innovation while 
        raising questions about the concentration of AI capabilities in few companies.
        """,
        
        """
        Renewable energy adoption continues to accelerate globally as costs decrease and efficiency 
        improves. Solar and wind power installations reached record levels last year, supported by 
        government incentives and corporate sustainability commitments. Energy storage technologies 
        are advancing rapidly, addressing intermittency challenges. The transition is creating new 
        job opportunities while reducing carbon emissions.
        """,
        
        """
        Remote work trends have fundamentally changed how businesses operate and employees collaborate. 
        Companies are adopting hybrid models that combine office and remote work arrangements. 
        Digital collaboration tools have become essential infrastructure for distributed teams. 
        The shift is influencing commercial real estate markets and urban planning decisions as 
        organizations reassess their space requirements.
        """
    ]
    
    summarizer = DocumentSummarizer(client)
    
    print("📰 Batch News Article Summarization")
    print("=" * 45)
    
    results = summarizer.summarize_documents_batch(news_articles)
    
    if isinstance(results, list):
        for i, result in enumerate(results):
            print(f"\n📄 Article {i + 1}:")
            print(f"Original: {result['original_text']}")
            
            if result["extractive_summary"]:
                print(f"Extractive: {result['extractive_summary']['summary_text']}")
            
            if result["abstractive_summary"]:
                print(f"Abstractive: {result['abstractive_summary']['primary_summary']}")
            
            if result["errors"]:
                print(f"Errors: {', '.join(result['errors'])}")
            
            print("-" * 40)
    else:
        print(f"Error: {results['error']}")

# Run batch demo
demo_batch_summarization()
```

### Summary Quality Assessment

```python
import re
from textstat import flesch_reading_ease, flesch_kincaid_grade

class SummaryQualityAnalyzer:
    """Analyze and score summary quality"""
    
    def __init__(self, client):
        self.client = client
        
    def analyze_summary_quality(self, original_text, summary_text):
        """Comprehensive summary quality analysis"""
        
        # Basic metrics
        original_words = len(original_text.split())
        summary_words = len(summary_text.split())
        compression_ratio = summary_words / original_words
        
        # Readability scores
        try:
            original_readability = flesch_reading_ease(original_text)
            summary_readability = flesch_reading_ease(summary_text)
            readability_improvement = summary_readability - original_readability
        except:
            original_readability = summary_readability = readability_improvement = None
        
        # Sentence structure analysis
        original_sentences = len(re.split(r'[.!?]+', original_text.strip()))
        summary_sentences = len(re.split(r'[.!?]+', summary_text.strip()))
        
        # Key terms preservation (simple approach)
        original_key_terms = self._extract_key_terms(original_text)
        summary_key_terms = self._extract_key_terms(summary_text)
        term_preservation = len(summary_key_terms.intersection(original_key_terms)) / len(original_key_terms) if original_key_terms else 0
        
        quality_metrics = {
            "compression_ratio": compression_ratio,
            "word_reduction": original_words - summary_words,
            "sentence_reduction": original_sentences - summary_sentences,
            "readability": {
                "original_score": original_readability,
                "summary_score": summary_readability,
                "improvement": readability_improvement
            },
            "key_term_preservation": term_preservation,
            "avg_sentence_length": {
                "original": original_words / original_sentences if original_sentences else 0,
                "summary": summary_words / summary_sentences if summary_sentences else 0
            }
        }
        
        # Overall quality score (0-100)
        quality_score = self._calculate_quality_score(quality_metrics)
        quality_metrics["overall_quality_score"] = quality_score
        
        return quality_metrics
    
    def _extract_key_terms(self, text):
        """Extract key terms using simple frequency analysis"""
        # Remove punctuation and convert to lowercase
        clean_text = re.sub(r'[^\w\s]', ' ', text.lower())
        words = clean_text.split()
        
        # Filter common words (simple stopwords)
        stopwords = {'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with', 'by', 'is', 'are', 'was', 'were', 'be', 'been', 'have', 'has', 'had', 'do', 'does', 'did', 'will', 'would', 'could', 'should', 'may', 'might', 'must', 'this', 'that', 'these', 'those'}
        
        # Get words longer than 3 characters that aren't stopwords
        key_terms = {word for word in words if len(word) > 3 and word not in stopwords}
        
        return key_terms
    
    def _calculate_quality_score(self, metrics):
        """Calculate overall quality score"""
        score = 0
        
        # Compression efficiency (30% weight)
        if 0.2 <= metrics["compression_ratio"] <= 0.5:
            score += 30
        elif 0.1 <= metrics["compression_ratio"] < 0.2:
            score += 25
        elif 0.5 < metrics["compression_ratio"] <= 0.7:
            score += 20
        else:
            score += 10
        
        # Key term preservation (25% weight)
        score += metrics["key_term_preservation"] * 25
        
        # Readability improvement (20% weight)
        if metrics["readability"]["improvement"] and metrics["readability"]["improvement"] > 0:
            score += min(20, metrics["readability"]["improvement"] / 5 * 20)
        
        # Sentence structure (25% weight)
        if metrics["avg_sentence_length"]["summary"] > 0:
            length_ratio = metrics["avg_sentence_length"]["summary"] / metrics["avg_sentence_length"]["original"]
            if 0.8 <= length_ratio <= 1.2:
                score += 25
            elif 0.6 <= length_ratio < 0.8 or 1.2 < length_ratio <= 1.4:
                score += 20
            else:
                score += 10
        
        return min(100, max(0, score))

# Example usage
def demo_quality_analysis():
    """Demonstrate summary quality analysis"""
    
    analyzer = SummaryQualityAnalyzer(client)
    
    original = """
    Artificial intelligence has revolutionized multiple industries through machine learning algorithms, 
    natural language processing, and computer vision technologies. Companies worldwide are implementing 
    AI solutions to automate processes, enhance decision-making, and improve customer experiences. 
    The healthcare sector benefits from AI-powered diagnostic tools, while financial institutions 
    use algorithms for fraud detection and risk assessment. Manufacturing companies deploy AI for 
    predictive maintenance and quality control. Despite these advances, challenges remain regarding 
    data privacy, algorithmic bias, and the need for skilled professionals to develop and maintain 
    AI systems effectively.
    """
    
    summary = """
    AI has transformed industries through machine learning and automation. Healthcare uses AI for 
    diagnostics, finance for fraud detection, and manufacturing for quality control. Challenges 
    include data privacy and algorithmic bias.
    """
    
    quality_metrics = analyzer.analyze_summary_quality(original, summary)
    
    print("📊 Summary Quality Analysis")
    print("=" * 35)
    print(f"Compression Ratio: {quality_metrics['compression_ratio']:.1%}")
    print(f"Word Reduction: {quality_metrics['word_reduction']} words")
    print(f"Key Term Preservation: {quality_metrics['key_term_preservation']:.1%}")
    print(f"Overall Quality Score: {quality_metrics['overall_quality_score']:.1f}/100")
    
    if quality_metrics["readability"]["improvement"]:
        print(f"Readability Improvement: {quality_metrics['readability']['improvement']:+.1f}")

# Run quality analysis demo
demo_quality_analysis()
```

---

## 🔷 C# Implementation

### Basic Text Summarization

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class TextSummarizationService
{
    private readonly TextAnalyticsClient _client;
    
    public TextSummarizationService(string endpoint, string key)
    {
        var credential = new AzureKeyCredential(key);
        _client = new TextAnalyticsClient(new Uri(endpoint), credential);
    }
    
    public async Task ExtractiveSummarizationAsync()
    {
        string document = @"
            The rise of artificial intelligence has transformed industries across the globe. 
            Machine learning algorithms now power recommendation systems, autonomous vehicles, 
            and medical diagnostic tools. Companies are investing billions in AI research and 
            development to gain competitive advantages. However, the rapid advancement also 
            raises concerns about job displacement and ethical implications. Experts argue that 
            proper regulation and retraining programs are essential to harness AI's benefits 
            while mitigating risks.";

        var actions = new List<TextAnalyticsAction>
        {
            new ExtractSummaryAction()
            {
                MaxSentenceCount = 3,
                OrderBy = SummarySentencesOrder.Rank
            }
        };

        try
        {
            var operation = await _client.StartAnalyzeActionsAsync(new[] { document }, actions);
            await operation.WaitForCompletionAsync();

            await foreach (var documentResult in operation.Value)
            {
                foreach (var actionResult in documentResult)
                {
                    if (actionResult is ExtractSummaryActionResult summarizeResult)
                    {
                        if (!summarizeResult.HasError)
                        {
                            Console.WriteLine("📄 Extractive Summary:");
                            Console.WriteLine(new string('=', 30));
                            
                            foreach (var sentence in summarizeResult.DocumentsResults[0].Sentences)
                            {
                                Console.WriteLine($"Rank: {sentence.RankScore:F3}");
                                Console.WriteLine($"Text: {sentence.Text}");
                                Console.WriteLine($"Offset: {sentence.Offset}");
                                Console.WriteLine("---");
                            }
                        }
                        else
                        {
                            Console.WriteLine($"Error: {summarizeResult.Error.Message}");
                        }
                    }
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

### Advanced Document Analysis

```csharp
public class DocumentAnalyzer
{
    private readonly TextAnalyticsClient _client;
    
    public DocumentAnalyzer(TextAnalyticsClient client)
    {
        _client = client;
    }
    
    public async Task<DocumentAnalysisResult> AnalyzeDocumentAsync(
        string document, 
        SummaryType summaryType = SummaryType.Both)
    {
        var actions = new List<TextAnalyticsAction>();
        
        if (summaryType == SummaryType.Extractive || summaryType == SummaryType.Both)
        {
            actions.Add(new ExtractSummaryAction
            {
                MaxSentenceCount = 5,
                OrderBy = SummarySentencesOrder.Rank
            });
        }
        
        if (summaryType == SummaryType.Abstractive || summaryType == SummaryType.Both)
        {
            actions.Add(new AbstractSummaryAction
            {
                SentenceCount = 3
            });
        }
        
        try
        {
            var operation = await _client.StartAnalyzeActionsAsync(new[] { document }, actions);
            await operation.WaitForCompletionAsync();
            
            var result = new DocumentAnalysisResult
            {
                OriginalLength = document.Length,
                WordCount = document.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length
            };
            
            await foreach (var documentResult in operation.Value)
            {
                foreach (var actionResult in documentResult)
                {
                    if (actionResult is ExtractSummaryActionResult extractResult && !extractResult.HasError)
                    {
                        var sentences = new List<ExtractedSentence>();
                        foreach (var sentence in extractResult.DocumentsResults[0].Sentences)
                        {
                            sentences.Add(new ExtractedSentence
                            {
                                Text = sentence.Text,
                                RankScore = sentence.RankScore,
                                Offset = sentence.Offset,
                                Length = sentence.Length
                            });
                        }
                        
                        result.ExtractiveSummary = new ExtractiveSummaryResult
                        {
                            Sentences = sentences,
                            SummaryText = string.Join(" ", sentences.Select(s => s.Text)),
                            CompressionRatio = string.Join(" ", sentences.Select(s => s.Text)).Length / (double)document.Length
                        };
                    }
                    
                    if (actionResult is AbstractSummaryActionResult abstractResult && !abstractResult.HasError)
                    {
                        var summaries = abstractResult.DocumentsResults[0].Summaries
                            .Select(s => s.Text).ToList();
                        
                        result.AbstractiveSummary = new AbstractiveSummaryResult
                        {
                            Summaries = summaries,
                            PrimarySummary = summaries.FirstOrDefault() ?? "",
                            CompressionRatio = (summaries.FirstOrDefault()?.Length ?? 0) / (double)document.Length
                        };
                    }
                }
            }
            
            return result;
        }
        catch (Exception ex)
        {
            return new DocumentAnalysisResult { Error = ex.Message };
        }
    }
}

// Supporting classes
public enum SummaryType
{
    Extractive,
    Abstractive,
    Both
}

public class DocumentAnalysisResult
{
    public int OriginalLength { get; set; }
    public int WordCount { get; set; }
    public ExtractiveSummaryResult ExtractiveSummary { get; set; }
    public AbstractiveSummaryResult AbstractiveSummary { get; set; }
    public string Error { get; set; }
}

public class ExtractiveSummaryResult
{
    public List<ExtractedSentence> Sentences { get; set; }
    public string SummaryText { get; set; }
    public double CompressionRatio { get; set; }
}

public class AbstractiveSummaryResult
{
    public List<string> Summaries { get; set; }
    public string PrimarySummary { get; set; }
    public double CompressionRatio { get; set; }
}

public class ExtractedSentence
{
    public string Text { get; set; }
    public double RankScore { get; set; }
    public int Offset { get; set; }
    public int Length { get; set; }
}
```

---

## 🌐 REST API Implementation

### Basic Summarization Request

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/analyze-text/jobs?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "The rise of artificial intelligence has transformed industries across the globe. Machine learning algorithms now power recommendation systems, autonomous vehicles, and medical diagnostic tools. Companies are investing billions in AI research and development to gain competitive advantages."
        }
      ]
    },
    "tasks": [
      {
        "kind": "ExtractiveSummarization",
        "taskName": "ExtractiveSummarization_Task",
        "parameters": {
          "sentenceCount": 3,
          "sortBy": "Rank"
        }
      }
    ]
  }'
```

### Python REST Implementation

```python
import requests
import json
import time
import os

class TextSummarizationREST:
    """REST API implementation for text summarization"""
    
    def __init__(self):
        self.endpoint = os.environ["LANGUAGE_ENDPOINT"]
        self.key = os.environ["LANGUAGE_KEY"]
        self.api_version = "2022-05-01"
        
    def submit_summarization_job(self, documents, summary_config=None):
        """Submit summarization job and return job ID"""
        
        if summary_config is None:
            summary_config = {
                "extractive": {
                    "sentenceCount": 3,
                    "sortBy": "Rank"
                },
                "abstractive": {
                    "sentenceCount": 2
                }
            }
        
    url = f"{self.endpoint}/language/analyze-text/jobs?api-version={self.api_version}"
        headers = {
            "Ocp-Apim-Subscription-Key": self.key,
            "Content-Type": "application/json"
        }
        
        # Format documents
        if isinstance(documents, list) and isinstance(documents[0], str):
            formatted_docs = [
                {"id": str(i+1), "language": "en", "text": text}
                for i, text in enumerate(documents)
            ]
        else:
            formatted_docs = documents
        
        tasks = []
        
        if "extractive" in summary_config:
            tasks.append({
                "kind": "ExtractiveSummarization",
                "taskName": "ExtractiveSummarization_Task",
                "parameters": summary_config["extractive"]
            })
        
        if "abstractive" in summary_config:
            tasks.append({
                "kind": "AbstractiveSummarization", 
                "taskName": "AbstractiveSummarization_Task",
                "parameters": summary_config["abstractive"]
            })
        
        payload = {
            "analysisInput": {
                "documents": formatted_docs
            },
            "tasks": tasks
        }
        
        try:
            response = requests.post(url, headers=headers, json=payload)
            response.raise_for_status()
            
            # Extract job ID from response headers
            operation_location = response.headers.get("operation-location")
            if operation_location:
                job_id = operation_location.split("/")[-1]
                return {"job_id": job_id, "status": "submitted"}
            else:
                return {"error": "No operation-location header found"}
                
        except requests.exceptions.RequestException as e:
            return {"error": str(e)}
    
    def get_job_status(self, job_id):
        """Get the status of a summarization job"""
        
    url = f"{self.endpoint}/language/analyze-text/jobs/{job_id}?api-version={self.api_version}"
        headers = {
            "Ocp-Apim-Subscription-Key": self.key
        }
        try:
            response = requests.get(url, headers=headers)
            response.raise_for_status()
            return response.json()
            
        except requests.exceptions.RequestException as e:
            return {"error": str(e)}
    
    def wait_for_completion(self, job_id, max_wait_time=300, poll_interval=5):
        """Wait for job completion and return results"""
        
        start_time = time.time()
        
        while time.time() - start_time < max_wait_time:
            status_response = self.get_job_status(job_id)
            
            if "error" in status_response:
                return status_response
            
            job_status = status_response.get("status", "unknown")
            
            if job_status == "succeeded":
                return self._process_results(status_response)
            elif job_status == "failed":
                return {"error": "Job failed", "details": status_response}
            elif job_status in ["running", "notStarted"]:
                time.sleep(poll_interval)
            else:
                return {"error": f"Unknown job status: {job_status}"}
        
        return {"error": "Job timed out"}
    
    def _process_results(self, job_response):
        """Process completed job results"""
        
        processed_results = {
            "job_id": job_response.get("jobId"),
            "status": job_response.get("status"),
            "documents": []
        }
        
        tasks = job_response.get("tasks", {})
        
        # Process each document
        for task_name, task_results in tasks.items():
            if "results" in task_results:
                for doc in task_results["results"]["documents"]:
                    doc_id = doc["id"]
                    
                    # Find or create document entry
                    doc_result = next(
                        (d for d in processed_results["documents"] if d["id"] == doc_id),
                        None
                    )
                    if not doc_result:
                        doc_result = {"id": doc_id}
                        processed_results["documents"].append(doc_result)
                    
                    # Add task-specific results
                    if "ExtractiveSummarization" in task_name:
                        doc_result["extractive_summary"] = {
                            "sentences": [
                                {
                                    "text": sentence["text"],
                                    "rankScore": sentence["rankScore"],
                                    "offset": sentence["offset"]
                                }
                                for sentence in doc.get("sentences", [])
                            ]
                        }
                    
                    elif "AbstractiveSummarization" in task_name:
                        doc_result["abstractive_summary"] = {
                            "summaries": [
                                summary["text"] for summary in doc.get("summaries", [])
                            ]
                        }
        
        return processed_results

# Example usage
def demo_rest_summarization():
    """Demonstrate REST API summarization"""
    
    summarizer = TextSummarizationREST()
    
    documents = [
        """
        Climate change represents one of the most pressing challenges of our time. 
        Rising global temperatures are causing widespread environmental impacts including 
        melting ice caps, rising sea levels, and extreme weather events. Scientists 
        agree that immediate action is needed to reduce greenhouse gas emissions and 
        transition to renewable energy sources.
        """
    ]
    
    print("🚀 Submitting Summarization Job")
    job_response = summarizer.submit_summarization_job(documents)
    
    if "job_id" in job_response:
        print(f"Job ID: {job_response['job_id']}")
        print("⏳ Waiting for completion...")
        
        results = summarizer.wait_for_completion(job_response["job_id"])
        
        if "error" not in results:
            print("\n📄 Summarization Results:")
            print("=" * 30)
            
            for doc in results["documents"]:
                print(f"Document {doc['id']}:")
                
                if "extractive_summary" in doc:
                    sentences = doc["extractive_summary"]["sentences"]
                    summary_text = " ".join([s["text"] for s in sentences])
                    print(f"  Extractive: {summary_text}")
                
                if "abstractive_summary" in doc:
                    summaries = doc["abstractive_summary"]["summaries"]
                    print(f"  Abstractive: {summaries[0] if summaries else 'None'}")
                
                print()
        else:
            print(f"Error: {results['error']}")
    else:
        print(f"Error submitting job: {job_response.get('error', 'Unknown error')}")

# Run REST demo
demo_rest_summarization()
```

---

## 📊 Response Format

### Extractive Summarization Response

```json
{
  "jobId": "12345678-1234-1234-1234-123456789012",
  "lastUpdateDateTime": "2023-10-01T12:00:00Z",
  "createdDateTime": "2023-10-01T11:59:00Z",
  "expirationDateTime": "2023-10-02T11:59:00Z",
  "status": "succeeded",
  "errors": [],
  "tasks": {
    "ExtractiveSummarization_Task": {
      "lastUpdateDateTime": "2023-10-01T12:00:00Z",
      "taskName": "ExtractiveSummarization_Task",
      "state": "succeeded",
      "results": {
        "documents": [
          {
            "id": "1",
            "sentences": [
              {
                "text": "Climate change represents one of the most pressing challenges of our time.",
                "rankScore": 0.95,
                "offset": 0,
                "length": 75
              },
              {
                "text": "Scientists agree that immediate action is needed to reduce greenhouse gas emissions.",
                "rankScore": 0.87,
                "offset": 245,
                "length": 83
              }
            ],
            "warnings": []
          }
        ],
        "errors": [],
        "modelVersion": "2023-04-01"
      }
    }
  }
}
```

### Abstractive Summarization Response

```json
{
  "tasks": {
    "AbstractiveSummarization_Task": {
      "results": {
        "documents": [
          {
            "id": "1",
            "summaries": [
              {
                "text": "Climate change poses significant environmental challenges requiring immediate action to reduce emissions and adopt renewable energy.",
                "contexts": [
                  {
                    "offset": 0,
                    "length": 150
                  }
                ]
              }
            ],
            "warnings": []
          }
        ]
      }
    }
  }
}
```

---

## 🎯 Real-World Use Cases

### 1. Research Paper Summarization

```python
class ResearchPaperSummarizer:
    """Specialized summarizer for research papers"""
    
    def __init__(self, client):
        self.client = client
        
    def summarize_research_paper(self, paper_sections):
        """Summarize different sections of a research paper"""
        
        section_summaries = {}
        
        for section_name, content in paper_sections.items():
            if len(content.strip()) < 100:
                section_summaries[section_name] = {
                    "summary": content,
                    "note": "Section too short for summarization"
                }
                continue
            
            try:
                # Use different summarization approaches for different sections
                if section_name.lower() in ["abstract", "conclusion"]:
                    # Abstractive for high-level sections
                    actions = [
                        self.client.AbstractSummaryAction(sentence_count=2)
                    ]
                else:
                    # Extractive for detailed sections
                    actions = [
                        self.client.ExtractSummaryAction(
                            max_sentence_count=3,
                            order_by="Rank"
                        )
                    ]
                
                poller = self.client.begin_analyze_actions([content], actions)
                result = poller.result()
                
                for doc_result in result:
                    for action_result in doc_result:
                        if action_result.kind == "ExtractiveSummarization":
                            summary_text = " ".join([s.text for s in action_result.sentences])
                            section_summaries[section_name] = {
                                "type": "extractive",
                                "summary": summary_text,
                                "sentence_count": len(action_result.sentences)
                            }
                        
                        elif action_result.kind == "AbstractiveSummarization":
                            summary_text = action_result.summaries[0].text if action_result.summaries else ""
                            section_summaries[section_name] = {
                                "type": "abstractive", 
                                "summary": summary_text
                            }
            
            except Exception as e:
                section_summaries[section_name] = {
                    "error": str(e)
                }
        
        return section_summaries

# Example usage
def demo_research_paper_summarization():
    """Demonstrate research paper summarization"""
    
    paper_sections = {
        "Introduction": """
        Artificial intelligence has emerged as a transformative technology across numerous domains. 
        This paper investigates the application of machine learning techniques in healthcare 
        diagnostics, specifically focusing on medical image analysis. Recent advances in deep 
        learning have shown promising results in automated disease detection and classification. 
        However, challenges remain in terms of model interpretability, data privacy, and 
        regulatory compliance.
        """,
        
        "Methodology": """
        We developed a convolutional neural network architecture specifically designed for 
        medical image classification. The model was trained on a dataset of 50,000 medical 
        images from multiple hospitals. Data preprocessing included image normalization, 
        augmentation techniques, and quality filtering. The network architecture consists of 
        12 convolutional layers with batch normalization and dropout regularization. Training 
        was performed using transfer learning from ImageNet pretrained weights.
        """,
        
        "Results": """
        Our model achieved 94.2% accuracy on the test dataset, outperforming existing baseline 
        methods by 7.3%. The sensitivity and specificity were 92.8% and 95.6% respectively. 
        Processing time per image was reduced to 0.3 seconds compared to 2.1 seconds for 
        traditional methods. The model showed consistent performance across different hospitals 
        and imaging equipment types.
        """,
        
        "Conclusion": """
        This study demonstrates the effectiveness of deep learning approaches for automated 
        medical image analysis. The proposed method offers significant improvements in both 
        accuracy and processing speed while maintaining clinical reliability. Future work 
        will focus on expanding the model to additional medical conditions and improving 
        interpretability for clinical adoption.
        """
    }
    
    summarizer = ResearchPaperSummarizer(client)
    summaries = summarizer.summarize_research_paper(paper_sections)
    
    print("📚 Research Paper Section Summaries")
    print("=" * 45)
    
    for section, summary_info in summaries.items():
        print(f"\n📖 {section}:")
        if "error" in summary_info:
            print(f"  Error: {summary_info['error']}")
        else:
            print(f"  Type: {summary_info.get('type', 'N/A')}")
            print(f"  Summary: {summary_info['summary']}")

# Run research paper demo
demo_research_paper_summarization()
```

### 2. News Article Aggregation

```python
class NewsAggregator:
    """Aggregate and summarize news articles by topic"""
    
    def __init__(self, client):
        self.client = client
        
    def aggregate_news_by_topic(self, articles_by_topic):
        """Create topic-based news summaries"""
        
        topic_summaries = {}
        
        for topic, articles in articles_by_topic.items():
            # Combine all articles for the topic
            combined_text = "\n\n".join([
                f"Article {i+1}: {article['title']}\n{article['content']}"
                for i, article in enumerate(articles)
            ])
            
            # Generate both extractive and abstractive summaries
            try:
                actions = [
                    self.client.ExtractSummaryAction(
                        max_sentence_count=5,
                        order_by="Rank"
                    ),
                    self.client.AbstractSummaryAction(
                        sentence_count=3
                    )
                ]
                
                poller = self.client.begin_analyze_actions([combined_text], actions)
                result = poller.result()
                
                topic_summary = {
                    "topic": topic,
                    "article_count": len(articles),
                    "total_length": len(combined_text),
                    "extractive_summary": None,
                    "abstractive_summary": None,
                    "key_articles": [article["title"] for article in articles[:3]]
                }
                
                for doc_result in result:
                    for action_result in doc_result:
                        if action_result.kind == "ExtractiveSummarization":
                            key_sentences = [s.text for s in action_result.sentences]
                            topic_summary["extractive_summary"] = " ".join(key_sentences)
                        
                        elif action_result.kind == "AbstractiveSummarization":
                            if action_result.summaries:
                                topic_summary["abstractive_summary"] = action_result.summaries[0].text
                
                topic_summaries[topic] = topic_summary
                
            except Exception as e:
                topic_summaries[topic] = {
                    "topic": topic,
                    "error": str(e),
                    "article_count": len(articles)
                }
        
        return topic_summaries

# Example usage
def demo_news_aggregation():
    """Demonstrate news aggregation and summarization"""
    
    news_by_topic = {
        "Technology": [
            {
                "title": "AI Breakthrough in Medical Diagnostics",
                "content": "Researchers have developed a new AI system that can diagnose diseases with 99% accuracy. The system uses advanced machine learning algorithms to analyze medical images and patient data."
            },
            {
                "title": "Quantum Computing Milestone Achieved",
                "content": "Scientists have successfully demonstrated quantum supremacy in solving complex optimization problems. This breakthrough could revolutionize cryptography and drug discovery."
            }
        ],
        
        "Environment": [
            {
                "title": "Renewable Energy Growth Accelerates",
                "content": "Solar and wind power installations reached record levels this year, with costs continuing to decline. Government incentives and corporate commitments are driving adoption."
            },
            {
                "title": "Ocean Cleanup Project Shows Promise",
                "content": "A new ocean cleanup system has successfully removed thousands of tons of plastic waste from the Pacific Ocean. The technology could be scaled to address marine pollution globally."
            }
        ]
    }
    
    aggregator = NewsAggregator(client)
    summaries = aggregator.aggregate_news_by_topic(news_by_topic)
    
    print("📰 News Topic Summaries")
    print("=" * 35)
    
    for topic, summary in summaries.items():
        print(f"\n🏷️ {topic} ({summary.get('article_count', 0)} articles)")
        
        if "error" in summary:
            print(f"  Error: {summary['error']}")
        else:
            print(f"  Key Articles: {', '.join(summary['key_articles'])}")
            print(f"  Extractive: {summary['extractive_summary']}")
            print(f"  Abstractive: {summary['abstractive_summary']}")

# Run news aggregation demo
demo_news_aggregation()
```

---

## ✅ Best Practices

### 1. Optimal Document Preparation

```python
class DocumentPreprocessor:
    """Prepare documents for optimal summarization"""
    
    def __init__(self):
        self.min_length = 40  # Minimum words for summarization
        self.max_length = 125000  # Maximum characters for Text Analytics
        
    def prepare_document(self, text, preserve_structure=True):
        """Prepare document for summarization"""
        
        # Basic cleaning
        cleaned_text = self._clean_text(text)
        
        # Check length constraints
        word_count = len(cleaned_text.split())
        char_count = len(cleaned_text)
        
        preparation_info = {
            "original_length": len(text),
            "cleaned_length": char_count,
            "word_count": word_count,
            "is_suitable": True,
            "warnings": [],
            "processed_text": cleaned_text
        }
        
        # Validate length
        if word_count < self.min_length:
            preparation_info["is_suitable"] = False
            preparation_info["warnings"].append(f"Document too short ({word_count} words, minimum {self.min_length})")
        
        if char_count > self.max_length:
            preparation_info["warnings"].append(f"Document too long ({char_count} chars, maximum {self.max_length})")
            # Truncate to fit
            preparation_info["processed_text"] = cleaned_text[:self.max_length]
            preparation_info["truncated"] = True
        
        # Structure preservation
        if preserve_structure:
            preparation_info["processed_text"] = self._preserve_structure(preparation_info["processed_text"])
        
        return preparation_info
    
    def _clean_text(self, text):
        """Clean text while preserving important content"""
        import re
        
        # Remove excessive whitespace
        text = re.sub(r'\s+', ' ', text)
        
        # Fix common encoding issues
        text = text.replace('\u2019', "'").replace('\u2018', "'")
        text = text.replace('\u201c', '"').replace('\u201d', '"')
        text = text.replace('\u2013', '-').replace('\u2014', '--')
        
        # Remove excessive punctuation
        text = re.sub(r'[.]{3,}', '...', text)
        text = re.sub(r'[!]{2,}', '!', text)
        text = re.sub(r'[?]{2,}', '?', text)
        
        return text.strip()
    
    def _preserve_structure(self, text):
        """Preserve important structural elements"""
        import re
        
        # Ensure sentences end properly
        text = re.sub(r'([.!?])\s*([A-Z])', r'\1 \2', text)
        
        # Preserve paragraph breaks for longer documents
        if len(text) > 5000:
            text = re.sub(r'\n\s*\n', '\n\n', text)
        
        return text

# Example usage
def demo_document_preparation():
    """Demonstrate document preparation"""
    
    preprocessor = DocumentPreprocessor()
    
    raw_document = """
    This    is   a document    with    excessive   whitespace.
    
    
    It also has... too many dots!!! And questions???
    
    Some "smart quotes" and –dashes– need fixing.
    
    The document contains important information about artificial intelligence applications.
    Machine learning models are being deployed across various industries for automation.
    """
    
    prep_result = preprocessor.prepare_document(raw_document)
    
    print("🔧 Document Preparation Results")
    print("=" * 40)
    print(f"Original Length: {prep_result['original_length']} chars")
    print(f"Cleaned Length: {prep_result['cleaned_length']} chars")
    print(f"Word Count: {prep_result['word_count']} words")
    print(f"Suitable for Summarization: {prep_result['is_suitable']}")
    
    if prep_result["warnings"]:
        print(f"Warnings: {'; '.join(prep_result['warnings'])}")
    
    print(f"\nProcessed Text:")
    print(prep_result["processed_text"])

# Run preparation demo
demo_document_preparation()
```

### 2. Summary Configuration Optimization

```python
class SummaryConfigurationOptimizer:
    """Optimize summarization parameters based on document characteristics"""
    
    def __init__(self, client):
        self.client = client
        
    def optimize_config(self, document, target_compression=0.3):
        """Determine optimal summarization configuration"""
        
        # Analyze document characteristics
        word_count = len(document.split())
        sentence_count = len([s for s in document.split('.') if s.strip()])
        avg_sentence_length = word_count / sentence_count if sentence_count else 0
        
        # Calculate optimal parameters
        config = {
            "extractive": {},
            "abstractive": {},
            "recommendations": []
        }
        
        # Extractive configuration
        if word_count < 200:
            config["extractive"]["max_sentence_count"] = 2
            config["recommendations"].append("Short document: using minimal extractive sentences")
        elif word_count < 1000:
            config["extractive"]["max_sentence_count"] = 3
        elif word_count < 3000:
            config["extractive"]["max_sentence_count"] = 5
        else:
            config["extractive"]["max_sentence_count"] = 7
            config["recommendations"].append("Long document: using maximum extractive sentences")
        
        # Abstractive configuration
        target_sentences = max(1, int(sentence_count * target_compression))
        config["abstractive"]["sentence_count"] = min(target_sentences, 4)  # API limit
        
        # Ordering preference
        if avg_sentence_length > 25:
            config["extractive"]["order_by"] = "Rank"
            config["recommendations"].append("Long sentences: ranking by importance")
        else:
            config["extractive"]["order_by"] = "Offset"
            config["recommendations"].append("Short sentences: preserving order")
        
        # Quality recommendations
        if word_count < 100:
            config["recommendations"].append("Warning: Document may be too short for effective summarization")
        
        if sentence_count < 3:
            config["recommendations"].append("Warning: Very few sentences may limit summary quality")
        
        return config

# Example usage
def demo_config_optimization():
    """Demonstrate configuration optimization"""
    
    optimizer = SummaryConfigurationOptimizer(client)
    
    test_documents = [
        {
            "name": "Short Article",
            "text": "AI is transforming healthcare. Machine learning models help doctors diagnose diseases faster and more accurately."
        },
        {
            "name": "Medium Article", 
            "text": """
            Artificial intelligence is revolutionizing healthcare delivery across multiple domains. 
            Machine learning algorithms are being deployed for medical image analysis, enabling 
            radiologists to detect diseases with greater accuracy and speed. Natural language 
            processing systems help extract insights from electronic health records and clinical 
            notes. Predictive analytics models identify patients at risk for complications, 
            allowing for proactive interventions. However, challenges remain in ensuring model 
            transparency, addressing algorithmic bias, and maintaining patient privacy.
            """
        }
    ]
    
    for doc in test_documents:
        print(f"\n📊 Configuration for {doc['name']}")
        print("=" * 40)
        
        config = optimizer.optimize_config(doc["text"])
        
        print(f"Extractive Config: {config['extractive']}")
        print(f"Abstractive Config: {config['abstractive']}")
        print("Recommendations:")
        for rec in config["recommendations"]:
            print(f"  • {rec}")

# Run configuration demo
demo_config_optimization()
```

---

## 🎯 AI-102 Exam Tips

### Key Concepts to Remember

1. **Summarization Types**
   - **Extractive**: Selects important sentences from original text
   - **Abstractive**: Generates new summary text using AI
   - Choose based on use case and quality requirements

2. **Configuration Parameters**
   - `max_sentence_count` for extractive (1-20)
   - `sentence_count` for abstractive (1-4)
   - `order_by`: "Rank" or "Offset"

3. **Document Requirements**
   - Minimum 40 words for effective summarization
   - Maximum 125,000 characters per document
   - Optimal range: 200-5000 words

4. **Performance Considerations**
   - Async operations for larger documents
   - Batch processing for multiple documents
   - Monitor compression ratios for quality assessment

5. **Integration Patterns**
   - Content management systems
   - Document analysis pipelines
   - News aggregation services
   - Research paper processing

### Common Exam Scenarios

- **Document analysis workflows**
- **Content curation systems**
- **Research paper summarization**
- **News article aggregation**
- **Meeting notes processing**

---

## 📚 Additional Resources

- **📖 [Text Summarization Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/summarization/overview)**
- **🔧 [Python SDK Reference](https://docs.microsoft.com/python/api/azure-ai-textanalytics/azure.ai.textanalytics.textanalyticsclient.begin_analyze_actions)**
- **🌐 [REST API Reference](https://docs.microsoft.com/rest/api/language/text-analysis-runtime/submit-job)**
- **💡 [Best Practices Guide](https://docs.microsoft.com/azure/cognitive-services/language-service/summarization/how-to/call-api)**
- **🎓 [AI-102 Study Guide](https://docs.microsoft.com/learn/certifications/exams/ai-102)**

---

*This guide provides comprehensive coverage of Azure AI Text Summarization capabilities for the AI-102 certification exam.*