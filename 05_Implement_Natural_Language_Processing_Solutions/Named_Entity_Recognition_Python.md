# 🔷 Named Entity Recognition Python SDK Examples

> **Python SDK examples for Named Entity Recognition using Azure AI Language services**

## 📋 Setup and Authentication

### Installation

```bash
pip install azure-ai-textanalytics azure-identity python-dotenv
```

### Basic Authentication Setup

```python
import os
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

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

### Simple Entity Recognition

```python
def recognize_entities_basic(client, text):
    """Basic entity recognition example"""
    try:
        documents = [text]
        response = client.recognize_entities(documents=documents)[0]
        
        if not response.is_error:
            print(f"Analyzing: '{text[:50]}...'")
            print("Entities found:")
            for entity in response.entities:
                print(f"  Text: {entity.text}")
                print(f"  Category: {entity.category}")
                print(f"  Subcategory: {entity.subcategory}")
                print(f"  Confidence Score: {entity.confidence_score:.2f}")
                print(f"  Offset: {entity.offset}")
                print(f"  Length: {entity.length}")
                print("---")
        else:
            print(f"Error: {response.error}")
            
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example usage
text = "Microsoft was founded by Bill Gates and Paul Allen in 1975. The company is headquartered in Redmond, Washington."
recognize_entities_basic(text_analytics_client, text)
```

### Batch Entity Recognition

```python
def recognize_entities_batch(client, documents):
    """Process multiple documents at once"""
    try:
        response = client.recognize_entities(documents=documents)
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"Document {idx + 1}:")
                entities_by_category = {}
                
                for entity in doc.entities:
                    category = entity.category
                    if category not in entities_by_category:
                        entities_by_category[category] = []
                    entities_by_category[category].append({
                        'text': entity.text,
                        'confidence': entity.confidence_score
                    })
                
                for category, entities in entities_by_category.items():
                    print(f"  {category}:")
                    for entity in entities:
                        print(f"    - {entity['text']} (confidence: {entity['confidence']:.2f})")
                print()
            else:
                print(f"Document {idx + 1} - Error: {doc.error}")
                
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example usage
documents = [
    "Apple Inc. is an American multinational technology company headquartered in Cupertino, California.",
    "Google was founded by Larry Page and Sergey Brin while they were Ph.D. students at Stanford University.",
    "Amazon.com, Inc. is an American multinational technology company based in Seattle, Washington."
]

recognize_entities_batch(text_analytics_client, documents)
```

## 🔧 Advanced Examples

### Entity Recognition with Language Detection

```python
from azure.ai.textanalytics import TextDocumentInput

def recognize_entities_multilingual(client, documents_with_languages):
    """Handle multiple documents with different languages"""
    try:
        # Create TextDocumentInput objects with language specification
        text_documents = []
        for doc_id, text, language in documents_with_languages:
            text_documents.append(
                TextDocumentInput(id=doc_id, text=text, language=language)
            )
        
        response = client.recognize_entities(documents=text_documents)
        
        for doc in response:
            if not doc.is_error:
                print(f"Document ID: {doc.id}")
                
                # Group entities by category
                entities_by_category = {}
                for entity in doc.entities:
                    if entity.category not in entities_by_category:
                        entities_by_category[entity.category] = []
                    entities_by_category[entity.category].append(entity)
                
                # Display results
                for category, entities in entities_by_category.items():
                    print(f"  {category} entities:")
                    for entity in entities:
                        print(f"    - {entity.text} (confidence: {entity.confidence_score:.2f})")
                print()
            else:
                print(f"Document {doc.id} - Error: {doc.error}")
                
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example with multiple languages
multilingual_docs = [
    ("doc1", "Microsoft Corporation is headquartered in Redmond, Washington.", "en"),
    ("doc2", "Microsoft Corporation est basée à Redmond, Washington.", "fr"),
    ("doc3", "Microsoft Corporation tiene su sede en Redmond, Washington.", "es"),
    ("doc4", "Microsoft Corporation hat ihren Hauptsitz in Redmond, Washington.", "de")
]

recognize_entities_multilingual(text_analytics_client, multilingual_docs)
```

### Entity Linking (Knowledge Base Integration)

```python
def recognize_linked_entities(client, documents):
    """Extract entities and link them to knowledge base"""
    try:
        response = client.recognize_linked_entities(documents=documents)
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"Document {idx + 1}:")
                
                for linked_entity in doc.entities:
                    print(f"  Entity: {linked_entity.name}")
                    print(f"  Language: {linked_entity.language}")
                    print(f"  Data Source: {linked_entity.data_source}")
                    print(f"  URL: {linked_entity.url}")
                    print(f"  Data Source Entity ID: {linked_entity.data_source_entity_id}")
                    
                    print("  Matches:")
                    for match in linked_entity.matches:
                        print(f"    - Text: '{match.text}'")
                        print(f"      Confidence Score: {match.confidence_score:.2f}")
                        print(f"      Offset: {match.offset}, Length: {match.length}")
                    print("---")
            else:
                print(f"Document {idx + 1} - Error: {doc.error}")
                
    except Exception as err:
        print(f"Encountered exception: {err}")

# Example usage
knowledge_texts = [
    "Microsoft was founded by Bill Gates and Paul Allen. Steve Jobs founded Apple.",
    "Einstein developed the theory of relativity. Newton formulated the laws of motion.",
    "The Eiffel Tower is located in Paris, France. The Statue of Liberty is in New York."
]

recognize_linked_entities(text_analytics_client, knowledge_texts)
```

## 🔧 Specialized Use Cases

### Business Document Analysis

```python
class BusinessEntityAnalyzer:
    def __init__(self, client):
        self.client = client
        
    def analyze_business_document(self, text):
        """Extract business-relevant entities from documents"""
        try:
            response = self.client.recognize_entities(documents=[text])[0]
            
            if response.is_error:
                return {"error": response.error}
            
            business_entities = {
                'organizations': [],
                'people': [],
                'locations': [],
                'financial': [],
                'dates': [],
                'contact_info': []
            }
            
            for entity in response.entities:
                if entity.category == 'Organization':
                    business_entities['organizations'].append({
                        'name': entity.text,
                        'confidence': entity.confidence_score
                    })
                elif entity.category == 'Person':
                    business_entities['people'].append({
                        'name': entity.text,
                        'confidence': entity.confidence_score
                    })
                elif entity.category in ['Location', 'GPE']:  # GPE = Geo-Political Entity
                    business_entities['locations'].append({
                        'name': entity.text,
                        'confidence': entity.confidence_score
                    })
                elif entity.category in ['Quantity', 'Money']:
                    business_entities['financial'].append({
                        'value': entity.text,
                        'confidence': entity.confidence_score
                    })
                elif entity.category in ['DateTime', 'DateRange']:
                    business_entities['dates'].append({
                        'date': entity.text,
                        'confidence': entity.confidence_score
                    })
                elif entity.category in ['Email', 'PhoneNumber', 'URL']:
                    business_entities['contact_info'].append({
                        'type': entity.category,
                        'value': entity.text,
                        'confidence': entity.confidence_score
                    })
            
            return business_entities
            
        except Exception as err:
            return {"error": str(err)}
    
    def analyze_contract(self, contract_text):
        """Specialized analysis for contract documents"""
        entities = self.analyze_business_document(contract_text)
        
        # Additional contract-specific processing
        contract_summary = {
            'parties': entities.get('organizations', []) + entities.get('people', []),
            'locations': entities.get('locations', []),
            'important_dates': entities.get('dates', []),
            'contact_information': entities.get('contact_info', []),
            'financial_terms': entities.get('financial', [])
        }
        
        return contract_summary

# Example usage
analyzer = BusinessEntityAnalyzer(text_analytics_client)

contract_text = """
This Software License Agreement is entered into on January 15, 2024, between 
TechCorp Inc., a corporation organized under the laws of Delaware, with offices 
at 123 Innovation Drive, San Francisco, CA 94105, and Client Solutions LLC, 
with principal offices at 456 Business Ave, New York, NY 10001. 

The total contract value is $150,000 payable over 12 months. For questions, 
contact John Smith at john.smith@techcorp.com or call +1-555-123-4567.
"""

contract_analysis = analyzer.analyze_contract(contract_text)

print("Contract Analysis Results:")
for section, items in contract_analysis.items():
    if items:
        print(f"\n{section.replace('_', ' ').title()}:")
        for item in items:
            if isinstance(item, dict):
                print(f"  - {list(item.values())[0]} (confidence: {item.get('confidence', 'N/A')})")
```

### Customer Feedback Entity Extraction

```python
def analyze_customer_feedback(client, feedback_list):
    """Extract entities from customer feedback for analysis"""
    try:
        response = client.recognize_entities(documents=feedback_list)
        
        feedback_analysis = {
            'companies_mentioned': set(),
            'products_mentioned': set(),
            'locations_mentioned': set(),
            'people_mentioned': set(),
            'contact_attempts': []
        }
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"Feedback {idx + 1}: '{feedback_list[idx][:60]}...'")
                
                for entity in doc.entities:
                    if entity.confidence_score > 0.7:  # Filter by confidence
                        if entity.category == 'Organization':
                            feedback_analysis['companies_mentioned'].add(entity.text)
                        elif entity.category == 'Product':
                            feedback_analysis['products_mentioned'].add(entity.text)
                        elif entity.category in ['Location', 'GPE']:
                            feedback_analysis['locations_mentioned'].add(entity.text)
                        elif entity.category == 'Person':
                            feedback_analysis['people_mentioned'].add(entity.text)
                        elif entity.category in ['Email', 'PhoneNumber']:
                            feedback_analysis['contact_attempts'].append(entity.text)
                
                print(f"  Entities found: {len(doc.entities)}")
        
        # Convert sets to lists for JSON serialization
        for key in ['companies_mentioned', 'products_mentioned', 'locations_mentioned', 'people_mentioned']:
            feedback_analysis[key] = list(feedback_analysis[key])
        
        return feedback_analysis
        
    except Exception as err:
        print(f"Error analyzing feedback: {err}")
        return None

# Example customer feedback
customer_feedback = [
    "I contacted Apple support about my iPhone 13 Pro battery issue. The representative Sarah was very helpful.",
    "Amazon Prime delivery to Seattle was delayed. I called customer service and spoke with Mike Johnson.",
    "Tesla Model S charging issue at the Fremont service center. The technician John fixed it quickly.",
    "Microsoft Teams integration with our CRM system works great. Our IT director Jane Smith is impressed."
]

analysis_results = analyze_customer_feedback(text_analytics_client, customer_feedback)

if analysis_results:
    print("\n" + "="*50)
    print("CUSTOMER FEEDBACK ANALYSIS SUMMARY")
    print("="*50)
    
    for category, items in analysis_results.items():
        if items:
            print(f"\n{category.replace('_', ' ').title()}:")
            for item in items:
                print(f"  • {item}")
```

## 🚀 Async Operations

### Asynchronous Entity Recognition

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

async def recognize_entities_async(endpoint, key, documents):
    """Asynchronous entity recognition"""
    credential = AzureKeyCredential(key)
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        try:
            response = await client.recognize_entities(documents=documents)
            
            results = []
            for idx, doc in enumerate(response):
                if not doc.is_error:
                    entities = []
                    for entity in doc.entities:
                        entities.append({
                            'text': entity.text,
                            'category': entity.category,
                            'confidence': entity.confidence_score,
                            'offset': entity.offset,
                            'length': entity.length
                        })
                    results.append({
                        'document_id': idx,
                        'entities': entities
                    })
                else:
                    results.append({
                        'document_id': idx,
                        'error': str(doc.error)
                    })
            
            return results
            
        except Exception as err:
            print(f"Async operation failed: {err}")
            return None

# Example usage
async def main():
    documents = [
        "Google CEO Sundar Pichai announced new AI features at Google I/O in Mountain View.",
        "Apple's Tim Cook presented the latest iPhone at the Steve Jobs Theater in Cupertino.",
        "Microsoft's Satya Nadella discussed Azure services at the Seattle headquarters."
    ]
    
    results = await recognize_entities_async(endpoint, key, documents)
    
    if results:
        for result in results:
            if 'error' not in result:
                print(f"Document {result['document_id']}:")
                for entity in result['entities']:
                    print(f"  {entity['text']} ({entity['category']}) - confidence: {entity['confidence']:.2f}")
            else:
                print(f"Document {result['document_id']} failed: {result['error']}")

# Run async function
# asyncio.run(main())
```

### Concurrent Processing

```python
import asyncio
import aiohttp
from concurrent.futures import ThreadPoolExecutor
import time

def process_large_dataset_concurrent(client, documents, max_workers=5):
    """Process large dataset using concurrent threads"""
    
    def process_batch(batch):
        """Process a batch of documents"""
        try:
            response = client.recognize_entities(documents=batch)
            return [(doc.entities if not doc.is_error else doc.error) for doc in response]
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
    
    return all_results

# Generate sample dataset
sample_documents = [
    f"Company {i} is located in City {i % 10} and was founded by Person {i}."
    for i in range(50)
]

# Process with concurrency
results = process_large_dataset_concurrent(text_analytics_client, sample_documents)

# Analyze results
successful_analyses = [r for r in results if not isinstance(r, str)]
print(f"Successfully processed: {len(successful_analyses)} documents")
```

## 📊 Data Analysis Integration

### Entity Frequency Analysis with Pandas

```python
import pandas as pd
from collections import Counter

def entity_analysis_with_pandas(client, documents):
    """Comprehensive entity analysis using pandas"""
    
    # Recognize entities
    response = client.recognize_entities(documents=documents)
    
    # Extract all entities into a list
    all_entities = []
    
    for doc_idx, doc in enumerate(response):
        if not doc.is_error:
            for entity in doc.entities:
                all_entities.append({
                    'document_id': doc_idx,
                    'text': entity.text,
                    'category': entity.category,
                    'subcategory': entity.subcategory,
                    'confidence_score': entity.confidence_score,
                    'offset': entity.offset,
                    'length': entity.length
                })
    
    # Create DataFrame
    df = pd.DataFrame(all_entities)
    
    if df.empty:
        print("No entities found in documents")
        return None
    
    # Analysis
    print("ENTITY ANALYSIS REPORT")
    print("=" * 50)
    
    # 1. Entity category distribution
    print("\n1. Entity Categories:")
    category_counts = df['category'].value_counts()
    print(category_counts)
    
    # 2. Most frequent entities
    print("\n2. Most Frequent Entities:")
    entity_counts = df['text'].value_counts().head(10)
    print(entity_counts)
    
    # 3. Average confidence by category
    print("\n3. Average Confidence by Category:")
    avg_confidence = df.groupby('category')['confidence_score'].mean().sort_values(ascending=False)
    for category, conf in avg_confidence.items():
        print(f"  {category}: {conf:.3f}")
    
    # 4. High-confidence entities (>0.8)
    print("\n4. High-Confidence Entities (>0.8):")
    high_conf = df[df['confidence_score'] > 0.8]['text'].value_counts().head(10)
    print(high_conf)
    
    return df

# Example with news articles
news_articles = [
    "Apple Inc. announced record quarterly earnings. CEO Tim Cook praised the team's performance.",
    "Microsoft Azure cloud services expanded to three new regions including Tokyo, Japan.",
    "Tesla CEO Elon Musk visited the Berlin Gigafactory to oversee Model Y production.",
    "Amazon Web Services reported strong growth in artificial intelligence and machine learning services.",
    "Google's parent company Alphabet posted solid financial results for Q3 2023."
]

entity_df = entity_analysis_with_pandas(text_analytics_client, news_articles)

# Additional pandas operations
if entity_df is not None:
    # Create pivot table
    print("\n5. Entity Distribution by Document:")
    pivot = entity_df.pivot_table(
        values='confidence_score', 
        index='document_id', 
        columns='category', 
        aggfunc='count', 
        fill_value=0
    )
    print(pivot)
```

### Entity Network Analysis

```python
import networkx as nx
import matplotlib.pyplot as plt
from collections import defaultdict

def create_entity_network(client, documents):
    """Create a network of co-occurring entities"""
    
    response = client.recognize_entities(documents=documents)
    
    # Build co-occurrence graph
    G = nx.Graph()
    
    for doc_idx, doc in enumerate(response):
        if not doc.is_error:
            # Get entities from this document
            doc_entities = [entity.text for entity in doc.entities if entity.confidence_score > 0.7]
            
            # Add nodes
            for entity in doc_entities:
                if not G.has_node(entity):
                    G.add_node(entity)
            
            # Add edges for co-occurring entities
            for i, entity1 in enumerate(doc_entities):
                for entity2 in doc_entities[i+1:]:
                    if G.has_edge(entity1, entity2):
                        G[entity1][entity2]['weight'] += 1
                    else:
                        G.add_edge(entity1, entity2, weight=1)
    
    # Analyze network
    print("ENTITY NETWORK ANALYSIS")
    print("=" * 30)
    print(f"Number of entities: {G.number_of_nodes()}")
    print(f"Number of relationships: {G.number_of_edges()}")
    
    # Most connected entities
    centrality = nx.degree_centrality(G)
    top_entities = sorted(centrality.items(), key=lambda x: x[1], reverse=True)[:5]
    
    print("\nMost Connected Entities:")
    for entity, centrality_score in top_entities:
        print(f"  {entity}: {centrality_score:.3f}")
    
    return G

# Example usage
business_documents = [
    "Microsoft and Apple are competing in the personal computing market.",
    "Google and Amazon are major players in cloud computing services.",
    "Tesla and Apple are both innovative technology companies based in California.",
    "Microsoft Azure competes with Amazon Web Services and Google Cloud Platform."
]

entity_graph = create_entity_network(text_analytics_client, business_documents)

# Visualize network (optional - requires matplotlib)
# plt.figure(figsize=(12, 8))
# pos = nx.spring_layout(entity_graph, k=1, iterations=50)
# nx.draw(entity_graph, pos, with_labels=True, node_color='lightblue', 
#         node_size=1000, font_size=8, font_weight='bold')
# plt.title("Entity Co-occurrence Network")
# plt.show()
```

## 📚 Additional Resources

- [Azure AI Language Python SDK Documentation](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)
- [Named Entity Recognition Samples](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics/samples)
- [SDK Source Code](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics/azure-ai-textanalytics)