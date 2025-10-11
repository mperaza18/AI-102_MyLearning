# 🎯 Classify Images

## 📚 Overview

Image classification is the process of training machine learning models to categorize images into predefined classes or labels. Azure Custom Vision service enables you to build, train, and deploy custom image classification models without deep machine learning expertise.

## 🎯 Learning Objectives

- ✅ Create and configure Custom Vision projects for image classification
- ✅ Upload and tag training images effectively
- ✅ Train custom image classification models
- ✅ Test and evaluate model performance
- ✅ Deploy models and make predictions via API

## 🛠️ Implementation Options

### 1️⃣ Azure Custom Vision SDK

#### Python Implementation

```python
from azure.cognitiveservices.vision.customvision.training import CustomVisionTrainingClient
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from azure.cognitiveservices.vision.customvision.training.models import ImageFileCreateBatch, ImageFileCreateEntry
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

def create_classification_project(trainer):
    """Create a new image classification project"""
    
    # Create a new project
    print("Creating project...")
    project_name = uuid.uuid4()
    project = trainer.create_project(project_name)
    print(f"Project created with ID: {project.id}")
    
    return project

def create_tags(trainer, project):
    """Create tags for different image categories"""
    
    # Make two tags in the new project
    hemlock_tag = trainer.create_tag(project.id, "Hemlock")
    cherry_tag = trainer.create_tag(project.id, "Japanese Cherry")
    
    print(f"Created tags: {hemlock_tag.name}, {cherry_tag.name}")
    
    return hemlock_tag, cherry_tag

def upload_and_tag_images(trainer, project, hemlock_tag, cherry_tag):
    """Upload and tag training images"""
    
    base_image_location = os.path.join(os.path.dirname(__file__), "Images")
    
    print("Adding images...")
    image_list = []
    
    # Upload Hemlock images
    for image_num in range(1, 11):
        file_name = f"hemlock_{image_num}.jpg"
        try:
            with open(os.path.join(base_image_location, "Hemlock", file_name), "rb") as image_contents:
                image_list.append(ImageFileCreateEntry(
                    name=file_name, 
                    contents=image_contents.read(), 
                    tag_ids=[hemlock_tag.id]
                ))
        except FileNotFoundError:
            print(f"Image not found: {file_name}")
    
    # Upload Japanese Cherry images
    for image_num in range(1, 11):
        file_name = f"japanese_cherry_{image_num}.jpg"
        try:
            with open(os.path.join(base_image_location, "Japanese_Cherry", file_name), "rb") as image_contents:
                image_list.append(ImageFileCreateEntry(
                    name=file_name, 
                    contents=image_contents.read(), 
                    tag_ids=[cherry_tag.id]
                ))
        except FileNotFoundError:
            print(f"Image not found: {file_name}")
    
    # Upload the batch
    upload_result = trainer.create_images_from_files(project.id, ImageFileCreateBatch(images=image_list))
    
    if not upload_result.is_batch_successful:
        print("Image batch upload failed.")
        for image in upload_result.images:
            print("Image status: ", image.status)
        return False
    
    print(f"Successfully uploaded {len(image_list)} images")
    return True

def train_project(trainer, project):
    """Train the classification model"""
    
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

def publish_iteration(trainer, project, iteration):
    """Publish the trained iteration"""
    
    # The iteration is now trained. Publish it to the project endpoint
    publish_iteration_name = "classifyModel"
    trainer.publish_iteration(project.id, iteration.id, publish_iteration_name, prediction_resource_id)
    print(f"Published iteration: {publish_iteration_name}")
    
    return publish_iteration_name

def test_prediction(predictor, project, publish_iteration_name):
    """Test the prediction endpoint with a sample image"""
    
    base_image_location = os.path.join(os.path.dirname(__file__), "Images")
    
    try:
        with open(os.path.join(base_image_location, "Test", "test_image.jpg"), "rb") as image_contents:
            results = predictor.classify_image(
                project.id, 
                publish_iteration_name, 
                image_contents.read()
            )
            
            # Display the results
            print("Prediction results:")
            for prediction in results.predictions:
                print(f"\t{prediction.tag_name}: {prediction.probability * 100:.2f}%")
                
    except FileNotFoundError:
        print("Test image not found")

def classify_image_from_url(predictor, project, publish_iteration_name, image_url):
    """Classify an image from URL"""
    
    results = predictor.classify_image_url(
        project.id,
        publish_iteration_name,
        image_url
    )
    
    print(f"Classification results for {image_url}:")
    for prediction in results.predictions:
        print(f"\t{prediction.tag_name}: {prediction.probability * 100:.2f}%")
    
    return results

# Complete workflow
def main():
    """Main function demonstrating the complete workflow"""
    
    try:
        # Step 1: Authenticate
        trainer, predictor = authenticate_clients()
        
        # Step 2: Create project
        project = create_classification_project(trainer)
        
        # Step 3: Create tags
        hemlock_tag, cherry_tag = create_tags(trainer, project)
        
        # Step 4: Upload and tag images
        if upload_and_tag_images(trainer, project, hemlock_tag, cherry_tag):
            
            # Step 5: Train the model
            iteration = train_project(trainer, project)
            
            # Step 6: Publish the iteration
            publish_iteration_name = publish_iteration(trainer, project, iteration)
            
            # Step 7: Test predictions
            print("\nTesting predictions...")
            test_prediction(predictor, project, publish_iteration_name)
            
            # Test with URL
            test_url = "https://example.com/test-image.jpg"
            classify_image_from_url(predictor, project, publish_iteration_name, test_url)
            
        print("Classification workflow completed!")
        
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

public class ImageClassificationService
{
    private readonly string _trainingEndpoint;
    private readonly string _trainingKey;
    private readonly string _predictionEndpoint;
    private readonly string _predictionKey;
    private readonly string _predictionResourceId;
    
    public ImageClassificationService(string trainingEndpoint, string trainingKey, 
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
    
    public Project CreateProject(CustomVisionTrainingClient trainingApi)
    {
        Console.WriteLine("Creating new project:");
        return trainingApi.CreateProject("My Classification Project");
    }
    
    public (Tag, Tag) AddTags(CustomVisionTrainingClient trainingApi, Project project)
    {
        var hemlockTag = trainingApi.CreateTag(project.Id, "Hemlock");
        var cherryTag = trainingApi.CreateTag(project.Id, "Japanese Cherry");
        
        Console.WriteLine($"Created tags: {hemlockTag.Name}, {cherryTag.Name}");
        return (hemlockTag, cherryTag);
    }
    
    public void UploadImages(CustomVisionTrainingClient trainingApi, Project project, Tag hemlockTag, Tag cherryTag)
    {
        var hemlockImages = Directory.GetFiles(Path.Combine("Images", "Hemlock")).ToList();
        foreach (var image in hemlockImages)
        {
            using (var stream = new MemoryStream(File.ReadAllBytes(image)))
            {
                trainingApi.CreateImagesFromData(project.Id, stream, new List<Guid>() { hemlockTag.Id });
            }
        }

        var cherryImages = Directory.GetFiles(Path.Combine("Images", "Japanese_Cherry")).ToList();
        foreach (var image in cherryImages)
        {
            using (var stream = new MemoryStream(File.ReadAllBytes(image)))
            {
                trainingApi.CreateImagesFromData(project.Id, stream, new List<Guid>() { cherryTag.Id });
            }
        }
        
        Console.WriteLine("Images uploaded successfully");
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
        trainingApi.PublishIteration(project.Id, iteration.Id, "Iteration1", _predictionResourceId);
        Console.WriteLine("Model published successfully");
    }
    
    public void TestIteration(CustomVisionPredictionClient predictionApi, Project project, string publishedModelName)
    {
        Console.WriteLine("Making a prediction:");
        var testImage = File.ReadAllBytes(Path.Combine("Images", "Test", "test_image.jpg"));
        
        using (var stream = new MemoryStream(testImage))
        {
            var result = predictionApi.ClassifyImage(project.Id, publishedModelName, stream);

            foreach (var prediction in result.Predictions)
            {
                Console.WriteLine($"\t{prediction.TagName}: {prediction.Probability:P1}");
            }
        }
    }
    
    public async Task<ImagePrediction> ClassifyImageFromUrlAsync(CustomVisionPredictionClient predictionApi, 
                                                                Project project, string publishedModelName, string imageUrl)
    {
        var result = predictionApi.ClassifyImageUrl(project.Id, publishedModelName, new ImageUrl(imageUrl));
        
        Console.WriteLine($"Classification results for {imageUrl}:");
        foreach (var prediction in result.Predictions)
        {
            Console.WriteLine($"\t{prediction.TagName}: {prediction.Probability:P1}");
        }
        
        return result;
    }
}

// Usage example
class Program
{
    static void Main(string[] args)
    {
        var service = new ImageClassificationService(
            "https://your-training-endpoint.cognitiveservices.azure.com/",
            "your-training-key",
            "https://your-prediction-endpoint.cognitiveservices.azure.com/",
            "your-prediction-key",
            "/subscriptions/your-subscription/resourceGroups/your-rg/providers/Microsoft.CognitiveServices/accounts/your-prediction-resource"
        );
        
        var trainingApi = service.AuthenticateTraining();
        var predictionApi = service.AuthenticatePrediction();
        
        var project = service.CreateProject(trainingApi);
        var (hemlockTag, cherryTag) = service.AddTags(trainingApi, project);
        
        service.UploadImages(trainingApi, project, hemlockTag, cherryTag);
        var iteration = service.TrainProject(trainingApi, project);
        service.PublishIteration(trainingApi, project, iteration);
        
        service.TestIteration(predictionApi, project, "Iteration1");
    }
}
```

### 2️⃣ REST API Implementation

#### Python - REST API

```python
import requests
import json
import base64

class CustomVisionRestClient:
    def __init__(self, endpoint, training_key, prediction_key):
        self.endpoint = endpoint
        self.training_key = training_key
        self.prediction_key = prediction_key
        
    def create_project(self, project_name, domain_id=None):
        """Create a new Custom Vision project"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/json"
        }
        
        data = {"name": project_name}
        if domain_id:
            data["domainId"] = domain_id
            
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()
    
    def create_tag(self, project_id, tag_name):
        """Create a tag for the project"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects/{project_id}/tags"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/json"
        }
        
        data = {"name": tag_name}
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()
    
    def upload_image(self, project_id, image_data, tag_ids):
        """Upload an image with tags"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects/{project_id}/images"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/octet-stream"
        }
        
        params = {"tagIds": ",".join(tag_ids)}
        response = requests.post(url, headers=headers, params=params, data=image_data)
        response.raise_for_status()
        
        return response.json()
    
    def train_project(self, project_id):
        """Start training the project"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects/{project_id}/train"
        headers = {
            "Training-Key": self.training_key,
            "Content-Type": "application/json"
        }
        
        response = requests.post(url, headers=headers)
        response.raise_for_status()
        
        return response.json()
    
    def get_iteration(self, project_id, iteration_id):
        """Get iteration status"""
        
        url = f"{self.endpoint}/customvision/v3.3/training/projects/{project_id}/iterations/{iteration_id}"
        headers = {"Training-Key": self.training_key}
        
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        
        return response.json()
    
    def classify_image(self, project_id, published_name, image_data):
        """Classify an image using the trained model"""
        
        url = f"{self.endpoint}/customvision/v3.0/prediction/{project_id}/classify/iterations/{published_name}/image"
        headers = {
            "Prediction-Key": self.prediction_key,
            "Content-Type": "application/octet-stream"
        }
        
        response = requests.post(url, headers=headers, data=image_data)
        response.raise_for_status()
        
        return response.json()
    
    def classify_image_url(self, project_id, published_name, image_url):
        """Classify an image from URL"""
        
        url = f"{self.endpoint}/customvision/v3.0/prediction/{project_id}/classify/iterations/{published_name}/url"
        headers = {
            "Prediction-Key": self.prediction_key,
            "Content-Type": "application/json"
        }
        
        data = {"Url": image_url}
        response = requests.post(url, headers=headers, json=data)
        response.raise_for_status()
        
        return response.json()

# Usage example
def demo_rest_api():
    client = CustomVisionRestClient(
        "https://your-endpoint.cognitiveservices.azure.com/",
        "your-training-key",
        "your-prediction-key"
    )
    
    # Create project
    project = client.create_project("REST API Demo Project")
    project_id = project["id"]
    
    # Create tags
    tag1 = client.create_tag(project_id, "Category1")
    tag2 = client.create_tag(project_id, "Category2")
    
    # Upload images (example with local file)
    with open("sample_image.jpg", "rb") as image_file:
        image_data = image_file.read()
        client.upload_image(project_id, image_data, [tag1["id"]])
    
    # Train project
    iteration = client.train_project(project_id)
    print(f"Training started: {iteration}")
    
    # Check for results (after training completes and model is published)
    # results = client.classify_image_url(project_id, "published_model_name", "https://example.com/test.jpg")
    # print(f"Classification results: {results}")
```

## 🔧 Configuration and Setup

### Environment Variables

```bash
# Custom Vision configuration
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
pip install msrest

# C# packages (via NuGet)
# Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training
# Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction
```

## 🎓 Key Concepts

### Project Types
- **Classification**: Assign single or multiple labels to images
- **Object Detection**: Detect and locate objects within images
- **Multi-class vs Multi-label**: Single category vs multiple categories per image

### Training Data Requirements
- **Minimum Images**: 5 images per tag for basic training
- **Recommended**: 50+ images per tag for better accuracy
- **Image Quality**: Clear, well-lit images with good resolution
- **Variety**: Include different angles, lighting, backgrounds

### Model Performance Metrics
- **Precision**: Accuracy of positive predictions
- **Recall**: Coverage of actual positive cases
- **AP (Average Precision)**: Overall model performance score

## 📝 Best Practices

### 1. Data Preparation
```python
def prepare_training_data():
    """Best practices for preparing training data"""
    
    guidelines = {
        "image_count": "50+ images per category for production models",
        "image_quality": "High resolution, clear, well-lit images",
        "diversity": "Include various angles, lighting, backgrounds",
        "balance": "Similar number of images across all categories",
        "format": "JPEG, PNG formats supported",
        "size": "Maximum 6MB per image"
    }
    
    return guidelines
```

### 2. Training Optimization
```python
def training_best_practices():
    """Training optimization strategies"""
    
    strategies = {
        "iterative_training": "Start with small dataset, gradually add more images",
        "negative_examples": "Include images that might confuse the model",
        "data_augmentation": "Use Custom Vision's built-in augmentation",
        "domain_selection": "Choose appropriate domain (General, Food, Landmarks, etc.)",
        "model_evaluation": "Test with separate validation dataset"
    }
    
    return strategies
```

### 3. Error Handling
```python
def robust_prediction_handling(predictor, project_id, published_name, image_data):
    """Implement robust error handling for predictions"""
    
    try:
        results = predictor.classify_image(project_id, published_name, image_data)
        
        # Filter predictions by confidence threshold
        confident_predictions = [
            pred for pred in results.predictions 
            if pred.probability > 0.5  # 50% confidence threshold
        ]
        
        if confident_predictions:
            # Sort by probability
            confident_predictions.sort(key=lambda x: x.probability, reverse=True)
            return confident_predictions
        else:
            print("No confident predictions found")
            return []
            
    except Exception as e:
        print(f"Prediction error: {e}")
        return None
```

## 🚀 Advanced Features

### Multi-Class Classification
```python
def handle_multiclass_results(results):
    """Handle multi-class classification results"""
    
    # Get top prediction
    top_prediction = max(results.predictions, key=lambda x: x.probability)
    
    # Get all predictions above threshold
    threshold = 0.3
    confident_predictions = [
        pred for pred in results.predictions 
        if pred.probability > threshold
    ]
    
    return {
        "top_prediction": {
            "tag": top_prediction.tag_name,
            "confidence": top_prediction.probability
        },
        "all_confident": [
            {"tag": pred.tag_name, "confidence": pred.probability}
            for pred in confident_predictions
        ]
    }
```

### Model Versioning
```python
def manage_model_versions(trainer, project_id):
    """Manage different model versions"""
    
    # Get all iterations
    iterations = trainer.get_iterations(project_id)
    
    # Find best performing iteration
    best_iteration = max(
        [iter for iter in iterations if iter.status == "Completed"],
        key=lambda x: getattr(x, 'precision', 0),
        default=None
    )
    
    if best_iteration:
        print(f"Best iteration: {best_iteration.name}")
        print(f"Precision: {best_iteration.precision}")
        print(f"Recall: {best_iteration.recall}")
    
    return best_iteration
```

## 📚 Additional Resources

- [Custom Vision Documentation](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/)
- [Custom Vision API Reference](https://docs.microsoft.com/rest/api/customvision/)
- [Training Best Practices](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/getting-started-improving-your-classifier)
- [Pricing and Limits](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/)