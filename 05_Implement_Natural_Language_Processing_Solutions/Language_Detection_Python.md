# 🔷 Language Detection Python SDK Examples

> **Python SDK examples for Language Detection using Azure AI Language services**

## 📦 Installation and Setup

```bash
# Install required packages
pip install azure-ai-textanalytics azure-identity pandas numpy
```

## 🛠️ Authentication and Client Setup

### Using API Key Authentication

```python
import os
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# Environment setup
endpoint = os.getenv('LANGUAGE_ENDPOINT')
key = os.getenv('LANGUAGE_KEY')

# Create client with API key
credential = AzureKeyCredential(key)
text_analytics_client = TextAnalyticsClient(
    endpoint=endpoint, 
    credential=credential
)

print("✅ Text Analytics Client initialized successfully")
```

### Using Azure Active Directory Authentication

```python
from azure.identity import DefaultAzureCredential, ClientSecretCredential
from azure.ai.textanalytics import TextAnalyticsClient

# Using DefaultAzureCredential (recommended for production)
credential = DefaultAzureCredential()
text_analytics_client = TextAnalyticsClient(
    endpoint=endpoint,
    credential=credential
)

# Alternative: Using Client Secret Credential
# credential = ClientSecretCredential(
#     tenant_id="your-tenant-id",
#     client_id="your-client-id", 
#     client_secret="your-client-secret"
# )
```

## 🔧 Basic Examples

### Simple Language Detection

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

def simple_language_detection():
    """Basic language detection example"""
    
    # Sample texts in different languages
    documents = [
        "Hello, how are you today? I hope you're having a wonderful day!",
        "Bonjour, comment allez-vous aujourd'hui? J'espère que vous passez une excellente journée!",
        "Hola, ¿cómo estás hoy? ¡Espero que tengas un día maravilloso!",
        "Hallo, wie geht es dir heute? Ich hoffe, du hast einen wunderbaren Tag!",
        "你好，你今天怎么样？我希望你度过美好的一天！"
    ]
    
    # Detect languages
    try:
        response = text_analytics_client.detect_language(documents=documents)
        
        print("BASIC LANGUAGE DETECTION RESULTS")
        print("=" * 40)
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                print(f"\nDocument {idx + 1}:")
                print(f"  Text: '{documents[idx][:50]}...'")
                print(f"  Detected Language: {doc.primary_language.name}")
                print(f"  ISO 639-1 Code: {doc.primary_language.iso6391_name}")
                print(f"  Confidence Score: {doc.primary_language.confidence_score:.3f}")
            else:
                print(f"\nDocument {idx + 1}: Error - {doc.error}")
                
    except Exception as e:
        print(f"Error occurred: {e}")

# Run the example
simple_language_detection()
```

### Language Detection with Detailed Analysis

```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class LanguageResult:
    text: str
    language_name: str
    iso_code: str
    confidence_score: float
    is_high_confidence: bool
    warnings: List[str] = None

def detailed_language_detection(texts: List[str]) -> List[LanguageResult]:
    """Perform language detection with detailed analysis"""
    
    try:
        response = text_analytics_client.detect_language(documents=texts)
        results = []
        
        for idx, doc in enumerate(response):
            if not doc.is_error:
                # Determine confidence level
                is_high_confidence = doc.primary_language.confidence_score >= 0.8
                
                # Collect warnings
                warnings = []
                if doc.primary_language.confidence_score < 0.5:
                    warnings.append("Low confidence detection")
                if len(texts[idx]) < 20:
                    warnings.append("Short text - detection may be less reliable")
                
                result = LanguageResult(
                    text=texts[idx],
                    language_name=doc.primary_language.name,
                    iso_code=doc.primary_language.iso6391_name,
                    confidence_score=doc.primary_language.confidence_score,
                    is_high_confidence=is_high_confidence,
                    warnings=warnings
                )
                results.append(result)
            else:
                # Handle error case
                result = LanguageResult(
                    text=texts[idx],
                    language_name="Error",
                    iso_code="",
                    confidence_score=0.0,
                    is_high_confidence=False,
                    warnings=[f"Detection error: {doc.error}"]
                )
                results.append(result)
        
        return results
        
    except Exception as e:
        print(f"Batch detection error: {e}")
        return []

# Example usage
test_texts = [
    "The quick brown fox jumps over the lazy dog.",  # Clear English
    "Le renard brun rapide saute par-dessus le chien paresseux.",  # Clear French
    "Hello, ¿cómo estás? I am learning español.",  # Mixed languages
    "Hi there!",  # Short text
    "𝓣𝓱𝓲𝓼 𝓲𝓼 𝓪 𝓽𝓮𝓼𝓽 𝔀𝓲𝓽𝓱 𝓼𝓹𝓮𝓬𝓲𝓪𝓵 𝓬𝓱𝓪𝓻𝓪𝓬𝓽𝓮𝓻𝓼.",  # Special characters
    "123-456-7890"  # Numbers only
]

results = detailed_language_detection(test_texts)

print("DETAILED LANGUAGE DETECTION ANALYSIS")
print("=" * 45)

for idx, result in enumerate(results, 1):
    print(f"\n📄 Text {idx}:")
    print(f"   Content: '{result.text[:60]}{'...' if len(result.text) > 60 else ''}'")
    print(f"   Language: {result.language_name} ({result.iso_code})")
    print(f"   Confidence: {result.confidence_score:.3f} {'✅' if result.is_high_confidence else '⚠️'}")
    
    if result.warnings:
        print(f"   Warnings: {', '.join(result.warnings)}")
```

## 🚀 Advanced Examples

### Batch Language Detection with Statistics

```python
import pandas as pd
from collections import Counter, defaultdict
from typing import Dict, Any

class LanguageDetectionAnalyzer:
    def __init__(self, client: TextAnalyticsClient):
        self.client = client
        
    def analyze_batch(self, texts: List[str], batch_size: int = 10) -> Dict[str, Any]:
        """Process large batches of text with comprehensive analysis"""
        
        all_results = []
        total_batches = (len(texts) + batch_size - 1) // batch_size
        
        print(f"Processing {len(texts)} documents in {total_batches} batches...")
        
        # Process in batches
        for i in range(0, len(texts), batch_size):
            batch = texts[i:i + batch_size]
            batch_number = i // batch_size + 1
            
            print(f"  Processing batch {batch_number}/{total_batches}")
            
            try:
                response = self.client.detect_language(documents=batch)
                
                for idx, doc in enumerate(response):
                    original_idx = i + idx
                    
                    if not doc.is_error:
                        result = {
                            'document_id': original_idx,
                            'text': texts[original_idx],
                            'text_length': len(texts[original_idx]),
                            'language_name': doc.primary_language.name,
                            'iso_code': doc.primary_language.iso6391_name,
                            'confidence_score': doc.primary_language.confidence_score,
                            'has_error': False
                        }
                    else:
                        result = {
                            'document_id': original_idx,
                            'text': texts[original_idx],
                            'text_length': len(texts[original_idx]),
                            'language_name': None,
                            'iso_code': None,
                            'confidence_score': 0.0,
                            'has_error': True,
                            'error_message': str(doc.error)
                        }
                    
                    all_results.append(result)
                    
            except Exception as e:
                print(f"  Error in batch {batch_number}: {e}")
                # Add error results for this batch
                for idx in range(len(batch)):
                    original_idx = i + idx
                    all_results.append({
                        'document_id': original_idx,
                        'text': texts[original_idx],
                        'text_length': len(texts[original_idx]),
                        'language_name': None,
                        'iso_code': None,
                        'confidence_score': 0.0,
                        'has_error': True,
                        'error_message': str(e)
                    })
        
        return self._generate_statistics(all_results)
    
    def _generate_statistics(self, results: List[Dict]) -> Dict[str, Any]:
        """Generate comprehensive statistics from results"""
        
        # Filter successful results
        successful_results = [r for r in results if not r['has_error']]
        
        # Basic statistics
        stats = {
            'total_documents': len(results),
            'successful_detections': len(successful_results),
            'failed_detections': len(results) - len(successful_results),
            'success_rate': len(successful_results) / len(results) if results else 0
        }
        
        if not successful_results:
            stats['detailed_analysis'] = "No successful detections to analyze"
            return stats
        
        # Language distribution
        language_counts = Counter(r['language_name'] for r in successful_results)
        stats['language_distribution'] = dict(language_counts.most_common())
        
        # Confidence statistics
        confidence_scores = [r['confidence_score'] for r in successful_results]
        stats['confidence_stats'] = {
            'mean': sum(confidence_scores) / len(confidence_scores),
            'min': min(confidence_scores),
            'max': max(confidence_scores),
            'high_confidence_count': sum(1 for score in confidence_scores if score >= 0.8),
            'medium_confidence_count': sum(1 for score in confidence_scores if 0.5 <= score < 0.8),
            'low_confidence_count': sum(1 for score in confidence_scores if score < 0.5)
        }
        
        # Text length analysis
        text_lengths = [r['text_length'] for r in successful_results]
        stats['text_length_stats'] = {
            'mean': sum(text_lengths) / len(text_lengths),
            'min': min(text_lengths),
            'max': max(text_lengths)
        }
        
        # Language-specific confidence analysis
        lang_confidence = defaultdict(list)
        for r in successful_results:
            lang_confidence[r['language_name']].append(r['confidence_score'])
        
        stats['language_confidence_analysis'] = {}
        for lang, confidences in lang_confidence.items():
            stats['language_confidence_analysis'][lang] = {
                'count': len(confidences),
                'avg_confidence': sum(confidences) / len(confidences),
                'min_confidence': min(confidences),
                'max_confidence': max(confidences)
            }
        
        # Create DataFrame for detailed analysis
        df = pd.DataFrame(successful_results)
        stats['dataframe'] = df
        
        return stats
    
    def print_analysis_report(self, stats: Dict[str, Any]):
        """Print comprehensive analysis report"""
        
        print("\n" + "="*50)
        print("LANGUAGE DETECTION BATCH ANALYSIS REPORT")
        print("="*50)
        
        print(f"\n📊 OVERVIEW:")
        print(f"   Total Documents: {stats['total_documents']}")
        print(f"   Successful Detections: {stats['successful_detections']}")
        print(f"   Failed Detections: {stats['failed_detections']}")
        print(f"   Success Rate: {stats['success_rate']:.2%}")
        
        if stats['successful_detections'] == 0:
            return
        
        print(f"\n🌍 LANGUAGE DISTRIBUTION:")
        for lang, count in list(stats['language_distribution'].items())[:10]:  # Top 10
            percentage = (count / stats['successful_detections']) * 100
            print(f"   {lang}: {count} documents ({percentage:.1f}%)")
        
        print(f"\n📈 CONFIDENCE STATISTICS:")
        conf_stats = stats['confidence_stats']
        print(f"   Average Confidence: {conf_stats['mean']:.3f}")
        print(f"   Confidence Range: {conf_stats['min']:.3f} - {conf_stats['max']:.3f}")
        print(f"   High Confidence (≥0.8): {conf_stats['high_confidence_count']} documents")
        print(f"   Medium Confidence (0.5-0.8): {conf_stats['medium_confidence_count']} documents")
        print(f"   Low Confidence (<0.5): {conf_stats['low_confidence_count']} documents")
        
        print(f"\n📝 TEXT LENGTH STATISTICS:")
        length_stats = stats['text_length_stats']
        print(f"   Average Length: {length_stats['mean']:.0f} characters")
        print(f"   Length Range: {length_stats['min']} - {length_stats['max']} characters")
        
        print(f"\n🔍 LANGUAGE-SPECIFIC CONFIDENCE ANALYSIS:")
        for lang, analysis in list(stats['language_confidence_analysis'].items())[:5]:  # Top 5
            print(f"   {lang}:")
            print(f"     Documents: {analysis['count']}")
            print(f"     Average Confidence: {analysis['avg_confidence']:.3f}")
            print(f"     Confidence Range: {analysis['min_confidence']:.3f} - {analysis['max_confidence']:.3f}")

# Example usage with multilingual dataset
analyzer = LanguageDetectionAnalyzer(text_analytics_client)

# Simulate a large multilingual dataset
multilingual_dataset = [
    # English samples
    "The weather today is absolutely perfect for a walk in the park.",
    "Technology is advancing at an unprecedented rate in today's world.",
    "Customer service representatives should always be helpful and courteous.",
    
    # Spanish samples  
    "El clima de hoy es absolutamente perfecto para caminar por el parque.",
    "La tecnología avanza a un ritmo sin precedentes en el mundo actual.",
    "Los representantes de servicio al cliente siempre deben ser útiles y corteses.",
    
    # French samples
    "Le temps aujourd'hui est absolument parfait pour une promenade dans le parc.",
    "La technologie progresse à un rythme sans précédent dans le monde d'aujourd'hui.",
    "Les représentants du service client doivent toujours être serviables et courtois.",
    
    # German samples
    "Das Wetter heute ist absolut perfekt für einen Spaziergang im Park.",
    "Die Technologie entwickelt sich in der heutigen Welt mit beispielloser Geschwindigkeit.",
    "Kundendienstmitarbeiter sollten immer hilfsbereit und höflich sein.",
    
    # Mixed/problematic samples
    "Hello, ¿cómo estás? Je vais bien, danke!",  # Mixed languages
    "123-456-7890",  # Numbers only
    "😀🎉🌟",  # Emojis only
    "Hi!",  # Very short
]

# Run comprehensive analysis
analysis_results = analyzer.analyze_batch(multilingual_dataset, batch_size=5)
analyzer.print_analysis_report(analysis_results)

# Access the DataFrame for further analysis
if 'dataframe' in analysis_results:
    df = analysis_results['dataframe']
    print(f"\n📋 DATA PREVIEW:")
    print(df[['language_name', 'confidence_score', 'text_length']].head())
```

### Asynchronous Language Detection

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
from typing import List

async def async_language_detection():
    """Asynchronous language detection for better performance"""
    
    # Create async client
    async with TextAnalyticsClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    ) as client:
        
        # Large dataset for async processing
        large_dataset = [
            "English text for processing in batch mode with high efficiency.",
            "Texto en español para procesamiento en modo por lotes con alta eficiencia.",
            "Texte français pour traitement en mode lot avec une efficacité élevée.",
            "Deutscher Text für die Stapelverarbeitung mit hoher Effizienz.",
            "用于高效批处理模式处理的中文文本。",
            "高効率バッチモード処理のための日本語テキスト。",
            "Testo italiano per elaborazione in modalità batch ad alta efficienza.",
            "Nederlandse tekst voor batchverwerking met hoge efficiëntie.",
        ] * 5  # Multiply to create larger dataset
        
        print(f"Processing {len(large_dataset)} documents asynchronously...")
        
        start_time = asyncio.get_event_loop().time()
        
        try:
            # Async language detection
            response = await client.detect_language(documents=large_dataset)
            
            end_time = asyncio.get_event_loop().time()
            processing_time = end_time - start_time
            
            print(f"✅ Async processing completed in {processing_time:.2f} seconds")
            
            # Analyze results
            language_counts = Counter()
            confidence_scores = []
            
            for doc in response:
                if not doc.is_error:
                    language_counts[doc.primary_language.name] += 1
                    confidence_scores.append(doc.primary_language.confidence_score)
            
            print(f"\n📊 ASYNC PROCESSING RESULTS:")
            print(f"   Documents processed: {len(large_dataset)}")
            print(f"   Processing rate: {len(large_dataset)/processing_time:.1f} docs/second")
            print(f"   Average confidence: {sum(confidence_scores)/len(confidence_scores):.3f}")
            
            print(f"\n🌍 Language distribution:")
            for lang, count in language_counts.most_common():
                print(f"   {lang}: {count} documents")
                
        except Exception as e:
            print(f"Async processing error: {e}")

# Run async example
asyncio.run(async_language_detection())
```

## 🔄 Specialized Use Cases

### Content Routing System

```python
from enum import Enum
from dataclasses import dataclass
from typing import Dict, List, Optional

class SupportTeam(Enum):
    ENGLISH = "support-team-en"
    SPANISH = "support-team-es" 
    FRENCH = "support-team-fr"
    GERMAN = "support-team-de"
    MULTILINGUAL = "support-team-multilingual"
    ESCALATION = "support-escalation"

@dataclass
class TicketRouting:
    ticket_id: str
    content: str
    detected_language: str
    confidence_score: float
    assigned_team: SupportTeam
    routing_reason: str
    priority: str = "normal"

class CustomerSupportRouter:
    def __init__(self, client: TextAnalyticsClient):
        self.client = client
        self.routing_rules = {
            'English': SupportTeam.ENGLISH,
            'Spanish': SupportTeam.SPANISH,
            'French': SupportTeam.FRENCH,
            'German': SupportTeam.GERMAN,
        }
        self.confidence_threshold = 0.7
    
    def route_tickets(self, tickets: List[str]) -> List[TicketRouting]:
        """Route customer support tickets based on language detection"""
        
        try:
            # Detect languages for all tickets
            response = self.client.detect_language(documents=tickets)
            
            routing_results = []
            
            for idx, doc in enumerate(response):
                ticket_id = f"TICKET-{1000 + idx:04d}"
                content = tickets[idx]
                
                if doc.is_error:
                    # Route errors to escalation team
                    routing = TicketRouting(
                        ticket_id=ticket_id,
                        content=content,
                        detected_language="Unknown",
                        confidence_score=0.0,
                        assigned_team=SupportTeam.ESCALATION,
                        routing_reason=f"Language detection error: {doc.error}",
                        priority="high"
                    )
                else:
                    language = doc.primary_language.name
                    confidence = doc.primary_language.confidence_score
                    
                    # Determine routing based on confidence and language
                    if confidence < self.confidence_threshold:
                        assigned_team = SupportTeam.MULTILINGUAL
                        reason = f"Low confidence detection ({confidence:.3f})"
                        priority = "normal"
                    elif language in [team_lang for team_lang in self.routing_rules.keys()]:
                        assigned_team = self.routing_rules[language]
                        reason = f"High confidence {language} detection"
                        priority = "normal"
                    else:
                        assigned_team = SupportTeam.MULTILINGUAL
                        reason = f"Unsupported language: {language}"
                        priority = "normal"
                    
                    # Check for urgent keywords
                    urgent_keywords = ['urgent', 'emergency', 'critical', 'down', 'outage', 'urgent', 'urgente', 'critique']
                    if any(keyword.lower() in content.lower() for keyword in urgent_keywords):
                        priority = "high"
                    
                    routing = TicketRouting(
                        ticket_id=ticket_id,
                        content=content,
                        detected_language=language,
                        confidence_score=confidence,
                        assigned_team=assigned_team,
                        routing_reason=reason,
                        priority=priority
                    )
                
                routing_results.append(routing)
            
            return routing_results
            
        except Exception as e:
            print(f"Routing error: {e}")
            return []
    
    def generate_routing_report(self, routing_results: List[TicketRouting]) -> Dict:
        """Generate comprehensive routing report"""
        
        report = {
            'total_tickets': len(routing_results),
            'team_distribution': Counter(),
            'language_distribution': Counter(),
            'priority_distribution': Counter(),
            'confidence_stats': [],
            'high_priority_tickets': [],
            'low_confidence_tickets': []
        }
        
        for routing in routing_results:
            report['team_distribution'][routing.assigned_team.value] += 1
            report['language_distribution'][routing.detected_language] += 1
            report['priority_distribution'][routing.priority] += 1
            
            if routing.confidence_score > 0:
                report['confidence_stats'].append(routing.confidence_score)
            
            if routing.priority == "high":
                report['high_priority_tickets'].append(routing)
            
            if 0 < routing.confidence_score < self.confidence_threshold:
                report['low_confidence_tickets'].append(routing)
        
        return report
    
    def print_routing_report(self, report: Dict):
        """Print detailed routing report"""
        
        print("\n" + "="*60)
        print("CUSTOMER SUPPORT TICKET ROUTING REPORT")
        print("="*60)
        
        print(f"\n📋 OVERVIEW:")
        print(f"   Total Tickets: {report['total_tickets']}")
        
        print(f"\n👥 TEAM ASSIGNMENT:")
        for team, count in report['team_distribution'].most_common():
            percentage = (count / report['total_tickets']) * 100
            print(f"   {team}: {count} tickets ({percentage:.1f}%)")
        
        print(f"\n🌍 LANGUAGE DISTRIBUTION:")
        for lang, count in report['language_distribution'].most_common():
            percentage = (count / report['total_tickets']) * 100
            print(f"   {lang}: {count} tickets ({percentage:.1f}%)")
        
        print(f"\n⚡ PRIORITY DISTRIBUTION:")
        for priority, count in report['priority_distribution'].most_common():
            percentage = (count / report['total_tickets']) * 100
            print(f"   {priority.capitalize()}: {count} tickets ({percentage:.1f}%)")
        
        if report['confidence_stats']:
            avg_confidence = sum(report['confidence_stats']) / len(report['confidence_stats'])
            print(f"\n📊 CONFIDENCE STATISTICS:")
            print(f"   Average Confidence: {avg_confidence:.3f}")
            print(f"   Min/Max Confidence: {min(report['confidence_stats']):.3f} / {max(report['confidence_stats']):.3f}")
        
        if report['high_priority_tickets']:
            print(f"\n🚨 HIGH PRIORITY TICKETS ({len(report['high_priority_tickets'])}):")
            for ticket in report['high_priority_tickets'][:3]:  # Show first 3
                print(f"   {ticket.ticket_id}: {ticket.detected_language} -> {ticket.assigned_team.value}")
                print(f"     Content: {ticket.content[:80]}...")
        
        if report['low_confidence_tickets']:
            print(f"\n⚠️ LOW CONFIDENCE DETECTIONS ({len(report['low_confidence_tickets'])}):")
            for ticket in report['low_confidence_tickets'][:3]:  # Show first 3
                print(f"   {ticket.ticket_id}: {ticket.detected_language} ({ticket.confidence_score:.3f})")
                print(f"     Content: {ticket.content[:80]}...")

# Example usage
router = CustomerSupportRouter(text_analytics_client)

# Sample customer support tickets
support_tickets = [
    "URGENT: My account has been locked and I cannot access my funds. Please help immediately!",
    "Hello, I'm having trouble with my password reset. Can you assist me?",
    "URGENT: Mi cuenta ha sido bloqueada y no puedo acceder a mis fondos. ¡Por favor ayúdenme inmediatamente!",
    "Bonjour, j'ai des problèmes avec la réinitialisation de mon mot de passe. Pouvez-vous m'aider?",
    "KRITISCH: Mein Konto wurde gesperrt und ich kann nicht auf meine Gelder zugreifen. Bitte helfen Sie mir sofort!",
    "Hi there, just wanted to say thanks for the great service!",
    "Ciao, sto avendo problemi con il mio account. Potete aiutarmi?",
    "こんにちは、パスワードのリセットで問題があります。助けていただけますか？",
    "Hello, ¿cómo puedo cambiar mi contraseña? Je ne comprends pas.",  # Mixed language
    "The system is down! Emergency! Critical outage affecting all users!",
]

# Route tickets
routing_results = router.route_tickets(support_tickets)

# Generate and print report
report = router.generate_routing_report(routing_results)
router.print_routing_report(report)

# Show individual routing decisions
print(f"\n📝 INDIVIDUAL ROUTING DECISIONS:")
for routing in routing_results:
    print(f"\n{routing.ticket_id} [{routing.priority.upper()}]:")
    print(f"  Language: {routing.detected_language} (confidence: {routing.confidence_score:.3f})")
    print(f"  Team: {routing.assigned_team.value}")
    print(f"  Reason: {routing.routing_reason}")
    print(f"  Content: {routing.content[:100]}...")
```

### Multi-Language Content Analytics

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta
import numpy as np

class ContentLanguageAnalytics:
    def __init__(self, client: TextAnalyticsClient):
        self.client = client
    
    def analyze_content_trends(self, content_data: List[Dict]) -> pd.DataFrame:
        """Analyze language trends in content over time"""
        
        # Extract texts for language detection
        texts = [item['content'] for item in content_data]
        
        # Detect languages
        response = self.client.detect_language(documents=texts)
        
        # Create enhanced dataset
        enhanced_data = []
        for idx, (original_item, detection_result) in enumerate(zip(content_data, response)):
            
            enhanced_item = original_item.copy()
            
            if not detection_result.is_error:
                enhanced_item.update({
                    'detected_language': detection_result.primary_language.name,
                    'language_code': detection_result.primary_language.iso6391_name,
                    'confidence_score': detection_result.primary_language.confidence_score,
                    'detection_error': False
                })
            else:
                enhanced_item.update({
                    'detected_language': 'Unknown',
                    'language_code': 'unk',
                    'confidence_score': 0.0,
                    'detection_error': True,
                    'error_message': str(detection_result.error)
                })
            
            enhanced_data.append(enhanced_item)
        
        return pd.DataFrame(enhanced_data)
    
    def generate_language_insights(self, df: pd.DataFrame) -> Dict:
        """Generate comprehensive language insights"""
        
        successful_df = df[~df['detection_error']].copy()
        
        insights = {
            'total_content_pieces': len(df),
            'successful_detections': len(successful_df),
            'detection_success_rate': len(successful_df) / len(df) if len(df) > 0 else 0,
            
            # Language distribution
            'language_distribution': successful_df['detected_language'].value_counts().to_dict(),
            'language_percentages': (successful_df['detected_language'].value_counts(normalize=True) * 100).round(2).to_dict(),
            
            # Confidence analysis
            'avg_confidence_by_language': successful_df.groupby('detected_language')['confidence_score'].mean().round(3).to_dict(),
            'overall_avg_confidence': successful_df['confidence_score'].mean(),
            
            # Content length analysis
            'avg_content_length_by_language': successful_df.groupby('detected_language')['content'].apply(lambda x: x.str.len().mean()).round(1).to_dict(),
        }
        
        # Time-based analysis if timestamp available
        if 'timestamp' in df.columns:
            successful_df['timestamp'] = pd.to_datetime(successful_df['timestamp'])
            successful_df['hour'] = successful_df['timestamp'].dt.hour
            successful_df['day_of_week'] = successful_df['timestamp'].dt.day_name()
            
            insights['temporal_analysis'] = {
                'languages_by_hour': successful_df.groupby('hour')['detected_language'].apply(lambda x: x.value_counts().to_dict()).to_dict(),
                'languages_by_day': successful_df.groupby('day_of_week')['detected_language'].apply(lambda x: x.value_counts().to_dict()).to_dict(),
            }
        
        # Content type analysis if available
        if 'content_type' in df.columns:
            insights['content_type_analysis'] = {
                'languages_by_content_type': successful_df.groupby('content_type')['detected_language'].apply(lambda x: x.value_counts().to_dict()).to_dict()
            }
        
        return insights
    
    def print_insights_report(self, insights: Dict):
        """Print comprehensive insights report"""
        
        print("\n" + "="*60)
        print("CONTENT LANGUAGE ANALYTICS REPORT")
        print("="*60)
        
        print(f"\n📊 OVERVIEW:")
        print(f"   Total Content Pieces: {insights['total_content_pieces']}")
        print(f"   Successful Detections: {insights['successful_detections']}")
        print(f"   Detection Success Rate: {insights['detection_success_rate']:.2%}")
        print(f"   Overall Average Confidence: {insights['overall_avg_confidence']:.3f}")
        
        print(f"\n🌍 LANGUAGE DISTRIBUTION:")
        for lang, percentage in list(insights['language_percentages'].items())[:10]:
            count = insights['language_distribution'][lang]
            print(f"   {lang}: {count} pieces ({percentage:.1f}%)")
        
        print(f"\n📈 CONFIDENCE ANALYSIS BY LANGUAGE:")
        for lang, avg_conf in sorted(insights['avg_confidence_by_language'].items(), 
                                   key=lambda x: x[1], reverse=True)[:10]:
            print(f"   {lang}: {avg_conf:.3f}")
        
        print(f"\n📝 AVERAGE CONTENT LENGTH BY LANGUAGE:")
        for lang, avg_length in sorted(insights['avg_content_length_by_language'].items(), 
                                     key=lambda x: x[1], reverse=True)[:10]:
            print(f"   {lang}: {avg_length:.0f} characters")
        
        # Temporal analysis if available
        if 'temporal_analysis' in insights:
            print(f"\n⏰ TEMPORAL PATTERNS:")
            print("   Language distribution varies by time of day and day of week")
            print("   (See detailed temporal_analysis in returned insights dict)")
        
        # Content type analysis if available  
        if 'content_type_analysis' in insights:
            print(f"\n📑 CONTENT TYPE PATTERNS:")
            print("   Language usage varies by content type")
            print("   (See detailed content_type_analysis in returned insights dict)")

# Example usage with simulated social media data
analytics = ContentLanguageAnalytics(text_analytics_client)

# Simulate social media content with timestamps and types
simulated_content = [
    {
        'content': 'Just had the best coffee at this new café! ☕ #coffee #morning',
        'timestamp': '2024-01-15 08:30:00',
        'content_type': 'social_post',
        'platform': 'twitter'
    },
    {
        'content': 'Acabamos de probar el mejor café en esta nueva cafetería! ☕ #café #mañana',
        'timestamp': '2024-01-15 08:45:00', 
        'content_type': 'social_post',
        'platform': 'twitter'
    },
    {
        'content': 'Je viens de prendre le meilleur café dans ce nouveau café! ☕ #café #matin',
        'timestamp': '2024-01-15 09:00:00',
        'content_type': 'social_post', 
        'platform': 'twitter'
    },
    {
        'content': 'Product feedback: The new update is working perfectly. Great improvements!',
        'timestamp': '2024-01-15 14:20:00',
        'content_type': 'product_review',
        'platform': 'app_store'
    },
    {
        'content': 'Comentarios del producto: La nueva actualización funciona perfectamente. ¡Grandes mejoras!',
        'timestamp': '2024-01-15 14:35:00',
        'content_type': 'product_review', 
        'platform': 'app_store'
    },
    {
        'content': 'Customer support ticket: Having trouble with login process.',
        'timestamp': '2024-01-15 16:45:00',
        'content_type': 'support_ticket',
        'platform': 'helpdesk'
    }
]

# Analyze content
df = analytics.analyze_content_trends(simulated_content)
insights = analytics.generate_language_insights(df)
analytics.print_insights_report(insights)

# Display DataFrame sample
print(f"\n📋 ENHANCED DATA SAMPLE:")
print(df[['detected_language', 'confidence_score', 'content_type', 'timestamp']].head())
```

## 📚 Data Analysis Integration

### Pandas Integration for Large Datasets

```python
import pandas as pd
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

class PandasLanguageDetector:
    def __init__(self, client: TextAnalyticsClient, max_workers: int = 5):
        self.client = client
        self.max_workers = max_workers
    
    def detect_languages_parallel(self, df: pd.DataFrame, text_column: str, batch_size: int = 10) -> pd.DataFrame:
        """Parallel language detection for large pandas DataFrames"""
        
        texts = df[text_column].tolist()
        total_texts = len(texts)
        
        print(f"Processing {total_texts} texts with {self.max_workers} workers...")
        
        # Split into batches for parallel processing
        batches = [texts[i:i + batch_size] for i in range(0, total_texts, batch_size)]
        
        results = []
        completed_batches = 0
        
        start_time = time.time()
        
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            # Submit all batch jobs
            future_to_batch = {
                executor.submit(self._process_batch, batch, batch_idx): batch_idx 
                for batch_idx, batch in enumerate(batches)
            }
            
            # Collect results as they complete
            for future in as_completed(future_to_batch):
                batch_idx = future_to_batch[future]
                try:
                    batch_results = future.result()
                    results.extend(batch_results)
                    completed_batches += 1
                    
                    progress = (completed_batches / len(batches)) * 100
                    print(f"  Progress: {progress:.1f}% ({completed_batches}/{len(batches)} batches)")
                    
                except Exception as e:
                    print(f"  Batch {batch_idx} failed: {e}")
                    # Add empty results for failed batch
                    batch_size_actual = len(batches[batch_idx])
                    for _ in range(batch_size_actual):
                        results.append({
                            'language_name': None,
                            'language_code': None,
                            'confidence_score': 0.0,
                            'has_error': True,
                            'error_message': str(e)
                        })
        
        end_time = time.time()
        processing_time = end_time - start_time
        
        print(f"✅ Processing completed in {processing_time:.2f} seconds")
        print(f"   Rate: {total_texts/processing_time:.1f} texts per second")
        
        # Create results DataFrame and merge with original
        results_df = pd.DataFrame(results)
        
        # Ensure results are in the same order as input
        enhanced_df = df.copy()
        enhanced_df = pd.concat([enhanced_df, results_df], axis=1)
        
        return enhanced_df
    
    def _process_batch(self, batch_texts: List[str], batch_idx: int) -> List[Dict]:
        """Process a single batch of texts"""
        
        try:
            response = self.client.detect_language(documents=batch_texts)
            
            batch_results = []
            for doc in response:
                if not doc.is_error:
                    result = {
                        'language_name': doc.primary_language.name,
                        'language_code': doc.primary_language.iso6391_name,
                        'confidence_score': doc.primary_language.confidence_score,
                        'has_error': False,
                        'error_message': None
                    }
                else:
                    result = {
                        'language_name': None,
                        'language_code': None,
                        'confidence_score': 0.0,
                        'has_error': True,
                        'error_message': str(doc.error)
                    }
                
                batch_results.append(result)
            
            return batch_results
            
        except Exception as e:
            # Return error results for entire batch
            return [
                {
                    'language_name': None,
                    'language_code': None, 
                    'confidence_score': 0.0,
                    'has_error': True,
                    'error_message': str(e)
                }
                for _ in batch_texts
            ]
    
    def analyze_dataframe(self, df: pd.DataFrame) -> Dict:
        """Comprehensive DataFrame analysis with language detection results"""
        
        successful_df = df[~df['has_error']].copy()
        
        analysis = {
            'total_rows': len(df),
            'successful_detections': len(successful_df),
            'error_rate': (len(df) - len(successful_df)) / len(df) if len(df) > 0 else 0,
            
            # Language statistics
            'language_counts': successful_df['language_name'].value_counts().to_dict() if len(successful_df) > 0 else {},
            'language_percentages': (successful_df['language_name'].value_counts(normalize=True) * 100).round(2).to_dict() if len(successful_df) > 0 else {},
            
            # Confidence statistics  
            'confidence_stats': {
                'mean': successful_df['confidence_score'].mean() if len(successful_df) > 0 else 0,
                'std': successful_df['confidence_score'].std() if len(successful_df) > 0 else 0,
                'min': successful_df['confidence_score'].min() if len(successful_df) > 0 else 0,
                'max': successful_df['confidence_score'].max() if len(successful_df) > 0 else 0,
                'q25': successful_df['confidence_score'].quantile(0.25) if len(successful_df) > 0 else 0,
                'q50': successful_df['confidence_score'].quantile(0.50) if len(successful_df) > 0 else 0,
                'q75': successful_df['confidence_score'].quantile(0.75) if len(successful_df) > 0 else 0,
            }
        }
        
        # Cross-tabulations if additional columns exist
        categorical_columns = df.select_dtypes(include=['object', 'category']).columns
        categorical_columns = [col for col in categorical_columns if col not in ['language_name', 'language_code', 'error_message']]
        
        if len(successful_df) > 0 and len(categorical_columns) > 0:
            analysis['cross_tabulations'] = {}
            for col in categorical_columns[:3]:  # Limit to first 3 categorical columns
                if col in successful_df.columns:
                    crosstab = pd.crosstab(successful_df['language_name'], successful_df[col])
                    analysis['cross_tabulations'][col] = crosstab.to_dict()
        
        return analysis

# Example usage with large simulated dataset
detector = PandasLanguageDetector(text_analytics_client, max_workers=3)

# Create a large simulated dataset
np.random.seed(42)
large_dataset_size = 100

# Simulated multilingual customer feedback
feedback_templates = {
    'English': [
        "Great product, very satisfied with the quality and service.",
        "The delivery was fast and the packaging was excellent.",
        "Customer support was helpful and responsive to my questions.",
        "Product quality exceeded my expectations, will buy again.",
        "Easy to use interface and great user experience overall."
    ],
    'Spanish': [
        "Excelente producto, muy satisfecho con la calidad y el servicio.",
        "La entrega fue rápida y el empaquetado fue excelente.",
        "El servicio al cliente fue útil y receptivo a mis preguntas.",
        "La calidad del producto superó mis expectativas, compraré de nuevo.",
        "Interfaz fácil de usar y gran experiencia de usuario en general."
    ],
    'French': [
        "Excellent produit, très satisfait de la qualité et du service.",
        "La livraison était rapide et l'emballage était excellent.",
        "Le support client était utile et réactif à mes questions.",
        "La qualité du produit a dépassé mes attentes, j'achèterai à nouveau.",
        "Interface facile à utiliser et excellente expérience utilisateur."
    ],
    'German': [
        "Tolles Produkt, sehr zufrieden mit Qualität und Service.",
        "Die Lieferung war schnell und die Verpackung war ausgezeichnet.",
        "Der Kundensupport war hilfreich und reagierte auf meine Fragen.",
        "Produktqualität übertraf meine Erwartungen, werde wieder kaufen.",
        "Einfach zu bedienende Oberfläche und insgesamt tolle Benutzererfahrung."
    ]
}

# Generate large dataset
large_data = []
languages = list(feedback_templates.keys())
categories = ['electronics', 'clothing', 'books', 'home_garden', 'sports']
ratings = [1, 2, 3, 4, 5]

for i in range(large_dataset_size):
    lang = np.random.choice(languages)
    template = np.random.choice(feedback_templates[lang])
    
    large_data.append({
        'feedback_id': f'FB-{i+1:04d}',
        'customer_feedback': template,
        'category': np.random.choice(categories),
        'rating': np.random.choice(ratings),
        'purchase_date': pd.Timestamp('2024-01-01') + pd.Timedelta(days=np.random.randint(0, 365)),
        'actual_language': lang  # For validation
    })

# Create DataFrame
df = pd.DataFrame(large_data)

print("ORIGINAL DATASET SAMPLE:")
print(df[['feedback_id', 'customer_feedback', 'category', 'rating', 'actual_language']].head())

# Process with parallel language detection
enhanced_df = detector.detect_languages_parallel(df, 'customer_feedback', batch_size=10)

# Analyze results
analysis_results = detector.analyze_dataframe(enhanced_df)

print(f"\n" + "="*60)
print("PANDAS LANGUAGE DETECTION ANALYSIS")
print("="*60)

print(f"\n📊 PROCESSING SUMMARY:")
print(f"   Total Feedback Records: {analysis_results['total_rows']}")
print(f"   Successful Detections: {analysis_results['successful_detections']}")
print(f"   Error Rate: {analysis_results['error_rate']:.2%}")

print(f"\n🌍 DETECTED LANGUAGE DISTRIBUTION:")
for lang, count in list(analysis_results['language_counts'].items())[:10]:
    percentage = analysis_results['language_percentages'][lang]
    print(f"   {lang}: {count} records ({percentage:.1f}%)")

print(f"\n📈 CONFIDENCE STATISTICS:")
conf_stats = analysis_results['confidence_stats']
print(f"   Mean Confidence: {conf_stats['mean']:.3f}")
print(f"   Std Deviation: {conf_stats['std']:.3f}")
print(f"   Confidence Range: {conf_stats['min']:.3f} - {conf_stats['max']:.3f}")
print(f"   Quartiles: Q1={conf_stats['q25']:.3f}, Q2={conf_stats['q50']:.3f}, Q3={conf_stats['q75']:.3f}")

# Validation against actual language (for simulated data)
if 'actual_language' in enhanced_df.columns:
    successful_enhanced = enhanced_df[~enhanced_df['has_error']].copy()
    if len(successful_enhanced) > 0:
        accuracy = (successful_enhanced['language_name'] == successful_enhanced['actual_language']).mean()
        print(f"\n✅ DETECTION ACCURACY: {accuracy:.2%}")

# Show sample of enhanced data
print(f"\n📋 ENHANCED DATASET SAMPLE:")
sample_cols = ['feedback_id', 'language_name', 'confidence_score', 'category', 'rating']
print(enhanced_df[sample_cols].head(10))

# Cross-tabulation analysis
if 'cross_tabulations' in analysis_results:
    print(f"\n🔍 LANGUAGE VS CATEGORY ANALYSIS:")
    for category, lang_dist in list(analysis_results['cross_tabulations'].get('category', {}).items())[:5]:
        print(f"   {category}: {lang_dist}")
```

## 📚 Additional Resources

- [Azure AI Text Analytics Client Library for Python](https://docs.microsoft.com/python/api/azure-ai-textanalytics/)
- [Language Detection Documentation](https://docs.microsoft.com/azure/cognitive-services/language-service/language-detection/)
- [Python SDK Samples](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/textanalytics)
- [Async Programming with Azure SDK](https://docs.microsoft.com/python/api/azure-ai-textanalytics/azure.ai.textanalytics.aio)