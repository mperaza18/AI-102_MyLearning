# 🔍 Detect, Analyze, and Recognize Faces

## 📚 Overview

Face detection, analysis, and recognition are fundamental capabilities for building intelligent applications that can identify and understand human faces in images and videos. This guide covers Azure AI Face service implementation patterns and best practices.

## 🎯 Learning Objectives

- ✅ Detect faces in images using Azure AI Face service
- ✅ Analyze face attributes (age, gender, emotion, pose)
- ✅ Implement face recognition and verification
- ✅ Build person groups for face identification
- ✅ Handle face landmarks and quality assessment

## 🛠️ Implementation Options

### 1️⃣ New Azure AI Face SDK (Recommended)

#### Python Implementation

```python
from azure.core.credentials import AzureKeyCredential
from azure.ai.vision.face import FaceClient
from azure.ai.vision.face.models import (
    FaceDetectionModel,
    FaceRecognitionModel,
    FaceAttributeTypeDetection03,
    FaceAttributeTypeRecognition04,
)

# Configuration
endpoint = "https://your-face-resource.cognitiveservices.azure.com/"
key = "your-api-key"

def detect_faces_with_attributes():
    """Detect faces with detailed attributes"""
    
    with FaceClient(endpoint=endpoint, credential=AzureKeyCredential(key)) as face_client:
        sample_file_path = "path/to/your/image.jpg"
        with open(sample_file_path, "rb") as fd:
            file_content = fd.read()

        result = face_client.detect(
            file_content,
            detection_model=FaceDetectionModel.DETECTION03,  # Latest detection model
            recognition_model=FaceRecognitionModel.RECOGNITION04,  # Latest recognition model
            return_face_id=True,
            return_face_attributes=[
                FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.MASK,
                FaceAttributeTypeRecognition04.QUALITY_FOR_RECOGNITION,
            ],
            return_face_landmarks=True,
            return_recognition_model=True,
            face_id_time_to_live=120,
        )

        print(f"Detected {len(result)} face(s)")
        for idx, face in enumerate(result):
            print(f"----- Face #{idx+1} -----")
            print(f"Face ID: {face.face_id}")
            print(f"Rectangle: {face.face_rectangle}")
            print(f"Head Pose: {face.face_attributes.head_pose}")
            print(f"Mask: {face.face_attributes.mask}")
            print(f"Quality: {face.face_attributes.quality_for_recognition}")
            print(f"Landmarks: {face.face_landmarks}")

def detect_faces_from_url():
    """Detect faces from image URL"""
    
    with FaceClient(endpoint=endpoint, credential=AzureKeyCredential(key)) as face_client:
        image_url = "https://example.com/image.jpg"
        
        result = face_client.detect_from_url(
            url=image_url,
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=True,
            return_face_attributes=[
                FaceAttributeTypeRecognition04.QUALITY_FOR_RECOGNITION
            ]
        )
        
        print(f"Detected {len(result)} face(s) from URL")
        for face in result:
            print(f"Face ID: {face.face_id}")
            print(f"Quality: {face.face_attributes.quality_for_recognition}")
```

#### C# Implementation

```csharp
using Azure.AI.Vision.Face;
using Azure.Core;

public class FaceDetectionService
{
    private readonly FaceClient _faceClient;
    
    public FaceDetectionService(string endpoint, string apiKey)
    {
        _faceClient = new FaceClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
    }
    
    public async Task DetectFacesWithAttributesAsync(string imagePath)
    {
        using var stream = new FileStream(imagePath, FileMode.Open, FileAccess.Read);

        var detectResponse = await _faceClient.DetectAsync(
            BinaryData.FromStream(stream),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: true,
            returnFaceAttributes: new[] { 
                FaceAttributeType.Detection03.HeadPose, 
                FaceAttributeType.Detection03.Mask, 
                FaceAttributeType.Recognition04.QualityForRecognition 
            },
            returnFaceLandmarks: true,
            returnRecognitionModel: true,
            faceIdTimeToLive: 120);

        var detectedFaces = detectResponse.Value;
        Console.WriteLine($"Detected {detectedFaces.Count} face(s) in the image.");
        
        foreach (var detectedFace in detectedFaces)
        {
            Console.WriteLine($"Face Rectangle: left={detectedFace.FaceRectangle.Left}, " +
                            $"top={detectedFace.FaceRectangle.Top}, " +
                            $"width={detectedFace.FaceRectangle.Width}, " +
                            $"height={detectedFace.FaceRectangle.Height}");
            Console.WriteLine($"Head pose: pitch={detectedFace.FaceAttributes.HeadPose.Pitch}, " +
                            $"roll={detectedFace.FaceAttributes.HeadPose.Roll}, " +
                            $"yaw={detectedFace.FaceAttributes.HeadPose.Yaw}");
            Console.WriteLine($"Mask: NoseAndMouthCovered={detectedFace.FaceAttributes.Mask.NoseAndMouthCovered}, " +
                            $"Type={detectedFace.FaceAttributes.Mask.Type}");
            Console.WriteLine($"Quality: {detectedFace.FaceAttributes.QualityForRecognition}");
            Console.WriteLine($"Recognition model: {detectedFace.RecognitionModel}");
        }
    }
    
    public async Task DetectFacesFromUrlAsync(string imageUrl)
    {
        var response = await _faceClient.DetectAsync(
            new Uri(imageUrl), 
            FaceDetectionModel.Detection03, 
            FaceRecognitionModel.Recognition04, 
            returnFaceId: true, 
            returnFaceLandmarks: true, 
            returnRecognitionModel: true);
            
        var faces = response.Value;
        Console.WriteLine($"Detected {faces.Count} face(s) from URL");
        
        foreach (var face in faces)
        {
            Console.WriteLine($"Face ID: {face.FaceId}");
            Console.WriteLine($"Face Rectangle: {face.FaceRectangle}");
        }
    }
}
```

### 2️⃣ Face Recognition and Identification

#### Python - Person Group Management

```python
import os
import time
import uuid
from azure.core.credentials import AzureKeyCredential
from azure.ai.vision.face import FaceAdministrationClient, FaceClient
from azure.ai.vision.face.models import (
    FaceAttributeTypeRecognition04, 
    FaceDetectionModel, 
    FaceRecognitionModel, 
    QualityForRecognition
)

# Configuration
KEY = os.environ["FACE_APIKEY"]
ENDPOINT = os.environ["FACE_ENDPOINT"]
LARGE_PERSON_GROUP_ID = str(uuid.uuid4())

def create_and_train_person_group():
    """Create and train a person group for face identification"""
    
    with FaceAdministrationClient(endpoint=ENDPOINT, credential=AzureKeyCredential(KEY)) as face_admin_client, \
         FaceClient(endpoint=ENDPOINT, credential=AzureKeyCredential(KEY)) as face_client:
        
        # Create Large Person Group
        print("Creating person group:", LARGE_PERSON_GROUP_ID)
        face_admin_client.large_person_group.create(
            large_person_group_id=LARGE_PERSON_GROUP_ID,
            name=LARGE_PERSON_GROUP_ID,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
        )

        # Create persons
        woman = face_admin_client.large_person_group.create_person(
            large_person_group_id=LARGE_PERSON_GROUP_ID,
            name="Woman",
        )
        man = face_admin_client.large_person_group.create_person(
            large_person_group_id=LARGE_PERSON_GROUP_ID,
            name="Man",
        )
        child = face_admin_client.large_person_group.create_person(
            large_person_group_id=LARGE_PERSON_GROUP_ID,
            name="Child",
        )

        # Sample images
        woman_images = [
            "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/Face/images/Family1-Mom1.jpg",
            "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/Face/images/Family1-Mom2.jpg",
        ]
        
        # Add faces to persons
        for image in woman_images:
            # Check image quality
            sufficient_quality = True
            detected_faces = face_client.detect_from_url(
                url=image,
                detection_model=FaceDetectionModel.DETECTION03,
                recognition_model=FaceRecognitionModel.RECOGNITION04,
                return_face_id=True,
                return_face_attributes=[FaceAttributeTypeRecognition04.QUALITY_FOR_RECOGNITION],
            )
            
            for face in detected_faces:
                if face.face_attributes.quality_for_recognition != QualityForRecognition.HIGH:
                    sufficient_quality = False
                    break

            if not sufficient_quality or len(detected_faces) != 1:
                continue

            face_admin_client.large_person_group.add_face_from_url(
                large_person_group_id=LARGE_PERSON_GROUP_ID,
                person_id=woman.person_id,
                url=image,
                detection_model=FaceDetectionModel.DETECTION03,
            )
            print(f"Face added to person {woman.person_id}")

        # Train the person group
        print(f"Training person group {LARGE_PERSON_GROUP_ID}")
        poller = face_admin_client.large_person_group.begin_train(
            large_person_group_id=LARGE_PERSON_GROUP_ID,
            polling_interval=5,
        )
        poller.wait()
        print(f"Person group {LARGE_PERSON_GROUP_ID} trained successfully")

def identify_faces():
    """Identify faces against the trained person group"""
    
    with FaceClient(endpoint=ENDPOINT, credential=AzureKeyCredential(KEY)) as face_client:
        test_image = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/Face/images/identification1.jpg"
        
        # Detect faces in test image
        faces = face_client.detect_from_url(
            url=test_image,
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=True,
            return_face_attributes=[FaceAttributeTypeRecognition04.QUALITY_FOR_RECOGNITION],
        )
        
        face_ids = []
        for face in faces:
            if face.face_attributes.quality_for_recognition != QualityForRecognition.LOW:
                face_ids.append(face.face_id)

        # Identify faces
        identify_results = face_client.identify_from_large_person_group(
            face_ids=face_ids,
            large_person_group_id=LARGE_PERSON_GROUP_ID,
        )
        
        print("Identifying faces in image")
        for identify_result in identify_results:
            if identify_result.candidates:
                print(f"Person identified for face ID {identify_result.face_id} " +
                      f"with confidence {identify_result.candidates[0].confidence}")
                
                # Verify the identification
                verify_result = face_client.verify_from_large_person_group(
                    face_id=identify_result.face_id,
                    large_person_group_id=LARGE_PERSON_GROUP_ID,
                    person_id=identify_result.candidates[0].person_id,
                )
                print(f"Verification result: {verify_result.is_identical}, " +
                      f"confidence: {verify_result.confidence}")
            else:
                print(f"No person identified for face ID {identify_result.face_id}")
```

#### C# - Face Identification

```csharp
public class FaceIdentificationService
{
    private readonly FaceClient _faceClient;
    private readonly string _personGroupId;
    
    public FaceIdentificationService(string endpoint, string apiKey, string personGroupId)
    {
        _faceClient = new FaceClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
        _personGroupId = personGroupId;
    }
    
    private async Task<List<FaceDetectionResult>> DetectFaceRecognizeAsync(string url)
    {
        var response = await _faceClient.DetectAsync(
            new Uri(url), 
            FaceDetectionModel.Detection03, 
            FaceRecognitionModel.Recognition04, 
            true, 
            [FaceAttributeType.QualityForRecognition]);
            
        IReadOnlyList<FaceDetectionResult> detectedFaces = response.Value;
        List<FaceDetectionResult> sufficientQualityFaces = new List<FaceDetectionResult>();
        
        foreach (FaceDetectionResult detectedFace in detectedFaces)
        {
            QualityForRecognition? faceQualityForRecognition = detectedFace.FaceAttributes.QualityForRecognition;
            if (faceQualityForRecognition.HasValue && 
                (faceQualityForRecognition.Value != QualityForRecognition.Low))
            {
                sufficientQualityFaces.Add(detectedFace);
            }
        }
        
        Console.WriteLine($"{detectedFaces.Count} face(s) with {sufficientQualityFaces.Count} " +
                         $"having sufficient quality detected from image `{Path.GetFileName(url)}`");

        return sufficientQualityFaces;
    }
    
    public async Task FindSimilarFacesAsync()
    {
        Console.WriteLine("========FIND SIMILAR========");
        
        List<string> targetImageFileNames = new List<string>
        {
            "Family1-Dad1.jpg",
            "Family1-Daughter1.jpg",
            "Family1-Mom1.jpg",
            "Family1-Son1.jpg",
            "Family2-Lady1.jpg",
            "Family2-Man1.jpg",
            "Family3-Lady1.jpg",
            "Family3-Man1.jpg"
        };

        string baseUrl = "https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/Face/images/";
        string sourceImageFileName = "findsimilar.jpg";
        List<Guid> targetFaceIds = new List<Guid>();
        
        foreach (string targetImageFileName in targetImageFileNames)
        {
            List<FaceDetectionResult> faces = await DetectFaceRecognizeAsync($"{baseUrl}{targetImageFileName}");
            targetFaceIds.Add(faces[0].FaceId.Value);
        }

        List<FaceDetectionResult> detectedFaces = await DetectFaceRecognizeAsync($"{baseUrl}{sourceImageFileName}");
        Console.WriteLine($"Source image has {detectedFaces.Count} faces");
    }
}
```

### 3️⃣ REST API Implementation

#### Python - REST API

```python
import requests
import json
import os

def create_liveness_session():
    """Create a liveness detection session"""
    
    endpoint = os.environ["FACE_ENDPOINT"]
    key = os.environ["FACE_APIKEY"]
    
    url = f"{endpoint}/face/v1.2/detectLiveness-sessions"
    body = {
        "livenessOperationMode": "PassiveActive",
        "deviceCorrelationId": "723d6d03-ef33-40a8-9682-23a1feb7bccd",
        "enableSessionImage": True
    }
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    
    res = requests.post(url, headers=headers, data=json.dumps(body))
    res.raise_for_status()
    
    data = res.json()
    print("Session created")
    print("sessionId :", data["sessionId"])
    print("authToken :", data["authToken"])
    
    return data["sessionId"], data["authToken"]

def detect_faces_rest_api(image_url):
    """Detect faces using REST API"""
    
    endpoint = os.environ["FACE_ENDPOINT"]
    key = os.environ["FACE_APIKEY"]
    
    detect_url = f"{endpoint}/face/v1.0/detect"
    
    headers = {
        'Ocp-Apim-Subscription-Key': key,
        'Content-Type': 'application/json'
    }
    
    params = {
        'returnFaceId': 'true',
        'returnFaceLandmarks': 'true',
        'returnFaceAttributes': 'age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise',
        'recognitionModel': 'recognition_04',
        'returnRecognitionModel': 'true',
        'detectionModel': 'detection_03',
        'faceIdTimeToLive': '86400'
    }
    
    data = {'url': image_url}
    
    response = requests.post(detect_url, params=params, headers=headers, json=data)
    faces = response.json()
    
    print(f"Detected {len(faces)} face(s)")
    for face in faces:
        print(f"Face ID: {face['faceId']}")
        print(f"Rectangle: {face['faceRectangle']}")
        print(f"Attributes: {face.get('faceAttributes', {})}")
```

## 🔧 Configuration and Setup

### Environment Variables

```bash
# Face service configuration
export FACE_ENDPOINT="https://your-face-resource.cognitiveservices.azure.com/"
export FACE_APIKEY="your-face-api-key"
```

### Package Installation

```bash
# Python packages
pip install azure-ai-vision-face
pip install azure-core

# C# packages (via NuGet)
# Azure.AI.Vision.Face
# Azure.Core
```

## 🎓 Key Concepts

### Face Detection Models
- **Detection 01**: Basic face detection
- **Detection 02**: Improved accuracy and performance
- **Detection 03**: Latest model with best accuracy (recommended)

### Recognition Models
- **Recognition 01**: Original model
- **Recognition 04**: Latest model with improved accuracy (recommended)

### Face Attributes
- **Demographics**: Age, gender
- **Pose**: Head pose (pitch, roll, yaw)
- **Facial Features**: Smile, facial hair, glasses
- **Image Quality**: Blur, exposure, noise
- **Accessories**: Mask detection, accessories

### Quality Assessment
- **High**: Suitable for recognition
- **Medium**: May work for recognition
- **Low**: Not recommended for recognition

## 📝 Best Practices

1. **Use Latest Models**: Always use Detection03 and Recognition04 for best results
2. **Quality Filtering**: Filter faces based on quality scores for recognition tasks
3. **Single Face Validation**: Ensure only one face per image for person enrollment
4. **Error Handling**: Implement retry logic for API calls
5. **Rate Limiting**: Respect API rate limits and implement backoff strategies
6. **Security**: Store API keys securely using environment variables or key vaults

## 🚀 Advanced Features

### Face Verification
```python
# Verify if two faces belong to the same person
verify_result = face_client.verify_face_to_face(
    face_id1=face_id_1,
    face_id2=face_id_2
)
print(f"Same person: {verify_result.is_identical}")
print(f"Confidence: {verify_result.confidence}")
```

### Liveness Detection
```python
# Detect if face is from a live person
session_id, auth_token = create_liveness_session()
# Use session for liveness verification in client app
```

### Face Landmarks
```python
# Get detailed facial landmarks for 27 key points
result = face_client.detect(
    image_data,
    return_face_landmarks=True
)
for face in result:
    landmarks = face.face_landmarks
    print(f"Left eye: ({landmarks.eye_left_outer.x}, {landmarks.eye_left_outer.y})")
    print(f"Right eye: ({landmarks.eye_right_outer.x}, {landmarks.eye_right_outer.y})")
    print(f"Nose tip: ({landmarks.nose_tip.x}, {landmarks.nose_tip.y})")
```

## 📚 Additional Resources

- [Azure AI Face Documentation](https://docs.microsoft.com/azure/cognitive-services/face/)
- [Face API Reference](https://docs.microsoft.com/rest/api/faceapi/)
- [Best Practices Guide](https://docs.microsoft.com/azure/cognitive-services/face/concepts/face-detection)
- [Privacy and Security Guidelines](https://docs.microsoft.com/azure/cognitive-services/face/concepts/responsible-ai-use)