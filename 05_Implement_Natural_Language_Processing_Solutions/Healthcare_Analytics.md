# 🏥 Healthcare Text Analytics with Azure AI Language

> **Comprehensive code examples for healthcare entity recognition and analysis using Azure AI Language for Health services**

## 📋 Overview

Azure AI Language for Health (formerly Text Analytics for Health) is a specialized NLP service that extracts and labels relevant medical information from unstructured text such as clinical notes, discharge summaries, clinical documents, and electronic health records (EHRs).

## 🔧 Key Features

- **🩺 Medical Entity Recognition** - Medications, dosages, symptoms, diagnoses, procedures
- **🔗 Entity Relations** - Relationships between medical entities (e.g., dosage of medication)
- **✅ Assertion Detection** - Negation, uncertainty, and conditional statements
- **📊 Entity Linking** - Links to medical ontologies (UMLS, ICD-10, SNOMED CT)
- **🏥 PHI Detection** - Protected Health Information identification and redaction
- **📝 Clinical Document Processing** - Structured extraction from clinical narratives
- **⚡ Async Processing** - Long-running operations for large document analysis
- **🌐 Multi-language Support** - English and other languages for healthcare content

## 🐍 Python Examples

### Basic Healthcare Entity Recognition

```python
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import asyncio
import os

# Authentication
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]
credential = AzureKeyCredential(key)

async def healthcare_entities_basic():
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        documents = [
            """
            Patient John Smith presented to the emergency department with severe chest pain.
            He was prescribed 100mg aspirin daily and 5mg lisinopril twice daily.
            Blood pressure was 140/90 mmHg. ECG showed normal sinus rhythm.
            Patient has a history of hypertension and diabetes mellitus type 2.
            """,
            """
            The patient underwent successful appendectomy under general anesthesia.
            Post-operative medications include 400mg ibuprofen every 6 hours for pain
            and 500mg cephalexin twice daily for infection prevention.
            Patient will follow up in clinic in 2 weeks.
            """
        ]
        
        # Start healthcare analysis
        operation = await client.begin_analyze_healthcare_entities(documents)
        
        # Wait for completion
        result = await operation.result()
        
        async for page in result:
            for idx, doc_result in enumerate(page):
                if not doc_result.is_error:
                    print(f"Healthcare Analysis - Document {idx + 1}:")
                    print("=" * 50)
                    
                    # Display entities by category
                    entities_by_category = {}
                    for entity in doc_result.entities:
                        category = entity.category
                        if category not in entities_by_category:
                            entities_by_category[category] = []
                        entities_by_category[category].append(entity)
                    
                    for category, entities in entities_by_category.items():
                        print(f"\n{category.upper()}:")
                        for entity in entities:
                            print(f"  - Text: {entity.text}")
                            print(f"    Confidence: {entity.confidence_score:.2f}")
                            if entity.normalized_text:
                                print(f"    Normalized: {entity.normalized_text}")
                            if hasattr(entity, 'assertion') and entity.assertion:
                                print(f"    Assertion: {entity.assertion}")
                            print()
                else:
                    print(f"Document {idx + 1} has an error: {doc_result.error}")

# Run basic healthcare analysis
asyncio.run(healthcare_entities_basic())
```

### Advanced Healthcare Analysis with Relations

```python
async def healthcare_entities_with_relations():
    """Analyze healthcare entities and their relationships"""
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        clinical_document = """
        CHIEF COMPLAINT: Chest pain and shortness of breath.
        
        HISTORY: 
        Patient is a 58-year-old male with a history of hypertension, diabetes mellitus type 2,
        and hyperlipidemia. He presents with acute onset chest pain radiating to the left arm,
        associated with diaphoresis and nausea that started 2 hours ago.
        
        MEDICATIONS:
        - Metformin 500mg twice daily for diabetes
        - Lisinopril 10mg daily for hypertension  
        - Atorvastatin 20mg at bedtime for cholesterol
        - Aspirin 81mg daily for cardioprotection
        
        PHYSICAL EXAMINATION:
        Blood pressure: 165/95 mmHg
        Heart rate: 88 bpm
        Temperature: 98.6°F
        Respiratory rate: 22/min
        
        ASSESSMENT AND PLAN:
        1. Acute coronary syndrome - rule out myocardial infarction
           - Serial cardiac enzymes
           - 12-lead ECG
           - Start heparin protocol
           - Continue aspirin, increase to 325mg daily
        
        2. Hypertension - poorly controlled
           - Increase lisinopril to 20mg daily
           - Add amlodipine 5mg daily if needed
        
        3. Type 2 diabetes mellitus - well controlled
           - Continue current metformin dose
           - Monitor blood glucose closely
        """
        
        operation = await client.begin_analyze_healthcare_entities([clinical_document])
        result = await operation.result()
        
        async for page in result:
            for doc_result in page:
                if not doc_result.is_error:
                    print("HEALTHCARE ENTITY ANALYSIS")
                    print("=" * 60)
                    
                    # Analyze entity relations
                    print("\nENTITY RELATIONS:")
                    for relation in doc_result.entity_relations:
                        print(f"\nRelation Type: {relation.relation_type}")
                        if relation.confidence_score:
                            print(f"Confidence: {relation.confidence_score:.2f}")
                        
                        for role in relation.roles:
                            entity = role.entity
                            print(f"  {role.name}: {entity.text}")
                            print(f"    Category: {entity.category}")
                            if entity.normalized_text:
                                print(f"    Normalized: {entity.normalized_text}")
                    
                    # Analyze assertions (negation, uncertainty, etc.)
                    print("\n" + "=" * 60)
                    print("ASSERTION ANALYSIS:")
                    
                    assertion_entities = [e for e in doc_result.entities if hasattr(e, 'assertion') and e.assertion]
                    
                    for entity in assertion_entities:
                        print(f"\nEntity: {entity.text} ({entity.category})")
                        print(f"  Certainty: {entity.assertion.certainty}")
                        print(f"  Conditionality: {entity.assertion.conditionality}")
                        print(f"  Association: {entity.assertion.association}")
                    
                    # Medical concepts with UMLS linking
                    print("\n" + "=" * 60)
                    print("MEDICAL CONCEPT LINKING:")
                    
                    linked_entities = [e for e in doc_result.entities if hasattr(e, 'data_sources') and e.data_sources]
                    
                    for entity in linked_entities:
                        print(f"\nEntity: {entity.text}")
                        print(f"Category: {entity.category}")
                        if entity.data_sources:
                            for data_source in entity.data_sources:
                                print(f"  Data Source: {data_source.name}")
                                print(f"  Entity ID: {data_source.entity_id}")

asyncio.run(healthcare_entities_with_relations())
```

### Batch Healthcare Document Processing

```python
async def batch_healthcare_processing():
    """Process multiple healthcare documents in batch"""
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        healthcare_documents = [
            {
                "id": "discharge_summary_1",
                "text": """
                DISCHARGE SUMMARY
                Patient: Jane Doe
                Admission Date: 01/15/2024
                Discharge Date: 01/18/2024
                
                DIAGNOSIS: 
                Primary: Pneumonia, community-acquired
                Secondary: COPD exacerbation
                
                TREATMENT:
                Patient treated with IV ceftriaxone 1g daily for 3 days,
                then switched to oral amoxicillin 500mg TID for 7 days.
                Bronchodilator therapy with albuterol inhaler 2 puffs q4h PRN.
                
                DISCHARGE MEDICATIONS:
                1. Amoxicillin 500mg PO TID x 7 days
                2. Albuterol inhaler 90mcg, 2 puffs q4h PRN shortness of breath
                3. Prednisone 20mg PO daily x 5 days, then taper
                """
            },
            {
                "id": "progress_note_1", 
                "text": """
                PROGRESS NOTE - Day 3
                Patient continues to show improvement with antibiotic therapy.
                Oxygen saturation improved to 96% on room air.
                Patient reports decreased cough and improved appetite.
                
                VITAL SIGNS:
                BP: 125/80 mmHg
                HR: 78 bpm
                Temp: 99.2°F
                RR: 18/min
                O2 sat: 96% RA
                
                PLAN:
                - Continue current antibiotics
                - Discontinue oxygen therapy
                - Physical therapy evaluation
                - Discharge planning if stable tomorrow
                """
            }
        ]
        
        # Process documents
        operation = await client.begin_analyze_healthcare_entities(
            [doc["text"] for doc in healthcare_documents]
        )
        
        result = await operation.result()
        
        async for page in result:
            for idx, doc_result in enumerate(page):
                document_info = healthcare_documents[idx]
                
                print(f"\nDOCUMENT: {document_info['id']}")
                print("=" * 50)
                
                if not doc_result.is_error:
                    # Summary statistics
                    medication_count = len([e for e in doc_result.entities if e.category == "MedicationName"])
                    condition_count = len([e for e in doc_result.entities if e.category in ["SymptomOrSign", "Diagnosis"]])
                    treatment_count = len([e for e in doc_result.entities if e.category == "TreatmentName"])
                    
                    print(f"Summary: {len(doc_result.entities)} total entities")
                    print(f"  - Medications: {medication_count}")
                    print(f"  - Conditions: {condition_count}")
                    print(f"  - Treatments: {treatment_count}")
                    
                    # Key findings
                    medications = [e.text for e in doc_result.entities if e.category == "MedicationName"]
                    if medications:
                        print(f"\nMedications mentioned: {', '.join(medications)}")
                    
                    conditions = [e.text for e in doc_result.entities if e.category in ["SymptomOrSign", "Diagnosis"]]
                    if conditions:
                        print(f"Conditions/Symptoms: {', '.join(conditions)}")
                
                else:
                    print(f"Error processing document: {doc_result.error}")

asyncio.run(batch_healthcare_processing())
```

### Healthcare PHI Detection

```python
async def healthcare_phi_detection():
    """Detect and redact PHI in healthcare documents"""
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        clinical_note = """
        PATIENT: John Smith
        DOB: 03/15/1965
        MRN: 123456789
        SSN: 987-65-4321
        Phone: (206) 555-0123
        Email: john.smith@email.com
        Address: 123 Medical Center Dr, Seattle, WA 98101
        
        PROVIDER: Dr. Sarah Johnson, MD
        Date of Service: January 20, 2024
        
        CLINICAL NOTE:
        Mr. Smith is a 58-year-old gentleman who presents for routine follow-up
        of his diabetes mellitus type 2 and hypertension. His last HbA1c was 7.2%.
        Current medications include metformin 1000mg twice daily and lisinopril 10mg daily.
        
        He reports good adherence to medications but admits to occasional dietary indiscretions
        during the holidays. Blood pressure today is 138/85 mmHg.
        
        PLAN:
        - Continue current diabetes management
        - Increase lisinopril to 20mg daily
        - Follow up in 3 months
        - Patient counseled on dietary modifications
        """
        
        # First, detect healthcare entities
        print("HEALTHCARE ENTITY ANALYSIS:")
        print("=" * 50)
        
        healthcare_operation = await client.begin_analyze_healthcare_entities([clinical_note])
        healthcare_result = await healthcare_operation.result()
        
        async for page in healthcare_result:
            for doc_result in page:
                if not doc_result.is_error:
                    medical_entities = {}
                    for entity in doc_result.entities:
                        category = entity.category
                        if category not in medical_entities:
                            medical_entities[category] = []
                        medical_entities[category].append(entity.text)
                    
                    for category, entities in medical_entities.items():
                        print(f"{category}: {', '.join(set(entities))}")
        
        # Then detect PHI
        print(f"\n\nPHI DETECTION AND REDACTION:")
        print("=" * 50)
        
        phi_response = client.recognize_pii_entities([clinical_note], domain_filter="phi")
        
        for doc in phi_response:
            if not doc.is_error:
                print(f"Original note length: {len(clinical_note)} characters")
                print(f"Redacted note length: {len(doc.redacted_text)} characters")
                print(f"\nDetected PHI entities: {len(doc.entities)}")
                
                phi_by_category = {}
                for entity in doc.entities:
                    category = entity.category
                    if category not in phi_by_category:
                        phi_by_category[category] = []
                    phi_by_category[category].append(entity.text)
                
                for category, entities in phi_by_category.items():
                    print(f"  {category}: {len(entities)} entities")
                
                print(f"\nRedacted Clinical Note:")
                print("-" * 30)
                print(doc.redacted_text[:500] + "..." if len(doc.redacted_text) > 500 else doc.redacted_text)

asyncio.run(healthcare_phi_detection())
```

## 🔷 C# Examples

### Basic Healthcare Entity Recognition

```csharp
using Azure;
using Azure.AI.TextAnalytics;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static string endpoint = Environment.GetEnvironmentVariable("LANGUAGE_ENDPOINT");
    static string key = Environment.GetEnvironmentVariable("LANGUAGE_KEY");
    
    static async Task Main(string[] args)
    {
        var credential = new AzureKeyCredential(key);
        var client = new TextAnalyticsClient(new Uri(endpoint), credential);
        
        await HealthcareEntityRecognitionExample(client);
        await HealthcareEntityRelationsExample(client);
    }
    
    static async Task HealthcareEntityRecognitionExample(TextAnalyticsClient client)
    {
        var documents = new List<string>()
        {
            "Patient was prescribed 100mg ibuprofen twice daily for chronic pain management. " +
            "Blood pressure reading was 140/90 mmHg. Patient has history of diabetes and hypertension."
        };

        AnalyzeHealthcareEntitiesOperation operation = await client.StartAnalyzeHealthcareEntitiesAsync(documents);

        await operation.WaitForCompletionAsync();

        Console.WriteLine("Healthcare Entity Recognition Results:");
        Console.WriteLine("=====================================");

        await foreach (AnalyzeHealthcareEntitiesResultCollection documentsInPage in operation.Value)
        {
            foreach (AnalyzeHealthcareEntitiesResult entitiesResult in documentsInPage)
            {
                if (entitiesResult.HasError)
                {
                    Console.WriteLine($"Error: {entitiesResult.Error.ErrorCode} - {entitiesResult.Error.Message}");
                }
                else
                {
                    Console.WriteLine($"Recognized {entitiesResult.Entities.Count} healthcare entities:");
                    
                    // Group entities by category
                    var entitiesByCategory = new Dictionary<string, List<HealthcareEntity>>();
                    
                    foreach (HealthcareEntity entity in entitiesResult.Entities)
                    {
                        if (!entitiesByCategory.ContainsKey(entity.Category))
                            entitiesByCategory[entity.Category] = new List<HealthcareEntity>();
                        
                        entitiesByCategory[entity.Category].Add(entity);
                    }
                    
                    foreach (var categoryGroup in entitiesByCategory)
                    {
                        Console.WriteLine($"\n{categoryGroup.Key.ToUpper()}:");
                        foreach (var entity in categoryGroup.Value)
                        {
                            Console.WriteLine($"  - Text: {entity.Text}");
                            Console.WriteLine($"    Confidence: {entity.ConfidenceScore:F2}");
                            if (!string.IsNullOrEmpty(entity.NormalizedText))
                                Console.WriteLine($"    Normalized: {entity.NormalizedText}");
                            Console.WriteLine();
                        }
                    }
                }
            }
        }
    }
    
    static async Task HealthcareEntityRelationsExample(TextAnalyticsClient client)
    {
        var document = "Patient was administered 40mg of Prednisone daily for asthma treatment.";

        AnalyzeHealthcareEntitiesOperation operation = await client.StartAnalyzeHealthcareEntitiesAsync(new[] { document });
        await operation.WaitForCompletionAsync();

        Console.WriteLine("\nHealthcare Entity Relations:");
        Console.WriteLine("============================");

        await foreach (AnalyzeHealthcareEntitiesResultCollection documentsInPage in operation.Value)
        {
            foreach (AnalyzeHealthcareEntitiesResult entitiesResult in documentsInPage)
            {
                foreach (HealthcareEntityRelation relation in entitiesResult.EntityRelations)
                {
                    Console.WriteLine($"Relation Type: {relation.RelationType}");
                    if (relation.ConfidenceScore.HasValue)
                        Console.WriteLine($"Confidence: {relation.ConfidenceScore:F2}");
                    
                    foreach (HealthcareEntityRelationRole role in relation.Roles)
                    {
                        Console.WriteLine($"  {role.Name}: {role.Entity.Text} ({role.Entity.Category})");
                    }
                    Console.WriteLine();
                }
            }
        }
    }
}
```

### Advanced Healthcare Processing

```csharp
static async Task AdvancedHealthcareProcessing(TextAnalyticsClient client)
{
    var clinicalDocuments = new List<TextDocumentInput>()
    {
        new TextDocumentInput("discharge_summary", 
            "Patient John Doe was treated for pneumonia with Amoxicillin 500mg TID. " +
            "Blood pressure was elevated at 160/95 mmHg. Patient has diabetes type 2.")
        {
            Language = "en"
        }
    };

    AnalyzeHealthcareEntitiesOperation operation = await client.StartAnalyzeHealthcareEntitiesAsync(clinicalDocuments);
    await operation.WaitForCompletionAsync();

    Console.WriteLine("Advanced Healthcare Analysis:");
    Console.WriteLine("=============================");

    await foreach (AnalyzeHealthcareEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (AnalyzeHealthcareEntitiesResult result in documentsInPage)
        {
            Console.WriteLine($"Document ID: {result.Id}");
            
            if (!result.HasError)
            {
                // Entity statistics
                var medications = result.Entities.Where(e => e.Category == "MedicationName").ToList();
                var conditions = result.Entities.Where(e => e.Category == "SymptomOrSign" || e.Category == "Diagnosis").ToList();
                var measurements = result.Entities.Where(e => e.Category == "MeasurementValue").ToList();
                
                Console.WriteLine($"\nSummary:");
                Console.WriteLine($"  Total entities: {result.Entities.Count}");
                Console.WriteLine($"  Medications: {medications.Count}");
                Console.WriteLine($"  Conditions: {conditions.Count}");
                Console.WriteLine($"  Measurements: {measurements.Count}");
                Console.WriteLine($"  Relations: {result.EntityRelations.Count}");
                
                // Assertion analysis
                var assertionEntities = result.Entities.Where(e => e.Assertion != null).ToList();
                if (assertionEntities.Any())
                {
                    Console.WriteLine($"\nAssertion Analysis:");
                    foreach (var entity in assertionEntities)
                    {
                        Console.WriteLine($"  Entity: {entity.Text}");
                        Console.WriteLine($"    Certainty: {entity.Assertion.Certainty}");
                        Console.WriteLine($"    Conditionality: {entity.Assertion.Conditionality}");
                        Console.WriteLine($"    Association: {entity.Assertion.Association}");
                    }
                }
                
                // Data source linking (UMLS, etc.)
                var linkedEntities = result.Entities.Where(e => e.DataSources?.Any() == true).ToList();
                if (linkedEntities.Any())
                {
                    Console.WriteLine($"\nLinked Medical Concepts:");
                    foreach (var entity in linkedEntities)
                    {
                        Console.WriteLine($"  {entity.Text} ({entity.Category}):");
                        foreach (var dataSource in entity.DataSources)
                        {
                            Console.WriteLine($"    {dataSource.Name}: {dataSource.EntityId}");
                        }
                    }
                }
            }
            else
            {
                Console.WriteLine($"Error: {result.Error.ErrorCode} - {result.Error.Message}");
            }
        }
    }
}
```

## 🌐 REST API Examples

### Healthcare Entity Recognition

```bash
curl -X POST "https://<your-endpoint>.cognitiveservices.azure.com/language/analyze-text/jobs?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "Healthcare Analysis Job",
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Patient was prescribed 100mg ibuprofen twice daily for pain management."
        }
      ]
    },
    "tasks": [
      {
        "kind": "Healthcare",
        "taskName": "Healthcare Entity Recognition"
      }
    ]
  }'
```

### Check Healthcare Job Status

```bash
# Use the operation-location URL from the response header
curl -X GET "https://<your-endpoint>.cognitiveservices.azure.com/language/analyze-text/jobs/<job-id>?api-version=2022-05-01" \
  -H "Ocp-Apim-Subscription-Key: <your-key>"
```

### Python REST Implementation

```python
import requests
import json
import time

def healthcare_analysis_rest():
    endpoint = "https://<your-endpoint>.cognitiveservices.azure.com"
    key = "<your-key>"
    
    # Submit healthcare analysis job
    url = f"{endpoint}/language/analyze-text/jobs?api-version=2022-05-01"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    payload = {
        "displayName": "Healthcare Entity Analysis",
        "analysisInput": {
            "documents": [
                {
                    "id": "clinical_note_1",
                    "language": "en",
                    "text": """
                    Patient John Smith (DOB: 01/15/1970) presented with chest pain.
                    Prescribed aspirin 81mg daily and lisinopril 10mg daily for hypertension.
                    Blood pressure: 150/90 mmHg. Patient has history of diabetes type 2.
                    Follow-up appointment scheduled in 2 weeks.
                    """
                }
            ]
        },
        "tasks": [
            {
                "kind": "Healthcare",
                "taskName": "Healthcare Analysis",
                "parameters": {
                    "modelVersion": "latest"
                }
            }
        ]
    }
    
    # Submit job
    response = requests.post(url, json=payload, headers=headers)
    
    if response.status_code == 202:
        # Get operation location
        operation_location = response.headers.get('operation-location')
        print("Healthcare analysis job submitted...")
        
        # Poll for completion
        while True:
            status_response = requests.get(operation_location, headers={
                "Ocp-Apim-Subscription-Key": key
            })
            
            if status_response.status_code == 200:
                result = status_response.json()
                
                if result['status'] == 'succeeded':
                    print("Analysis completed!")
                    
                    # Process results
                    for task in result['tasks']['items']:
                        if task['kind'] == 'HealthcareLROResults':
                            for document in task['results']['documents']:
                                print(f"\nDocument: {document['id']}")
                                print("Healthcare Entities:")
                                
                                # Group entities by category
                                entities_by_category = {}
                                for entity in document['entities']:
                                    category = entity['category']
                                    if category not in entities_by_category:
                                        entities_by_category[category] = []
                                    entities_by_category[category].append(entity)
                                
                                for category, entities in entities_by_category.items():
                                    print(f"  {category}:")
                                    for entity in entities:
                                        print(f"    - {entity['text']} (confidence: {entity['confidenceScore']:.2f})")
                                        if 'normalizedText' in entity:
                                            print(f"      Normalized: {entity['normalizedText']}")
                                
                                # Display relations
                                if 'relations' in document and document['relations']:
                                    print("\n  Entity Relations:")
                                    for relation in document['relations']:
                                        print(f"    Relation: {relation['relationType']}")
                                        for role in relation['roles']:
                                            print(f"      {role['name']}: {role['entity']['text']}")
                    break
                    
                elif result['status'] == 'failed':
                    print(f"Analysis failed: {result.get('errors', 'Unknown error')}")
                    break
                else:
                    print(f"Status: {result['status']}")
                    time.sleep(2)
            else:
                print(f"Error checking status: {status_response.status_code}")
                break
    else:
        print(f"Error submitting job: {response.status_code} - {response.text}")

healthcare_analysis_rest()
```

## 📊 Response Format

### Healthcare Entities Response

```json
{
  "jobId": "healthcare-job-456",
  "lastUpdateDateTime": "2024-01-20T15:30:00Z",
  "createdDateTime": "2024-01-20T15:29:30Z",
  "expirationDateTime": "2024-01-21T15:29:30Z",
  "status": "succeeded",
  "tasks": {
    "completed": 1,
    "failed": 0,
    "inProgress": 0,
    "total": 1,
    "items": [
      {
        "kind": "HealthcareLROResults",
        "taskName": "Healthcare Analysis",
        "lastUpdateDateTime": "2024-01-20T15:30:00Z",
        "results": {
          "documents": [
            {
              "id": "1",
              "entities": [
                {
                  "offset": 26,
                  "length": 5,
                  "text": "100mg",
                  "category": "Dosage",
                  "confidenceScore": 0.99
                },
                {
                  "offset": 32,
                  "length": 9,
                  "text": "ibuprofen",
                  "category": "MedicationName",
                  "confidenceScore": 0.97,
                  "normalizedText": "Ibuprofen",
                  "dataSources": [
                    {
                      "entityId": "C0020740",
                      "name": "UMLS"
                    }
                  ]
                },
                {
                  "offset": 42,
                  "length": 11,
                  "text": "twice daily",
                  "category": "Frequency",
                  "confidenceScore": 0.95
                },
                {
                  "offset": 58,
                  "length": 15,
                  "text": "pain management",
                  "category": "TreatmentName",
                  "confidenceScore": 0.89,
                  "assertion": {
                    "certainty": "positive",
                    "conditionality": "none",
                    "association": "other"
                  }
                }
              ],
              "relations": [
                {
                  "relationType": "DosageOfMedication",
                  "roles": [
                    {
                      "entity": {
                        "offset": 26,
                        "length": 5,
                        "text": "100mg"
                      },
                      "name": "Dosage"
                    },
                    {
                      "entity": {
                        "offset": 32,
                        "length": 9,
                        "text": "ibuprofen"
                      },
                      "name": "Medication"
                    }
                  ]
                },
                {
                  "relationType": "FrequencyOfMedication",
                  "roles": [
                    {
                      "entity": {
                        "offset": 42,
                        "length": 11,
                        "text": "twice daily"
                      },
                      "name": "Frequency"
                    },
                    {
                      "entity": {
                        "offset": 32,
                        "length": 9,
                        "text": "ibuprofen"
                      },
                      "name": "Medication"
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```

## 🎯 Healthcare Entity Categories

### Medication-Related

| Category | Description | Examples |
|----------|-------------|----------|
| **MedicationName** | Drug names | "aspirin", "metformin", "lisinopril" |
| **Dosage** | Medication dosage | "100mg", "2 tablets", "5ml" |
| **Frequency** | Administration frequency | "twice daily", "BID", "q6h" |
| **RouteOrMode** | Administration route | "oral", "IV", "topical" |
| **MedicationClass** | Drug classifications | "antibiotic", "beta blocker" |

### Clinical Conditions

| Category | Description | Examples |
|----------|-------------|----------|
| **SymptomOrSign** | Clinical symptoms | "chest pain", "fever", "shortness of breath" |
| **Diagnosis** | Medical diagnoses | "pneumonia", "diabetes", "hypertension" |
| **ConditionName** | General conditions | "asthma", "arthritis", "depression" |

### Medical Procedures

| Category | Description | Examples |
|----------|-------------|----------|
| **TreatmentName** | Medical treatments | "surgery", "physical therapy", "radiation" |
| **ExaminationName** | Medical examinations | "CT scan", "blood test", "ECG" |

### Measurements and Values

| Category | Description | Examples |
|----------|-------------|----------|
| **MeasurementValue** | Numeric measurements | "120/80", "98.6°F", "15%" |
| **MeasurementUnit** | Units of measurement | "mmHg", "mg/dL", "bpm" |

### Anatomical References

| Category | Description | Examples |
|----------|-------------|----------|
| **BodyStructure** | Body parts/organs | "heart", "liver", "left arm" |
| **Direction** | Anatomical directions | "left", "right", "anterior" |

## 🔧 Best Practices

### Clinical Document Processing Pipeline

```python
async def clinical_document_pipeline(documents):
    """Complete pipeline for clinical document analysis"""
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        results = {
            'healthcare_entities': [],
            'phi_entities': [],
            'summary_stats': {}
        }
        
        # Step 1: Healthcare entity analysis
        healthcare_operation = await client.begin_analyze_healthcare_entities(documents)
        healthcare_result = await healthcare_operation.result()
        
        async for page in healthcare_result:
            for idx, doc_result in enumerate(page):
                if not doc_result.is_error:
                    # Extract structured information
                    doc_analysis = {
                        'document_id': idx,
                        'entities_by_category': {},
                        'relations': [],
                        'assertions': []
                    }
                    
                    # Group entities by category
                    for entity in doc_result.entities:
                        category = entity.category
                        if category not in doc_analysis['entities_by_category']:
                            doc_analysis['entities_by_category'][category] = []
                        
                        entity_info = {
                            'text': entity.text,
                            'confidence': entity.confidence_score,
                            'normalized_text': entity.normalized_text,
                            'offset': entity.offset,
                            'length': entity.length
                        }
                        
                        # Add assertion information if available
                        if hasattr(entity, 'assertion') and entity.assertion:
                            entity_info['assertion'] = {
                                'certainty': entity.assertion.certainty,
                                'conditionality': entity.assertion.conditionality,
                                'association': entity.assertion.association
                            }
                            doc_analysis['assertions'].append(entity_info)
                        
                        doc_analysis['entities_by_category'][category].append(entity_info)
                    
                    # Extract relations
                    for relation in doc_result.entity_relations:
                        relation_info = {
                            'type': relation.relation_type,
                            'confidence': relation.confidence_score if hasattr(relation, 'confidence_score') else None,
                            'roles': []
                        }
                        
                        for role in relation.roles:
                            relation_info['roles'].append({
                                'name': role.name,
                                'entity_text': role.entity.text,
                                'entity_category': role.entity.category
                            })
                        
                        doc_analysis['relations'].append(relation_info)
                    
                    results['healthcare_entities'].append(doc_analysis)
        
        # Step 2: PHI detection
        phi_response = client.recognize_pii_entities(documents, domain_filter="phi")
        
        for idx, doc in enumerate(phi_response):
            if not doc.is_error:
                phi_info = {
                    'document_id': idx,
                    'redacted_text': doc.redacted_text,
                    'phi_entities': []
                }
                
                for entity in doc.entities:
                    phi_info['phi_entities'].append({
                        'text': entity.text,
                        'category': entity.category,
                        'confidence': entity.confidence_score,
                        'offset': entity.offset,
                        'length': entity.length
                    })
                
                results['phi_entities'].append(phi_info)
        
        # Step 3: Generate summary statistics
        total_healthcare_entities = sum(len(doc['entities_by_category']) for doc in results['healthcare_entities'])
        total_phi_entities = sum(len(doc['phi_entities']) for doc in results['phi_entities'])
        
        results['summary_stats'] = {
            'total_documents': len(documents),
            'total_healthcare_entities': total_healthcare_entities,
            'total_phi_entities': total_phi_entities,
            'processing_timestamp': asyncio.get_event_loop().time()
        }
        
        return results

# Example usage
clinical_documents = [
    "Patient John Smith was prescribed metformin 500mg twice daily for diabetes management.",
    "Blood pressure reading was 140/90 mmHg. Patient has history of hypertension."
]

pipeline_results = asyncio.run(clinical_document_pipeline(clinical_documents))
print(json.dumps(pipeline_results, indent=2))
```

### Medical Concept Extraction

```python
async def extract_medical_concepts(clinical_text):
    """Extract and categorize key medical concepts"""
    
    async with TextAnalyticsClient(endpoint=endpoint, credential=credential) as client:
        
        operation = await client.begin_analyze_healthcare_entities([clinical_text])
        result = await operation.result()
        
        medical_concepts = {
            'medications': [],
            'conditions': [],
            'procedures': [],
            'measurements': [],
            'anatomy': []
        }
        
        async for page in result:
            for doc_result in page:
                if not doc_result.is_error:
                    for entity in doc_result.entities:
                        concept = {
                            'text': entity.text,
                            'confidence': entity.confidence_score,
                            'normalized_text': entity.normalized_text,
                            'umls_codes': []
                        }
                        
                        # Add UMLS codes if available
                        if hasattr(entity, 'data_sources') and entity.data_sources:
                            for data_source in entity.data_sources:
                                if data_source.name == "UMLS":
                                    concept['umls_codes'].append(data_source.entity_id)
                        
                        # Categorize by entity type
                        if entity.category in ['MedicationName', 'Dosage', 'Frequency']:
                            medical_concepts['medications'].append(concept)
                        elif entity.category in ['Diagnosis', 'SymptomOrSign', 'ConditionName']:
                            medical_concepts['conditions'].append(concept)
                        elif entity.category in ['TreatmentName', 'ExaminationName']:
                            medical_concepts['procedures'].append(concept)
                        elif entity.category in ['MeasurementValue', 'MeasurementUnit']:
                            medical_concepts['measurements'].append(concept)
                        elif entity.category in ['BodyStructure', 'Direction']:
                            medical_concepts['anatomy'].append(concept)
        
        return medical_concepts

# Example usage
clinical_note = """
Patient underwent cardiac catheterization which revealed 70% stenosis of the LAD.
Treatment plan includes atorvastatin 40mg daily and clopidogrel 75mg daily.
Blood pressure was 135/85 mmHg. Patient will follow up in cardiology clinic.
"""

concepts = asyncio.run(extract_medical_concepts(clinical_note))
for category, items in concepts.items():
    if items:
        print(f"{category.upper()}:")
        for item in items:
            print(f"  - {item['text']} (confidence: {item['confidence']:.2f})")
            if item['umls_codes']:
                print(f"    UMLS: {', '.join(item['umls_codes'])}")
```

## 📚 Additional Resources

- **[What is Azure AI Language for Health?](https://learn.microsoft.com/en-us/azure/ai-services/language-service/text-analytics-for-health/overview)**
- **[Healthcare Entity Categories](https://learn.microsoft.com/en-us/azure/ai-services/language-service/text-analytics-for-health/concepts/health-entity-categories)**
- **[Assertion Detection](https://learn.microsoft.com/en-us/azure/ai-services/language-service/text-analytics-for-health/concepts/assertion-detection)**
- **[Entity Relations](https://learn.microsoft.com/en-us/azure/ai-services/language-service/text-analytics-for-health/concepts/relation-extraction)**
- **[REST API Reference](https://learn.microsoft.com/rest/api/language/text-analysis-runtime/)**

## 🎯 AI-102 Exam Tips

- Understand the difference between general NER and healthcare-specific entity recognition
- Know the main healthcare entity categories and their relationships
- Practice with assertion detection (negation, uncertainty, conditional)
- Understand entity linking to medical ontologies (UMLS, ICD-10, SNOMED)
- Know when to use synchronous vs asynchronous processing for healthcare documents
- Practice with PHI detection in healthcare contexts
- Understand the relationship between dosage, medication, and frequency entities