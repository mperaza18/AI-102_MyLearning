# 🔷 Key Phrase Extraction REST API Examples

> **REST API examples for Key Phrase Extraction using Azure AI Language services**

## 🛠️ Setup and Authentication

### Authentication Methods

```bash
# Set environment variables
export LANGUAGE_ENDPOINT="https://your-resource.cognitiveservices.azure.com/"
export LANGUAGE_KEY="your-api-key"
```

### Basic Headers

```bash
# Standard headers for all requests
HEADERS=(-H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" -H "Content-Type: application/json")
```

## 🔧 Basic Examples

### Simple Key Phrase Extraction

```bash
# Basic key phrase extraction
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Microsoft Azure is a comprehensive cloud computing platform that offers a wide range of services including virtual machines, databases, and artificial intelligence tools. It helps businesses scale their operations efficiently."
        }
      ]
    }
  }' | jq '.results.documents[0].keyPhrases[]'
```

### Extract Key Phrases with Document Metadata

```bash
# Key phrase extraction with metadata display
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
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
          "text": "Artificial Intelligence and Machine Learning technologies are transforming the healthcare industry. Medical professionals can now leverage predictive analytics to improve patient outcomes and reduce costs."
        }
      ]
    }
  }' | jq '.results.documents[0] | {id: .id, keyPhrases: .keyPhrases, warnings: .warnings}'
```

## 🔧 Advanced Examples

### Batch Processing Multiple Documents

```bash
# Process multiple documents simultaneously
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "tech_news",
          "language": "en",
          "text": "Apple announced its new iPhone lineup with advanced camera features and improved battery life. The devices include cutting-edge processors and enhanced security features."
        },
        {
          "id": "business_update",
          "language": "en", 
          "text": "The quarterly earnings report shows significant growth in cloud computing revenue. Digital transformation initiatives continue to drive market expansion across multiple sectors."
        },
        {
          "id": "research_paper",
          "language": "en",
          "text": "Recent studies in renewable energy demonstrate promising advances in solar panel efficiency. Researchers have developed new materials that significantly improve energy conversion rates."
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, keyPhrases: .keyPhrases}'
```

### Multi-Language Key Phrase Extraction

```bash
# Extract key phrases from different languages
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "english",
          "language": "en",
          "text": "Cloud computing and artificial intelligence are revolutionizing modern business operations."
        },
        {
          "id": "spanish",
          "language": "es",
          "text": "La computación en la nube y la inteligencia artificial están revolucionando las operaciones comerciales modernas."
        },
        {
          "id": "french",
          "language": "fr",
          "text": "Le cloud computing et l'intelligence artificielle révolutionnent les opérations commerciales modernes."
        },
        {
          "id": "german",
          "language": "de",
          "text": "Cloud Computing und künstliche Intelligenz revolutionieren moderne Geschäftsabläufe."
        }
      ]
    }
  }' | jq '.results.documents[] | {language: .id, keyPhrases: .keyPhrases}'
```

## 🚀 PowerShell Examples

### Basic Key Phrase Extraction with PowerShell

```powershell
# PowerShell script for key phrase extraction
$endpoint = $env:LANGUAGE_ENDPOINT
$key = $env:LANGUAGE_KEY

$headers = @{
    'Ocp-Apim-Subscription-Key' = $key
    'Content-Type' = 'application/json'
}

$body = @{
    kind = "KeyPhraseExtraction"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = @(
            @{
                id = "1"
                language = "en"
                text = "The company's digital transformation strategy focuses on cloud migration, data analytics, and automation technologies to improve operational efficiency and customer experience."
            }
        )
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Display key phrases
Write-Host "Key Phrases Found:"
$response.results.documents[0].keyPhrases | ForEach-Object {
    Write-Host "  • $_"
}
```

### Batch Processing with PowerShell

```powershell
# Process multiple business documents
$businessTexts = @(
    "Our quarterly results demonstrate strong performance in cloud services and artificial intelligence solutions.",
    "The new cybersecurity framework implementation will enhance data protection and regulatory compliance.",
    "Digital marketing campaigns leveraging social media analytics have increased customer engagement significantly.",
    "Supply chain optimization through predictive analytics has reduced operational costs and improved delivery times."
)

$documents = @()
for ($i = 0; $i -lt $businessTexts.Length; $i++) {
    $documents += @{
        id = "doc_$($i + 1)"
        language = "en"
        text = $businessTexts[$i]
    }
}

$body = @{
    kind = "KeyPhraseExtraction"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = $documents
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Analyze results
Write-Host "BUSINESS DOCUMENT KEY PHRASES ANALYSIS"
Write-Host "=" * 45

foreach ($doc in $response.results.documents) {
    Write-Host "`nDocument $($doc.id):"
    Write-Host "Key Phrases ($($doc.keyPhrases.Count) found):"
    $doc.keyPhrases | Sort-Object | ForEach-Object {
        Write-Host "  • $_"
    }
}

# Find common key phrases across documents
$allKeyPhrases = $response.results.documents | ForEach-Object { $_.keyPhrases } | Group-Object | Where-Object { $_.Count -gt 1 }
if ($allKeyPhrases) {
    Write-Host "`nCommon Key Phrases Across Documents:"
    $allKeyPhrases | ForEach-Object {
        Write-Host "  • $($_.Name) (appears in $($_.Count) documents)"
    }
}
```

### Industry-Specific Analysis

```powershell
# Analyze key phrases in healthcare documents
$healthcareTexts = @(
    "Patient care quality improvement through electronic health records and telemedicine solutions.",
    "Clinical trial data analysis using machine learning algorithms for drug discovery and development.",
    "Healthcare data privacy and HIPAA compliance in cloud-based medical information systems.",
    "Medical imaging technology advances in diagnostic accuracy and treatment planning."
)

$documents = @()
for ($i = 0; $i -lt $healthcareTexts.Length; $i++) {
    $documents += @{
        id = "healthcare_$($i + 1)"
        language = "en"
        text = $healthcareTexts[$i]
    }
}

$body = @{
    kind = "KeyPhraseExtraction"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = $documents
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

Write-Host "HEALTHCARE INDUSTRY KEY PHRASES"
Write-Host "=" * 35

# Create a hashtable to count phrase frequency
$phraseFrequency = @{}
foreach ($doc in $response.results.documents) {
    foreach ($phrase in $doc.keyPhrases) {
        if ($phraseFrequency.ContainsKey($phrase)) {
            $phraseFrequency[$phrase]++
        } else {
            $phraseFrequency[$phrase] = 1
        }
    }
}

# Display most frequent phrases
Write-Host "`nMost Frequent Healthcare Key Phrases:"
$phraseFrequency.GetEnumerator() | Sort-Object Value -Descending | ForEach-Object {
    Write-Host "  • $($_.Key) (frequency: $($_.Value))"
}
```

## 🐍 Python Requests Examples

### Basic Key Phrase Extraction with Python

```python
import requests
import json
import os
from collections import Counter

endpoint = os.getenv('LANGUAGE_ENDPOINT')
key = os.getenv('LANGUAGE_KEY')

def extract_key_phrases(text, language='en', document_id='1'):
    """Extract key phrases from a single document"""
    url = f"{endpoint}language/:analyze-text?api-version=2023-04-01"
    
    headers = {
        'Ocp-Apim-Subscription-Key': key,
        'Content-Type': 'application/json'
    }
    
    body = {
        "kind": "KeyPhraseExtraction",
        "parameters": {
            "modelVersion": "latest"
        },
        "analysisInput": {
            "documents": [
                {
                    "id": document_id,
                    "language": language,
                    "text": text
                }
            ]
        }
    }
    
    response = requests.post(url, headers=headers, json=body)
    
    if response.status_code == 200:
        result = response.json()
        return result['results']['documents'][0]['keyPhrases']
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return None

# Example usage
sample_text = """
Artificial intelligence and machine learning are transforming the healthcare industry. 
Medical professionals can now use predictive analytics to improve patient outcomes, 
reduce costs, and enhance diagnostic accuracy. Cloud computing platforms enable 
secure storage and analysis of large medical datasets.
"""

key_phrases = extract_key_phrases(sample_text)

if key_phrases:
    print("Key Phrases Extracted:")
    for i, phrase in enumerate(key_phrases, 1):
        print(f"{i:2d}. {phrase}")
else:
    print("No key phrases found or error occurred.")
```

### Advanced Batch Processing Pipeline

```python
import requests
import json
from typing import List, Dict, Optional
from dataclasses import dataclass

@dataclass
class KeyPhraseDocument:
    id: str
    text: str
    language: str = 'en'

@dataclass
class KeyPhraseResult:
    document_id: str
    key_phrases: List[str]
    has_error: bool = False
    error_message: Optional[str] = None

class KeyPhraseExtractor:
    def __init__(self, endpoint: str, key: str):
        self.endpoint = endpoint
        self.key = key
        self.headers = {
            'Ocp-Apim-Subscription-Key': key,
            'Content-Type': 'application/json'
        }
    
    def extract_batch(self, documents: List[KeyPhraseDocument]) -> List[KeyPhraseResult]:
        """Extract key phrases from multiple documents"""
        url = f"{self.endpoint}language/:analyze-text?api-version=2023-04-01"
        
        # Convert documents to API format
        api_documents = []
        for doc in documents:
            api_documents.append({
                "id": doc.id,
                "language": doc.language,
                "text": doc.text
            })
        
        body = {
            "kind": "KeyPhraseExtraction",
            "parameters": {
                "modelVersion": "latest"
            },
            "analysisInput": {
                "documents": api_documents
            }
        }
        
        response = requests.post(url, headers=self.headers, json=body)
        
        if response.status_code == 200:
            result = response.json()
            return self._process_results(result)
        else:
            raise Exception(f"API Error: {response.status_code} - {response.text}")
    
    def _process_results(self, api_result: Dict) -> List[KeyPhraseResult]:
        """Process API results into KeyPhraseResult objects"""
        results = []
        
        for doc in api_result['results']['documents']:
            if 'keyPhrases' in doc:
                results.append(KeyPhraseResult(
                    document_id=doc['id'],
                    key_phrases=doc['keyPhrases']
                ))
            else:
                # Handle error case
                error_msg = doc.get('error', {}).get('message', 'Unknown error')
                results.append(KeyPhraseResult(
                    document_id=doc['id'],
                    key_phrases=[],
                    has_error=True,
                    error_message=error_msg
                ))
        
        return results
    
    def analyze_content_themes(self, documents: List[KeyPhraseDocument]) -> Dict:
        """Analyze themes across multiple documents"""
        results = self.extract_batch(documents)
        
        # Collect all key phrases
        all_phrases = []
        successful_docs = 0
        
        for result in results:
            if not result.has_error:
                all_phrases.extend(result.key_phrases)
                successful_docs += 1
            else:
                print(f"Error in document {result.document_id}: {result.error_message}")
        
        # Analyze phrase frequency
        phrase_counter = Counter(all_phrases)
        
        # Find common themes (phrases appearing in multiple documents)
        phrase_distribution = {}
        for result in results:
            if not result.has_error:
                for phrase in result.key_phrases:
                    if phrase not in phrase_distribution:
                        phrase_distribution[phrase] = 0
                    phrase_distribution[phrase] += 1
        
        common_themes = {phrase: count for phrase, count in phrase_distribution.items() if count > 1}
        
        return {
            'total_documents': len(documents),
            'successful_analyses': successful_docs,
            'total_key_phrases': len(all_phrases),
            'unique_phrases': len(phrase_counter),
            'most_frequent_phrases': phrase_counter.most_common(10),
            'common_themes': sorted(common_themes.items(), key=lambda x: x[1], reverse=True),
            'phrase_frequency_distribution': dict(phrase_counter)
        }

# Example usage
extractor = KeyPhraseExtractor(endpoint, key)

# Create sample documents for analysis
sample_documents = [
    KeyPhraseDocument("tech_1", "Cloud computing enables scalable and flexible IT infrastructure for modern businesses."),
    KeyPhraseDocument("tech_2", "Artificial intelligence and machine learning drive innovation in data analysis and automation."),
    KeyPhraseDocument("tech_3", "Cybersecurity solutions protect digital assets and ensure data privacy compliance."),
    KeyPhraseDocument("tech_4", "Digital transformation initiatives leverage cloud platforms and AI technologies."),
    KeyPhraseDocument("tech_5", "Big data analytics provide insights for strategic business decision making.")
]

# Perform comprehensive analysis
analysis_results = extractor.analyze_content_themes(sample_documents)

print("KEY PHRASE ANALYSIS RESULTS")
print("=" * 40)
print(f"Total Documents: {analysis_results['total_documents']}")
print(f"Successful Analyses: {analysis_results['successful_analyses']}")
print(f"Total Key Phrases: {analysis_results['total_key_phrases']}")
print(f"Unique Phrases: {analysis_results['unique_phrases']}")

print("\nMost Frequent Key Phrases:")
for phrase, count in analysis_results['most_frequent_phrases']:
    print(f"  • {phrase}: {count}")

print("\nCommon Themes Across Documents:")
for phrase, doc_count in analysis_results['common_themes']:
    print(f"  • {phrase} (in {doc_count} documents)")
```

### Content Analysis Pipeline

```python
class ContentAnalyzer:
    def __init__(self, extractor: KeyPhraseExtractor):
        self.extractor = extractor
    
    def analyze_customer_feedback(self, feedback_texts: List[str]) -> Dict:
        """Analyze customer feedback for key insights"""
        documents = [
            KeyPhraseDocument(f"feedback_{i+1}", text) 
            for i, text in enumerate(feedback_texts)
        ]
        
        results = self.extractor.extract_batch(documents)
        
        # Categorize key phrases by type
        categories = {
            'product_features': [],
            'service_aspects': [],
            'technical_terms': [],
            'emotional_indicators': [],
            'business_outcomes': []
        }
        
        # Simple keyword-based categorization
        product_keywords = ['product', 'feature', 'functionality', 'design', 'quality', 'performance']
        service_keywords = ['service', 'support', 'help', 'assistance', 'response', 'team']
        technical_keywords = ['integration', 'API', 'system', 'platform', 'technology', 'software']
        emotional_keywords = ['satisfied', 'disappointed', 'excellent', 'terrible', 'amazing', 'frustrated']
        business_keywords = ['efficiency', 'productivity', 'cost', 'revenue', 'growth', 'ROI']
        
        for result in results:
            if not result.has_error:
                for phrase in result.key_phrases:
                    phrase_lower = phrase.lower()
                    
                    if any(keyword in phrase_lower for keyword in product_keywords):
                        categories['product_features'].append(phrase)
                    elif any(keyword in phrase_lower for keyword in service_keywords):
                        categories['service_aspects'].append(phrase)
                    elif any(keyword in phrase_lower for keyword in technical_keywords):
                        categories['technical_terms'].append(phrase)
                    elif any(keyword in phrase_lower for keyword in emotional_keywords):
                        categories['emotional_indicators'].append(phrase)
                    elif any(keyword in phrase_lower for keyword in business_keywords):
                        categories['business_outcomes'].append(phrase)
        
        # Remove duplicates and count occurrences
        for category in categories:
            categories[category] = Counter(categories[category])
        
        return {
            'feedback_count': len(feedback_texts),
            'successful_extractions': len([r for r in results if not r.has_error]),
            'categories': categories,
            'top_insights': {
                category: list(phrases.most_common(3))
                for category, phrases in categories.items()
                if phrases
            }
        }

# Example usage with customer feedback
analyzer = ContentAnalyzer(extractor)

customer_feedback = [
    "The product quality is excellent and the customer service team was very helpful during integration.",
    "We've seen significant efficiency improvements since implementing this platform in our workflow.",
    "The API documentation could be better, but the technical support response time is impressive.",
    "Cost reduction and productivity gains have exceeded our expectations for this software solution.",
    "User interface design is intuitive, though some advanced features require better training materials."
]

feedback_analysis = analyzer.analyze_customer_feedback(customer_feedback)

print("CUSTOMER FEEDBACK ANALYSIS")
print("=" * 30)
print(f"Feedback Count: {feedback_analysis['feedback_count']}")
print(f"Successful Extractions: {feedback_analysis['successful_extractions']}")

print("\nTop Insights by Category:")
for category, insights in feedback_analysis['top_insights'].items():
    if insights:
        print(f"\n{category.replace('_', ' ').title()}:")
        for phrase, count in insights:
            print(f"  • {phrase}: {count}")
```

## 📊 Analysis and Filtering Examples

### Key Phrase Frequency Analysis

```bash
# Create analysis script for phrase frequency
cat > analyze_key_phrases.sh << 'EOF'
#!/bin/bash

ENDPOINT=$LANGUAGE_ENDPOINT
KEY=$LANGUAGE_KEY

# Sample business articles
ARTICLES=(
    "Digital transformation initiatives are driving organizational change and technological innovation across industries."
    "Cloud computing platforms enable scalable infrastructure and data analytics for modern enterprises."
    "Artificial intelligence applications in healthcare improve diagnostic accuracy and patient care outcomes."
    "Cybersecurity frameworks protect sensitive data and ensure regulatory compliance in digital environments."
    "Remote work technologies facilitate collaboration and productivity in distributed team environments."
)

echo "KEY PHRASE FREQUENCY ANALYSIS"
echo "============================="

# Collect all key phrases
ALL_PHRASES_FILE="/tmp/all_phrases.txt"
> "$ALL_PHRASES_FILE"

for i in "${!ARTICLES[@]}"; do
    echo "Processing article $((i+1))..."
    
    curl -s -X POST "${ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
      -H "Ocp-Apim-Subscription-Key: ${KEY}" \
      -H "Content-Type: application/json" \
      -d "{
        \"kind\": \"KeyPhraseExtraction\",
        \"parameters\": {
          \"modelVersion\": \"latest\"
        },
        \"analysisInput\": {
          \"documents\": [
            {
              \"id\": \"$((i+1))\",
              \"language\": \"en\",
              \"text\": \"${ARTICLES[i]}\"
            }
          ]
        }
      }" | jq -r '.results.documents[0].keyPhrases[]' >> "$ALL_PHRASES_FILE"
done

echo -e "\nKey Phrase Frequency:"
echo "===================="
sort "$ALL_PHRASES_FILE" | uniq -c | sort -nr | head -10

echo -e "\nUnique Key Phrases Found:"
echo "========================"
sort "$ALL_PHRASES_FILE" | uniq | wc -l

rm "$ALL_PHRASES_FILE"
EOF

chmod +x analyze_key_phrases.sh
./analyze_key_phrases.sh
```

### Topic Clustering by Key Phrases

```bash
# Analyze technology topics
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "ai_ml",
          "language": "en",
          "text": "Machine learning algorithms and deep neural networks enable predictive analytics and automated decision making in business applications."
        },
        {
          "id": "cloud_computing",
          "language": "en",
          "text": "Cloud infrastructure services provide scalable computing resources, data storage solutions, and platform-as-a-service offerings."
        },
        {
          "id": "cybersecurity",
          "language": "en",
          "text": "Information security frameworks implement threat detection systems, encryption protocols, and access control mechanisms."
        }
      ]
    }
  }' | jq '.results.documents[] | {topic: .id, keyPhrases: .keyPhrases} | select(.keyPhrases | length > 0)'
```

## 🔍 Industry-Specific Examples

### Financial Services Key Phrases

```bash
# Extract key phrases from financial documents
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "financial_report",
          "language": "en",
          "text": "Quarterly earnings demonstrate strong performance in investment banking and wealth management sectors. Digital banking transformation initiatives continue to drive customer acquisition and operational efficiency improvements."
        },
        {
          "id": "risk_assessment",
          "language": "en",
          "text": "Credit risk analysis and regulatory compliance frameworks ensure portfolio diversification and capital adequacy requirements. Market volatility monitoring systems provide real-time risk assessment capabilities."
        }
      ]
    }
  }' | jq '.results.documents[] | {document: .id, financial_keyphrases: .keyPhrases}'
```

### Healthcare Domain Analysis

```bash
# Healthcare-specific key phrase extraction
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "clinical_research",
          "language": "en",
          "text": "Clinical trial data analysis reveals significant improvements in patient outcomes through personalized treatment protocols and precision medicine approaches."
        },
        {
          "id": "medical_technology",
          "language": "en",
          "text": "Medical imaging technology advances enable early disease detection and diagnostic accuracy improvements in radiology departments and surgical planning."
        }
      ]
    }
  }' | jq '.results.documents[] | {category: .id, medical_keyphrases: .keyPhrases}'
```

## 📚 Additional Resources

- [Key Phrase Extraction API Reference](https://docs.microsoft.com/rest/api/language/text-analysis-runtime/analyze-text)
- [Language Support for Key Phrase Extraction](https://docs.microsoft.com/azure/cognitive-services/language-service/key-phrase-extraction/language-support)
- [Best Practices for Text Analysis](https://docs.microsoft.com/azure/cognitive-services/language-service/concepts/best-practices)