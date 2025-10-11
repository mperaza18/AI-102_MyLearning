# 🎯 Detect Objects in Images

## 📚 Overview

Object detection identifies and locates multiple objects within images, providing both classification labels and bounding box coordinates. Azure Custom Vision and other Azure services enable you to build custom object detection models for specific use cases.

## 🎯 Learning Objectives

- ✅ Create and configure Custom Vision projects for object detection
- ✅ Upload and annotate training images with bounding boxes
- ✅ Train custom object detection models
- ✅ Handle prediction results with confidence scores and coordinates
- ✅ Visualize detection results with bounding boxes

## 🛠️ Implementation Options

### 1️⃣ Azure Custom Vision Object Detection

#### Python Implementation

```python
from azure.cognitiveservices.vision.customvision.training import CustomVisionTrainingClient
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from azure.cognitiveservices.vision.customvision.training.models import ImageFileCreateBatch, ImageFileCreateEntry, Region
from msrest.authentication import ApiKeyCredentials
import os
import time
import uuid

# Configuration
ENDPOINT = "https://your-custom-vision-resource.cognitiveservices.azure.com/"
training_key = "your-training-key"
prediction_key = "your-prediction-key"
prediction_resource_id = "/subscriptions/your-subscription/resourceGroups/your-rg/providers/Microsoft.CognitiveServices/accounts/your-prediction-resource"

def authenticate_clients():
    """Authenticate training and prediction clients"""
    
    # Training client
    credentials = ApiKeyCredentials(in_headers={"Training-key": training_key})
    trainer = CustomVisionTrainingClient(ENDPOINT, credentials)
    
    # Prediction client
    prediction_credentials = ApiKeyCredentials(in_headers={"Prediction-key": prediction_key})
    predictor = CustomVisionPredictionClient(ENDPOINT, prediction_credentials)
    
    return trainer, predictor

def create_object_detection_project(trainer):
    """Create a new object detection project"""
    
    publish_iteration_name = "detectModel"
    
    # Find the object detection domain
    obj_detection_domain = next(
        domain for domain in trainer.get_domains() 
        if domain.type == "ObjectDetection" and domain.name == "General"
    )
    
    # Create a new project
    print("Creating object detection project...")
    project = trainer.create_project(str(uuid.uuid4()), domain_id=obj_detection_domain.id)
    print(f"Project created with ID: {project.id}")
    
    return project, publish_iteration_name

def create_tags(trainer, project):
    """Create tags for different object categories"""
    
    # Create tags for objects you want to detect
    fork_tag = trainer.create_tag(project.id, "fork")
    scissors_tag = trainer.create_tag(project.id, "scissors")
    
    print(f"Created tags: {fork_tag.name}, {scissors_tag.name}")
    
    return fork_tag, scissors_tag

def upload_and_tag_images_with_regions(trainer, project, fork_tag, scissors_tag):
    """Upload images with bounding box annotations"""
    
    base_image_location = os.path.join(os.path.dirname(__file__), "Images")
    
    print("Adding images with regions...")
    
    # Fork images with bounding boxes
    fork_image_regions = {
        "fork_1.jpg": [Region(tag_id=fork_tag.id, left=0.145, top=0.348, width=0.314, height=0.616)],
        "fork_2.jpg": [Region(tag_id=fork_tag.id, left=0.294, top=0.12, width=0.412, height=0.81)],
        "fork_3.jpg": [Region(tag_id=fork_tag.id, left=0.09, top=0.42, width=0.618, height=0.496)]
    }
    
    # Scissors images with bounding boxes
    scissors_image_regions = {
        "scissors_1.jpg": [Region(tag_id=scissors_tag.id, left=0.279, top=0.12, width=0.255, height=0.614)],
        "scissors_2.jpg": [Region(tag_id=scissors_tag.id, left=0.348, top=0.07, width=0.456, height=0.849)],
        "scissors_3.jpg": [Region(tag_id=scissors_tag.id, left=0.318, top=0.023, width=0.361, height=0.746)]
    }
    
    # Upload fork images
    image_list = []
    for file_name, regions in fork_image_regions.items():
        try:
            with open(os.path.join(base_image_location, "fork", file_name), "rb") as image_contents:
                image_list.append(ImageFileCreateEntry(
                    name=file_name,
                    contents=image_contents.read(),
                    regions=regions
                ))
        except FileNotFoundError:
            print(f"Fork image not found: {file_name}")
    
    # Upload scissors images
    for file_name, regions in scissors_image_regions.items():
        try:
            with open(os.path.join(base_image_location, "scissors", file_name), "rb") as image_contents:
                image_list.append(ImageFileCreateEntry(
                    name=file_name,
                    contents=image_contents.read(),
                    regions=regions
                ))
        except FileNotFoundError:
            print(f"Scissors image not found: {file_name}")
    
    # Upload the batch
    upload_result = trainer.create_images_from_files(project.id, ImageFileCreateBatch(images=image_list))
    
    if not upload_result.is_batch_successful:
        print("Image batch upload failed.")
        for image in upload_result.images:
            print("Image status: ", image.status)
        return False
    
    print(f"Successfully uploaded {len(image_list)} images with regions")
    return True

def train_object_detection_model(trainer, project):
    """Train the object detection model"""
    
    print("Training...")
    iteration = trainer.train_project(project.id)
    
    # Wait for training to complete
    while iteration.status != "Completed":
        iteration = trainer.get_iteration(project.id, iteration.id)
        print("Training status: " + iteration.status)
        print("Waiting 10 seconds...")
        time.sleep(10)
    
    print("Training completed!")
    return iteration

def publish_iteration(trainer, project, iteration, publish_iteration_name):
    """Publish the trained iteration"""
    
    # Publish the iteration to the project endpoint
    trainer.publish_iteration(project.id, iteration.id, publish_iteration_name, prediction_resource_id)
    print(f"Published iteration: {publish_iteration_name}")

def test_object_detection(predictor, project, publish_iteration_name):
    """Test object detection with a sample image"""
    
    base_image_location = os.path.join(os.path.dirname(__file__), "Images")
    
    try:
        with open(os.path.join(base_image_location, "test", "test_od_image.jpg"), "rb") as test_data:
            results = predictor.detect_image(project.id, publish_iteration_name, test_data)
            
            # Display the results
            print("Object Detection Results:")
            for prediction in results.predictions:
                print(f"\t{prediction.tag_name}: {prediction.probability * 100:.2f}% " +
                      f"bbox.left = {prediction.bounding_box.left:.2f}, " +
                      f"bbox.top = {prediction.bounding_box.top:.2f}, " +
                      f"bbox.width = {prediction.bounding_box.width:.2f}, " +
                      f"bbox.height = {prediction.bounding_box.height:.2f}")
                
    except FileNotFoundError:
        print("Test image not found")

def detect_objects_from_url(predictor, project, publish_iteration_name, image_url):
    """Detect objects in an image from URL"""
    
    results = predictor.detect_image_url(
        project.id,
        publish_iteration_name,
        image_url
    )
    
    print(f"Object Detection Results for {image_url}:")
    detections = []
    
    for prediction in results.predictions:
        if prediction.probability > 0.5:  # Filter by confidence threshold
            detection = {
                "tag": prediction.tag_name,
                "confidence": prediction.probability,
                "bounding_box": {
                    "left": prediction.bounding_box.left,
                    "top": prediction.bounding_box.top,
                    "width": prediction.bounding_box.width,
                    "height": prediction.bounding_box.height
                }
            }
            detections.append(detection)
            print(f"\t{prediction.tag_name}: {prediction.probability * 100:.2f}% " +
                  f"[{prediction.bounding_box.left:.3f}, {prediction.bounding_box.top:.3f}, " +
                  f"{prediction.bounding_box.width:.3f}, {prediction.bounding_box.height:.3f}]")
    
    return detections

# Complete workflow
def main():
    """Main function demonstrating the complete object detection workflow"""
    
    try:
        # Step 1: Authenticate
        trainer, predictor = authenticate_clients()
        
        # Step 2: Create project
        project, publish_iteration_name = create_object_detection_project(trainer)
        
        # Step 3: Create tags
        fork_tag, scissors_tag = create_tags(trainer, project)
        
        # Step 4: Upload and annotate images
        if upload_and_tag_images_with_regions(trainer, project, fork_tag, scissors_tag):
            
            # Step 5: Train the model
            iteration = train_object_detection_model(trainer, project)
            
            # Step 6: Publish the iteration
            publish_iteration(trainer, project, iteration, publish_iteration_name)
            
            # Step 7: Test object detection
            print("\nTesting object detection...")
            test_object_detection(predictor, project, publish_iteration_name)
            
            # Test with URL
            test_url = "https://example.com/test-objects.jpg"
            detect_objects_from_url(predictor, project, publish_iteration_name, test_url)
            
        print("Object detection workflow completed!")
        
    except Exception as e:
        print(f"Error: {e}")

if __name__ == "__main__":
    main()
```

#### C# Implementation

```csharp
using Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training;
using Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training.Models;
using Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;

public class ObjectDetectionService
{
    private readonly string _trainingEndpoint;
    private readonly string _trainingKey;
    private readonly string _predictionEndpoint;
    private readonly string _predictionKey;
    private readonly string _predictionResourceId;
    private readonly string _publishedModelName = "objectDetectionModel";
    
    public ObjectDetectionService(string trainingEndpoint, string trainingKey, 
                                string predictionEndpoint, string predictionKey, 
                                string predictionResourceId)
    {
        _trainingEndpoint = trainingEndpoint;
        _trainingKey = trainingKey;
        _predictionEndpoint = predictionEndpoint;
        _predictionKey = predictionKey;
        _predictionResourceId = predictionResourceId;
    }
    
    public CustomVisionTrainingClient AuthenticateTraining()
    {
        return new CustomVisionTrainingClient(new Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training.ApiKeyServiceClientCredentials(_trainingKey))
        {
            Endpoint = _trainingEndpoint
        };
    }

    public CustomVisionPredictionClient AuthenticatePrediction()
    {
        return new CustomVisionPredictionClient(new Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction.ApiKeyServiceClientCredentials(_predictionKey))
        {
            Endpoint = _predictionEndpoint
        };
    }
    
    public Project CreateObjectDetectionProject(CustomVisionTrainingClient trainingApi)
    {
        Console.WriteLine("Creating object detection project:");
        
        // Find the object detection domain
        var domains = trainingApi.GetDomains();
        var objDetectionDomain = domains.FirstOrDefault(d => d.Type == "ObjectDetection" && d.Name == "General");
        
        return trainingApi.CreateProject("My Object Detection Project", null, objDetectionDomain.Id);
    }
    
    public (Tag, Tag) AddTags(CustomVisionTrainingClient trainingApi, Project project)
    {
        var forkTag = trainingApi.CreateTag(project.Id, "fork");
        var scissorsTag = trainingApi.CreateTag(project.Id, "scissors");
        
        Console.WriteLine($"Created tags: {forkTag.Name}, {scissorsTag.Name}");
        return (forkTag, scissorsTag);
    }
    
    public void UploadImagesWithRegions(CustomVisionTrainingClient trainingApi, Project project, Tag forkTag, Tag scissorsTag)
    {
        var imageFiles = new List<ImageFileCreateEntry>();
        
        // Fork images with bounding boxes
        var forkImages = new Dictionary<string, IList<Region>>
        {
            {"fork_1.jpg", new List<Region> { new Region(forkTag.Id, 0.145, 0.348, 0.314, 0.616) }},
            {"fork_2.jpg", new List<Region> { new Region(forkTag.Id, 0.294, 0.12, 0.412, 0.81) }},
            {"fork_3.jpg", new List<Region> { new Region(forkTag.Id, 0.09, 0.42, 0.618, 0.496) }}
        };
        
        // Scissors images with bounding boxes
        var scissorsImages = new Dictionary<string, IList<Region>>
        {
            {"scissors_1.jpg", new List<Region> { new Region(scissorsTag.Id, 0.279, 0.12, 0.255, 0.614) }},
            {"scissors_2.jpg", new List<Region> { new Region(scissorsTag.Id, 0.348, 0.07, 0.456, 0.849) }},
            {"scissors_3.jpg", new List<Region> { new Region(scissorsTag.Id, 0.318, 0.023, 0.361, 0.746) }}
        };
        
        // Add fork images
        foreach (var imageInfo in forkImages)
        {
            var imagePath = Path.Combine("Images", "fork", imageInfo.Key);
            if (File.Exists(imagePath))
            {
                var imageData = File.ReadAllBytes(imagePath);
                imageFiles.Add(new ImageFileCreateEntry(imageInfo.Key, imageData, null, imageInfo.Value));
            }
        }
        
        // Add scissors images
        foreach (var imageInfo in scissorsImages)
        {
            var imagePath = Path.Combine("Images", "scissors", imageInfo.Key);
            if (File.Exists(imagePath))
            {
                var imageData = File.ReadAllBytes(imagePath);
                imageFiles.Add(new ImageFileCreateEntry(imageInfo.Key, imageData, null, imageInfo.Value));
            }
        }
        
        // Upload images
        var imageBatch = new ImageFileCreateBatch(imageFiles);
        var result = trainingApi.CreateImagesFromFiles(project.Id, imageBatch);
        
        if (!result.IsBatchSuccessful)
        {
            Console.WriteLine("Image batch upload failed.");
            foreach (var image in result.Images)
            {
                Console.WriteLine($"Image status: {image.Status}");
            }
        }
        else
        {
            Console.WriteLine($"Successfully uploaded {imageFiles.Count} images with regions");
        }
    }
    
    public Iteration TrainProject(CustomVisionTrainingClient trainingApi, Project project)
    {
        Console.WriteLine("Training...");
        var iteration = trainingApi.TrainProject(project.Id);

        // Wait for training to complete
        while (iteration.Status == "Training")
        {
            Thread.Sleep(10000);
            iteration = trainingApi.GetIteration(project.Id, iteration.Id);
            Console.WriteLine($"Training status: {iteration.Status}");
        }
        
        Console.WriteLine("Training completed!");
        return iteration;
    }
    
    public void PublishIteration(CustomVisionTrainingClient trainingApi, Project project, Iteration iteration)
    {
        trainingApi.PublishIteration(project.Id, iteration.Id, _publishedModelName, _predictionResourceId);
        Console.WriteLine("Model published successfully");
    }
    
    public void TestIteration(CustomVisionPredictionClient predictionApi, Project project)
    {
        Console.WriteLine("Making object detection prediction:");
        var imageFile = Path.Combine("Images", "test", "test_od_image.jpg");
        
        if (File.Exists(imageFile))
        {
            using (var stream = File.OpenRead(imageFile))
            {
                var result = predictionApi.DetectImage(project.Id, _publishedModelName, stream);

                // Loop over each prediction and write out the results
                foreach (var prediction in result.Predictions)
                {
                    Console.WriteLine($"\t{prediction.TagName}: {prediction.Probability:P1} " +
                                    $"[ {prediction.BoundingBox.Left:F3}, {prediction.BoundingBox.Top:F3}, " +
                                    f"{prediction.BoundingBox.Width:F3}, {prediction.BoundingBox.Height:F3} ]");
                }
            }
        }
        else
        {
            Console.WriteLine("Test image not found");
        }
    }
    
    public List<DetectionResult> DetectObjectsFromUrl(CustomVisionPredictionClient predictionApi, 
                                                     Project project, string imageUrl, double confidenceThreshold = 0.5)
    {
        var result = predictionApi.DetectImageUrl(project.Id, _publishedModelName, new ImageUrl(imageUrl));
        var detections = new List<DetectionResult>();
        
        Console.WriteLine($"Object Detection Results for {imageUrl}:");
        foreach (var prediction in result.Predictions)
        {
            if (prediction.Probability > confidenceThreshold)
            {
                var detection = new DetectionResult
                {
                    TagName = prediction.TagName,
                    Confidence = prediction.Probability,
                    BoundingBox = new BoundingBoxInfo
                    {
                        Left = prediction.BoundingBox.Left,
                        Top = prediction.BoundingBox.Top,
                        Width = prediction.BoundingBox.Width,
                        Height = prediction.BoundingBox.Height
                    }
                };
                
                detections.Add(detection);
                Console.WriteLine($"\t{prediction.TagName}: {prediction.Probability:P1} " +
                                f"[{prediction.BoundingBox.Left:F3}, {prediction.BoundingBox.Top:F3}, " +
                                f"{prediction.BoundingBox.Width:F3}, {prediction.BoundingBox.Height:F3}]");
            }
        }
        
        return detections;
    }
}

// Helper classes
public class DetectionResult
{
    public string TagName { get; set; }
    public double Confidence { get; set; }
    public BoundingBoxInfo BoundingBox { get; set; }
}

public class BoundingBoxInfo
{
    public double Left { get; set; }
    public double Top { get; set; }
    public double Width { get; set; }
    public double Height { get; set; }
}

// Usage example
class Program
{
    static void Main(string[] args)
    {
        var service = new ObjectDetectionService(
            "https://your-training-endpoint.cognitiveservices.azure.com/",
            "your-training-key",
            "https://your-prediction-endpoint.cognitiveservices.azure.com/",
            "your-prediction-key",
            "/subscriptions/your-subscription/resourceGroups/your-rg/providers/Microsoft.CognitiveServices/accounts/your-prediction-resource"
        );
        
        var trainingApi = service.AuthenticateTraining();
        var predictionApi = service.AuthenticatePrediction();
        
        var project = service.CreateObjectDetectionProject(trainingApi);
        var (forkTag, scissorsTag) = service.AddTags(trainingApi, project);
        
        service.UploadImagesWithRegions(trainingApi, project, forkTag, scissorsTag);
        var iteration = service.TrainProject(trainingApi, project);
        service.PublishIteration(trainingApi, project, iteration);
        
        service.TestIteration(predictionApi, project);
        var detections = service.DetectObjectsFromUrl(predictionApi, project, "https://example.com/test.jpg");
    }
}
```

### 2️⃣ Visualization and Post-Processing

#### Python - Visualization with Matplotlib

```python
import matplotlib.pyplot as plt
import matplotlib.patches as patches
import matplotlib.image as mpimg
from PIL import Image
import numpy as np

class ObjectDetectionVisualizer:
    def __init__(self):
        self.colors = [
            'red', 'blue', 'green', 'yellow', 'orange', 'purple', 'pink', 'brown'
        ]
    
    def visualize_detections(self, image_path, detections, confidence_threshold=0.5):
        """Visualize object detection results with bounding boxes"""
        
        # Load image
        img_np = mpimg.imread(image_path)
        img = Image.fromarray(img_np.astype('uint8'), 'RGB')
        x, y = img.size
        
        # Create figure
        IMAGE_SIZE = (15, 10)
        fig, ax = plt.subplots(1, figsize=IMAGE_SIZE)
        ax.imshow(img_np)
        
        # Draw bounding boxes
        color_index = 0
        for detection in detections:
            if detection['confidence'] >= confidence_threshold:
                # Get bounding box coordinates
                bbox = detection['bounding_box']
                left = bbox['left'] * x
                top = bbox['top'] * y
                width = bbox['width'] * x
                height = bbox['height'] * y
                
                # Select color
                color = self.colors[color_index % len(self.colors)]
                color_index += 1
                
                # Draw rectangle
                rect = patches.Rectangle(
                    (left, top), width, height,
                    linewidth=2, edgecolor=color, facecolor='none'
                )
                ax.add_patch(rect)
                
                # Add label
                label = f"{detection['tag']}: {detection['confidence']:.2f}"
                plt.text(left, top - 10, label, color=color, fontsize=12, 
                        bbox=dict(boxstyle="round,pad=0.3", facecolor='white', alpha=0.7))
        
        plt.title("Object Detection Results")
        plt.axis('off')
        plt.tight_layout()
        plt.show()
    
    def visualize_detections_from_custom_vision(self, image_path, predictions, confidence_threshold=0.5):
        """Visualize Custom Vision predictions"""
        
        detections = []
        for prediction in predictions:
            if prediction.probability >= confidence_threshold:
                detection = {
                    'tag': prediction.tag_name,
                    'confidence': prediction.probability,
                    'bounding_box': {
                        'left': prediction.bounding_box.left,
                        'top': prediction.bounding_box.top,
                        'width': prediction.bounding_box.width,
                        'height': prediction.bounding_box.height
                    }
                }
                detections.append(detection)
        
        self.visualize_detections(image_path, detections, confidence_threshold)

# Usage example
def demo_visualization():
    visualizer = ObjectDetectionVisualizer()
    
    # Example detection results
    sample_detections = [
        {
            'tag': 'fork',
            'confidence': 0.95,
            'bounding_box': {'left': 0.2, 'top': 0.3, 'width': 0.3, 'height': 0.4}
        },
        {
            'tag': 'scissors',
            'confidence': 0.87,
            'bounding_box': {'left': 0.6, 'top': 0.1, 'width': 0.25, 'height': 0.6}
        }
    ]
    
    # Visualize results
    visualizer.visualize_detections("sample_image.jpg", sample_detections)
```

#### Non-Maximum Suppression (NMS)

```python
def non_max_suppression(detections, confidence_threshold=0.5, iou_threshold=0.4):
    """Apply Non-Maximum Suppression to filter overlapping detections"""
    
    if not detections:
        return []
    
    # Filter by confidence threshold
    filtered_detections = [
        det for det in detections 
        if det['confidence'] >= confidence_threshold
    ]
    
    if not filtered_detections:
        return []
    
    # Sort by confidence (descending)
    filtered_detections.sort(key=lambda x: x['confidence'], reverse=True)
    
    # Apply NMS
    keep = []
    while filtered_detections:
        # Take the detection with highest confidence
        current = filtered_detections.pop(0)
        keep.append(current)
        
        # Remove detections with high IoU with current detection
        filtered_detections = [
            det for det in filtered_detections 
            if calculate_iou(current['bounding_box'], det['bounding_box']) < iou_threshold
        ]
    
    return keep

def calculate_iou(box1, box2):
    """Calculate Intersection over Union (IoU) of two bounding boxes"""
    
    # Calculate intersection coordinates
    x1 = max(box1['left'], box2['left'])
    y1 = max(box1['top'], box2['top'])
    x2 = min(box1['left'] + box1['width'], box2['left'] + box2['width'])
    y2 = min(box1['top'] + box1['height'], box2['top'] + box2['height'])
    
    # Calculate intersection area
    if x2 <= x1 or y2 <= y1:
        intersection = 0
    else:
        intersection = (x2 - x1) * (y2 - y1)
    
    # Calculate union area
    area1 = box1['width'] * box1['height']
    area2 = box2['width'] * box2['height']
    union = area1 + area2 - intersection
    
    # Calculate IoU
    if union == 0:
        return 0
    
    return intersection / union
```

### 3️⃣ REST API Implementation

#### Python - REST API for Object Detection

```python
import requests
import json
import base64

class CustomVisionObjectDetectionAPI:
    def __init__(self, endpoint, training_key, prediction_key):
        self.endpoint = endpoint
        self.training_key = training_key
        self.prediction_key = prediction_key
    
    def create_object_detection_project(self, project_name):
        """Create object detection project via REST API"""
        
        # Get domains first
        domains_url = f"{self.endpoint}/customvision/v3.3/training/domains"
        headers = {"Training-Key": self.training_key}
        
        response = requests.get(domains_url, headers=headers)
        response.raise_for_status()
        domains = response.json()
        
        # Find object detection domain
        obj_detection_domain = next(
            (d for d in domains if d['type'] == 'ObjectDetection' and d['name'] == 'General'),
            None
        )
        
        if not obj_detection_domain:
            raise Exception("Object detection domain not found")
        
        # Create project
        url = f"{self.endpoint}/customvision/v3.3/training/projects"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/json"
        }
        
        data = {
            "name": project_name,
            "domainId": obj_detection_domain['id']
        }
        
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()
    
    def upload_image_with_regions(self, project_id, image_data, regions):
        """Upload image with bounding box regions"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects/{project_id}/images/regions"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/json"
        }
        
        # Convert image to base64
        image_b64 = base64.b64encode(image_data).decode('utf-8')
        
        data = {
            "images": [
                {
                    "contents": image_b64,
                    "regions": regions
                }
            ]
        }
        
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()
    
    def detect_objects(self, project_id, published_name, image_data):
        """Detect objects in image"""
        
        url = f"{self.endpoint}/customvision/v3.0/prediction/{project_id}/detect/iterations/{published_name}/image"
        headers = {
            "Prediction-Key": self.prediction_key,
            "Content-Type": "application/octet-stream"
        }
        
        response = requests.post(url, headers=headers, data=image_data)
        response.raise_for_status()
        
        return response.json()
    
    def detect_objects_from_url(self, project_id, published_name, image_url):
        """Detect objects in image from URL"""
        
        url = f"{self.endpoint}/customvision/v3.0/prediction/{project_id}/detect/iterations/{published_name}/url"
        headers = {
            "Prediction-Key": self.prediction_key,
            "Content-Type": "application/json"
        }
        
        data = {"Url": image_url}
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()
```

## 🔧 Configuration and Setup

### Environment Variables

```bash
# Custom Vision configuration for Object Detection
export CUSTOM_VISION_TRAINING_ENDPOINT="https://your-training-resource.cognitiveservices.azure.com/"
export CUSTOM_VISION_TRAINING_KEY="your-training-key"
export CUSTOM_VISION_PREDICTION_ENDPOINT="https://your-prediction-resource.cognitiveservices.azure.com/"
export CUSTOM_VISION_PREDICTION_KEY="your-prediction-key"
export CUSTOM_VISION_PREDICTION_RESOURCE_ID="/subscriptions/your-subscription/resourceGroups/your-rg/providers/Microsoft.CognitiveServices/accounts/your-prediction-resource"
```

### Package Installation

```bash
# Python packages
pip install azure-cognitiveservices-vision-customvision
pip install matplotlib pillow numpy
pip install msrest

# C# packages (via NuGet)
# Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training
# Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction
```

## 🎓 Key Concepts

### Bounding Box Coordinates
- **Normalized coordinates**: Values between 0 and 1 relative to image dimensions
- **Left**: X-coordinate of top-left corner
- **Top**: Y-coordinate of top-left corner
- **Width**: Width of bounding box
- **Height**: Height of bounding box

### Object Detection vs Classification
- **Classification**: What is in the image?
- **Object Detection**: What is in the image and where is it located?
- **Instance Segmentation**: What is in the image, where is it, and what exact pixels belong to it?

### Training Requirements
- **Minimum**: 15 images per object with bounding box annotations
- **Recommended**: 50+ images per object for production models
- **Annotations**: Precise bounding boxes around each object instance

## 📝 Best Practices

### 1. Annotation Quality
```python
def validate_annotation_quality():
    """Guidelines for high-quality annotations"""
    
    guidelines = {
        "tight_boxes": "Bounding boxes should tightly fit around objects",
        "complete_objects": "Include entire object, don't crop important parts",
        "consistent_labeling": "Use consistent labeling across all images",
        "edge_cases": "Include partially visible and occluded objects",
        "variety": "Include objects at different scales, angles, and lighting"
    }
    
    return guidelines
```

### 2. Model Performance Optimization
```python
def optimize_object_detection_model():
    """Strategies for improving model performance"""
    
    strategies = {
        "data_augmentation": "Use various lighting, backgrounds, orientations",
        "negative_samples": "Include images without target objects",
        "balanced_dataset": "Similar number of instances per object class",
        "quality_images": "High resolution, clear, well-lit images",
        "iterative_training": "Start small, add more data based on errors"
    }
    
    return strategies
```

### 3. Post-Processing Pipeline
```python
def create_detection_pipeline(predictions, confidence_threshold=0.5, nms_threshold=0.4):
    """Complete detection post-processing pipeline"""
    
    # Step 1: Filter by confidence
    confident_predictions = [
        pred for pred in predictions 
        if pred.probability > confidence_threshold
    ]
    
    # Step 2: Convert to standard format
    detections = []
    for pred in confident_predictions:
        detection = {
            'tag': pred.tag_name,
            'confidence': pred.probability,
            'bounding_box': {
                'left': pred.bounding_box.left,
                'top': pred.bounding_box.top,
                'width': pred.bounding_box.width,
                'height': pred.bounding_box.height
            }
        }
        detections.append(detection)
    
    # Step 3: Apply Non-Maximum Suppression
    final_detections = non_max_suppression(detections, confidence_threshold, nms_threshold)
    
    return final_detections
```

## 🚀 Advanced Features

### Multi-Object Tracking
```python
class ObjectTracker:
    def __init__(self, max_disappeared=10):
        self.next_object_id = 0
        self.objects = {}
        self.disappeared = {}
        self.max_disappeared = max_disappeared
    
    def register(self, centroid):
        """Register new object"""
        self.objects[self.next_object_id] = centroid
        self.disappeared[self.next_object_id] = 0
        self.next_object_id += 1
    
    def deregister(self, object_id):
        """Remove object that has disappeared"""
        del self.objects[object_id]
        del self.disappeared[object_id]
    
    def update(self, detections):
        """Update object tracking with new detections"""
        # Implementation for object tracking across frames
        pass
```

### Performance Metrics
```python
def calculate_detection_metrics(predictions, ground_truth, iou_threshold=0.5):
    """Calculate object detection performance metrics"""
    
    true_positives = 0
    false_positives = 0
    false_negatives = len(ground_truth)
    
    for pred in predictions:
        matched = False
        for gt in ground_truth:
            if (pred['tag'] == gt['tag'] and 
                calculate_iou(pred['bounding_box'], gt['bounding_box']) >= iou_threshold):
                true_positives += 1
                false_negatives -= 1
                matched = True
                break
        
        if not matched:
            false_positives += 1
    
    precision = true_positives / (true_positives + false_positives) if (true_positives + false_positives) > 0 else 0
    recall = true_positives / (true_positives + false_negatives) if (true_positives + false_negatives) > 0 else 0
    f1_score = 2 * (precision * recall) / (precision + recall) if (precision + recall) > 0 else 0
    
    return {
        "precision": precision,
        "recall": recall,
        "f1_score": f1_score,
        "true_positives": true_positives,
        "false_positives": false_positives,
        "false_negatives": false_negatives
    }
```

## 📚 Additional Resources

- [Custom Vision Object Detection Documentation](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/get-started-build-detector)
- [Object Detection Best Practices](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/suggested-tags)
- [Custom Vision API Reference](https://docs.microsoft.com/rest/api/customvision/)
- [COCO Dataset Format](https://cocodataset.org/#format-data)
- [Object Detection Metrics](https://blog.zenggyu.com/en/post/2018-12-16/an-introduction-to-evaluation-metrics-for-object-detection/)