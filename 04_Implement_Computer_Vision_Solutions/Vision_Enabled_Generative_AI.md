# 🤖 Vision-Enabled Generative AI

## 📚 Overview

Vision-enabled generative AI combines computer vision capabilities with large language models to create multimodal AI systems that can understand, analyze, and generate content based on visual inputs. This includes using models like GPT-4o Vision, DALL-E, and other multimodal AI services to create sophisticated AI applications that work with both text and images.

## 🎯 Learning Objectives

- ✅ Implement GPT-4o Vision for image understanding and analysis
- ✅ Build multimodal chat applications with image inputs
- ✅ Use DALL-E for AI image generation and editing
- ✅ Create vision-enabled chatbots and assistants
- ✅ Integrate computer vision with generative AI workflows

## 🛠️ Implementation Options

### 1️⃣ GPT-4o Vision Implementation

#### Python Implementation - Multimodal Chat with Vision

```python
import os
import base64
import requests
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential, get_bearer_token_provider
from PIL import Image
import io
import json

class VisionEnabledAI:
    def __init__(self, endpoint=None, api_key=None, deployment_name="gpt-4o"):
        self.endpoint = endpoint or os.getenv("AZURE_OPENAI_ENDPOINT")
        self.api_key = api_key or os.getenv("AZURE_OPENAI_API_KEY")
        self.deployment_name = deployment_name
        
        # Initialize client with authentication
        if self.api_key:
            self.client = AzureOpenAI(
                azure_endpoint=self.endpoint,
                api_key=self.api_key,
                api_version="2024-05-01-preview"
            )
        else:
            # Use managed identity authentication
            token_provider = get_bearer_token_provider(
                DefaultAzureCredential(), 
                "https://cognitiveservices.azure.com/.default"
            )
            self.client = AzureOpenAI(
                azure_endpoint=self.endpoint,
                azure_ad_token_provider=token_provider,
                api_version="2024-05-01-preview"
            )
    
    def encode_image_from_path(self, image_path):
        """Encode image file to base64 string"""
        with open(image_path, "rb") as image_file:
            return base64.b64encode(image_file.read()).decode("utf-8")
    
    def encode_image_from_url(self, image_url):
        """Download and encode image from URL to base64"""
        response = requests.get(image_url)
        response.raise_for_status()
        return base64.b64encode(response.content).decode("utf-8")
    
    def analyze_image_with_prompt(self, image_input, prompt, max_tokens=1000):
        """
        Analyze image with custom prompt using GPT-4o Vision
        
        Args:
            image_input: Can be file path, URL, or base64 string
            prompt: Text prompt for analysis
            max_tokens: Maximum tokens in response
        """
        
        # Determine image input type and prepare content
        if image_input.startswith("data:image"):
            # Already base64 encoded with data URI
            image_url = image_input
        elif image_input.startswith("http"):
            # URL
            image_url = image_input
        elif os.path.exists(image_input):
            # File path
            encoded_image = self.encode_image_from_path(image_input)
            image_url = f"data:image/jpeg;base64,{encoded_image}"
        else:
            # Assume it's base64 string
            image_url = f"data:image/jpeg;base64,{image_input}"
        
        messages = [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url}
                    }
                ]
            }
        ]
        
        try:
            response = self.client.chat.completions.create(
                model=self.deployment_name,
                messages=messages,
                max_tokens=max_tokens,
                temperature=0.7
            )
            
            return {
                "success": True,
                "content": response.choices[0].message.content,
                "usage": response.usage._asdict(),
                "model": response.model
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def compare_images(self, image1, image2, comparison_prompt=None):
        """Compare two images using GPT-4o Vision"""
        
        if comparison_prompt is None:
            comparison_prompt = "Compare these two images. What are the similarities and differences?"
        
        # Process both images
        if image1.startswith("http"):
            image1_url = image1
        else:
            encoded_image1 = self.encode_image_from_path(image1)
            image1_url = f"data:image/jpeg;base64,{encoded_image1}"
        
        if image2.startswith("http"):
            image2_url = image2
        else:
            encoded_image2 = self.encode_image_from_path(image2)
            image2_url = f"data:image/jpeg;base64,{encoded_image2}"
        
        messages = [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": comparison_prompt},
                    {
                        "type": "image_url",
                        "image_url": {"url": image1_url}
                    },
                    {
                        "type": "image_url", 
                        "image_url": {"url": image2_url}
                    }
                ]
            }
        ]
        
        try:
            response = self.client.chat.completions.create(
                model=self.deployment_name,
                messages=messages,
                max_tokens=1500,
                temperature=0.7
            )
            
            return {
                "success": True,
                "content": response.choices[0].message.content,
                "usage": response.usage._asdict()
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def create_multimodal_conversation(self):
        """Interactive multimodal conversation with vision capabilities"""
        
        conversation_history = []
        
        print("🤖 Vision-Enabled AI Assistant")
        print("Commands: 'image <path/url>' to add image, 'quit' to exit")
        print("-" * 50)
        
        while True:
            user_input = input("\n👤 You: ").strip()
            
            if user_input.lower() == 'quit':
                break
            
            if user_input.lower().startswith('image '):
                # Handle image input
                image_path = user_input[6:].strip()
                prompt = input("📝 Prompt for this image: ").strip()
                
                if not prompt:
                    prompt = "What do you see in this image? Provide a detailed description."
                
                print("\n🔍 Analyzing image...")
                result = self.analyze_image_with_prompt(image_path, prompt)
                
                if result["success"]:
                    print(f"\n🤖 AI: {result['content']}")
                    
                    # Add to conversation history
                    conversation_history.append({
                        "type": "image_analysis",
                        "image": image_path,
                        "prompt": prompt,
                        "response": result['content']
                    })
                else:
                    print(f"\n❌ Error: {result['error']}")
            
            else:
                # Handle text-only conversation
                if conversation_history:
                    # Build context from conversation history
                    context_messages = []
                    for item in conversation_history[-3:]:  # Last 3 interactions
                        if item["type"] == "image_analysis":
                            context_messages.append({
                                "role": "user",
                                "content": f"[Previous image analysis] {item['prompt']}"
                            })
                            context_messages.append({
                                "role": "assistant",
                                "content": item["response"]
                            })
                    
                    context_messages.append({
                        "role": "user",
                        "content": user_input
                    })
                else:
                    context_messages = [
                        {"role": "user", "content": user_input}
                    ]
                
                try:
                    response = self.client.chat.completions.create(
                        model=self.deployment_name,
                        messages=context_messages,
                        max_tokens=500,
                        temperature=0.7
                    )
                    
                    ai_response = response.choices[0].message.content
                    print(f"\n🤖 AI: {ai_response}")
                    
                    conversation_history.append({
                        "type": "text",
                        "user_input": user_input,
                        "ai_response": ai_response
                    })
                    
                except Exception as e:
                    print(f"\n❌ Error: {str(e)}")
    
    def extract_structured_data(self, image_input, schema):
        """Extract structured data from image based on provided schema"""
        
        schema_prompt = f"""
        Analyze this image and extract information according to the following schema.
        Return the data as a JSON object that matches this structure:
        
        {json.dumps(schema, indent=2)}
        
        If a field cannot be determined from the image, use null as the value.
        Ensure the response is valid JSON.
        """
        
        result = self.analyze_image_with_prompt(image_input, schema_prompt)
        
        if result["success"]:
            try:
                # Try to parse JSON from response
                json_start = result["content"].find('{')
                json_end = result["content"].rfind('}') + 1
                
                if json_start != -1 and json_end != 0:
                    json_str = result["content"][json_start:json_end]
                    parsed_data = json.loads(json_str)
                    result["structured_data"] = parsed_data
                else:
                    result["structured_data"] = None
                    result["parse_error"] = "No valid JSON found in response"
                    
            except json.JSONDecodeError as e:
                result["structured_data"] = None
                result["parse_error"] = str(e)
        
        return result

# Usage examples
def demonstrate_vision_ai():
    # Initialize the vision AI
    vision_ai = VisionEnabledAI()
    
    # Example 1: Basic image analysis
    print("=== Basic Image Analysis ===")
    result = vision_ai.analyze_image_with_prompt(
        "https://example.com/sample-image.jpg",
        "Describe this image in detail. What objects, people, and activities do you see?"
    )
    
    if result["success"]:
        print(f"Analysis: {result['content']}")
        print(f"Tokens used: {result['usage']['total_tokens']}")
    
    # Example 2: Structured data extraction
    print("\n=== Structured Data Extraction ===")
    schema = {
        "scene_type": "string",
        "objects": ["string"],
        "people_count": "integer",
        "colors": ["string"],
        "mood": "string",
        "text_content": "string"
    }
    
    result = vision_ai.extract_structured_data(
        "path/to/image.jpg",
        schema
    )
    
    if result["success"] and result.get("structured_data"):
        print("Extracted data:")
        print(json.dumps(result["structured_data"], indent=2))
    
    # Example 3: Image comparison
    print("\n=== Image Comparison ===")
    comparison_result = vision_ai.compare_images(
        "image1.jpg",
        "image2.jpg",
        "Compare these product images. Which one appears higher quality and why?"
    )
    
    if comparison_result["success"]:
        print(f"Comparison: {comparison_result['content']}")

if __name__ == "__main__":
    demonstrate_vision_ai()
```

#### C# Implementation - Vision-Enabled Chat Application

```csharp
using Azure;
using Azure.AI.OpenAI;
using Azure.Identity;
using OpenAI.Chat;
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

public class VisionEnabledAI
{
    private readonly AzureOpenAIClient _client;
    private readonly ChatClient _chatClient;
    private readonly string _deploymentName;

    public VisionEnabledAI(string endpoint, string apiKey = null, string deploymentName = "gpt-4o")
    {
        _deploymentName = deploymentName;

        if (!string.IsNullOrEmpty(apiKey))
        {
            _client = new AzureOpenAIClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
        }
        else
        {
            _client = new AzureOpenAIClient(new Uri(endpoint), new DefaultAzureCredential());
        }

        _chatClient = _client.GetChatClient(deploymentName);
    }

    public static string EncodeImageToBase64(string imagePath)
    {
        byte[] imageBytes = File.ReadAllBytes(imagePath);
        return Convert.ToBase64String(imageBytes);
    }

    public static async Task<string> EncodeImageFromUrlAsync(string imageUrl)
    {
        using var httpClient = new HttpClient();
        byte[] imageBytes = await httpClient.GetByteArrayAsync(imageUrl);
        return Convert.ToBase64String(imageBytes);
    }

    public async Task<VisionAnalysisResult> AnalyzeImageWithPromptAsync(
        string imageInput, 
        string prompt, 
        int maxTokens = 1000)
    {
        try
        {
            string imageUrl;

            // Determine image input type
            if (imageInput.StartsWith("data:image"))
            {
                imageUrl = imageInput;
            }
            else if (imageInput.StartsWith("http"))
            {
                imageUrl = imageInput;
            }
            else if (File.Exists(imageInput))
            {
                string base64Image = EncodeImageToBase64(imageInput);
                imageUrl = $"data:image/jpeg;base64,{base64Image}";
            }
            else
            {
                imageUrl = $"data:image/jpeg;base64,{imageInput}";
            }

            var messages = new List<ChatMessage>
            {
                new UserChatMessage(
                    ChatMessageContentPart.CreateTextPart(prompt),
                    ChatMessageContentPart.CreateImagePart(imageUrl)
                )
            };

            var chatCompletionOptions = new ChatCompletionOptions
            {
                MaxTokens = maxTokens,
                Temperature = 0.7f
            };

            ChatCompletion response = await _chatClient.CompleteChatAsync(messages, chatCompletionOptions);

            return new VisionAnalysisResult
            {
                Success = true,
                Content = response.Content[0].Text,
                Usage = new TokenUsage
                {
                    PromptTokens = response.Usage.InputTokenCount,
                    CompletionTokens = response.Usage.OutputTokenCount,
                    TotalTokens = response.Usage.TotalTokenCount
                },
                Model = response.Model
            };
        }
        catch (Exception ex)
        {
            return new VisionAnalysisResult
            {
                Success = false,
                Error = ex.Message
            };
        }
    }

    public async Task<VisionAnalysisResult> CompareImagesAsync(
        string image1, 
        string image2, 
        string comparisonPrompt = null)
    {
        comparisonPrompt ??= "Compare these two images. What are the similarities and differences?";

        try
        {
            // Process both images
            string image1Url = await PrepareImageUrlAsync(image1);
            string image2Url = await PrepareImageUrlAsync(image2);

            var messages = new List<ChatMessage>
            {
                new UserChatMessage(
                    ChatMessageContentPart.CreateTextPart(comparisonPrompt),
                    ChatMessageContentPart.CreateImagePart(image1Url),
                    ChatMessageContentPart.CreateImagePart(image2Url)
                )
            };

            var chatCompletionOptions = new ChatCompletionOptions
            {
                MaxTokens = 1500,
                Temperature = 0.7f
            };

            ChatCompletion response = await _chatClient.CompleteChatAsync(messages, chatCompletionOptions);

            return new VisionAnalysisResult
            {
                Success = true,
                Content = response.Content[0].Text,
                Usage = new TokenUsage
                {
                    PromptTokens = response.Usage.InputTokenCount,
                    CompletionTokens = response.Usage.OutputTokenCount,
                    TotalTokens = response.Usage.TotalTokenCount
                }
            };
        }
        catch (Exception ex)
        {
            return new VisionAnalysisResult
            {
                Success = false,
                Error = ex.Message
            };
        }
    }

    private async Task<string> PrepareImageUrlAsync(string imageInput)
    {
        if (imageInput.StartsWith("http"))
        {
            return imageInput;
        }
        else if (File.Exists(imageInput))
        {
            string base64Image = EncodeImageToBase64(imageInput);
            return $"data:image/jpeg;base64,{base64Image}";
        }
        else
        {
            return $"data:image/jpeg;base64,{imageInput}";
        }
    }

    public async Task<VisionAnalysisResult> ExtractStructuredDataAsync<T>(
        string imageInput, 
        string schemaDescription) where T : class
    {
        string schemaPrompt = $@"
        Analyze this image and extract information according to the following schema.
        Return the data as a valid JSON object that matches this structure:
        
        {schemaDescription}
        
        If a field cannot be determined from the image, use null as the value.
        Ensure the response is valid JSON only, no additional text.";

        var result = await AnalyzeImageWithPromptAsync(imageInput, schemaPrompt);

        if (result.Success)
        {
            try
            {
                // Find JSON in response
                string content = result.Content.Trim();
                int jsonStart = content.IndexOf('{');
                int jsonEnd = content.LastIndexOf('}') + 1;

                if (jsonStart >= 0 && jsonEnd > jsonStart)
                {
                    string jsonStr = content.Substring(jsonStart, jsonEnd - jsonStart);
                    T structuredData = JsonSerializer.Deserialize<T>(jsonStr);
                    result.StructuredData = structuredData;
                }
                else
                {
                    result.ParseError = "No valid JSON found in response";
                }
            }
            catch (JsonException ex)
            {
                result.ParseError = ex.Message;
            }
        }

        return result;
    }

    public async Task RunInteractiveSessionAsync()
    {
        var conversationHistory = new List<ConversationItem>();

        Console.WriteLine("🤖 Vision-Enabled AI Assistant");
        Console.WriteLine("Commands: 'image <path/url>' to add image, 'quit' to exit");
        Console.WriteLine(new string('-', 50));

        while (true)
        {
            Console.Write("\n👤 You: ");
            string userInput = Console.ReadLine()?.Trim();

            if (string.IsNullOrEmpty(userInput) || userInput.ToLower() == "quit")
                break;

            if (userInput.ToLower().StartsWith("image "))
            {
                string imagePath = userInput.Substring(6).Trim();
                Console.Write("📝 Prompt for this image: ");
                string prompt = Console.ReadLine()?.Trim();

                if (string.IsNullOrEmpty(prompt))
                    prompt = "What do you see in this image? Provide a detailed description.";

                Console.WriteLine("\n🔍 Analyzing image...");
                var result = await AnalyzeImageWithPromptAsync(imagePath, prompt);

                if (result.Success)
                {
                    Console.WriteLine($"\n🤖 AI: {result.Content}");
                    conversationHistory.Add(new ConversationItem
                    {
                        Type = "image_analysis",
                        ImagePath = imagePath,
                        Prompt = prompt,
                        Response = result.Content
                    });
                }
                else
                {
                    Console.WriteLine($"\n❌ Error: {result.Error}");
                }
            }
            else
            {
                // Handle text-only conversation with context
                var messages = new List<ChatMessage>();

                // Add recent conversation context
                foreach (var item in conversationHistory.TakeLast(3))
                {
                    if (item.Type == "image_analysis")
                    {
                        messages.Add(new UserChatMessage($"[Previous image analysis] {item.Prompt}"));
                        messages.Add(new AssistantChatMessage(item.Response));
                    }
                }

                messages.Add(new UserChatMessage(userInput));

                try
                {
                    var chatCompletionOptions = new ChatCompletionOptions
                    {
                        MaxTokens = 500,
                        Temperature = 0.7f
                    };

                    ChatCompletion response = await _chatClient.CompleteChatAsync(messages, chatCompletionOptions);
                    string aiResponse = response.Content[0].Text;

                    Console.WriteLine($"\n🤖 AI: {aiResponse}");

                    conversationHistory.Add(new ConversationItem
                    {
                        Type = "text",
                        UserInput = userInput,
                        Response = aiResponse
                    });
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"\n❌ Error: {ex.Message}");
                }
            }
        }
    }
}

// Data models
public class VisionAnalysisResult
{
    public bool Success { get; set; }
    public string Content { get; set; }
    public string Error { get; set; }
    public TokenUsage Usage { get; set; }
    public string Model { get; set; }
    public object StructuredData { get; set; }
    public string ParseError { get; set; }
}

public class TokenUsage
{
    public int PromptTokens { get; set; }
    public int CompletionTokens { get; set; }
    public int TotalTokens { get; set; }
}

public class ConversationItem
{
    public string Type { get; set; }
    public string ImagePath { get; set; }
    public string Prompt { get; set; }
    public string UserInput { get; set; }
    public string Response { get; set; }
}

// Example structured data models
public class ImageAnalysisSchema
{
    public string SceneType { get; set; }
    public List<string> Objects { get; set; }
    public int PeopleCount { get; set; }
    public List<string> Colors { get; set; }
    public string Mood { get; set; }
    public string TextContent { get; set; }
}

// Usage example
class Program
{
    static async Task Main(string[] args)
    {
        string endpoint = Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT");
        string apiKey = Environment.GetEnvironmentVariable("AZURE_OPENAI_API_KEY");

        var visionAI = new VisionEnabledAI(endpoint, apiKey);

        // Example 1: Basic image analysis
        Console.WriteLine("=== Basic Image Analysis ===");
        var result = await visionAI.AnalyzeImageWithPromptAsync(
            "https://example.com/sample-image.jpg",
            "Describe this image in detail. What objects, people, and activities do you see?"
        );

        if (result.Success)
        {
            Console.WriteLine($"Analysis: {result.Content}");
            Console.WriteLine($"Tokens used: {result.Usage.TotalTokens}");
        }

        // Example 2: Interactive session
        await visionAI.RunInteractiveSessionAsync();
    }
}
```

### 2️⃣ DALL-E Image Generation

#### Python Implementation - AI Image Generation

```python
import os
import requests
from openai import AzureOpenAI
from PIL import Image
import json
import base64
from io import BytesIO

class ImageGenerator:
    def __init__(self, endpoint=None, api_key=None, deployment_name="dalle3"):
        self.endpoint = endpoint or os.getenv("AZURE_OPENAI_ENDPOINT")
        self.api_key = api_key or os.getenv("AZURE_OPENAI_API_KEY")
        self.deployment_name = deployment_name
        
        self.client = AzureOpenAI(
            azure_endpoint=self.endpoint,
            api_key=self.api_key,
            api_version="2024-02-01"
        )
    
    def generate_image(self, prompt, size="1024x1024", quality="standard", n=1):
        """Generate image using DALL-E 3"""
        
        try:
            response = self.client.images.generate(
                model=self.deployment_name,
                prompt=prompt,
                size=size,
                quality=quality,
                n=n
            )
            
            result = {
                "success": True,
                "images": [],
                "revised_prompt": None
            }
            
            for image_data in response.data:
                image_info = {
                    "url": image_data.url,
                    "revised_prompt": image_data.revised_prompt
                }
                result["images"].append(image_info)
                
                if image_data.revised_prompt and not result["revised_prompt"]:
                    result["revised_prompt"] = image_data.revised_prompt
            
            return result
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def download_and_save_image(self, image_url, save_path):
        """Download generated image and save locally"""
        
        try:
            response = requests.get(image_url)
            response.raise_for_status()
            
            image = Image.open(BytesIO(response.content))
            image.save(save_path)
            
            return {
                "success": True,
                "path": save_path,
                "size": image.size
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def edit_image(self, image_path, mask_path, prompt, size="1024x1024", n=1):
        """Edit existing image using DALL-E"""
        
        try:
            with open(image_path, "rb") as image_file:
                image_data = image_file.read()
            
            files = {
                "image": ("image.png", image_data, "image/png")
            }
            
            # Add mask if provided
            if mask_path and os.path.exists(mask_path):
                with open(mask_path, "rb") as mask_file:
                    mask_data = mask_file.read()
                files["mask"] = ("mask.png", mask_data, "image/png")
            
            # Use REST API for image editing
            url = f"{self.endpoint}/openai/deployments/{self.deployment_name}/images/edits"
            headers = {
                "Api-Key": self.api_key
            }
            
            data = {
                "prompt": prompt,
                "n": n,
                "size": size
            }
            
            response = requests.post(url, headers=headers, data=data, files=files)
            response.raise_for_status()
            
            result_data = response.json()
            
            return {
                "success": True,
                "images": [{"url": img["url"]} for img in result_data["data"]]
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def create_image_variations(self, image_path, n=2, size="1024x1024"):
        """Create variations of an existing image"""
        
        try:
            with open(image_path, "rb") as image_file:
                image_data = image_file.read()
            
            files = {
                "image": ("image.png", image_data, "image/png")
            }
            
            url = f"{self.endpoint}/openai/deployments/{self.deployment_name}/images/variations"
            headers = {
                "Api-Key": self.api_key
            }
            
            data = {
                "n": n,
                "size": size
            }
            
            response = requests.post(url, headers=headers, data=data, files=files)
            response.raise_for_status()
            
            result_data = response.json()
            
            return {
                "success": True,
                "images": [{"url": img["url"]} for img in result_data["data"]]
            }
            
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }

# Creative AI workflow combining vision and generation
class CreativeAIWorkflow:
    def __init__(self, vision_ai, image_generator):
        self.vision_ai = vision_ai
        self.image_generator = image_generator
    
    def analyze_and_recreate(self, source_image, style_modification=""):
        """Analyze an image and create a new version with modifications"""
        
        # Step 1: Analyze the source image
        analysis_prompt = """
        Describe this image in detail for the purpose of recreating it.
        Include:
        - Main subjects and objects
        - Composition and layout
        - Colors and lighting
        - Style and artistic elements
        - Background and setting
        
        Provide a detailed description that could be used as a prompt for image generation.
        """
        
        analysis_result = self.vision_ai.analyze_image_with_prompt(
            source_image, 
            analysis_prompt
        )
        
        if not analysis_result["success"]:
            return {"success": False, "error": "Failed to analyze source image"}
        
        # Step 2: Create generation prompt
        base_description = analysis_result["content"]
        
        if style_modification:
            generation_prompt = f"{base_description}\n\nStyle modification: {style_modification}"
        else:
            generation_prompt = base_description
        
        # Step 3: Generate new image
        generation_result = self.image_generator.generate_image(
            generation_prompt,
            quality="hd"
        )
        
        return {
            "success": generation_result["success"],
            "original_analysis": base_description,
            "generation_prompt": generation_prompt,
            "generated_images": generation_result.get("images", []),
            "revised_prompt": generation_result.get("revised_prompt"),
            "error": generation_result.get("error")
        }
    
    def iterative_improvement(self, initial_prompt, feedback_rounds=3):
        """Iteratively improve image generation based on AI feedback"""
        
        results = []
        current_prompt = initial_prompt
        
        for round_num in range(feedback_rounds):
            print(f"\n=== Round {round_num + 1} ===")
            
            # Generate image
            generation_result = self.image_generator.generate_image(
                current_prompt,
                quality="hd"
            )
            
            if not generation_result["success"]:
                results.append({
                    "round": round_num + 1,
                    "success": False,
                    "error": generation_result["error"]
                })
                break
            
            # Download the generated image
            image_url = generation_result["images"][0]["url"]
            temp_path = f"temp_round_{round_num + 1}.png"
            
            download_result = self.image_generator.download_and_save_image(
                image_url, 
                temp_path
            )
            
            if not download_result["success"]:
                results.append({
                    "round": round_num + 1,
                    "success": False,
                    "error": "Failed to download image"
                })
                break
            
            # Analyze the generated image for improvements
            if round_num < feedback_rounds - 1:  # Don't analyze the last image
                feedback_prompt = f"""
                Analyze this generated image based on the prompt: "{current_prompt}"
                
                Provide specific suggestions for improvement:
                1. What elements could be enhanced?
                2. What's missing or could be added?
                3. How could the composition be improved?
                4. What style adjustments would help?
                
                Based on your analysis, provide an improved prompt for the next iteration.
                """
                
                feedback_result = self.vision_ai.analyze_image_with_prompt(
                    temp_path,
                    feedback_prompt
                )
                
                if feedback_result["success"]:
                    # Extract improved prompt from feedback
                    feedback_text = feedback_result["content"]
                    # Simple extraction - in practice, you might want more sophisticated parsing
                    if "improved prompt:" in feedback_text.lower():
                        improved_section = feedback_text.lower().split("improved prompt:")[-1]
                        current_prompt = improved_section.strip()
                    else:
                        # Combine original prompt with feedback
                        current_prompt = f"{current_prompt}. Improvements: {feedback_text}"
            
            results.append({
                "round": round_num + 1,
                "success": True,
                "prompt": current_prompt,
                "image_url": image_url,
                "image_path": temp_path,
                "revised_prompt": generation_result.get("revised_prompt"),
                "feedback": feedback_result["content"] if round_num < feedback_rounds - 1 else None
            })
        
        return results

# Usage examples
def demonstrate_creative_ai():
    # Initialize components
    vision_ai = VisionEnabledAI()
    image_generator = ImageGenerator()
    creative_workflow = CreativeAIWorkflow(vision_ai, image_generator)
    
    # Example 1: Basic image generation
    print("=== Image Generation ===")
    result = image_generator.generate_image(
        "A serene landscape with mountains reflected in a crystal-clear lake at sunset, painted in the style of Bob Ross",
        quality="hd"
    )
    
    if result["success"]:
        print(f"Generated {len(result['images'])} image(s)")
        print(f"Revised prompt: {result['revised_prompt']}")
        
        # Download the first image
        download_result = image_generator.download_and_save_image(
            result["images"][0]["url"],
            "generated_landscape.png"
        )
        
        if download_result["success"]:
            print(f"Image saved: {download_result['path']}")
    
    # Example 2: Analyze and recreate workflow
    print("\n=== Analyze and Recreate ===")
    recreate_result = creative_workflow.analyze_and_recreate(
        "source_image.jpg",
        "in a cyberpunk style with neon lighting"
    )
    
    if recreate_result["success"]:
        print("Original analysis:")
        print(recreate_result["original_analysis"][:200] + "...")
        print(f"\nGenerated {len(recreate_result['generated_images'])} variations")
    
    # Example 3: Iterative improvement
    print("\n=== Iterative Improvement ===")
    improvement_results = creative_workflow.iterative_improvement(
        "A futuristic city with flying cars and holographic advertisements",
        feedback_rounds=2
    )
    
    for result in improvement_results:
        if result["success"]:
            print(f"Round {result['round']}: {result['image_path']}")
        else:
            print(f"Round {result['round']} failed: {result['error']}")

if __name__ == "__main__":
    demonstrate_creative_ai()
```

### 3️⃣ Multimodal AI Applications

#### Python Implementation - Document Intelligence with Vision

```python
import os
import json
from typing import List, Dict, Any
import base64
from dataclasses import dataclass
from enum import Enum

class DocumentType(Enum):
    INVOICE = "invoice"
    RECEIPT = "receipt"
    CONTRACT = "contract"
    REPORT = "report"
    PRESENTATION = "presentation"
    UNKNOWN = "unknown"

@dataclass
class DocumentInsight:
    document_type: DocumentType
    key_information: Dict[str, Any]
    summary: str
    confidence: float
    extracted_text: str
    tables: List[Dict]
    recommendations: List[str]

class DocumentIntelligenceAI:
    def __init__(self, vision_ai):
        self.vision_ai = vision_ai
        
    def analyze_document(self, document_image_path) -> DocumentInsight:
        """Comprehensive document analysis using vision AI"""
        
        # Step 1: Identify document type
        type_prompt = """
        Analyze this document image and determine its type.
        Possible types: invoice, receipt, contract, report, presentation, other
        
        Return just the document type in lowercase.
        """
        
        type_result = self.vision_ai.analyze_image_with_prompt(
            document_image_path, 
            type_prompt
        )
        
        if not type_result["success"]:
            raise Exception(f"Failed to analyze document type: {type_result['error']}")
        
        # Parse document type
        detected_type = type_result["content"].lower().strip()
        try:
            doc_type = DocumentType(detected_type)
        except ValueError:
            doc_type = DocumentType.UNKNOWN
        
        # Step 2: Extract structured information based on document type
        extraction_result = self._extract_structured_info(
            document_image_path, 
            doc_type
        )
        
        # Step 3: Generate summary and recommendations
        summary_result = self._generate_summary_and_recommendations(
            document_image_path,
            doc_type,
            extraction_result
        )
        
        # Step 4: Extract all text content
        text_result = self.vision_ai.analyze_image_with_prompt(
            document_image_path,
            "Extract all text content from this document. Maintain the original structure and formatting as much as possible."
        )
        
        # Step 5: Extract tables if present
        tables_result = self._extract_tables(document_image_path)
        
        return DocumentInsight(
            document_type=doc_type,
            key_information=extraction_result.get("data", {}),
            summary=summary_result.get("summary", ""),
            confidence=min(
                type_result.get("confidence", 0.8),
                extraction_result.get("confidence", 0.8)
            ),
            extracted_text=text_result.get("content", ""),
            tables=tables_result.get("tables", []),
            recommendations=summary_result.get("recommendations", [])
        )
    
    def _extract_structured_info(self, image_path, doc_type: DocumentType):
        """Extract structured information based on document type"""
        
        schemas = {
            DocumentType.INVOICE: {
                "invoice_number": "string",
                "date": "string",
                "due_date": "string",
                "vendor_name": "string",
                "vendor_address": "string",
                "customer_name": "string",
                "customer_address": "string",
                "total_amount": "number",
                "tax_amount": "number",
                "line_items": [
                    {
                        "description": "string",
                        "quantity": "number",
                        "unit_price": "number",
                        "total": "number"
                    }
                ]
            },
            DocumentType.RECEIPT: {
                "store_name": "string",
                "store_address": "string",
                "date": "string",
                "time": "string",
                "total_amount": "number",
                "tax_amount": "number",
                "payment_method": "string",
                "items": [
                    {
                        "name": "string",
                        "quantity": "number",
                        "price": "number"
                    }
                ]
            },
            DocumentType.CONTRACT: {
                "contract_title": "string",
                "parties": ["string"],
                "effective_date": "string",
                "expiration_date": "string",
                "key_terms": ["string"],
                "signatures_present": "boolean",
                "contract_value": "string"
            },
            DocumentType.REPORT: {
                "report_title": "string",
                "author": "string",
                "date": "string",
                "executive_summary": "string",
                "key_findings": ["string"],
                "recommendations": ["string"],
                "sections": ["string"]
            }
        }
        
        schema = schemas.get(doc_type, {
            "title": "string",
            "date": "string",
            "key_points": ["string"],
            "summary": "string"
        })
        
        schema_prompt = f"""
        Extract structured information from this {doc_type.value} document according to the following schema.
        Return valid JSON only:
        
        {json.dumps(schema, indent=2)}
        
        If a field cannot be found, use null. Ensure all monetary values are numbers without currency symbols.
        """
        
        return self.vision_ai.extract_structured_data(image_path, schema)
    
    def _generate_summary_and_recommendations(self, image_path, doc_type: DocumentType, extraction_result):
        """Generate summary and actionable recommendations"""
        
        summary_prompt = f"""
        Based on this {doc_type.value} document, provide:
        
        1. A concise summary (2-3 sentences) of the document's key content
        2. A list of 3-5 actionable recommendations or next steps
        
        Format your response as JSON:
        {{
            "summary": "string",
            "recommendations": ["string", "string", ...]
        }}
        """
        
        result = self.vision_ai.analyze_image_with_prompt(image_path, summary_prompt)
        
        if result["success"]:
            try:
                # Parse JSON from response
                content = result["content"].strip()
                json_start = content.find('{')
                json_end = content.rfind('}') + 1
                
                if json_start >= 0 and json_end > json_start:
                    json_str = content[json_start:json_end]
                    return json.loads(json_str)
            except json.JSONDecodeError:
                pass
        
        return {
            "summary": "Unable to generate summary",
            "recommendations": []
        }
    
    def _extract_tables(self, image_path):
        """Extract tables from document"""
        
        table_prompt = """
        Identify and extract all tables from this document.
        For each table found, provide:
        1. Table headers (column names)
        2. All data rows
        3. Table caption or title if present
        
        Format as JSON:
        {
            "tables": [
                {
                    "title": "string or null",
                    "headers": ["column1", "column2", ...],
                    "rows": [
                        ["cell1", "cell2", ...],
                        ["cell1", "cell2", ...]
                    ]
                }
            ]
        }
        
        If no tables are found, return {"tables": []}
        """
        
        result = self.vision_ai.analyze_image_with_prompt(image_path, table_prompt)
        
        if result["success"]:
            try:
                content = result["content"].strip()
                json_start = content.find('{')
                json_end = content.rfind('}') + 1
                
                if json_start >= 0 and json_end > json_start:
                    json_str = content[json_start:json_end]
                    return json.loads(json_str)
            except json.JSONDecodeError:
                pass
        
        return {"tables": []}
    
    def batch_process_documents(self, document_paths: List[str]) -> List[DocumentInsight]:
        """Process multiple documents in batch"""
        
        results = []
        
        for i, doc_path in enumerate(document_paths):
            print(f"Processing document {i+1}/{len(document_paths)}: {doc_path}")
            
            try:
                insight = self.analyze_document(doc_path)
                results.append(insight)
                print(f"✅ Successfully processed: {insight.document_type.value}")
            except Exception as e:
                print(f"❌ Failed to process {doc_path}: {str(e)}")
                results.append(None)
        
        return results
    
    def generate_document_report(self, insights: List[DocumentInsight]) -> Dict[str, Any]:
        """Generate summary report from multiple document insights"""
        
        # Filter out None results
        valid_insights = [insight for insight in insights if insight is not None]
        
        if not valid_insights:
            return {"error": "No valid document insights to process"}
        
        # Aggregate statistics
        doc_types = {}
        total_confidence = 0
        
        for insight in valid_insights:
            doc_type = insight.document_type.value
            doc_types[doc_type] = doc_types.get(doc_type, 0) + 1
            total_confidence += insight.confidence
        
        avg_confidence = total_confidence / len(valid_insights)
        
        # Extract common themes and recommendations
        all_recommendations = []
        for insight in valid_insights:
            all_recommendations.extend(insight.recommendations)
        
        return {
            "total_documents": len(valid_insights),
            "document_types": doc_types,
            "average_confidence": round(avg_confidence, 2),
            "common_recommendations": list(set(all_recommendations)),
            "insights": [
                {
                    "type": insight.document_type.value,
                    "summary": insight.summary,
                    "confidence": insight.confidence,
                    "key_info_keys": list(insight.key_information.keys())
                }
                for insight in valid_insights
            ]
        }

# Usage example
def demonstrate_document_intelligence():
    # Initialize components
    vision_ai = VisionEnabledAI()
    doc_ai = DocumentIntelligenceAI(vision_ai)
    
    # Example 1: Single document analysis
    print("=== Single Document Analysis ===")
    try:
        insight = doc_ai.analyze_document("sample_invoice.jpg")
        
        print(f"Document Type: {insight.document_type.value}")
        print(f"Confidence: {insight.confidence:.2f}")
        print(f"Summary: {insight.summary}")
        print(f"Key Information: {json.dumps(insight.key_information, indent=2)}")
        print(f"Recommendations: {insight.recommendations}")
        
        if insight.tables:
            print(f"Tables found: {len(insight.tables)}")
            for i, table in enumerate(insight.tables):
                print(f"  Table {i+1}: {len(table.get('headers', []))} columns, {len(table.get('rows', []))} rows")
    
    except Exception as e:
        print(f"Error: {e}")
    
    # Example 2: Batch processing
    print("\n=== Batch Processing ===")
    document_files = [
        "invoice1.jpg",
        "receipt1.jpg", 
        "contract1.pdf",
        "report1.png"
    ]
    
    # Filter to existing files
    existing_files = [f for f in document_files if os.path.exists(f)]
    
    if existing_files:
        batch_results = doc_ai.batch_process_documents(existing_files)
        report = doc_ai.generate_document_report(batch_results)
        
        print("Batch Processing Report:")
        print(json.dumps(report, indent=2))
    else:
        print("No sample documents found for batch processing")

if __name__ == "__main__":
    demonstrate_document_intelligence()
```

## 🔧 Configuration and Setup

### Environment Variables

```bash
# Azure OpenAI configuration
export AZURE_OPENAI_ENDPOINT="https://your-openai-resource.openai.azure.com/"
export AZURE_OPENAI_API_KEY="your-openai-key"

# Model deployment names
export GPT4O_DEPLOYMENT="gpt-4o"
export DALLE_DEPLOYMENT="dalle3"

# Optional: Storage for generated images
export AZURE_STORAGE_CONNECTION_STRING="your-storage-connection-string"
```

### Package Installation

```bash
# Python packages
pip install openai
pip install azure-identity
pip install pillow requests
pip install azure-storage-blob  # Optional for cloud storage

# C# packages (via NuGet)
# Azure.AI.OpenAI
# Azure.Identity
# OpenAI
# System.Drawing.Common
```

## 🎓 Key Concepts

### Multimodal AI Principles
- **Vision-Language Understanding**: Combining visual and textual information
- **Prompt Engineering**: Crafting effective prompts for vision models
- **Context Management**: Maintaining conversation context across modalities
- **Output Parsing**: Extracting structured data from AI responses

### Image Generation Workflows
- **Prompt Optimization**: Iterative refinement of generation prompts
- **Style Transfer**: Applying artistic styles to generated content
- **Image Editing**: Modifying existing images with AI assistance
- **Quality Control**: Evaluating and improving generated outputs

## 📝 Best Practices

### 1. Effective Prompt Engineering
```python
def create_effective_vision_prompt(task_type, specific_requirements):
    """Create optimized prompts for vision tasks"""
    
    base_prompts = {
        "analysis": "Analyze this image in detail. Focus on [SPECIFIC_ASPECTS].",
        "extraction": "Extract [DATA_TYPE] from this image. Return as structured JSON.",
        "comparison": "Compare these images, focusing on [COMPARISON_CRITERIA].",
        "generation": "Create a detailed description for generating [TARGET_OUTPUT]."
    }
    
    prompt_template = base_prompts.get(task_type, base_prompts["analysis"])
    
    # Add specific requirements
    if specific_requirements:
        prompt_template += f"\n\nSpecific requirements:\n{specific_requirements}"
    
    # Add output format instructions
    prompt_template += "\n\nProvide clear, detailed, and actionable information."
    
    return prompt_template
```

### 2. Error Handling and Resilience
```python
def robust_vision_api_call(vision_ai, image_path, prompt, max_retries=3):
    """Implement robust API calls with retry logic"""
    
    for attempt in range(max_retries):
        try:
            result = vision_ai.analyze_image_with_prompt(image_path, prompt)
            
            if result["success"]:
                return result
            else:
                print(f"Attempt {attempt + 1} failed: {result.get('error', 'Unknown error')}")
                
        except Exception as e:
            print(f"Attempt {attempt + 1} exception: {str(e)}")
            
        if attempt < max_retries - 1:
            time.sleep(2 ** attempt)  # Exponential backoff
    
    return {"success": False, "error": "Max retries exceeded"}
```

### 3. Cost Optimization
```python
class CostOptimizedVisionAI:
    def __init__(self, vision_ai):
        self.vision_ai = vision_ai
        self.token_usage = {"total": 0, "sessions": []}
        
    def analyze_with_budget(self, image_path, prompt, max_tokens=1000, budget_limit=10000):
        """Analyze image with token budget management"""
        
        if self.token_usage["total"] + max_tokens > budget_limit:
            return {
                "success": False,
                "error": f"Budget limit exceeded. Used: {self.token_usage['total']}, Limit: {budget_limit}"
            }
        
        result = self.vision_ai.analyze_image_with_prompt(
            image_path, 
            prompt, 
            max_tokens=max_tokens
        )
        
        if result["success"] and "usage" in result:
            tokens_used = result["usage"]["total_tokens"]
            self.token_usage["total"] += tokens_used
            self.token_usage["sessions"].append({
                "timestamp": time.time(),
                "tokens": tokens_used,
                "image": image_path
            })
        
        return result
```

## 🚀 Advanced Features

### Multimodal RAG (Retrieval-Augmented Generation)
```python
class MultimodalRAG:
    def __init__(self, vision_ai, vector_db, embedding_client):
        self.vision_ai = vision_ai
        self.vector_db = vector_db
        self.embedding_client = embedding_client
    
    def query_with_image_context(self, query_image, text_query, top_k=5):
        """Query knowledge base with both image and text context"""
        
        # Analyze query image
        image_analysis = self.vision_ai.analyze_image_with_prompt(
            query_image,
            "Describe this image in detail for search purposes."
        )
        
        if not image_analysis["success"]:
            return {"error": "Failed to analyze query image"}
        
        # Combine image description with text query
        combined_query = f"{text_query}\n\nImage context: {image_analysis['content']}"
        
        # Get text embedding
        query_embedding = self.embedding_client.get_embedding(combined_query)
        
        # Search vector database
        similar_documents = self.vector_db.search(query_embedding, top_k=top_k)
        
        # Generate response with context
        context = "\n\n".join([doc["content"] for doc in similar_documents])
        
        final_prompt = f"""
        Context information:
        {context}
        
        Query: {text_query}
        Image context: {image_analysis['content']}
        
        Based on the context and image, provide a comprehensive answer.
        """
        
        response = self.vision_ai.client.chat.completions.create(
            model=self.vision_ai.deployment_name,
            messages=[{"role": "user", "content": final_prompt}],
            max_tokens=1500
        )
        
        return {
            "answer": response.choices[0].message.content,
            "image_analysis": image_analysis['content'],
            "relevant_documents": similar_documents
        }
```

### Real-time Vision Processing
```python
import asyncio
import websockets
import json

class RealTimeVisionProcessor:
    def __init__(self, vision_ai):
        self.vision_ai = vision_ai
        self.active_connections = set()
    
    async def handle_client(self, websocket, path):
        """Handle real-time vision processing requests"""
        
        self.active_connections.add(websocket)
        
        try:
            async for message in websocket:
                data = json.loads(message)
                
                if data["type"] == "analyze_image":
                    # Process image analysis request
                    result = await self.process_image_async(
                        data["image_data"],
                        data.get("prompt", "Analyze this image")
                    )
                    
                    await websocket.send(json.dumps({
                        "type": "analysis_result",
                        "result": result
                    }))
                    
        except websockets.exceptions.ConnectionClosed:
            pass
        finally:
            self.active_connections.remove(websocket)
    
    async def process_image_async(self, image_data, prompt):
        """Process image analysis asynchronously"""
        
        loop = asyncio.get_event_loop()
        
        # Run vision analysis in thread pool to avoid blocking
        result = await loop.run_in_executor(
            None,
            self.vision_ai.analyze_image_with_prompt,
            image_data,
            prompt
        )
        
        return result
    
    def start_server(self, host="localhost", port=8765):
        """Start WebSocket server for real-time processing"""
        
        start_server = websockets.serve(self.handle_client, host, port)
        
        print(f"Real-time vision server started on {host}:{port}")
        asyncio.get_event_loop().run_until_complete(start_server)
        asyncio.get_event_loop().run_forever()
```

## 📚 Additional Resources

- [Azure OpenAI Service Documentation](https://docs.microsoft.com/azure/cognitive-services/openai/)
- [GPT-4 Turbo with Vision API Reference](https://docs.microsoft.com/azure/cognitive-services/openai/reference)
- [DALL-E Image Generation Guide](https://docs.microsoft.com/azure/cognitive-services/openai/dall-e-quickstart)
- [Multimodal AI Best Practices](https://docs.microsoft.com/azure/cognitive-services/openai/concepts/multimodal)
- [Vision-Enabled Applications Architecture](https://docs.microsoft.com/azure/architecture/solution-ideas/articles/multimodal-ai)
- [Responsible AI Guidelines for Vision Systems](https://docs.microsoft.com/azure/cognitive-services/responsible-use-of-ai-overview)