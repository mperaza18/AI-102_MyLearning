# 🔷 Key Phrase Extraction Python SDK Examples

> **Python SDK examples for Key Phrase Extraction using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```bash
pip install azure-ai-textanalytics azure-identity python-dotenv pandas matplotlib
```

### Basic Authentication Setup

```python
import os
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv
from collections import Counter, defaultdict
import pandas as pd

# Load environment variables
load_dotenv()

# Method 1: Using API Key (simpler for development)
endpoint = os.getenv('LANGUAGE_ENDPOINT')
key = os.getenv('LANGUAGE_KEY')

credential = AzureKeyCredential(key)
text_analytics_client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

# Method 2: Using Azure Identity (recommended for production)
identity_credential = DefaultAzureCredential()
identity_client = TextAnalyticsClient(endpoint=endpoint, credential=identity_credential)
```

## 🔧 Basic Examples

### Simple Key Phrase Extraction

```python
def extract_key_phrases_basic(client, text):
    """Basic key phrase extraction example"""
    try:
        documents = [text]
        response = client.extract_key_phrases(documents=documents)[0]
        
        if not response.is_error:
            print(f"Analyzing: '{text[:60]}...'")
            print(f"Key phrases found: {len(response.key_phrases)}")
            print("Key Phrases:")
            for i, phrase in enumerate(response.key_phrases, 1):
                print(f"  {i:2d}. {phrase}")
        else:
            print(f"Error: {response.error}")
            
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example usage
sample_text = """
Microsoft Azure is a comprehensive cloud computing platform that offers a wide range 
of services including virtual machines, databases, artificial intelligence tools, 
and analytics services. It helps businesses scale their operations efficiently 
and reduce infrastructure costs through flexible pricing models.
"""

extract_key_phrases_basic(text_analytics_client, sample_text)
```

### Batch Key Phrase Extraction

```python
def extract_key_phrases_batch(client, documents):
    """Process multiple documents at once"""
    try:
        response = client.extract_key_phrases(documents=documents)
        
        for idx, doc in enumerate(response):
            print(f"\nDocument {idx + 1}:")
            if not doc.is_error:
                print(f"  Text preview: '{documents[idx][:50]}...'")
                print(f"  Key phrases ({len(doc.key_phrases)} found):")
                for phrase in doc.key_phrases:
                    print(f"    • {phrase}")
                
                # Show warnings if any
                if hasattr(doc, 'warnings') and doc.warnings:
                    print("  Warnings:")
                    for warning in doc.warnings:
                        print(f"    - {warning}")
            else:
                print(f"  Error: {doc.error}")
                
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example usage
business_documents = [
    "Our quarterly results demonstrate strong performance in cloud services and artificial intelligence solutions.",
    "Digital transformation initiatives are driving innovation and operational efficiency across multiple business units.",
    "Customer experience improvements through data analytics and machine learning have increased satisfaction scores.",
    "Supply chain optimization using predictive analytics has reduced costs and improved delivery performance."
]

extract_key_phrases_batch(text_analytics_client, business_documents)
```

## 🔧 Advanced Examples

### Multi-Language Key Phrase Extraction

```python
from azure.ai.textanalytics import TextDocumentInput

def extract_key_phrases_multilingual(client, multilingual_documents):
    """Handle multiple documents with different languages"""
    try:
        # Create TextDocumentInput objects with language specification
        text_documents = []
        for doc_id, text, language in multilingual_documents:
            text_documents.append(
                TextDocumentInput(id=doc_id, text=text, language=language)
            )
        
        response = client.extract_key_phrases(documents=text_documents)
        
        language_results = {}
        for doc in response:
            if not doc.is_error:
                doc_info = next((d for d in multilingual_documents if d[0] == doc.id), None)
                if doc_info:
                    language = doc_info[2]
                    if language not in language_results:
                        language_results[language] = []
                    language_results[language].extend(doc.key_phrases)
                
                print(f"Document ID: {doc.id}")
                print(f"Key phrases: {doc.key_phrases}")
                print()
            else:
                print(f"Document {doc.id} - Error: {doc.error}")
        
        # Analyze common themes across languages
        print("CROSS-LANGUAGE ANALYSIS:")
        for lang, phrases in language_results.items():
            print(f"\n{lang.upper()} key phrases:")
            phrase_freq = Counter(phrases)
            for phrase, count in phrase_freq.most_common(5):
                print(f"  • {phrase} ({count})")
                
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example with multiple languages
multilingual_docs = [
    ("en_doc", "Cloud computing and artificial intelligence are transforming business operations.", "en"),
    ("es_doc", "La computación en la nube y la inteligencia artificial están transformando las operaciones comerciales.", "es"),
    ("fr_doc", "Le cloud computing et l'intelligence artificielle transforment les opérations commerciales.", "fr"),
    ("de_doc", "Cloud Computing und künstliche Intelligenz transformieren Geschäftsabläufe.", "de")
]

extract_key_phrases_multilingual(text_analytics_client, multilingual_docs)
```

### Content Theme Analysis

```python
class ContentThemeAnalyzer:
    def __init__(self, client):
        self.client = client
        
    def analyze_content_themes(self, documents, doc_labels=None):
        """Comprehensive theme analysis across multiple documents"""
        try:
            response = self.client.extract_key_phrases(documents=documents)
            
            # Collect all key phrases
            all_phrases = []
            doc_phrases = {}
            
            for idx, doc in enumerate(response):
                doc_label = doc_labels[idx] if doc_labels else f"Document {idx + 1}"
                
                if not doc.is_error:
                    doc_phrases[doc_label] = doc.key_phrases
                    all_phrases.extend(doc.key_phrases)
                else:
                    print(f"{doc_label} - Error: {doc.error}")
            
            # Analyze themes
            analysis_results = {
                'total_documents': len(documents),
                'successful_analyses': len(doc_phrases),
                'total_key_phrases': len(all_phrases),
                'unique_phrases': len(set(all_phrases)),
                'phrase_frequency': Counter(all_phrases),
                'document_phrases': doc_phrases
            }
            
            # Find common themes (phrases appearing in multiple documents)
            phrase_distribution = defaultdict(int)
            for phrases in doc_phrases.values():
                for phrase in set(phrases):  # Use set to count each phrase once per document
                    phrase_distribution[phrase] += 1
            
            common_themes = {phrase: count for phrase, count in phrase_distribution.items() if count > 1}
            analysis_results['common_themes'] = common_themes
            
            # Categorize phrases by length and complexity
            short_phrases = [p for p in set(all_phrases) if len(p.split()) <= 2]
            medium_phrases = [p for p in set(all_phrases) if 3 <= len(p.split()) <= 4]
            long_phrases = [p for p in set(all_phrases) if len(p.split()) > 4]
            
            analysis_results['phrase_categories'] = {
                'short_phrases': short_phrases,
                'medium_phrases': medium_phrases,
                'long_phrases': long_phrases
            }
            
            return analysis_results
            
        except Exception as err:
            print(f"Theme analysis failed: {err}")
            return None
    
    def print_theme_report(self, analysis_results):
        """Print a comprehensive theme analysis report"""
        if not analysis_results:
            print("No analysis results to display")
            return
        
        print("CONTENT THEME ANALYSIS REPORT")
        print("=" * 50)
        print(f"Total Documents: {analysis_results['total_documents']}")
        print(f"Successful Analyses: {analysis_results['successful_analyses']}")
        print(f"Total Key Phrases: {analysis_results['total_key_phrases']}")
        print(f"Unique Phrases: {analysis_results['unique_phrases']}")
        
        print("\n📊 Most Frequent Key Phrases:")
        for phrase, count in analysis_results['phrase_frequency'].most_common(10):
            print(f"  • {phrase}: {count}")
        
        print("\n🔗 Common Themes Across Documents:")
        if analysis_results['common_themes']:
            sorted_themes = sorted(analysis_results['common_themes'].items(), 
                                 key=lambda x: x[1], reverse=True)
            for phrase, doc_count in sorted_themes[:10]:
                print(f"  • {phrase} (in {doc_count} documents)")
        else:
            print("  No common themes found across multiple documents")
        
        print("\n📏 Phrase Categories by Length:")
        categories = analysis_results['phrase_categories']
        print(f"  Short phrases (1-2 words): {len(categories['short_phrases'])}")
        print(f"  Medium phrases (3-4 words): {len(categories['medium_phrases'])}")
        print(f"  Long phrases (5+ words): {len(categories['long_phrases'])}")
        
        print("\n📄 Document-Specific Phrases:")
        for doc_label, phrases in analysis_results['document_phrases'].items():
            print(f"  {doc_label}: {len(phrases)} key phrases")
            print(f"    Top 3: {phrases[:3]}")

# Example usage
analyzer = ContentThemeAnalyzer(text_analytics_client)

technology_articles = [
    "Artificial intelligence and machine learning technologies are revolutionizing data analysis and business intelligence.",
    "Cloud computing platforms provide scalable infrastructure for modern applications and data storage solutions.",
    "Cybersecurity frameworks protect digital assets through advanced threat detection and prevention systems.",
    "Digital transformation strategies leverage emerging technologies to improve operational efficiency and customer experience.",
    "Internet of Things devices generate massive datasets that require advanced analytics and real-time processing capabilities."
]

article_labels = ["AI/ML Article", "Cloud Computing", "Cybersecurity", "Digital Transformation", "IoT Analytics"]

theme_analysis = analyzer.analyze_content_themes(technology_articles, article_labels)
analyzer.print_theme_report(theme_analysis)
```

## 🔧 Specialized Use Cases

### Customer Feedback Analysis

```python
class CustomerFeedbackAnalyzer:
    def __init__(self, client):
        self.client = client
        
    def analyze_feedback_themes(self, feedback_texts):
        """Extract and categorize key themes from customer feedback"""
        try:
            response = self.client.extract_key_phrases(documents=feedback_texts)
            
            # Define categories based on common feedback themes
            categories = {
                'product_features': [],
                'service_quality': [],
                'user_experience': [],
                'technical_issues': [],
                'pricing_value': [],
                'performance': [],
                'support': []
            }
            
            # Keywords for categorization
            category_keywords = {
                'product_features': ['product', 'feature', 'functionality', 'capability', 'tool', 'option'],
                'service_quality': ['service', 'quality', 'support', 'help', 'assistance', 'care'],
                'user_experience': ['experience', 'interface', 'usability', 'design', 'navigation', 'user'],
                'technical_issues': ['bug', 'error', 'issue', 'problem', 'crash', 'failure', 'technical'],
                'pricing_value': ['price', 'cost', 'value', 'expensive', 'affordable', 'pricing', 'budget'],
                'performance': ['speed', 'fast', 'slow', 'performance', 'efficiency', 'response', 'latency'],
                'support': ['support', 'help desk', 'customer service', 'documentation', 'training']
            }
            
            all_feedback_phrases = []
            
            for idx, doc in enumerate(response):
                if not doc.is_error:
                    all_feedback_phrases.extend(doc.key_phrases)
                    
                    # Categorize phrases
                    for phrase in doc.key_phrases:
                        phrase_lower = phrase.lower()
                        categorized = False
                        
                        for category, keywords in category_keywords.items():
                            if any(keyword in phrase_lower for keyword in keywords):
                                categories[category].append(phrase)
                                categorized = True
                                break
                        
                        if not categorized:
                            # Could add to a 'miscellaneous' category
                            pass
                else:
                    print(f"Feedback {idx + 1} - Error: {doc.error}")
            
            # Calculate category statistics
            category_stats = {}
            for category, phrases in categories.items():
                if phrases:
                    category_stats[category] = {
                        'count': len(phrases),
                        'unique_phrases': len(set(phrases)),
                        'top_phrases': Counter(phrases).most_common(5)
                    }
            
            return {
                'total_feedback_count': len(feedback_texts),
                'total_key_phrases': len(all_feedback_phrases),
                'categories': category_stats,
                'overall_top_phrases': Counter(all_feedback_phrases).most_common(10)
            }
            
        except Exception as err:
            print(f"Feedback analysis failed: {err}")
            return None
    
    def print_feedback_report(self, analysis_results):
        """Print customer feedback analysis report"""
        if not analysis_results:
            return
        
        print("CUSTOMER FEEDBACK ANALYSIS")
        print("=" * 40)
        print(f"Total Feedback Items: {analysis_results['total_feedback_count']}")
        print(f"Total Key Phrases: {analysis_results['total_key_phrases']}")
        
        print("\n🎯 Overall Top Key Phrases:")
        for phrase, count in analysis_results['overall_top_phrases']:
            print(f"  • {phrase}: {count}")
        
        print("\n📋 Category Analysis:")
        for category, stats in analysis_results['categories'].items():
            print(f"\n  {category.replace('_', ' ').title()}:")
            print(f"    Phrase Count: {stats['count']}")
            print(f"    Unique Phrases: {stats['unique_phrases']}")
            print("    Top Phrases:")
            for phrase, count in stats['top_phrases']:
                print(f"      • {phrase}: {count}")

# Example usage
feedback_analyzer = CustomerFeedbackAnalyzer(text_analytics_client)

customer_feedback = [
    "The product quality is excellent and the user interface is very intuitive. Customer support response time could be improved.",
    "Great value for money and performance is outstanding. Some advanced features are hard to find in the interface.",
    "Technical issues with the mobile app but the web version works perfectly. Documentation is comprehensive.",
    "Pricing is competitive and the service quality exceeds expectations. Would like more training materials.",
    "Fast performance and reliable service. The support team is knowledgeable and helpful with technical problems."
]

feedback_results = feedback_analyzer.analyze_feedback_themes(customer_feedback)
feedback_analyzer.print_feedback_report(feedback_results)
```

### Research Paper Analysis

```python
class ResearchPaperAnalyzer:
    def __init__(self, client):
        self.client = client
        
    def analyze_research_abstracts(self, abstracts, paper_titles=None):
        """Analyze research paper abstracts for key concepts and methodologies"""
        try:
            response = self.client.extract_key_phrases(documents=abstracts)
            
            # Categories for research papers
            research_categories = {
                'methodologies': [],
                'technologies': [],
                'applications': [],
                'metrics': [],
                'domains': [],
                'concepts': []
            }
            
            # Research-specific keywords
            research_keywords = {
                'methodologies': ['algorithm', 'method', 'approach', 'technique', 'framework', 'model', 'analysis'],
                'technologies': ['machine learning', 'deep learning', 'neural network', 'AI', 'blockchain', 'cloud'],
                'applications': ['application', 'system', 'platform', 'solution', 'implementation', 'deployment'],
                'metrics': ['accuracy', 'performance', 'efficiency', 'improvement', 'optimization', 'results'],
                'domains': ['healthcare', 'finance', 'education', 'manufacturing', 'retail', 'automotive'],
                'concepts': ['data', 'information', 'knowledge', 'intelligence', 'automation', 'prediction']
            }
            
            paper_analyses = []
            all_research_phrases = []
            
            for idx, doc in enumerate(response):
                paper_title = paper_titles[idx] if paper_titles else f"Paper {idx + 1}"
                
                if not doc.is_error:
                    all_research_phrases.extend(doc.key_phrases)
                    
                    paper_analysis = {
                        'title': paper_title,
                        'key_phrases': doc.key_phrases,
                        'categorized_phrases': {category: [] for category in research_categories}
                    }
                    
                    # Categorize phrases for this paper
                    for phrase in doc.key_phrases:
                        phrase_lower = phrase.lower()
                        for category, keywords in research_keywords.items():
                            if any(keyword in phrase_lower for keyword in keywords):
                                paper_analysis['categorized_phrases'][category].append(phrase)
                                research_categories[category].append(phrase)
                    
                    paper_analyses.append(paper_analysis)
                else:
                    print(f"{paper_title} - Error: {doc.error}")
            
            # Find trending research topics
            trending_topics = Counter(all_research_phrases).most_common(15)
            
            # Analyze research focus areas
            focus_areas = {}
            for category, phrases in research_categories.items():
                if phrases:
                    focus_areas[category] = Counter(phrases).most_common(5)
            
            return {
                'paper_count': len(abstracts),
                'successful_analyses': len(paper_analyses),
                'paper_analyses': paper_analyses,
                'trending_topics': trending_topics,
                'research_focus_areas': focus_areas,
                'cross_paper_themes': self._find_cross_paper_themes(paper_analyses)
            }
            
        except Exception as err:
            print(f"Research analysis failed: {err}")
            return None
    
    def _find_cross_paper_themes(self, paper_analyses):
        """Find themes that appear across multiple papers"""
        phrase_papers = defaultdict(set)
        
        for idx, paper in enumerate(paper_analyses):
            for phrase in paper['key_phrases']:
                phrase_papers[phrase].add(idx)
        
        # Find phrases that appear in multiple papers
        cross_themes = {
            phrase: len(papers) 
            for phrase, papers in phrase_papers.items() 
            if len(papers) > 1
        }
        
        return sorted(cross_themes.items(), key=lambda x: x[1], reverse=True)
    
    def print_research_report(self, analysis_results):
        """Print comprehensive research analysis report"""
        if not analysis_results:
            return
        
        print("RESEARCH PAPER ANALYSIS REPORT")
        print("=" * 45)
        print(f"Total Papers: {analysis_results['paper_count']}")
        print(f"Successful Analyses: {analysis_results['successful_analyses']}")
        
        print("\n🔬 Trending Research Topics:")
        for phrase, count in analysis_results['trending_topics']:
            print(f"  • {phrase}: {count}")
        
        print("\n🎯 Research Focus Areas:")
        for area, top_phrases in analysis_results['research_focus_areas'].items():
            print(f"\n  {area.replace('_', ' ').title()}:")
            for phrase, count in top_phrases:
                print(f"    • {phrase}: {count}")
        
        print("\n🔗 Cross-Paper Themes:")
        for phrase, paper_count in analysis_results['cross_paper_themes'][:10]:
            print(f"  • {phrase} (in {paper_count} papers)")
        
        print("\n📄 Individual Paper Analysis:")
        for paper in analysis_results['paper_analyses'][:3]:  # Show first 3 papers
            print(f"\n  {paper['title']}:")
            print(f"    Key phrases: {len(paper['key_phrases'])}")
            
            # Show top categories for this paper
            for category, phrases in paper['categorized_phrases'].items():
                if phrases:
                    print(f"    {category}: {len(phrases)} phrases")

# Example usage
research_analyzer = ResearchPaperAnalyzer(text_analytics_client)

research_abstracts = [
    "This paper presents a novel deep learning approach for natural language processing tasks. The proposed neural network architecture demonstrates significant improvements in text classification accuracy and computational efficiency.",
    "We investigate machine learning techniques for predictive analytics in healthcare systems. Our methodology combines data mining algorithms with clinical decision support systems to improve patient outcomes.",
    "The study examines blockchain technology applications in supply chain management. We develop a distributed framework that enhances transparency and traceability in manufacturing processes.",
    "This research explores computer vision algorithms for autonomous vehicle navigation. Our approach integrates sensor fusion techniques with real-time image processing for enhanced safety performance.",
]

paper_titles = [
    "Deep Learning for NLP Tasks",
    "ML in Healthcare Analytics", 
    "Blockchain Supply Chain Management",
    "Computer Vision for Autonomous Vehicles"
]

research_results = research_analyzer.analyze_research_abstracts(research_abstracts, paper_titles)
research_analyzer.print_research_report(research_results)
```

## 🚀 Async Operations

### Asynchronous Key Phrase Extraction

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

async def extract_key_phrases_async(endpoint, key, documents):
    """Asynchronous key phrase extraction"""
    credential = AzureKeyCredential(key)
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        try:
            response = await client.extract_key_phrases(documents=documents)
            
            results = []
            for idx, doc in enumerate(response):
                if not doc.is_error:
                    results.append({
                        'document_index': idx,
                        'key_phrases': doc.key_phrases,
                        'phrase_count': len(doc.key_phrases)
                    })
                else:
                    results.append({
                        'document_index': idx,
                        'error': str(doc.error),
                        'key_phrases': []
                    })
            
            return results
            
        except Exception as err:
            print(f"Async operation failed: {err}")
            return None

# Example usage
async def main():
    documents = [
        "Cloud computing platforms enable digital transformation and scalable business solutions.",
        "Artificial intelligence applications revolutionize data analysis and decision-making processes.",
        "Cybersecurity measures protect sensitive information and maintain data privacy compliance."
    ]
    
    results = await extract_key_phrases_async(endpoint, key, documents)
    
    if results:
        for result in results:
            if 'error' not in result:
                print(f"Document {result['document_index']}: {result['phrase_count']} key phrases")
                for phrase in result['key_phrases']:
                    print(f"  • {phrase}")
            else:
                print(f"Document {result['document_index']} failed: {result['error']}")

# Run async function
# asyncio.run(main())
```

### Concurrent Batch Processing

```python
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor

def process_large_dataset_concurrent(client, documents, max_workers=5):
    """Process large dataset using concurrent threads"""
    
    def process_batch(batch):
        """Process a batch of documents"""
        try:
            response = client.extract_key_phrases(documents=batch)
            return [(doc.key_phrases if not doc.is_error else str(doc.error)) for doc in response]
        except Exception as e:
            return [f"Error: {str(e)}" for _ in batch]
    
    # Split documents into batches of 10 (API limit)
    batch_size = 10
    batches = [documents[i:i + batch_size] for i in range(0, len(documents), batch_size)]
    
    start_time = time.time()
    
    # Process batches concurrently
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        batch_results = list(executor.map(process_batch, batches))
    
    # Flatten results
    all_results = []
    for batch_result in batch_results:
        all_results.extend(batch_result)
    
    end_time = time.time()
    
    print(f"Processed {len(documents)} documents in {end_time - start_time:.2f} seconds")
    print(f"Used {len(batches)} batches with max {max_workers} concurrent workers")
    
    # Analyze results
    successful_extractions = [r for r in all_results if isinstance(r, list)]
    all_phrases = [phrase for result in successful_extractions for phrase in result]
    
    print(f"Successfully processed: {len(successful_extractions)} documents")
    print(f"Total key phrases extracted: {len(all_phrases)}")
    print(f"Unique key phrases: {len(set(all_phrases))}")
    
    # Show top phrases
    if all_phrases:
        phrase_freq = Counter(all_phrases)
        print("\nTop 10 Key Phrases:")
        for phrase, count in phrase_freq.most_common(10):
            print(f"  • {phrase}: {count}")
    
    return all_results

# Generate sample dataset
sample_documents = [
    f"Technology company {i} focuses on cloud computing and artificial intelligence solutions."
    for i in range(30)
]

# Process with concurrency
concurrent_results = process_large_dataset_concurrent(text_analytics_client, sample_documents)
```

## 📊 Data Analysis Integration

### Integration with Pandas

```python
import pandas as pd
from datetime import datetime, timedelta

def create_key_phrase_dataframe(client, documents, document_metadata=None):
    """Create a comprehensive DataFrame for key phrase analysis"""
    
    response = client.extract_key_phrases(documents=documents)
    
    # Prepare data for DataFrame
    data_rows = []
    
    for doc_idx, doc in enumerate(response):
        doc_meta = document_metadata[doc_idx] if document_metadata else {}
        
        if not doc.is_error:
            for phrase in doc.key_phrases:
                data_rows.append({
                    'document_id': doc_idx,
                    'document_source': doc_meta.get('source', 'unknown'),
                    'document_category': doc_meta.get('category', 'general'),
                    'document_date': doc_meta.get('date', datetime.now()),
                    'key_phrase': phrase,
                    'phrase_length': len(phrase),
                    'word_count': len(phrase.split()),
                    'has_numbers': any(char.isdigit() for char in phrase),
                    'is_technical': any(term in phrase.lower() for term in ['technology', 'software', 'system', 'platform', 'algorithm'])
                })
    
    return pd.DataFrame(data_rows)

# Example with metadata
business_docs = [
    "Digital transformation initiatives drive operational efficiency and customer satisfaction improvements.",
    "Financial technology solutions enhance payment processing and regulatory compliance capabilities.",
    "Healthcare analytics platforms improve patient outcomes through predictive modeling and data insights.",
    "Supply chain automation reduces costs and improves inventory management across global operations."
]

metadata = [
    {'source': 'company_report', 'category': 'business', 'date': datetime.now() - timedelta(days=30)},
    {'source': 'industry_news', 'category': 'fintech', 'date': datetime.now() - timedelta(days=15)},
    {'source': 'research_paper', 'category': 'healthcare', 'date': datetime.now() - timedelta(days=7)},
    {'source': 'case_study', 'category': 'logistics', 'date': datetime.now()}
]

# Create DataFrame
df = create_key_phrase_dataframe(text_analytics_client, business_docs, metadata)

print("KEY PHRASE DATAFRAME ANALYSIS")
print("=" * 40)
print(f"Total records: {len(df)}")
print(f"Unique phrases: {df['key_phrase'].nunique()}")
print(f"Documents analyzed: {df['document_id'].nunique()}")

# Analysis examples
print("\n📊 Phrase Analysis by Category:")
category_analysis = df.groupby('document_category').agg({
    'key_phrase': 'count',
    'phrase_length': 'mean',
    'word_count': 'mean'
}).round(2)
print(category_analysis)

print("\n🔤 Most Common Key Phrases:")
top_phrases = df['key_phrase'].value_counts().head(10)
print(top_phrases)

print("\n📏 Phrase Length Distribution:")
length_dist = df.groupby('word_count')['key_phrase'].count()
print(length_dist)

print("\n🔧 Technical vs Non-Technical Phrases:")
tech_analysis = df.groupby('is_technical')['key_phrase'].count()
print(tech_analysis)

# Time-based analysis
print("\n📅 Key Phrases Over Time:")
df['date_group'] = pd.to_datetime(df['document_date']).dt.date
time_analysis = df.groupby('date_group')['key_phrase'].count()
print(time_analysis)
```

### Visualization with Matplotlib

```python
import matplotlib.pyplot as plt
import seaborn as sns

def visualize_key_phrase_analysis(df):
    """Create visualizations for key phrase analysis"""
    
    # Set up the plotting style
    plt.style.use('default')
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    fig.suptitle('Key Phrase Analysis Dashboard', fontsize=16, fontweight='bold')
    
    # 1. Top 10 Key Phrases
    top_phrases = df['key_phrase'].value_counts().head(10)
    axes[0, 0].barh(range(len(top_phrases)), top_phrases.values)
    axes[0, 0].set_yticks(range(len(top_phrases)))
    axes[0, 0].set_yticklabels([phrase[:25] + '...' if len(phrase) > 25 else phrase 
                               for phrase in top_phrases.index])
    axes[0, 0].set_xlabel('Frequency')
    axes[0, 0].set_title('Top 10 Key Phrases')
    axes[0, 0].invert_yaxis()
    
    # 2. Phrase Length Distribution
    axes[0, 1].hist(df['phrase_length'], bins=20, alpha=0.7, color='skyblue', edgecolor='black')
    axes[0, 1].set_xlabel('Phrase Length (characters)')
    axes[0, 1].set_ylabel('Frequency')
    axes[0, 1].set_title('Phrase Length Distribution')
    
    # 3. Word Count Distribution
    word_count_dist = df['word_count'].value_counts().sort_index()
    axes[1, 0].bar(word_count_dist.index, word_count_dist.values, color='lightcoral', alpha=0.7)
    axes[1, 0].set_xlabel('Number of Words')
    axes[1, 0].set_ylabel('Frequency')
    axes[1, 0].set_title('Word Count Distribution')
    
    # 4. Category Analysis
    if 'document_category' in df.columns:
        category_counts = df.groupby('document_category')['key_phrase'].count()
        axes[1, 1].pie(category_counts.values, labels=category_counts.index, autopct='%1.1f%%')
        axes[1, 1].set_title('Key Phrases by Document Category')
    else:
        # Alternative: Technical vs Non-Technical
        tech_counts = df.groupby('is_technical')['key_phrase'].count()
        labels = ['Non-Technical', 'Technical']
        axes[1, 1].pie(tech_counts.values, labels=labels, autopct='%1.1f%%')
        axes[1, 1].set_title('Technical vs Non-Technical Phrases')
    
    plt.tight_layout()
    return fig

# Generate visualization (uncomment to display)
# if len(df) > 0:
#     viz_fig = visualize_key_phrase_analysis(df)
#     plt.show()
```

## 📚 Additional Resources

- [Azure AI Language Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)
- [Key Phrase Extraction Samples](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics)