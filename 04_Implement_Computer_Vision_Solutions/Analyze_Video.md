# 🎬 Analyze Video

## 📚 Overview

Video analysis involves extracting insights from video content including scene detection, object tracking, face recognition, motion analysis, and content understanding. Azure provides several services for video analysis including Video Indexer, Azure AI Vision for video frames, and real-time video processing capabilities.

## 🎯 Learning Objectives

- ✅ Implement video frame analysis using Azure AI Vision
- ✅ Build real-time video processing with face detection and tracking
- ✅ Use Azure Video Indexer for comprehensive video insights
- ✅ Extract text, objects, and scenes from video content
- ✅ Implement motion detection and tracking algorithms

## 🛠️ Implementation Options

### 1️⃣ Real-Time Video Frame Analysis

#### Python Implementation - Video Frame Processing

```python
import cv2
import numpy as np
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.face import FaceClient
from azure.core.credentials import AzureKeyCredential
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel
import io
import time
from PIL import Image

# Configuration
VISION_ENDPOINT = "https://your-vision-resource.cognitiveservices.azure.com/"
VISION_KEY = "your-vision-key"
FACE_ENDPOINT = "https://your-face-resource.cognitiveservices.azure.com/"
FACE_KEY = "your-face-key"

class VideoAnalyzer:
    def __init__(self):
        self.vision_client = ImageAnalysisClient(
            endpoint=VISION_ENDPOINT,
            credential=AzureKeyCredential(VISION_KEY)
        )
        self.face_client = FaceClient(
            endpoint=FACE_ENDPOINT,
            credential=AzureKeyCredential(FACE_KEY)
        )
        self.frame_count = 0
        self.analysis_interval = 30  # Analyze every 30 frames (1 second at 30fps)
        
    def frame_to_bytes(self, frame):
        """Convert OpenCV frame to bytes for API calls"""
        # Convert BGR to RGB
        rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        pil_image = Image.fromarray(rgb_frame)
        
        # Convert to bytes
        byte_stream = io.BytesIO()
        pil_image.save(byte_stream, format='JPEG')
        return byte_stream.getvalue()
    
    def analyze_frame_content(self, frame):
        """Analyze frame content using Azure AI Vision"""
        try:
            frame_bytes = self.frame_to_bytes(frame)
            
            # Analyze image
            result = self.vision_client.analyze(
                image_data=frame_bytes,
                visual_features=[
                    VisualFeatures.CAPTION,
                    VisualFeatures.OBJECTS,
                    VisualFeatures.PEOPLE,
                    VisualFeatures.TAGS
                ]
            )
            
            analysis_result = {
                'caption': result.caption.text if result.caption else None,
                'confidence': result.caption.confidence if result.caption else 0,
                'objects': [],
                'people': [],
                'tags': []
            }
            
            # Extract objects
            if result.objects:
                for obj in result.objects.list:
                    analysis_result['objects'].append({
                        'name': obj.tags[0].name if obj.tags else 'unknown',
                        'confidence': obj.tags[0].confidence if obj.tags else 0,
                        'bounding_box': {
                            'x': obj.bounding_box.x,
                            'y': obj.bounding_box.y,
                            'w': obj.bounding_box.w,
                            'h': obj.bounding_box.h
                        }
                    })
            
            # Extract people
            if result.people:
                for person in result.people.list:
                    analysis_result['people'].append({
                        'confidence': person.confidence,
                        'bounding_box': {
                            'x': person.bounding_box.x,
                            'y': person.bounding_box.y,
                            'w': person.bounding_box.w,
                            'h': person.bounding_box.h
                        }
                    })
            
            # Extract tags
            if result.tags:
                for tag in result.tags.list:
                    analysis_result['tags'].append({
                        'name': tag.name,
                        'confidence': tag.confidence
                    })
            
            return analysis_result
            
        except Exception as e:
            print(f"Frame analysis error: {e}")
            return None
    
    def detect_faces_in_frame(self, frame):
        """Detect faces in frame using Azure Face service"""
        try:
            frame_bytes = self.frame_to_bytes(frame)
            
            result = self.face_client.detect(
                frame_bytes,
                detection_model=FaceDetectionModel.DETECTION03,
                recognition_model=FaceRecognitionModel.RECOGNITION04,
                return_face_id=True,
                return_face_landmarks=True,
                return_face_attributes=[
                    'age', 'gender', 'headPose', 'smile', 'emotion'
                ]
            )
            
            faces = []
            for face in result:
                face_info = {
                    'face_id': str(face.face_id),
                    'rectangle': {
                        'left': face.face_rectangle.left,
                        'top': face.face_rectangle.top,
                        'width': face.face_rectangle.width,
                        'height': face.face_rectangle.height
                    },
                    'attributes': {
                        'age': face.face_attributes.age if face.face_attributes else None,
                        'gender': face.face_attributes.gender if face.face_attributes else None,
                        'emotion': face.face_attributes.emotion if face.face_attributes else None,
                        'smile': face.face_attributes.smile if face.face_attributes else None
                    }
                }
                faces.append(face_info)
            
            return faces
            
        except Exception as e:
            print(f"Face detection error: {e}")
            return []
    
    def draw_analysis_results(self, frame, analysis_result, faces):
        """Draw analysis results on frame"""
        # Draw objects
        if analysis_result and 'objects' in analysis_result:
            for obj in analysis_result['objects']:
                bbox = obj['bounding_box']
                cv2.rectangle(frame, 
                            (bbox['x'], bbox['y']), 
                            (bbox['x'] + bbox['w'], bbox['y'] + bbox['h']), 
                            (0, 255, 0), 2)
                cv2.putText(frame, f"{obj['name']} ({obj['confidence']:.2f})", 
                          (bbox['x'], bbox['y'] - 10), 
                          cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
        
        # Draw people
        if analysis_result and 'people' in analysis_result:
            for person in analysis_result['people']:
                bbox = person['bounding_box']
                cv2.rectangle(frame, 
                            (bbox['x'], bbox['y']), 
                            (bbox['x'] + bbox['w'], bbox['y'] + bbox['h']), 
                            (255, 0, 0), 2)
                cv2.putText(frame, f"Person ({person['confidence']:.2f})", 
                          (bbox['x'], bbox['y'] - 10), 
                          cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 0, 0), 1)
        
        # Draw faces
        for face in faces:
            rect = face['rectangle']
            cv2.rectangle(frame, 
                        (rect['left'], rect['top']), 
                        (rect['left'] + rect['width'], rect['top'] + rect['height']), 
                        (0, 0, 255), 2)
            
            # Draw face attributes
            attrs = face['attributes']
            if attrs['age'] and attrs['gender']:
                cv2.putText(frame, f"{attrs['gender']}, {attrs['age']}", 
                          (rect['left'], rect['top'] - 30), 
                          cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 1)
            
            if attrs['emotion']:
                dominant_emotion = max(attrs['emotion'].items(), key=lambda x: x[1])
                cv2.putText(frame, f"{dominant_emotion[0]} ({dominant_emotion[1]:.2f})", 
                          (rect['left'], rect['top'] - 10), 
                          cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 255), 1)
        
        # Draw caption
        if analysis_result and analysis_result['caption']:
            cv2.putText(frame, analysis_result['caption'], 
                      (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
        
        return frame
    
    def process_video_stream(self, video_source=0):
        """Process video stream with real-time analysis"""
        cap = cv2.VideoCapture(video_source)
        
        print("Starting video analysis. Press 'q' to quit.")
        
        while True:
            ret, frame = cap.read()
            if not ret:
                break
            
            self.frame_count += 1
            
            # Analyze frame periodically
            analysis_result = None
            faces = []
            
            if self.frame_count % self.analysis_interval == 0:
                print(f"Analyzing frame {self.frame_count}...")
                analysis_result = self.analyze_frame_content(frame)
                faces = self.detect_faces_in_frame(frame)
                
                if analysis_result:
                    print(f"Caption: {analysis_result['caption']}")
                    print(f"Objects detected: {len(analysis_result['objects'])}")
                    print(f"People detected: {len(analysis_result['people'])}")
                    print(f"Faces detected: {len(faces)}")
            
            # Draw results on frame
            annotated_frame = self.draw_analysis_results(frame, analysis_result, faces)
            
            # Display frame
            cv2.imshow('Video Analysis', annotated_frame)
            
            # Check for quit
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break
        
        cap.release()
        cv2.destroyAllWindows()

# Usage example
def main():
    analyzer = VideoAnalyzer()
    
    # Process webcam feed
    analyzer.process_video_stream(0)  # 0 for default webcam
    
    # Or process video file
    # analyzer.process_video_stream('path/to/video.mp4')

if __name__ == "__main__":
    main()
```

#### C# Implementation - Video Frame Analysis

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
using OpenCvSharp;
using OpenCvSharp.Extensions;
using System.Drawing;
using System.Collections.Generic;
using System.Linq;

public class VideoAnalyzer
{
    private readonly FaceClient _faceClient;
    private readonly string _visionEndpoint;
    private readonly string _visionKey;
    private int _frameCount = 0;
    private readonly int _analysisInterval = 30; // Analyze every 30 frames

    public VideoAnalyzer(string faceEndpoint, string faceKey, string visionEndpoint, string visionKey)
    {
        _faceClient = new FaceClient(new ApiKeyServiceClientCredentials(faceKey))
        {
            Endpoint = faceEndpoint
        };
        _visionEndpoint = visionEndpoint;
        _visionKey = visionKey;
    }

    public async Task ProcessVideoStreamAsync(int cameraIndex = 0)
    {
        using var capture = new VideoCapture(cameraIndex);
        if (!capture.IsOpened())
        {
            Console.WriteLine("Failed to open camera");
            return;
        }

        Console.WriteLine("Starting video analysis. Press ESC to quit.");

        using var window = new Window("Video Analysis");
        var frame = new Mat();

        while (true)
        {
            capture.Read(frame);
            if (frame.Empty())
                break;

            _frameCount++;

            // Analyze frame periodically
            List<DetectedFace> faces = null;
            if (_frameCount % _analysisInterval == 0)
            {
                Console.WriteLine($"Analyzing frame {_frameCount}...");
                faces = await DetectFacesInFrameAsync(frame);
                Console.WriteLine($"Detected {faces?.Count ?? 0} faces");
            }

            // Draw results on frame
            if (faces != null)
            {
                DrawFacesOnFrame(frame, faces);
            }

            // Display frame
            window.ShowImage(frame);

            // Check for ESC key
            if (Cv2.WaitKey(1) == 27) // ESC key
                break;
        }
    }

    private async Task<List<DetectedFace>> DetectFacesInFrameAsync(Mat frame)
    {
        try
        {
            // Convert frame to byte array
            using var bitmap = BitmapConverter.ToBitmap(frame);
            using var stream = new MemoryStream();
            bitmap.Save(stream, System.Drawing.Imaging.ImageFormat.Jpeg);
            stream.Position = 0;

            // Detect faces
            var faces = await _faceClient.Face.DetectWithStreamAsync(
                stream,
                returnFaceId: true,
                returnFaceLandmarks: true,
                returnFaceAttributes: new List<FaceAttributeType>
                {
                    FaceAttributeType.Age,
                    FaceAttributeType.Gender,
                    FaceAttributeType.Emotion,
                    FaceAttributeType.Smile
                }
            );

            return faces.ToList();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Face detection error: {ex.Message}");
            return new List<DetectedFace>();
        }
    }

    private void DrawFacesOnFrame(Mat frame, List<DetectedFace> faces)
    {
        foreach (var face in faces)
        {
            var rect = face.FaceRectangle;
            var rectangle = new Rect(rect.Left, rect.Top, rect.Width, rect.Height);
            
            // Draw face rectangle
            Cv2.Rectangle(frame, rectangle, Scalar.Red, 2);

            // Draw age and gender
            if (face.FaceAttributes != null)
            {
                var label = $"{face.FaceAttributes.Gender}, {face.FaceAttributes.Age}";
                Cv2.PutText(frame, label, 
                    new OpenCvSharp.Point(rect.Left, rect.Top - 30),
                    HersheyFonts.HersheySimplex, 0.5, Scalar.Red, 1);

                // Draw emotion
                if (face.FaceAttributes.Emotion != null)
                {
                    var dominantEmotion = GetDominantEmotion(face.FaceAttributes.Emotion);
                    Cv2.PutText(frame, dominantEmotion,
                        new OpenCvSharp.Point(rect.Left, rect.Top - 10),
                        HersheyFonts.HersheySimplex, 0.5, Scalar.Red, 1);
                }
            }
        }
    }

    private string GetDominantEmotion(Emotion emotion)
    {
        var emotions = new Dictionary<string, double>
        {
            { "Anger", emotion.Anger },
            { "Contempt", emotion.Contempt },
            { "Disgust", emotion.Disgust },
            { "Fear", emotion.Fear },
            { "Happiness", emotion.Happiness },
            { "Neutral", emotion.Neutral },
            { "Sadness", emotion.Sadness },
            { "Surprise", emotion.Surprise }
        };

        var dominant = emotions.OrderByDescending(e => e.Value).First();
        return $"{dominant.Key} ({dominant.Value:F2})";
    }
}

// Usage example
class Program
{
    static async Task Main(string[] args)
    {
        var analyzer = new VideoAnalyzer(
            "https://your-face-endpoint.cognitiveservices.azure.com/",
            "your-face-key",
            "https://your-vision-endpoint.cognitiveservices.azure.com/",
            "your-vision-key"
        );

        await analyzer.ProcessVideoStreamAsync(0); // 0 for default camera
    }
}
```

### 2️⃣ Azure Video Indexer Integration

#### Python Implementation - Video Indexer API

```python
import requests
import json
import time
from urllib.parse import urlencode

class VideoIndexerClient:
    def __init__(self, account_id, api_key, location="trial"):
        self.account_id = account_id
        self.api_key = api_key
        self.location = location
        self.base_url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}"
        self.access_token = None
        
    def get_access_token(self):
        """Get access token for Video Indexer API"""
        url = f"https://api.videoindexer.ai/Auth/{self.location}/Accounts/{self.account_id}/AccessToken"
        headers = {
            'Ocp-Apim-Subscription-Key': self.api_key
        }
        
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        
        self.access_token = response.json()
        return self.access_token
    
    def upload_video(self, video_url, video_name, description="", privacy="Private"):
        """Upload video to Video Indexer"""
        if not self.access_token:
            self.get_access_token()
        
        params = {
            'accessToken': self.access_token,
            'name': video_name,
            'description': description,
            'privacy': privacy,
            'videoUrl': video_url,
            'language': 'en-US',
            'indexingPreset': 'Default'
        }
        
        url = f"{self.base_url}/Videos?" + urlencode(params)
        response = requests.post(url)
        response.raise_for_status()
        
        return response.json()
    
    def upload_video_file(self, file_path, video_name, description="", privacy="Private"):
        """Upload video file to Video Indexer"""
        if not self.access_token:
            self.get_access_token()
        
        params = {
            'accessToken': self.access_token,
            'name': video_name,
            'description': description,
            'privacy': privacy,
            'language': 'en-US',
            'indexingPreset': 'Default'
        }
        
        url = f"{self.base_url}/Videos?" + urlencode(params)
        
        with open(file_path, 'rb') as video_file:
            files = {'file': video_file}
            response = requests.post(url, files=files)
            response.raise_for_status()
        
        return response.json()
    
    def get_video_index(self, video_id, language='en-US'):
        """Get video index results"""
        if not self.access_token:
            self.get_access_token()
        
        params = {
            'accessToken': self.access_token,
            'language': language
        }
        
        url = f"{self.base_url}/Videos/{video_id}/Index?" + urlencode(params)
        response = requests.get(url)
        response.raise_for_status()
        
        return response.json()
    
    def wait_for_indexing(self, video_id, polling_interval=30):
        """Wait for video indexing to complete"""
        print(f"Waiting for video {video_id} to be indexed...")
        
        while True:
            index = self.get_video_index(video_id)
            state = index['state']
            
            print(f"Indexing state: {state}")
            
            if state == 'Processed':
                print("Video indexing completed!")
                return index
            elif state == 'Failed':
                raise Exception("Video indexing failed")
            
            time.sleep(polling_interval)
    
    def get_video_insights(self, video_id):
        """Extract comprehensive insights from indexed video"""
        index = self.get_video_index(video_id)
        
        insights = {
            'summary': {
                'duration': index['durationInSeconds'],
                'thumbnail_id': index['thumbnailId'],
                'language': index['sourceLanguage']
            },
            'transcript': [],
            'faces': [],
            'keywords': [],
            'topics': [],
            'emotions': [],
            'scenes': [],
            'shots': [],
            'labels': [],
            'brands': []
        }
        
        # Extract transcript
        if 'videos' in index and index['videos']:
            video = index['videos'][0]
            
            # Transcript
            if 'insights' in video and 'transcript' in video['insights']:
                for transcript_item in video['insights']['transcript']:
                    insights['transcript'].append({
                        'id': transcript_item['id'],
                        'text': transcript_item['text'],
                        'confidence': transcript_item['confidence'],
                        'start': transcript_item['instances'][0]['start'],
                        'end': transcript_item['instances'][0]['end']
                    })
            
            # Faces
            if 'insights' in video and 'faces' in video['insights']:
                for face in video['insights']['faces']:
                    insights['faces'].append({
                        'id': face['id'],
                        'name': face['name'],
                        'confidence': face['confidence'],
                        'description': face.get('description', ''),
                        'title': face.get('title', ''),
                        'thumbnail_id': face.get('thumbnailId', '')
                    })
            
            # Keywords
            if 'insights' in video and 'keywords' in video['insights']:
                for keyword in video['insights']['keywords']:
                    insights['keywords'].append({
                        'name': keyword['name'],
                        'confidence': keyword['confidence'],
                        'text': keyword['text']
                    })
            
            # Topics
            if 'insights' in video and 'topics' in video['insights']:
                for topic in video['insights']['topics']:
                    insights['topics'].append({
                        'name': topic['name'],
                        'confidence': topic['confidence'],
                        'language': topic['language']
                    })
            
            # Emotions
            if 'insights' in video and 'emotions' in video['insights']:
                for emotion in video['insights']['emotions']:
                    insights['emotions'].append({
                        'type': emotion['type'],
                        'confidence': emotion['confidence'],
                        'instances': emotion['instances']
                    })
            
            # Scenes
            if 'insights' in video and 'scenes' in video['insights']:
                for scene in video['insights']['scenes']:
                    insights['scenes'].append({
                        'id': scene['id'],
                        'instances': scene['instances']
                    })
            
            # Labels (visual content)
            if 'insights' in video and 'labels' in video['insights']:
                for label in video['insights']['labels']:
                    insights['labels'].append({
                        'name': label['name'],
                        'confidence': label['confidence'],
                        'language': label['language']
                    })
        
        return insights
    
    def search_videos(self, query, page_size=25, skip=0):
        """Search for videos in the account"""
        if not self.access_token:
            self.get_access_token()
        
        params = {
            'accessToken': self.access_token,
            'query': query,
            'pageSize': page_size,
            'skip': skip
        }
        
        url = f"{self.base_url}/Videos/Search?" + urlencode(params)
        response = requests.get(url)
        response.raise_for_status()
        
        return response.json()

# Usage example
def analyze_video_with_indexer():
    # Initialize client
    client = VideoIndexerClient(
        account_id="your-account-id",
        api_key="your-api-key",
        location="trial"  # or your region
    )
    
    # Upload video
    print("Uploading video...")
    upload_result = client.upload_video(
        video_url="https://example.com/sample-video.mp4",
        video_name="Sample Video Analysis",
        description="Demo video for analysis"
    )
    
    video_id = upload_result['id']
    print(f"Video uploaded with ID: {video_id}")
    
    # Wait for indexing to complete
    indexed_video = client.wait_for_indexing(video_id)
    
    # Extract insights
    insights = client.get_video_insights(video_id)
    
    # Display results
    print("\n=== VIDEO ANALYSIS RESULTS ===")
    print(f"Duration: {insights['summary']['duration']} seconds")
    
    print(f"\nTranscript ({len(insights['transcript'])} segments):")
    for segment in insights['transcript'][:5]:  # Show first 5 segments
        print(f"  [{segment['start']} - {segment['end']}] {segment['text']}")
    
    print(f"\nFaces detected ({len(insights['faces'])}):")
    for face in insights['faces']:
        print(f"  - {face['name']} (confidence: {face['confidence']:.2f})")
    
    print(f"\nKeywords ({len(insights['keywords'])}):")
    for keyword in insights['keywords'][:10]:  # Show first 10 keywords
        print(f"  - {keyword['name']} (confidence: {keyword['confidence']:.2f})")
    
    print(f"\nScenes: {len(insights['scenes'])}")
    print(f"Labels: {len(insights['labels'])}")
    
    return insights

if __name__ == "__main__":
    analyze_video_with_indexer()
```

### 3️⃣ Motion Detection and Tracking

#### Python Implementation - Motion Detection

```python
import cv2
import numpy as np
from collections import deque
import json

class MotionDetector:
    def __init__(self, min_area=500, history_length=5):
        self.min_area = min_area
        self.history_length = history_length
        self.background_subtractor = cv2.createBackgroundSubtractorMOG2(
            detectShadows=True
        )
        self.motion_history = deque(maxlen=history_length)
        self.tracking_objects = {}
        self.next_object_id = 0
        
    def detect_motion(self, frame):
        """Detect motion in frame"""
        # Apply background subtraction
        fg_mask = self.background_subtractor.apply(frame)
        
        # Remove noise
        kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3))
        fg_mask = cv2.morphologyEx(fg_mask, cv2.MORPH_OPEN, kernel)
        
        # Find contours
        contours, _ = cv2.findContours(
            fg_mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
        )
        
        # Filter contours by area
        motion_areas = []
        for contour in contours:
            area = cv2.contourArea(contour)
            if area > self.min_area:
                x, y, w, h = cv2.boundingRect(contour)
                motion_areas.append({
                    'bbox': (x, y, w, h),
                    'area': area,
                    'centroid': (x + w // 2, y + h // 2)
                })
        
        # Update motion history
        self.motion_history.append({
            'timestamp': cv2.getTickCount(),
            'motion_areas': motion_areas
        })
        
        return motion_areas, fg_mask
    
    def track_objects(self, motion_areas, max_distance=50):
        """Simple object tracking based on centroid distance"""
        current_centroids = [area['centroid'] for area in motion_areas]
        
        if not self.tracking_objects:
            # Initialize tracking for first frame
            for i, area in enumerate(motion_areas):
                self.tracking_objects[self.next_object_id] = {
                    'centroid': area['centroid'],
                    'bbox': area['bbox'],
                    'area': area['area'],
                    'trail': deque(maxlen=10),
                    'disappeared': 0
                }
                self.next_object_id += 1
        else:
            # Update existing objects
            object_ids = list(self.tracking_objects.keys())
            
            if current_centroids:
                # Calculate distances between existing objects and current detections
                distances = np.zeros((len(object_ids), len(current_centroids)))
                
                for i, obj_id in enumerate(object_ids):
                    for j, centroid in enumerate(current_centroids):
                        distances[i, j] = np.linalg.norm(
                            np.array(self.tracking_objects[obj_id]['centroid']) - 
                            np.array(centroid)
                        )
                
                # Assign detections to existing objects
                used_rows = set()
                used_cols = set()
                
                for i in range(min(len(object_ids), len(current_centroids))):
                    min_idx = np.unravel_index(distances.argmin(), distances.shape)
                    row, col = min_idx
                    
                    if row in used_rows or col in used_cols:
                        distances[row, col] = np.inf
                        continue
                    
                    if distances[row, col] <= max_distance:
                        obj_id = object_ids[row]
                        area = motion_areas[col]
                        
                        # Update object
                        self.tracking_objects[obj_id]['centroid'] = area['centroid']
                        self.tracking_objects[obj_id]['bbox'] = area['bbox']
                        self.tracking_objects[obj_id]['area'] = area['area']
                        self.tracking_objects[obj_id]['trail'].append(area['centroid'])
                        self.tracking_objects[obj_id]['disappeared'] = 0
                        
                        used_rows.add(row)
                        used_cols.add(col)
                    
                    distances[row, col] = np.inf
                
                # Mark unused existing objects as disappeared
                for i, obj_id in enumerate(object_ids):
                    if i not in used_rows:
                        self.tracking_objects[obj_id]['disappeared'] += 1
                
                # Create new objects for unassigned detections
                for j in range(len(current_centroids)):
                    if j not in used_cols:
                        area = motion_areas[j]
                        self.tracking_objects[self.next_object_id] = {
                            'centroid': area['centroid'],
                            'bbox': area['bbox'],
                            'area': area['area'],
                            'trail': deque(maxlen=10),
                            'disappeared': 0
                        }
                        self.next_object_id += 1
            else:
                # No current detections, mark all as disappeared
                for obj_id in object_ids:
                    self.tracking_objects[obj_id]['disappeared'] += 1
            
            # Remove objects that have disappeared for too long
            to_remove = []
            for obj_id, obj in self.tracking_objects.items():
                if obj['disappeared'] > 10:  # Remove after 10 frames
                    to_remove.append(obj_id)
            
            for obj_id in to_remove:
                del self.tracking_objects[obj_id]
        
        return self.tracking_objects
    
    def draw_motion_analysis(self, frame, motion_areas, tracked_objects):
        """Draw motion detection and tracking results"""
        # Draw motion areas
        for area in motion_areas:
            x, y, w, h = area['bbox']
            cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
            cv2.putText(frame, f"Motion: {area['area']}", 
                       (x, y - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
        
        # Draw tracked objects
        for obj_id, obj in tracked_objects.items():
            x, y, w, h = obj['bbox']
            
            # Draw bounding box
            cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)
            
            # Draw object ID
            cv2.putText(frame, f"ID: {obj_id}", 
                       (x, y - 25), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 0, 0), 2)
            
            # Draw trail
            if len(obj['trail']) > 1:
                points = np.array(obj['trail'], dtype=np.int32)
                cv2.polylines(frame, [points], False, (0, 0, 255), 2)
            
            # Draw centroid
            cv2.circle(frame, obj['centroid'], 5, (0, 0, 255), -1)
        
        return frame
    
    def analyze_motion_patterns(self):
        """Analyze motion patterns from history"""
        if len(self.motion_history) < 2:
            return {}
        
        analysis = {
            'total_motion_events': len(self.motion_history),
            'average_motion_areas': 0,
            'motion_intensity': 0,
            'active_objects': len(self.tracking_objects)
        }
        
        total_areas = 0
        total_intensity = 0
        
        for frame_data in self.motion_history:
            frame_areas = len(frame_data['motion_areas'])
            total_areas += frame_areas
            
            frame_intensity = sum(area['area'] for area in frame_data['motion_areas'])
            total_intensity += frame_intensity
        
        analysis['average_motion_areas'] = total_areas / len(self.motion_history)
        analysis['motion_intensity'] = total_intensity / len(self.motion_history)
        
        return analysis

# Usage example
def main():
    cap = cv2.VideoCapture(0)  # Use webcam
    motion_detector = MotionDetector(min_area=1000)
    
    print("Motion detection started. Press 'q' to quit.")
    
    while True:
        ret, frame = cap.read()
        if not ret:
            break
        
        # Detect motion
        motion_areas, fg_mask = motion_detector.detect_motion(frame)
        
        # Track objects
        tracked_objects = motion_detector.track_objects(motion_areas)
        
        # Draw results
        result_frame = motion_detector.draw_motion_analysis(
            frame.copy(), motion_areas, tracked_objects
        )
        
        # Analyze patterns
        if len(motion_detector.motion_history) >= 5:
            analysis = motion_detector.analyze_motion_patterns()
            cv2.putText(result_frame, 
                       f"Objects: {analysis['active_objects']}, "
                       f"Intensity: {analysis['motion_intensity']:.0f}", 
                       (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2)
        
        # Display results
        cv2.imshow('Motion Detection', result_frame)
        cv2.imshow('Foreground Mask', fg_mask)
        
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    
    cap.release()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    main()
```

## 🔧 Configuration and Setup

### Environment Variables

```bash
# Azure AI Vision configuration
export VISION_ENDPOINT="https://your-vision-resource.cognitiveservices.azure.com/"
export VISION_KEY="your-vision-key"

# Azure Face service configuration
export FACE_ENDPOINT="https://your-face-resource.cognitiveservices.azure.com/"
export FACE_KEY="your-face-key"

# Video Indexer configuration
export VIDEO_INDEXER_ACCOUNT_ID="your-account-id"
export VIDEO_INDEXER_API_KEY="your-api-key"
export VIDEO_INDEXER_LOCATION="trial"
```

### Package Installation

```bash
# Python packages
pip install opencv-python
pip install azure-ai-vision-imageanalysis
pip install azure-ai-vision-face
pip install numpy pillow
pip install requests

# C# packages (via NuGet)
# OpenCvSharp4
# Microsoft.Azure.CognitiveServices.Vision.Face
# System.Drawing.Common
```

## 🎓 Key Concepts

### Video Analysis Types
- **Frame-by-frame**: Analyze individual frames for content
- **Temporal analysis**: Track changes across multiple frames
- **Scene detection**: Identify scene boundaries and transitions
- **Motion analysis**: Detect and track moving objects

### Real-time Processing
- **Frame rate optimization**: Balance analysis frequency with performance
- **Buffering strategies**: Manage frame queues for smooth processing
- **Parallel processing**: Use multiple threads for analysis tasks

### Object Tracking
- **Centroid tracking**: Simple distance-based object association
- **Kalman filtering**: Predict object movement for better tracking
- **Multi-object tracking**: Handle multiple objects simultaneously

## 📝 Best Practices

### 1. Performance Optimization
```python
def optimize_video_processing():
    """Performance optimization strategies"""
    
    strategies = {
        "frame_skipping": "Analyze every Nth frame instead of every frame",
        "resolution_scaling": "Downscale frames for faster processing",
        "roi_processing": "Focus analysis on regions of interest",
        "async_processing": "Use asynchronous API calls",
        "caching": "Cache analysis results for similar frames"
    }
    
    return strategies
```

### 2. Error Handling
```python
def robust_video_processing(frame_processor):
    """Implement robust error handling for video processing"""
    
    def process_with_retry(frame, max_retries=3):
        for attempt in range(max_retries):
            try:
                return frame_processor(frame)
            except Exception as e:
                print(f"Attempt {attempt + 1} failed: {e}")
                if attempt == max_retries - 1:
                    print("Max retries reached, skipping frame")
                    return None
                time.sleep(1)  # Wait before retry
        
        return None
    
    return process_with_retry
```

### 3. Resource Management
```python
class VideoProcessorManager:
    def __init__(self):
        self.active_connections = {}
        self.max_concurrent_requests = 5
        
    def __enter__(self):
        return self
        
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup_resources()
        
    def cleanup_resources(self):
        """Clean up video processing resources"""
        for connection in self.active_connections.values():
            if hasattr(connection, 'close'):
                connection.close()
        
        self.active_connections.clear()
```

## 🚀 Advanced Features

### Multi-threaded Video Processing
```python
import threading
from queue import Queue
import concurrent.futures

class MultiThreadedVideoProcessor:
    def __init__(self, max_workers=4):
        self.max_workers = max_workers
        self.frame_queue = Queue(maxsize=10)
        self.result_queue = Queue()
        
    def process_video_parallel(self, video_source):
        """Process video with multiple worker threads"""
        
        def frame_producer():
            cap = cv2.VideoCapture(video_source)
            while True:
                ret, frame = cap.read()
                if not ret:
                    break
                self.frame_queue.put(frame)
            cap.release()
            
            # Signal end of frames
            for _ in range(self.max_workers):
                self.frame_queue.put(None)
        
        def frame_processor(worker_id):
            while True:
                frame = self.frame_queue.get()
                if frame is None:
                    break
                
                # Process frame
                result = self.analyze_frame(frame)
                self.result_queue.put(result)
        
        # Start producer thread
        producer_thread = threading.Thread(target=frame_producer)
        producer_thread.start()
        
        # Start worker threads
        with concurrent.futures.ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            futures = [
                executor.submit(frame_processor, i) 
                for i in range(self.max_workers)
            ]
            
            # Process results
            self.process_results()
            
            # Wait for completion
            concurrent.futures.wait(futures)
        
        producer_thread.join()
```

### Real-time Analytics Dashboard
```python
class VideoAnalyticsDashboard:
    def __init__(self):
        self.metrics = {
            'frames_processed': 0,
            'faces_detected': 0,
            'objects_detected': 0,
            'motion_events': 0,
            'processing_fps': 0
        }
        self.start_time = time.time()
        
    def update_metrics(self, analysis_result):
        """Update analytics metrics"""
        self.metrics['frames_processed'] += 1
        
        if 'faces' in analysis_result:
            self.metrics['faces_detected'] += len(analysis_result['faces'])
        
        if 'objects' in analysis_result:
            self.metrics['objects_detected'] += len(analysis_result['objects'])
        
        if 'motion_detected' in analysis_result:
            self.metrics['motion_events'] += 1
        
        # Calculate FPS
        elapsed_time = time.time() - self.start_time
        self.metrics['processing_fps'] = self.metrics['frames_processed'] / elapsed_time
    
    def get_dashboard_data(self):
        """Get current dashboard metrics"""
        return {
            'metrics': self.metrics,
            'uptime': time.time() - self.start_time,
            'status': 'active'
        }
```

## 📚 Additional Resources

- [Azure Video Indexer Documentation](https://docs.microsoft.com/azure/media-services/video-indexer/)
- [Azure AI Vision Video Analysis](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-analyzing-videos)
- [OpenCV Python Documentation](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
- [Real-time Video Processing Best Practices](https://docs.microsoft.com/azure/architecture/solution-ideas/articles/video-analytics)
- [Azure Face API Documentation](https://docs.microsoft.com/azure/cognitive-services/face/)