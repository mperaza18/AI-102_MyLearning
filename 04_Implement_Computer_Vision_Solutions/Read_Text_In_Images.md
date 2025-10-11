# 📄 Read Text in Images (OCR)

> **Comprehensive code examples for optical character recognition using Azure AI Vision Read API**

## 📋 Overview

Azure AI Vision's Read API provides state-of-the-art optical character recognition (OCR) capabilities for extracting printed and handwritten text from images and documents. This powerful service can handle various languages, fonts, and document layouts with high accuracy.

## 🔧 Key Features

- **📝 Text Extraction** - Extract printed and handwritten text from images
- **🌍 Multi-language Support** - Support for over 70 languages
- **📐 Layout Analysis** - Preserve text layout with bounding boxes
- **🎯 High Accuracy** - Advanced OCR models with confidence scores
- **📄 Document Processing** - Handle complex documents and forms
- **🖋️ Handwriting Recognition** - Extract handwritten text with high precision
- **📊 Table Detection** - Identify and extract tabular data

## 🐍 Python Examples

### New SDK (Azure AI Vision 4.0) - Recommended

```python
import os
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential

# Set up authentication
endpoint = os.environ["VISION_ENDPOINT"]
key = os.environ["VISION_KEY"]

# Create client
client = ImageAnalysisClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)

# Extract text from URL
image_url = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"

result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[VisualFeatures.READ]
)

# Print text extraction results
print("Image analysis results:")
print(" Read:")
if result.read is not None:
    for line in result.read.blocks[0].lines:
        print(f"   Line: '{line.text}', Bounding box {line.bounding_polygon}")
        for word in line.words:
            print(f"     Word: '{word.text}', Confidence {word.confidence:.4f}, Bounding polygon {word.bounding_polygon}")
```

### Extract Text from Local File

```python
# Load image from local file
with open("sample.jpg", "rb") as f:
    image_data = f.read()

# Extract text from image data
result = client.analyze(
    image_data=image_data,
    visual_features=[VisualFeatures.READ]
)

# Print results
print("Image analysis results:")
print(" Read:")
if result.read is not None:
    for line in result.read.blocks[0].lines:
        print(f"   Line: '{line.text}', Bounding box {line.bounding_polygon}")
        for word in line.words:
            print(f"     Word: '{word.text}', Confidence {word.confidence:.4f}, Bounding polygon {word.bounding_polygon}")
```

### Legacy SDK (Computer Vision 3.2)

```python
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
import os
import time

# Authentication
subscription_key = os.environ["VISION_KEY"]
endpoint = os.environ["VISION_ENDPOINT"]

computervision_client = ComputerVisionClient(endpoint, CognitiveServicesCredentials(subscription_key))

# OCR: Read File using the Read API - remote
print("===== Read File - remote =====")
read_image_url = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"

# Call API with URL and raw response (allows you to get the operation location)
read_response = computervision_client.read(read_image_url, raw=True)

# Get the operation location (URL with an ID at the end) from the response
read_operation_location = read_response.headers["Operation-Location"]
# Grab the ID from the URL
operation_id = read_operation_location.split("/")[-1]

# Call the "GET" API and wait for it to retrieve the results 
while True:
    read_result = computervision_client.get_read_result(operation_id)
    if read_result.status not in ['notStarted', 'running']:
        break
    time.sleep(1)

# Print the detected text, line by line
if read_result.status == OperationStatusCodes.succeeded:
    for text_result in read_result.analyze_result.read_results:
        for line in text_result.lines:
            print(line.text)
            print(line.bounding_box)
print()
```

### REST API Example

```python
import requests
import json
import os
import time

# Configuration
endpoint = os.environ["VISION_ENDPOINT"]
subscription_key = os.environ["VISION_KEY"]
read_url = endpoint + "vision/v3.2/read/analyze"

# Image to analyze
image_url = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"

# Request headers
headers = {
    'Ocp-Apim-Subscription-Key': subscription_key,
    'Content-Type': 'application/json'
}

# Request body
data = {'url': image_url}

# Make the initial request
response = requests.post(read_url, headers=headers, json=data)
response.raise_for_status()

# Get operation location from headers
operation_location = response.headers["Operation-Location"]
operation_id = operation_location.split("/")[-1]

# Poll for results
get_read_result_url = endpoint + f"vision/v3.2/read/analyzeResults/{operation_id}"

while True:
    read_result = requests.get(get_read_result_url, headers={'Ocp-Apim-Subscription-Key': subscription_key})
    read_result_json = read_result.json()
    
    if read_result_json["status"] not in ["notStarted", "running"]:
        break
    time.sleep(1)

# Print results
if read_result_json["status"] == "succeeded":
    for read_result in read_result_json["analyzeResult"]["readResults"]:
        for line in read_result["lines"]:
            print(f"Text: {line['text']}")
            print(f"Bounding box: {line['boundingBox']}")
```

## 🔷 C# Examples

### New SDK (Azure AI Vision 4.0) - Recommended

```csharp
using Azure;
using Azure.AI.Vision.ImageAnalysis;
using System;

public class Program
{
    static void ExtractText()
    {
        string endpoint = Environment.GetEnvironmentVariable("VISION_ENDPOINT");
        string key = Environment.GetEnvironmentVariable("VISION_KEY");

        ImageAnalysisClient client = new ImageAnalysisClient(
            new Uri(endpoint),
            new AzureKeyCredential(key));

        // Extract text from URL
        ImageAnalysisResult result = client.Analyze(
            new Uri("https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"),
            VisualFeatures.Read);

        Console.WriteLine("Image analysis results:");
        Console.WriteLine(" Read:");

        foreach (DetectedTextBlock block in result.Read.Blocks)
            foreach (DetectedTextLine line in block.Lines)
            {
                Console.WriteLine($"   Line: '{line.Text}', Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
                foreach (DetectedTextWord word in line.Words)
                {
                    Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence.ToString("#.####")}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
                }
            }
    }

    static void Main()
    {
        try
        {
            ExtractText();
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }
    }
}
```

### Extract Text from Local File

```csharp
// Load image from local file
using FileStream stream = new FileStream("image-analysis-sample.jpg", FileMode.Open);

// Extract text from image stream
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Read);

// Print text analysis results
Console.WriteLine("Image analysis results:");
Console.WriteLine(" Read:");

foreach (DetectedTextBlock block in result.Read.Blocks)
    foreach (DetectedTextLine line in block.Lines)
    {
        Console.WriteLine($"   Line: '{line.Text}', Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
        foreach (DetectedTextWord word in line.Words)
        {
            Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence.ToString("#.####")}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        }
    }
```

### Legacy SDK (Computer Vision 3.2)

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
using System.Threading.Tasks;
using System.Threading;

namespace ComputerVisionOCR
{
    class Program
    {
        static string key = Environment.GetEnvironmentVariable("VISION_KEY");
        static string endpoint = Environment.GetEnvironmentVariable("VISION_ENDPOINT");

        static async Task Main(string[] args)
        {
            ComputerVisionClient client = new ComputerVisionClient(new ApiKeyServiceClientCredentials(key))
            { 
                Endpoint = endpoint 
            };

            string imageUrl = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png";

            await ReadFileUrl(client, imageUrl);
        }

        public static async Task ReadFileUrl(ComputerVisionClient client, string urlFile)
        {
            Console.WriteLine("----------------------------------------------------------");
            Console.WriteLine("READ FILE FROM URL");
            Console.WriteLine();

            // Read text from URL
            var textHeaders = await client.ReadAsync(urlFile);
            // After the request, get the operation location (operation ID)
            string operationLocation = textHeaders.OperationLocation;
            Thread.Sleep(2000);

            // Retrieve the URI where the extracted text will be stored from the Operation-Location header.
            // We only need the ID and not the full URL
            const int numberOfCharsInOperationId = 36;
            string operationId = operationLocation.Substring(operationLocation.Length - numberOfCharsInOperationId);

            // Extract the text
            ReadOperationResult results;
            Console.WriteLine($"Extracting text from URL file {Path.GetFileName(urlFile)}...");
            Console.WriteLine();
            do
            {
                results = await client.GetReadResultAsync(Guid.Parse(operationId));
            }
            while ((results.Status == OperationStatusCodes.Running ||
                results.Status == OperationStatusCodes.NotStarted));

            // Display the found text
            Console.WriteLine();
            var textUrlFileResults = results.AnalyzeResult.ReadResults;
            foreach (ReadResult page in textUrlFileResults)
            {
                foreach (Line line in page.Lines)
                {
                    Console.WriteLine(line.Text);
                    Console.WriteLine($"Bounding box: [{string.Join(",", line.BoundingBox)}]");
                }
            }
            Console.WriteLine();
        }
    }
}
```

### REST API Example

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.Threading;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

class Program
{
    private static readonly string endpoint = Environment.GetEnvironmentVariable("VISION_ENDPOINT");
    private static readonly string subscriptionKey = Environment.GetEnvironmentVariable("VISION_KEY");

    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            // Initial request to start OCR
            var readUri = $"{endpoint}vision/v3.2/read/analyze";
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

            var requestData = new
            {
                url = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"
            };

            var content = new StringContent(JsonConvert.SerializeObject(requestData), Encoding.UTF8, "application/json");
            var response = await client.PostAsync(readUri, content);

            // Get operation location
            string operationLocation = response.Headers.GetValues("Operation-Location").FirstOrDefault();
            string operationId = operationLocation.Split('/').Last();

            // Poll for results
            string getResultUri = $"{endpoint}vision/v3.2/read/analyzeResults/{operationId}";
            
            JObject result;
            do
            {
                Thread.Sleep(1000);
                var resultResponse = await client.GetAsync(getResultUri);
                var resultContent = await resultResponse.Content.ReadAsStringAsync();
                result = JObject.Parse(resultContent);
            }
            while (result["status"].ToString() == "running" || result["status"].ToString() == "notStarted");

            // Display results
            if (result["status"].ToString() == "succeeded")
            {
                var readResults = result["analyzeResult"]["readResults"];
                foreach (var page in readResults)
                {
                    foreach (var line in page["lines"])
                    {
                        Console.WriteLine($"Text: {line["text"]}");
                        Console.WriteLine($"Bounding box: [{string.Join(",", line["boundingBox"])}]");
                    }
                }
            }
        }
    }
}
```

## 🔗 Key Endpoints

- **Read API (3.2):** `https://<resource-name>.cognitiveservices.azure.com/vision/v3.2/read/analyze`
- **Image Analysis 4.0:** `https://<resource-name>.cognitiveservices.azure.com/computervision/imageanalysis:analyze?api-version=2024-02-01&features=read`

## 📋 Required NuGet Packages (C#)

```xml
<!-- New SDK (Recommended) -->
<PackageReference Include="Azure.AI.Vision.ImageAnalysis" Version="1.0.0-beta.3" />

<!-- Legacy SDK -->
<PackageReference Include="Microsoft.Azure.CognitiveServices.Vision.ComputerVision" Version="7.0.1" />
<PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
```

## 📋 Required Python Packages

```bash
# New SDK (Recommended)
pip install azure-ai-vision-imageanalysis

# Legacy SDK  
pip install azure-cognitiveservices-vision-computervision
pip install azure-core
```

## 🚀 Quick Start Tips

1. **🔑 Authentication:** Set up environment variables for `VISION_ENDPOINT` and `VISION_KEY`
2. **📄 File Support:** Supports JPEG, PNG, BMP, PDF, and TIFF formats
3. **📏 Size Limits:** Image files must be less than 50 MB
4. **🌍 Languages:** Supports 70+ languages for printed text and several for handwritten text
5. **⚡ Performance:** Use the new 4.0 SDK for synchronous processing
6. **📊 Confidence:** Review confidence scores to assess OCR quality

## 📖 Use Cases

- **📄 Document Digitization** - Convert physical documents to digital text
- **🏢 Business Card Processing** - Extract contact information
- **📝 Form Processing** - Digitize handwritten forms and applications
- **🔍 Content Moderation** - Extract text from images for content review
- **♿ Accessibility** - Create accessible content from image-based text
- **📚 Archive Processing** - Digitize historical documents and books

## 📖 Related Documentation

- [Azure AI Vision Read API Overview](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview-ocr)
- [OCR Quickstart Guide](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/client-library)
- [Language Support](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/language-support)