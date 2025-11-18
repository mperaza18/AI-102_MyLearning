# 🔷 Language Detection REST API Examples

> **REST API examples for Language Detection using Azure AI Language services**

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

### Simple Language Detection

```bash
# Basic language detection
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "text": "Hello, how are you today? I hope you are having a wonderful day!"
        }
      ]
    }
  }' | jq '.results.documents[0] | {id: .id, detectedLanguage: .detectedLanguage}'
```

### Language Detection with Confidence Scores

```bash
# Detect language with detailed confidence scores
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "text": "Bonjour! Comment allez-vous aujourd'hui? J'espère que vous passez une excellente journée!"
        }
      ]
    }
  }' | jq '.results.documents[0] | {id: .id, language: .detectedLanguage.name, iso6391Name: .detectedLanguage.iso6391Name, confidenceScore: .detectedLanguage.confidenceScore}'
```

## 🔧 Advanced Examples

### Batch Language Detection

```bash
# Detect languages for multiple texts
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "english",
          "text": "The quick brown fox jumps over the lazy dog. This is a sample English text."
        },
        {
          "id": "spanish",
          "text": "El zorro marrón rápido salta sobre el perro perezoso. Este es un texto de muestra en español."
        },
        {
          "id": "french",
          "text": "Le renard brun rapide saute par-dessus le chien paresseux. Ceci est un exemple de texte français."
        },
        {
          "id": "german",
          "text": "Der schnelle braune Fuchs springt über den faulen Hund. Dies ist ein deutscher Beispieltext."
        },
        {
          "id": "chinese",
          "text": "快速的棕色狐狸跳过懒惰的狗。这是一个中文示例文本。"
        },
        {
          "id": "japanese",
          "text": "速い茶色のキツネが怠け者の犬を飛び越えます。これは日本語のサンプルテキストです。"
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, language: .detectedLanguage.name, code: .detectedLanguage.iso6391Name, confidence: .detectedLanguage.confidenceScore}'
```

### Mixed Language Content Analysis

```bash
# Analyze mixed language content (code-switching)
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "mixed1",
          "text": "Hello, comment ça va? I am learning français."
        },
        {
          "id": "mixed2", 
          "text": "Me gusta programming in Python y JavaScript."
        },
        {
          "id": "mixed3",
          "text": "这个 application 很好用，I really like it!"
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, primaryLanguage: .detectedLanguage.name, confidence: .detectedLanguage.confidenceScore, warnings: .warnings}'
```

## 🚀 PowerShell Examples

### Basic Language Detection with PowerShell

```powershell
# PowerShell script for language detection
$endpoint = $env:LANGUAGE_ENDPOINT
$key = $env:LANGUAGE_KEY

$headers = @{
    'Ocp-Apim-Subscription-Key' = $key
    'Content-Type' = 'application/json'
}

$body = @{
    kind = "LanguageDetection"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = @(
            @{
                id = "1"
                text = "Hola, ¿cómo estás hoy? Espero que tengas un día maravilloso."
            }
        )
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Display language detection result
$result = $response.results.documents[0]
Write-Host "Language Detection Result:"
Write-Host "  Language: $($result.detectedLanguage.name)"
Write-Host "  ISO Code: $($result.detectedLanguage.iso6391Name)"
Write-Host "  Confidence Score: $($result.detectedLanguage.confidenceScore)"
```

### Multi-Language Document Processing

```powershell
# Process customer support tickets in multiple languages
$supportTickets = @(
    "I'm having trouble with my account login. Can you please help me?",
    "J'ai des problèmes avec la connexion à mon compte. Pouvez-vous m'aider?",
    "Tengo problemas para iniciar sesión en mi cuenta. ¿Pueden ayudarme?",
    "Ich habe Probleme bei der Anmeldung in meinem Konto. Können Sie mir helfen?",
    "Sto avendo problemi con l'accesso al mio account. Potete aiutarmi?",
    "我在登录我的账户时遇到了问题。你们能帮我吗？"
)

$documents = @()
for ($i = 0; $i -lt $supportTickets.Length; $i++) {
    $documents += @{
        id = "ticket_$($i + 1)"
        text = $supportTickets[$i]
    }
}

$body = @{
    kind = "LanguageDetection"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = $documents
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

# Analyze and categorize by language
Write-Host "CUSTOMER SUPPORT TICKET LANGUAGE ANALYSIS"
Write-Host "=" * 45

$languageGroups = $response.results.documents | Group-Object { $_.detectedLanguage.name }

Write-Host "`nLanguage Distribution:"
foreach ($group in $languageGroups | Sort-Object Count -Descending) {
    Write-Host "  $($group.Name): $($group.Count) tickets"
}

Write-Host "`nDetailed Results:"
foreach ($doc in $response.results.documents) {
    Write-Host "`nTicket $($doc.id):"
    Write-Host "  Language: $($doc.detectedLanguage.name)"
    Write-Host "  Confidence: $($doc.detectedLanguage.confidenceScore)"
    Write-Host "  Text: $($supportTickets[[int]($doc.id.Split('_')[1]) - 1].Substring(0, [Math]::Min(50, $supportTickets[[int]($doc.id.Split('_')[1]) - 1].Length)))..."
}
```

### Content Classification by Language

```powershell
# Classify social media content by language
$socialMediaPosts = @(
    "Just had the best coffee at this new café downtown! ☕ #coffee #downtown",
    "Acabamos de probar el mejor café de la ciudad. ¡Increíble! ☕ #café #ciudad",
    "Je viens de boire le meilleur café du centre-ville! ☕ #café #centerville",
    "Gerade den besten Kaffee in der Innenstadt getrunken! ☕ #Kaffee #Innenstadt",
    "今日在市中心喝到了最好的咖啡！☕ #咖啡 #市中心",
    "街の中心で最高のコーヒーを飲みました！☕ #コーヒー #ダウンタウン"
)

$documents = @()
for ($i = 0; $i -lt $socialMediaPosts.Length; $i++) {
    $documents += @{
        id = "post_$($i + 1)"
        text = $socialMediaPosts[$i]
    }
}

$body = @{
    kind = "LanguageDetection"
    parameters = @{
        modelVersion = "latest"
    }
    analysisInput = @{
        documents = $documents
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod -Uri "${endpoint}language/:analyze-text?api-version=2023-04-01" -Method Post -Headers $headers -Body $body

Write-Host "SOCIAL MEDIA LANGUAGE ANALYSIS"
Write-Host "=" * 32

# Group by language and show confidence stats
$languageStats = @{}
foreach ($doc in $response.results.documents) {
    $lang = $doc.detectedLanguage.name
    $confidence = $doc.detectedLanguage.confidenceScore
    
    if (-not $languageStats.ContainsKey($lang)) {
        $languageStats[$lang] = @{
            Count = 0
            TotalConfidence = 0
            MinConfidence = 1
            MaxConfidence = 0
        }
    }
    
    $languageStats[$lang].Count++
    $languageStats[$lang].TotalConfidence += $confidence
    $languageStats[$lang].MinConfidence = [Math]::Min($languageStats[$lang].MinConfidence, $confidence)
    $languageStats[$lang].MaxConfidence = [Math]::Max($languageStats[$lang].MaxConfidence, $confidence)
}

Write-Host "`nLanguage Statistics:"
foreach ($lang in $languageStats.Keys | Sort-Object) {
    $stats = $languageStats[$lang]
    $avgConfidence = $stats.TotalConfidence / $stats.Count
    
    Write-Host "`n  $lang ($($stats.Count) posts):"
    Write-Host "    Average Confidence: $($avgConfidence.ToString('F3'))"
    Write-Host "    Confidence Range: $($stats.MinConfidence.ToString('F3')) - $($stats.MaxConfidence.ToString('F3'))"
}
```

## 🐍 Python Requests Examples

### Basic Language Detection with Python

```python
import requests
import json
import os
from collections import Counter, defaultdict

endpoint = os.getenv('LANGUAGE_ENDPOINT')
key = os.getenv('LANGUAGE_KEY')

def detect_language(text, document_id='1'):
    """Detect language for a single text"""
    url = f"{endpoint}language/:analyze-text?api-version=2023-04-01"
    
    headers = {
        'Ocp-Apim-Subscription-Key': key,
        'Content-Type': 'application/json'
    }
    
    body = {
        "kind": "LanguageDetection",
        "parameters": {
            "modelVersion": "latest"
        },
        "analysisInput": {
            "documents": [
                {
                    "id": document_id,
                    "text": text
                }
            ]
        }
    }
    
    response = requests.post(url, headers=headers, json=body)
    
    if response.status_code == 200:
        result = response.json()
        doc_result = result['results']['documents'][0]
        return {
            'language_name': doc_result['detectedLanguage']['name'],
            'iso_code': doc_result['detectedLanguage']['iso6391Name'],
            'confidence_score': doc_result['detectedLanguage']['confidenceScore']
        }
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return None

# Example usage
sample_texts = [
    "Hello world! How are you doing today?",
    "Bonjour le monde! Comment allez-vous aujourd'hui?",
    "¡Hola mundo! ¿Cómo estás hoy?",
    "Hallo Welt! Wie geht es dir heute?",
    "你好世界！你今天好吗？"
]

print("LANGUAGE DETECTION RESULTS")
print("=" * 30)

for i, text in enumerate(sample_texts, 1):
    result = detect_language(text, f"sample_{i}")
    if result:
        print(f"\nText {i}: '{text[:40]}...'")
        print(f"  Language: {result['language_name']}")
        print(f"  ISO Code: {result['iso_code']}")
        print(f"  Confidence: {result['confidence_score']:.3f}")
```

### Advanced Batch Processing with Analysis

```python
import requests
import json
from typing import List, Dict, Optional
from dataclasses import dataclass

@dataclass
class LanguageDetectionResult:
    document_id: str
    text: str
    language_name: str
    iso_code: str
    confidence_score: float
    has_error: bool = False
    error_message: Optional[str] = None

class LanguageDetectionAnalyzer:
    def __init__(self, endpoint: str, key: str):
        self.endpoint = endpoint
        self.key = key
        self.headers = {
            'Ocp-Apim-Subscription-Key': key,
            'Content-Type': 'application/json'
        }
    
    def detect_languages_batch(self, texts: List[str], doc_ids: List[str] = None) -> List[LanguageDetectionResult]:
        """Detect languages for multiple texts"""
        url = f"{self.endpoint}language/:analyze-text?api-version=2023-04-01"
        
        # Create document objects
        documents = []
        for i, text in enumerate(texts):
            doc_id = doc_ids[i] if doc_ids else f"doc_{i+1}"
            documents.append({
                "id": doc_id,
                "text": text
            })
        
        body = {
            "kind": "LanguageDetection",
            "parameters": {
                "modelVersion": "latest"
            },
            "analysisInput": {
                "documents": documents
            }
        }
        
        response = requests.post(url, headers=self.headers, json=body)
        
        if response.status_code == 200:
            result = response.json()
            return self._process_results(result, texts)
        else:
            raise Exception(f"API Error: {response.status_code} - {response.text}")
    
    def _process_results(self, api_result: Dict, original_texts: List[str]) -> List[LanguageDetectionResult]:
        """Process API results into LanguageDetectionResult objects"""
        results = []
        
        for i, doc in enumerate(api_result['results']['documents']):
            text = original_texts[i]
            
            if 'detectedLanguage' in doc:
                results.append(LanguageDetectionResult(
                    document_id=doc['id'],
                    text=text,
                    language_name=doc['detectedLanguage']['name'],
                    iso_code=doc['detectedLanguage']['iso6391Name'],
                    confidence_score=doc['detectedLanguage']['confidenceScore']
                ))
            else:
                # Handle error case
                error_msg = doc.get('error', {}).get('message', 'Unknown error')
                results.append(LanguageDetectionResult(
                    document_id=doc['id'],
                    text=text,
                    language_name='',
                    iso_code='',
                    confidence_score=0.0,
                    has_error=True,
                    error_message=error_msg
                ))
        
        return results
    
    def analyze_language_distribution(self, texts: List[str]) -> Dict:
        """Comprehensive analysis of language distribution"""
        results = self.detect_languages_batch(texts)
        
        # Language statistics
        language_counts = Counter()
        confidence_stats = defaultdict(list)
        successful_detections = []
        
        for result in results:
            if not result.has_error:
                language_counts[result.language_name] += 1
                confidence_stats[result.language_name].append(result.confidence_score)
                successful_detections.append(result)
            else:
                print(f"Error in {result.document_id}: {result.error_message}")
        
        # Calculate confidence statistics
        language_confidence_stats = {}
        for lang, confidences in confidence_stats.items():
            language_confidence_stats[lang] = {
                'min': min(confidences),
                'max': max(confidences),
                'avg': sum(confidences) / len(confidences),
                'count': len(confidences)
            }
        
        return {
            'total_documents': len(texts),
            'successful_detections': len(successful_detections),
            'language_distribution': dict(language_counts),
            'confidence_statistics': language_confidence_stats,
            'results': successful_detections
        }

# Example usage
analyzer = LanguageDetectionAnalyzer(endpoint, key)

# Multilingual customer reviews
customer_reviews = [
    "This product is amazing! Great quality and fast shipping.",
    "Ce produit est incroyable! Excellente qualité et livraison rapide.",
    "Este producto es increíble! Gran calidad y envío rápido.",
    "Dieses Produkt ist fantastisch! Tolle Qualität und schneller Versand.",
    "这个产品太棒了！质量很好，运输很快。",
    "この製品は素晴らしいです！品質が良く、発送が早いです。",
    "Questo prodotto è fantastico! Ottima qualità e spedizione veloce.",
    "Dit product is geweldig! Geweldige kwaliteit en snelle verzending.",
    "Этот продукт потрясающий! Отличное качество и быстрая доставка.",
    "이 제품은 놀랍습니다! 훌륭한 품질과 빠른 배송입니다."
]

# Perform comprehensive analysis
analysis_results = analyzer.analyze_language_distribution(customer_reviews)

print("MULTILINGUAL CUSTOMER REVIEW ANALYSIS")
print("=" * 45)
print(f"Total Reviews: {analysis_results['total_documents']}")
print(f"Successful Detections: {analysis_results['successful_detections']}")

print("\nLanguage Distribution:")
for lang, count in sorted(analysis_results['language_distribution'].items()):
    percentage = (count / analysis_results['successful_detections']) * 100
    print(f"  {lang}: {count} reviews ({percentage:.1f}%)")

print("\nConfidence Statistics by Language:")
for lang, stats in analysis_results['confidence_statistics'].items():
    print(f"\n  {lang}:")
    print(f"    Average Confidence: {stats['avg']:.3f}")
    print(f"    Range: {stats['min']:.3f} - {stats['max']:.3f}")
    print(f"    Sample Count: {stats['count']}")
```

### Content Routing by Language

```python
class ContentRouter:
    def __init__(self, analyzer: LanguageDetectionAnalyzer):
        self.analyzer = analyzer
        
    def route_customer_support_tickets(self, tickets: List[str]) -> Dict:
        """Route customer support tickets based on detected language"""
        results = self.analyzer.detect_languages_batch(tickets)
        
        # Define routing rules
        routing_rules = {
            'English': 'support-team-us',
            'French': 'support-team-fr', 
            'Spanish': 'support-team-es',
            'German': 'support-team-de',
            'Chinese (Simplified)': 'support-team-cn',
            'Japanese': 'support-team-jp',
            'Italian': 'support-team-it',
            'Portuguese': 'support-team-pt'
        }
        
        routed_tickets = defaultdict(list)
        unrouted_tickets = []
        
        for result in results:
            if not result.has_error:
                # Check confidence threshold
                if result.confidence_score >= 0.7:
                    team = routing_rules.get(result.language_name, 'support-team-multilingual')
                    routed_tickets[team].append({
                        'ticket_id': result.document_id,
                        'language': result.language_name,
                        'confidence': result.confidence_score,
                        'content': result.text[:100] + '...' if len(result.text) > 100 else result.text
                    })
                else:
                    # Low confidence - route to multilingual team
                    unrouted_tickets.append({
                        'ticket_id': result.document_id,
                        'detected_language': result.language_name,
                        'confidence': result.confidence_score,
                        'reason': 'Low confidence detection',
                        'content': result.text[:100] + '...' if len(result.text) > 100 else result.text
                    })
            else:
                unrouted_tickets.append({
                    'ticket_id': result.document_id,
                    'error': result.error_message,
                    'reason': 'Detection failed',
                    'content': result.text[:100] + '...' if len(result.text) > 100 else result.text
                })
        
        return {
            'routed_tickets': dict(routed_tickets),
            'unrouted_tickets': unrouted_tickets,
            'routing_summary': {
                team: len(tickets) for team, tickets in routed_tickets.items()
            }
        }
    
    def print_routing_report(self, routing_results: Dict):
        """Print customer support routing report"""
        print("CUSTOMER SUPPORT TICKET ROUTING REPORT")
        print("=" * 45)
        
        print("\nRouting Summary:")
        total_routed = sum(routing_results['routing_summary'].values())
        for team, count in routing_results['routing_summary'].items():
            print(f"  {team}: {count} tickets")
        
        print(f"\nTotal Routed: {total_routed}")
        print(f"Unrouted: {len(routing_results['unrouted_tickets'])}")
        
        print("\nDetailed Routing:")
        for team, tickets in routing_results['routed_tickets'].items():
            print(f"\n  {team.upper()}:")
            for ticket in tickets[:3]:  # Show first 3 tickets
                print(f"    Ticket {ticket['ticket_id']}: {ticket['language']} (confidence: {ticket['confidence']:.3f})")
                print(f"      Content: {ticket['content']}")
        
        if routing_results['unrouted_tickets']:
            print("\n  UNROUTED TICKETS:")
            for ticket in routing_results['unrouted_tickets'][:3]:  # Show first 3
                print(f"    Ticket {ticket['ticket_id']}: {ticket['reason']}")
                if 'detected_language' in ticket:
                    print(f"      Detected: {ticket['detected_language']} (confidence: {ticket['confidence']:.3f})")

# Example usage
router = ContentRouter(analyzer)

support_tickets = [
    "I'm having trouble logging into my account. Can you help me reset my password?",
    "Je ne peux pas me connecter à mon compte. Pouvez-vous m'aider à réinitialiser mon mot de passe?",
    "No puedo iniciar sesión en mi cuenta. ¿Pueden ayudarme a restablecer mi contraseña?",
    "Ich kann mich nicht in mein Konto einloggen. Können Sie mir beim Zurücksetzen meines Passworts helfen?",
    "我无法登录我的账户。您能帮我重置密码吗？",
    "アカウントにログインできません。パスワードをリセットしてもらえますか？",
    "Non riesco ad accedere al mio account. Potete aiutarmi a reimpostare la mia password?",
    "Mixed language text: Hello, je ne comprends pas this error message.",  # Low confidence expected
]

ticket_ids = [f"TICKET-{1000+i}" for i in range(len(support_tickets))]
routing_results = router.route_customer_support_tickets(support_tickets)
router.print_routing_report(routing_results)
```

## 📊 Analysis and Filtering Examples

### Language Confidence Analysis

```bash
# Analyze confidence scores across different languages
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "clear_english",
          "text": "The artificial intelligence revolution is transforming businesses across all industries with unprecedented speed and efficiency."
        },
        {
          "id": "clear_spanish", 
          "text": "La revolución de la inteligencia artificial está transformando las empresas de todas las industrias con una velocidad y eficiencia sin precedentes."
        },
        {
          "id": "mixed_languages",
          "text": "Hello, ¿cómo estás? I am apprendre français et español."
        },
        {
          "id": "short_text",
          "text": "Bonjour!"
        },
        {
          "id": "numbers_symbols",
          "text": "123-456-7890 | user@domain.com | #hashtag @mention"
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, language: .detectedLanguage.name, confidence: .detectedLanguage.confidenceScore, category: (if .detectedLanguage.confidenceScore > 0.9 then "high" elif .detectedLanguage.confidenceScore > 0.7 then "medium" else "low" end)}'
```

### Language Pattern Analysis

```bash
# Create analysis script for language patterns
cat > analyze_language_patterns.sh << 'EOF'
#!/bin/bash

ENDPOINT=$LANGUAGE_ENDPOINT
KEY=$LANGUAGE_KEY

# Different types of content to analyze patterns
declare -a CONTENT_TYPES=(
    "formal_business:We are pleased to inform you that your application has been approved and processed successfully."
    "informal_chat:hey whats up? how r u doing today lol 😊"
    "technical_docs:The REST API endpoint accepts HTTP POST requests with JSON payload containing authentication headers."
    "social_media:Just had the best #coffee ever! Thanks @cafe_downtown for the amazing experience ☕️✨"
    "news_headline:Global technology companies report record quarterly earnings amid digital transformation surge."
)

echo "LANGUAGE DETECTION PATTERN ANALYSIS"
echo "=================================="

for content in "${CONTENT_TYPES[@]}"; do
    IFS=':' read -r type text <<< "$content"
    
    echo -e "\nAnalyzing $type content:"
    echo "Text: $text"
    
    result=$(curl -s -X POST "${ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
      -H "Ocp-Apim-Subscription-Key: ${KEY}" \
      -H "Content-Type: application/json" \
      -d "{
        \"kind\": \"LanguageDetection\",
        \"parameters\": {
          \"modelVersion\": \"latest\"
        },
        \"analysisInput\": {
          \"documents\": [
            {
              \"id\": \"$type\",
              \"text\": \"$text\"
            }
          ]
        }
      }" | jq -r '.results.documents[0] | "Language: \(.detectedLanguage.name) | Confidence: \(.detectedLanguage.confidenceScore)"')
    
    echo "Result: $result"
done
EOF

chmod +x analyze_language_patterns.sh
./analyze_language_patterns.sh
```

## 🔍 Specialized Use Cases

### Document Classification Pipeline

```bash
# Classify documents by language for processing pipeline
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "legal_en",
          "text": "This agreement shall be governed by and construed in accordance with the laws of the jurisdiction."
        },
        {
          "id": "legal_fr",
          "text": "Cet accord sera régi et interprété conformément aux lois de la juridiction compétente."
        },
        {
          "id": "medical_en",
          "text": "Patient presents with symptoms consistent with viral upper respiratory tract infection."
        },
        {
          "id": "medical_de",
          "text": "Patient zeigt Symptome, die mit einer viralen Infektion der oberen Atemwege vereinbar sind."
        }
      ]
    }
  }' | jq '.results.documents[] | {document_type: .id, language: .detectedLanguage.name, iso_code: .detectedLanguage.iso6391Name, confidence: .detectedLanguage.confidenceScore, processing_queue: (.id | split("_")[0])}'
```

### Multilingual Content Validation

```bash
# Validate expected vs detected languages
curl -X POST "${LANGUAGE_ENDPOINT}language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: ${LANGUAGE_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "LanguageDetection",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "expected_en",
          "text": "Welcome to our international customer service portal."
        },
        {
          "id": "expected_es",
          "text": "Bienvenido a nuestro portal de servicio al cliente internacional."
        },
        {
          "id": "expected_fr_but_mixed",
          "text": "Bienvenue to our service portal international."
        }
      ]
    }
  }' | jq '.results.documents[] | {id: .id, expected_lang: (.id | split("_")[1]), detected_lang: .detectedLanguage.iso6391Name, confidence: .detectedLanguage.confidenceScore, match: ((.id | split("_")[1]) == .detectedLanguage.iso6391Name)}'
```

## 📚 Additional Resources

- [Language Detection API Reference](https://docs.microsoft.com/rest/api/language/text-analysis-runtime/analyze-text)
- [Supported Languages for Detection](https://docs.microsoft.com/azure/cognitive-services/language-service/language-detection/language-support)
- [Language Detection Best Practices](https://docs.microsoft.com/azure/cognitive-services/language-service/language-detection/overview)