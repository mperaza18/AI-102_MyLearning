# 🖼️ Analyze Images with Azure AI Vision

> **Comprehensive code examples for image analysis using Azure AI Vision services**

## 📋 Overview

Azure AI Vision's Analyze Image API extracts rich visual information from images including objects, people, tags, colors, faces, and text. This capability supports both REST API and SDK approaches across multiple programming languages.

## 🔧 Key Features

- **📸 Object Detection** - Identify and locate objects with bounding boxes
- **🏷️ Tagging** - Generate descriptive tags with confidence scores  
- **📝 Descriptions** - Create natural language captions
- **👥 Face Detection** - Detect faces with attributes like age and gender
- **🎨 Color Analysis** - Extract dominant colors and schemes
- **🏢 Brand Recognition** - Identify well-known brands and logos
- **🌟 Celebrity Recognition** - Detect famous people
- **🏛️ Landmark Detection** - Identify famous landmarks

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

# Analyze image from URL
image_url = "https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"

result = client.analyze_from_url(
    image_url=image_url,
    visual_features=[
        VisualFeatures.CAPTION, 
        VisualFeatures.READ,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE
    ],
    gender_neutral_caption=True
)

# Print results
print("Image analysis results:")

# Caption
if result.caption is not None:
    print(f"Caption: '{result.caption.text}', Confidence {result.caption.confidence:.4f}")

# Tags
if result.tags is not None:
    print("Tags:")
    for tag in result.tags.list:
        print(f"  '{tag.name}' with confidence {tag.confidence:.4f}")

# Objects
if result.objects is not None:
    print("Objects:")
    for obj in result.objects.list:
        print(f"  '{obj.tags[0].name}' at {obj.bounding_box}")

# Read (OCR)
if result.read is not None:
    print("Text (OCR):")
    for line in result.read.blocks[0].lines:
        print(f"  Line: '{line.text}', Bounding box {line.bounding_polygon}")
```

### Legacy SDK (Computer Vision 3.2)

```python
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
import os

# Authentication
subscription_key = os.environ["VISION_KEY"]
endpoint = os.environ["VISION_ENDPOINT"]

computervision_client = ComputerVisionClient(endpoint, CognitiveServicesCredentials(subscription_key))

# Image URL to analyze
remote_image_url = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg"

# Select visual features
remote_image_features = [
    VisualFeatureTypes.categories,
    VisualFeatureTypes.brands,
    VisualFeatureTypes.adult,
    VisualFeatureTypes.color,
    VisualFeatureTypes.description,
    VisualFeatureTypes.faces,
    VisualFeatureTypes.image_type,
    VisualFeatureTypes.objects,
    VisualFeatureTypes.tags
]

# Analyze the image
results_remote = computervision_client.analyze_image(remote_image_url, remote_image_features)

# Print results
print("Categories:")
for category in results_remote.categories:
    print(f"'{category.name}' with confidence {category.score * 100:.2f}%")

print("\nDescription:")
for caption in results_remote.description.captions:
    print(f"'{caption.text}' with confidence {caption.confidence * 100:.2f}%")

print("\nTags:")
for tag in results_remote.tags:
    print(f"'{tag.name}' with confidence {tag.confidence * 100:.2f}%")

print("\nObjects:")
for obj in results_remote.objects:
    print(f"'{obj.object_property}' at location {obj.rectangle.x}, {obj.rectangle.y}, {obj.rectangle.w}, {obj.rectangle.h}")

print("\nFaces:")
for face in results_remote.faces:
    print(f"'{face.gender}' of age {face.age} at location {face.face_rectangle.left}, {face.face_rectangle.top}")
```

### REST API Example

```python
import requests
import json
import os

# Configuration
endpoint = os.environ["VISION_ENDPOINT"]
subscription_key = os.environ["VISION_KEY"]
analyze_url = endpoint + "vision/v3.2/analyze"

# Image to analyze
image_url = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg"

# Request headers
headers = {
    'Ocp-Apim-Subscription-Key': subscription_key,
    'Content-Type': 'application/json'
}

# Request parameters
params = {
    'visualFeatures': 'Categories,Description,Faces,Objects,Tags,Adult,Brands,Color,ImageType',
    'details': 'Celebrities,Landmarks',
    'language': 'en'
}

# Request body
data = {'url': image_url}

# Make the request
response = requests.post(analyze_url, headers=headers, params=params, json=data)
response.raise_for_status()

# Parse the response
analysis = response.json()
print(json.dumps(analysis, indent=2))
```

## 🔷 C# Examples

### New SDK (Azure AI Vision 4.0) - Recommended

```csharp
using Azure;
using Azure.AI.Vision.ImageAnalysis;
using System;

public class Program
{
    static void AnalyzeImage()
    {
        string endpoint = Environment.GetEnvironmentVariable("VISION_ENDPOINT");
        string key = Environment.GetEnvironmentVariable("VISION_KEY");

        ImageAnalysisClient client = new ImageAnalysisClient(
            new Uri(endpoint),
            new AzureKeyCredential(key));

        ImageAnalysisResult result = client.Analyze(
            new Uri("https://learn.microsoft.com/azure/ai-services/computer-vision/media/quickstarts/presentation.png"),
            VisualFeatures.Caption | VisualFeatures.Read | VisualFeatures.Tags | VisualFeatures.Objects,
            new ImageAnalysisOptions { GenderNeutralCaption = true });

        Console.WriteLine("Image analysis results:");
        
        // Caption
        Console.WriteLine(" Caption:");
        Console.WriteLine($"   '{result.Caption.Text}', Confidence {result.Caption.Confidence:F4}");

        // Tags
        Console.WriteLine(" Tags:");
        foreach (DetectedTag tag in result.Tags.Values)
        {
            Console.WriteLine($"   '{tag.Name}' with confidence {tag.Confidence:F4}");
        }

        // Objects
        Console.WriteLine(" Objects:");
        foreach (DetectedObject detectedObject in result.Objects.Values)
        {
            Console.WriteLine($"   '{detectedObject.Tags.First().Name}' with confidence {detectedObject.Tags.First().Confidence:F4}");
            Console.WriteLine($"     Bounding box: {detectedObject.BoundingBox}");
        }

        // Read (OCR)
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
            AnalyzeImage();
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }
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

namespace ComputerVisionAnalyze
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

            string imageUrl = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg";

            await AnalyzeImageUrl(client, imageUrl);
        }

        public static async Task AnalyzeImageUrl(ComputerVisionClient client, string imageUrl)
        {
            Console.WriteLine("ANALYZE IMAGE - URL");

            // Creating a list that defines the features to be extracted from the image
            List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
            {
                VisualFeatureTypes.Categories, VisualFeatureTypes.Description,
                VisualFeatureTypes.Faces, VisualFeatureTypes.ImageType,
                VisualFeatureTypes.Tags, VisualFeatureTypes.Adult,
                VisualFeatureTypes.Color, VisualFeatureTypes.Brands,
                VisualFeatureTypes.Objects
            };

            // Analyze the URL image 
            ImageAnalysis results = await client.AnalyzeImageAsync(imageUrl, visualFeatures: features);

            // Summarizes the image content
            Console.WriteLine("Summary:");
            foreach (var caption in results.Description.Captions)
            {
                Console.WriteLine($"{caption.Text} with confidence {caption.Confidence}");
            }

            // Display categories
            Console.WriteLine("Categories:");
            foreach (var category in results.Categories)
            {
                Console.WriteLine($"{category.Name} with confidence {category.Score}");
            }

            // Image tags and their confidence score
            Console.WriteLine("Tags:");
            foreach (var tag in results.Tags)
            {
                Console.WriteLine($"{tag.Name} {tag.Confidence}");
            }

            // Objects
            Console.WriteLine("Objects:");
            foreach (var obj in results.Objects)
            {
                Console.WriteLine($"{obj.ObjectProperty} with confidence {obj.Confidence} at location {obj.Rectangle.X}, " +
                  $"{obj.Rectangle.X + obj.Rectangle.W}, {obj.Rectangle.Y}, {obj.Rectangle.Y + obj.Rectangle.H}");
            }

            // Faces
            Console.WriteLine("Faces:");
            foreach (var face in results.Faces)
            {
                Console.WriteLine($"A {face.Gender} of age {face.Age} at location {face.FaceRectangle.Left}, " +
                  $"{face.FaceRectangle.Left}, {face.FaceRectangle.Top + face.FaceRectangle.Width}, " +
                  $"{face.FaceRectangle.Top + face.FaceRectangle.Height}");
            }

            // Color scheme
            Console.WriteLine("Color Scheme:");
            Console.WriteLine("Is black and white?: " + results.Color.IsBWImg);
            Console.WriteLine("Accent color: " + results.Color.AccentColor);
            Console.WriteLine("Dominant background color: " + results.Color.DominantColorBackground);
            Console.WriteLine("Dominant foreground color: " + results.Color.DominantColorForeground);
            Console.WriteLine("Dominant colors: " + string.Join(",", results.Color.DominantColors));
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
using Newtonsoft.Json;

class Program
{
    private static readonly string endpoint = Environment.GetEnvironmentVariable("VISION_ENDPOINT");
    private static readonly string subscriptionKey = Environment.GetEnvironmentVariable("VISION_KEY");

    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            var uri = $"{endpoint}vision/v3.2/analyze?visualFeatures=Categories,Description,Faces,Objects,Tags,Adult,Brands,Color,ImageType&details=Celebrities,Landmarks&language=en";

            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);

            var requestData = new
            {
                url = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/landmark.jpg"
            };

            var content = new StringContent(JsonConvert.SerializeObject(requestData), Encoding.UTF8, "application/json");

            var response = await client.PostAsync(uri, content);
            var responseContent = await response.Content.ReadAsStringAsync();

            Console.WriteLine(JsonConvert.SerializeObject(JsonConvert.DeserializeObject(responseContent), Formatting.Indented));
        }
    }
}
```

## 🔗 Key Endpoints

- **Analyze Image API:** `https://<resource-name>.cognitiveservices.azure.com/vision/v3.2/analyze`
- **Image Analysis 4.0:** `https://<resource-name>.cognitiveservices.azure.com/computervision/imageanalysis:analyze?api-version=2024-02-01`

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
2. **📊 Choose Features:** Select only the visual features you need to optimize performance
3. **🌐 Image Sources:** Support both URLs and local file uploads
4. **⚡ Performance:** Use the new 4.0 SDK for better performance and features
5. **🔒 Security:** Never hardcode credentials in your source code

## 📖 Related Documentation

- [Azure AI Vision Overview](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)
- [Image Analysis 4.0 Quickstart](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/quickstarts-sdk/image-analysis-client-library-40)
- [REST API Reference](https://learn.microsoft.com/en-us/rest/api/computervision/)